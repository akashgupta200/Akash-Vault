---
layout: default
title: "Locks & Blocking"
parent: "DBA Reference Sheet"
nav_order: 15
---

# Locks & Blocking

Reference guide for lock detection, blocking sessions, and lock history.

---

## Find Locked Objects

### Find locked object

```sql
SET LINESIZE 800 PAGES 9999
COL username       FORMAT A40
COL sess_id        FORMAT A10
COL object         FORMAT A50
COL mode_held      FORMAT A10

SELECT oracle_username || ' (' || s.osuser || ')' username,
       s.sid || ',' || s.serial# sess_id,
       owner || '.' || object_name object,
       object_type,
       DECODE(l.block, 0, 'Not Blocking', 1, 'Blocking', 2, 'Global') status,
       DECODE(v.locked_mode, 0, 'None', 1, 'Null', 2, 'Row-S (SS)', 3, 'Row-X (SX)',
              4, 'Share', 5, 'S/Row-X (SSX)', 6, 'Exclusive', TO_CHAR(lmode)) mode_held
FROM v$locked_object v, dba_objects d, v$lock l, v$session s
WHERE v.object_id = d.object_id
  AND v.object_id = l.id1
  AND v.session_id = s.sid
ORDER BY oracle_username, session_id;
```

**Alternative (RAC):**

```sql
COLUMN owner FORMAT A20
COLUMN username FORMAT A20
COLUMN object_owner FORMAT A20
COLUMN object_name FORMAT A30
COLUMN locked_mode FORMAT A15

SELECT b.inst_id, b.session_id AS sid,
       NVL(b.oracle_username, '(oracle)') AS username,
       a.owner AS object_owner, a.object_name,
       DECODE(b.locked_mode, 0, 'None', 1, 'Null (NULL)', 2, 'Row-S (SS)',
              3, 'Row-X (SX)', 4, 'Share (S)', 5, 'S/Row-X (SSX)', 6, 'Exclusive (X)',
              b.locked_mode) locked_mode,
       b.os_user_name
FROM dba_objects a, gv$locked_object b
WHERE a.object_id = b.object_id
ORDER BY 1, 2, 3, 4
/
```

**Alternative (holder/waiter with object):**

```sql
SELECT DECODE(request,0,'Holder: ','Waiter: ') || gv$lock.sid sess, machine,
       do.object_name AS locked_object, id1, id2, lmode, request, gv$lock.type
FROM gv$lock
JOIN gv$session ON gv$lock.sid = gv$session.sid AND gv$lock.inst_id = gv$session.inst_id
JOIN gv$locked_object lo ON gv$lock.SID = lo.SESSION_ID AND gv$lock.inst_id = lo.inst_id
JOIN dba_objects do ON lo.OBJECT_ID = do.OBJECT_ID
WHERE (id1, id2, gv$lock.type) IN (
  SELECT id1, id2, type FROM gv$lock WHERE request > 0
)
ORDER BY id1, request;
```

**Alternative (simple blocking check):**

```sql
SELECT * FROM dba_waiters;
```

**Alternative (blocking session from gv$session):**

```sql
SELECT INST_ID, sid, BLOCKING_SESSION, BLOCKING_instance
FROM gv$session
WHERE BLOCKING_SESSION IS NOT NULL;
```

---

## Object Access

### Find who is accessing a particular table

```sql
SELECT t1.sid SID, s.username, s.osuser, s.program, s.machine, o.object_name, t1.ctime
FROM v$access A, v$lock T1, v$session S, DBA_OBJECTS O
WHERE t1.TYPE = 'TM'
  AND A.SID = S.SID
  AND T1.sid = s.sid
  AND T1.id1 = o.object_id
  AND A.OBJECT LIKE 'TABLE_NAME'||'%'
ORDER BY T1.sid, T1.ctime;
```

### Who is accessing the object

```sql
SELECT b.status, a.INST_ID, a.SID, OWNER, OBJECT, a.TYPE
FROM gv$access a, gv$session b
WHERE a.object = '&object_name'
  AND a.sid = b.sid
  AND a.inst_id = b.inst_id
ORDER BY 1, 2;
```

---

## Blocking Sessions

### Find blocking sessions (good query)

```sql
SET LINES 750 PAGES 9999
COL blocking_status FOR A100

SELECT s1.inst_id, s2.inst_id,
       s1.username || '@' || s1.machine || ' ( SID=' || s1.sid || ' )  is blocking '
       || s2.username || '@' || s2.machine || ' ( SID=' || s2.sid || ' ) ' AS blocking_status,
       ROUND(s2.seconds_in_wait/60)||' mins' Blocking_mins
FROM gv$lock l1, gv$session s1, gv$lock l2, gv$session s2
WHERE s1.sid = l1.sid AND s2.sid = l2.sid
  AND s1.inst_id = l1.inst_id AND s2.inst_id = l2.inst_id
  AND l1.BLOCK = 1 AND l2.request > 0
  AND l1.id1 = l2.id1 AND l2.id2 = l2.id2
ORDER BY s1.inst_id;
```

**Alternative (RAC with more detail):**

```sql
SET LINES 750 PAGES 9999
COL blocking_status FOR A100

SELECT s1.inst_id, s2.inst_id,
       'Instance '||s1.INST_ID||' '|| s1.username || '@' || s1.machine || ' ( SID=' || s1.sid || ','|| s1.serial#||' '||s1.status||','||s1.sql_id||'  )
is blocking '|| s2.username || '@' || s2.machine || ' ( SID=' || s2.sid || ' ) '||s2.sql_id ||' since '||ROUND(s2.seconds_in_wait/60)||' mins' AS blocking_sessions
FROM gv$lock l1, gv$session s1, gv$lock l2, gv$session s2
WHERE s1.sid = l1.sid AND s1.inst_id = l1.inst_id
  AND s2.sid = l2.sid AND s2.inst_id = l2.inst_id
  AND l1.BLOCK = 1 AND l2.request > 0
  AND l1.id1 = l2.id1 AND l2.id2 = l2.id2;
```

**Alternative (simple blocking check):**

```sql
SELECT l1.sid || ' is blocking ' || l2.sid blocking_sessions
FROM v$lock l1, v$lock l2
WHERE l1.block = 1 AND l2.request > 0
  AND l1.id1 = l2.id1 AND l1.id2 = l2.id2;
```

**Alternative (v$session blocking_session):**

```sql
SELECT s.blocking_session, s.sid, s.serial#, s.seconds_in_wait
FROM v$session s
WHERE blocking_session IS NOT NULL;
```

### Check who is blocking who in RAC, including objects

```sql
SELECT DECODE(request,0,'Holder: ','Waiter: ') || gv$lock.sid sess, machine,
       do.object_name AS locked_object, id1, id2, lmode, request, gv$lock.type
FROM gv$lock
JOIN gv$session ON gv$lock.sid = gv$session.sid AND gv$lock.inst_id = gv$session.inst_id
JOIN gv$locked_object lo ON gv$lock.SID = lo.SESSION_ID AND gv$lock.inst_id = lo.inst_id
JOIN dba_objects do ON lo.OBJECT_ID = do.OBJECT_ID
WHERE (id1, id2, gv$lock.type) IN (
  SELECT id1, id2, type FROM gv$lock WHERE request > 0
)
ORDER BY id1, request;
```

---

## SQL of Blocker & Waiter

### All SQL executed by a SID

```sql
SELECT o.inst_id, o.sid, o.sql_text, o.address, o.hash_value, o.user_name,
       s.schemaname, o.sql_id
FROM gv$open_cursor o, gv$session s
WHERE o.saddr = s.saddr
  AND o.sid = s.sid
  AND (O.SID = &sid)
  AND o.inst_id = s.inst_id;
```

### Blocker SQL

```sql
SELECT sid AS blocker_sid, sql_text AS blocker_sql
FROM (SELECT s.sid, txt.sql_text
      FROM gv$sqltext txt, gv$session s, gv$lock l
      WHERE txt.address = s.sql_address
        AND s.sid = l.sid
        AND l.block = 1
      ORDER BY s.sid, txt.piece);
```

### Waiting SQL

```sql
SELECT sid AS waiting_sid, sql_text AS waiting_sql
FROM (SELECT s.sid, txt.sql_text
      FROM gv$sqltext txt, gv$session s, gv$lock lb, gv$lock lw
      WHERE txt.address = s.sql_address
        AND s.sid = lw.sid
        AND lw.id1 = lb.id1
        AND lb.block = 1
        AND lw.request > 0
      ORDER BY s.sid, txt.piece);
```

### What sql_id, sql_text are waiting due to blocking session

```sql
SELECT sn.USERNAME, m.SID, sn.SERIAL#, m.TYPE,
       DECODE(LMODE, 0, 'None', 1, 'Null', 2, 'Row-S (SS)', 3, 'Row-X (SX)',
              4, 'Share', 5, 'S/Row-X (SSX)', 6, 'Exclusive') lock_type,
       DECODE(REQUEST, 0, 'None', 1, 'Null', 2, 'Row-S (SS)', 3, 'Row-X (SX)',
              4, 'Share', 5, 'S/Row-X (SSX)', 6, 'Exclusive') lock_requested,
       m.ID1, m.ID2, t.sql_id, t.SQL_TEXT
FROM v$session sn, v$lock m, v$sqltext t
WHERE t.ADDRESS = sn.SQL_ADDRESS
  AND t.HASH_VALUE = sn.SQL_HASH_VALUE
  AND ((sn.SID = m.SID AND m.REQUEST != 0)
       OR (sn.SID = m.SID AND m.REQUEST = 0 AND LMODE != 4
           AND (ID1, ID2) IN (SELECT s.ID1, s.ID2 FROM v$lock S
                              WHERE REQUEST != 0 AND s.ID1 = m.ID1 AND s.ID2 = m.ID2)))
ORDER BY sn.USERNAME, sn.SID, t.PIECE;
```

---

## Blocking Session History

### Blocking session history (AWR)

```sql
SELECT sample_time, session_state, blocking_session,
       owner||'.'||object_name||':'||NVL(subobject_name,'-') obj_name,
       DBMS_ROWID.ROWID_CREATE(1, o.data_object_id, current_file#, current_block#, current_row#) row_id
FROM dba_hist_active_sess_history s, dba_objects o
WHERE user_id = 92
  AND sample_time BETWEEN TO_DATE('29-SEP-12 04.55.02 PM','dd-MON-yy hh:mi:ss PM')
                      AND TO_DATE('29-SEP-12 05.05.02 PM','dd-MON-yy hh:mi:ss PM')
  AND event = 'enq: TX - row lock contention'
  AND o.data_object_id = s.current_obj#
ORDER BY 1, 2;
```

**Alternative (ASH):**

```sql
COL sql_id FORMAT A15
COL inst_id FORMAT '9'
COL sql_text FORMAT A50
COL module FORMAT A10
COL blocker_ses FORMAT '999999'
COL blocker_ser FORMAT '999999'

SELECT DISTINCT a.sql_id, a.inst_id, a.blocking_session blocker_ses,
       a.blocking_session_serial# blocker_ser, a.user_id, s.sql_text, a.module
FROM GV$ACTIVE_SESSION_HISTORY a, gv$sql s
WHERE a.sql_id = s.sql_id
  AND blocking_session IS NOT NULL
  AND a.user_id <> 0
  AND a.sample_time > SYSDATE - 1
/
```

---

## Metalink Lock Script

### Blocking and locking information (Metalink style)

```sql
SET LINESIZE 200
SET PAGESIZE 1000

COLUMN username FORMAT A10
COLUMN module FORMAT A50
COLUMN blocker FORMAT A7
COLUMN waiter FORMAT A7
COLUMN lmode FORMAT 9999
COLUMN request FORMAT 9999
COLUMN inst_id FORMAT 9999
COLUMN sid FORMAT 9999

SPOOL locking_information

SELECT TO_CHAR(SYSDATE, 'DD-MON-YYYY HH24:MI:SS') current_time FROM dual
/

PROMPT ########################
PROMPT # Blocking Information #
PROMPT ########################

SELECT b.inst_id||'/'||b.sid blocker, w.inst_id||'/'||w.sid waiter,
       b.type, b.id1, b.id2, b.lmode, w.request
FROM gv$lock b,
     (SELECT inst_id, sid, type, id1, id2, lmode, request
      FROM gv$lock WHERE request > 0) w
WHERE b.lmode > 0
  AND (b.id1 = w.id1 AND b.id2 = w.id2 AND b.type = w.type)
ORDER BY b.inst_id, b.sid
/

PROMPT ########################
PROMPT # Locking Information #
PROMPT ########################

SELECT a.inst_id, a.sid, a.type, a.lmode, a.request, a.id1, a.id2, a.block
FROM gv$lock a
ORDER BY a.inst_id, a.sid
/

PROMPT ########################
PROMPT # Session Information #
PROMPT ########################

SELECT s.inst_id, s.sid, s.serial# s#, s.username, s.process client, p.spid server, p.pname,
       s.status, s.module, s.osuser
FROM gv$session s, gv$process p
WHERE s.paddr = p.addr AND s.inst_id = p.inst_id
ORDER BY s.inst_id, s.sid
/

SPOOL OFF
```

---

## Other Lock Queries

### Locked statistics table

```sql
SELECT owner, table_name, stattype_locked
FROM dba_tab_statistics
WHERE stattype_locked IS NOT NULL
  AND owner NOT IN ('SYS','SYSTEM');
```

### Reference object (ORA-02266)

```sql
SELECT p.table_name "Parent Table", c.table_name "Child Table",
       p.constraint_name "Parent Constraint", c.constraint_name "Child Constraint"
FROM user_constraints p
JOIN user_constraints c ON (p.constraint_name = c.r_constraint_name)
WHERE (p.constraint_type = 'P' OR p.constraint_type = 'U')
  AND c.constraint_type = 'R'
  AND p.table_name = UPPER('&table_name');
```

### Who is using my dblink

```sql
SELECT /*+ ORDERED */
       SUBSTR(s.ksusemnm,1,10)||'-'|| SUBSTR(s.ksusepid,1,10) "ORIGIN",
       SUBSTR(g.K2GTITID_ORA,1,35) "GTXID",
       SUBSTR(s.indx,1,4)||'.'|| SUBSTR(s.ksuseser,1,5) "LSESSION",
       s2.username,
       SUBSTR(DECODE(BITAND(ksuseidl,11), 1,'ACTIVE', 0, DECODE(BITAND(ksuseflg,4096), 0,'INACTIVE','CACHED'), 2,'SNIPED', 3,'SNIPED', 'KILLED'),1,1) "S",
       SUBSTR(w.event,1,10) "WAITING"
FROM x$k2gte g, x$ktcxb t, x$ksuse s, v$session_wait w, v$session s2
WHERE g.K2GTDXCB = t.ktcxbxba
  AND g.K2GTDSES = t.ktcxbses
  AND s.addr = g.K2GTDSES
  AND w.sid = s.indx
  AND s2.sid = w.sid;
```

---

## Reference Links

- Oracle DBA Scripts (locks): https://github.com/gwenshap/Oracle-DBA-Scripts/blob/master/locks.sql
- Locks and blocking sessions: http://www.oracle-ckpt.com/scripts-for-locks-and-blocking-sessions/

---

Back to [Main Index](README.md)
