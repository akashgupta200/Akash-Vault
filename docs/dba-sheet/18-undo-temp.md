---
layout: default
title: "Undo & Temp"
parent: "DBA Reference Sheet"
nav_order: 18
---

# Undo & Temp

Undo tablespace management, temp tablespace usage, and rollback monitoring.

---

## Undo Parameters

### v$parameter for undo

```sql
SELECT inst_id, name, value
FROM gv$parameter
WHERE name LIKE '%undo%';
```

---

## Undo Tablespace Usage

### UNDO tablespace free (including expired segments)

```sql
SELECT d.tablespace_name,
       ROUND(((NVL(f.bytes,0) + (a.maxbytes - a.bytes))/1048576 + u.exp_space),2) AS max_free_mb,
       ROUND(((a.bytes - (NVL(f.bytes,0) + (1024*1024*u.exp_space)))*100/a.maxbytes),2) used_pct
FROM sys.dba_tablespaces d,
     (SELECT tablespace_name, SUM(bytes) bytes, SUM(GREATEST(maxbytes,bytes)) maxbytes
      FROM sys.dba_data_files GROUP BY tablespace_name) a,
     (SELECT tablespace_name, SUM(bytes) bytes FROM sys.dba_free_space GROUP BY tablespace_name) f,
     (SELECT tablespace_name, SUM(blocks)*8/(1024) exp_space
      FROM dba_undo_extents WHERE status NOT IN ('ACTIVE','UNEXPIRED') GROUP BY tablespace_name) u
WHERE d.tablespace_name = a.tablespace_name(+)
  AND d.tablespace_name = f.tablespace_name(+)
  AND d.tablespace_name = u.tablespace_name
  AND d.contents = 'UNDO'
  AND u.tablespace_name = (SELECT UPPER(value) FROM v$parameter WHERE name = 'undo_tablespace');
```

### Check for expired segments in undo

```sql
SELECT status, COUNT(1)
FROM dba_undo_extents
GROUP BY status;
```

### Active, expired, unexpired undo (by size and percentage)

```sql
SELECT status,
  ROUND(sum_bytes / (1024*1024), 0) AS MB,
  ROUND((sum_bytes / undo_size) * 100, 0) AS PERC
FROM (
  SELECT status, SUM(bytes) sum_bytes
  FROM dba_undo_extents
  GROUP BY status
),
(
  SELECT SUM(a.bytes) undo_size
  FROM dba_tablespaces c
  JOIN v$tablespace b ON b.name = c.tablespace_name
  JOIN v$datafile a ON a.ts# = b.ts#
  WHERE c.contents = 'UNDO' AND c.status = 'ONLINE'
);
```

**Alternative (count by status):**

```sql
SELECT COUNT(status) FROM dba_undo_extents WHERE status = 'EXPIRED';
SELECT COUNT(status) FROM dba_undo_extents WHERE status = 'UNEXPIRED';
SELECT COUNT(status) FROM dba_undo_extents WHERE status = 'ACTIVE';
```

### Active, expired, unexpired undo by size in MB

```sql
SELECT status, TABLESPACE_NAME, SUM(bytes)/1024/1024||'MB'
FROM DBA_UNDO_EXTENTS
GROUP BY status, TABLESPACE_NAME;
```

**Alternative:**

```sql
SELECT status, SUM(bytes)/1024/1024||'MB'
FROM DBA_UNDO_EXTENTS
GROUP BY status;
```

---

## Undo Usage by Session

### User-generated UNDO

```sql
COL sql_text FORMAT a40
SET lines 130
SELECT sq.sql_text sql_text, t.USED_UREC Records, t.USED_UBLK Blocks,
       (t.USED_UBLK*8192/1024) KBytes
FROM v$transaction t, v$session s, v$sql sq
WHERE t.addr = s.taddr
  AND s.sql_id = sq.sql_id
  AND s.username = '<user>';
```

### UNDO tablespace usage by user

```sql
SELECT TO_CHAR(s.sid)||','||TO_CHAR(s.serial#) sid_serial,
       NVL(s.username, 'None') orauser,
       s.program,
       r.name undoseg,
       t.used_ublk * TO_NUMBER(x.value)/1024/1024||'M' "Undo"
FROM sys.v_$rollname r,
     sys.v_$session s,
     sys.v_$transaction t,
     sys.v_$parameter x
WHERE s.taddr = t.addr
  AND r.usn = t.xidusn(+)
  AND x.name = 'db_block_size';
```

**Alternative (with segment size and logon time):**

```sql
SELECT v$session.SID, v$session.SERIAL#, r.NAME "Undo Segment Name", dba_seg.size_mb,
  DECODE(TRUNC(SYSDATE - LOGON_TIME), 0, NULL, TRUNC(SYSDATE - LOGON_TIME) || ' Days' || ' + ') ||
  TO_CHAR(TO_DATE(TRUNC(MOD(SYSDATE-LOGON_TIME,1)*86400), 'SSSSS'), 'HH24:MI:SS') LOGON,
  p.SPID, v$session.process, v$session.USERNAME, v$session.STATUS, v$session.OSUSER,
  v$session.MACHINE, v$session.PROGRAM, v$session.module, action
FROM v$lock l, v$process p, v$rollname r, v$session,
  (SELECT segment_name, ROUND(bytes/(1024*1024),2) size_mb
   FROM dba_segments WHERE segment_type = 'TYPE2 UNDO' ORDER BY bytes DESC) dba_seg
WHERE l.SID = p.pid(+)
  AND v$session.SID = l.SID
  AND TRUNC(l.id1(+)/65536) = r.usn
  AND l.TYPE(+) = 'TX'
  AND l.lmode(+) = 6
  AND r.NAME = dba_seg.segment_name
ORDER BY size_mb DESC;
```

**Alternative (compact):**

```sql
SELECT DISTINCT RPAD(s.sid,3) "SID", S.USERNAME, E.SEGMENT_NAME, T.START_TIME "Start",
       RPAD(T.STATUS,9) "Status", ROUND((t.used_ublk*8)/1024) "Size(MB)"
FROM DBA_DATA_FILES DF, DBA_EXTENTS E, V$SESSION S, V$TRANSACTION T
WHERE DF.TABLESPACE_NAME = E.TABLESPACE_NAME
  AND DF.FILE_ID = UBAFIL
  AND S.SADDR = T.SES_ADDR
  AND T.UBABLK BETWEEN E.BLOCK_ID AND E.BLOCK_ID+E.BLOCKS
  AND E.SEGMENT_TYPE IN ('ROLLBACK','TYPE2 UNDO');
```

### Running SQLs in undo segments

```sql
COL o FORMAT a10
COL u FORMAT a10
SELECT osuser o, username u, sid, segment_name s, SUBSTR(sa.sql_text,1,200) txt
FROM v$session s, v$transaction t, dba_rollback_segs r, v$sqlarea sa
WHERE s.taddr = t.addr
  AND t.xidusn = r.segment_id(+)
  AND s.sql_address = sa.address(+)
  AND SUBSTR(sa.sql_text,1,200) IS NOT NULL
ORDER BY 3;
```

---

## Undo Advisor

### Undo tablespace advice

```sql
SELECT MAX(maxquerylen) AS "maxquerylen",
       dbms_undo_adv.required_undo_size(MAX(maxquerylen)) AS "its required UNDOTBS_n MB"
FROM gv$undostat;

SELECT dbms_undo_adv.required_retention AS "required_retention",
       dbms_undo_adv.required_undo_size(dbms_undo_adv.required_retention) AS "its required UNDOTBS_n MB"
FROM dual;
```

### Undo advisor (actual vs optimal retention)

```sql
SELECT d.undo_size/(1024*1024) "ACTUAL UNDO SIZE [MByte]",
       SUBSTR(e.value,1,25) "UNDO RETENTION [Sec]",
       ROUND((d.undo_size / (TO_NUMBER(f.value) * g.undo_block_per_sec))) "OPTIMAL UNDO RETENTION [Sec]"
FROM (
  SELECT SUM(a.bytes) undo_size
  FROM v$datafile a, v$tablespace b, dba_tablespaces c
  WHERE c.contents = 'UNDO' AND c.status = 'ONLINE'
    AND b.name = c.tablespace_name AND a.ts# = b.ts#
) d,
v$parameter e, v$parameter f,
(
  SELECT MAX(undoblks/((end_time-begin_time)*3600*24)) undo_block_per_sec
  FROM v$undostat
) g
WHERE e.name = 'undo_retention' AND f.name = 'db_block_size';
```

**Alternative (required size for retention):**

```sql
SELECT dbms_undo_adv.required_undo_size(1800, SYSDATE-30, SYSDATE) FROM dual;
```

---

## Monitoring Undo Segments

### Max query length and tuned retention

```sql
SELECT MAX(maxquerylen), MAX(tuned_undoretention) FROM v$undostat;
SELECT MAX(maxquerylen), MAX(tuned_undoretention) FROM dba_hist_undostat;
```

**Alternative (longest queries):**

```sql
SELECT maxquerysqlid, maxquerylen FROM dba_hist_undostat ORDER BY maxquerylen DESC;
SELECT maxqueryid, maxquerylen FROM v$undostat ORDER BY maxquerylen DESC;
```

### How long will rollback take?

Use the script at [williamrobertson.net – undo tracker](http://www.williamrobertson.net/documents/undo_tracker.shtml). It samples undo records over time and estimates minutes/hours to complete rollback.

---

## Undo Not Releasing Unexpired Segments

### Check undo-related parameters

```sql
SELECT a.ksppinm "Parameter", b.ksppstvl "Session Value", c.ksppstvl "Instance Value"
FROM sys.x$ksppi a, sys.x$ksppcv b, sys.x$ksppsv c
WHERE a.indx = b.indx AND a.indx = c.indx
  AND a.ksppinm IN ('_undo_autotune','_smu_debug_mode','_highthreshold_undoretention',
                    'undo_tablespace','undo_retention','undo_management')
ORDER BY 2;
```

---

## Recreating Undo Tablespace

### Check active sessions using undo before switch

```sql
SELECT a.name, b.status, d.username, d.sid, d.serial#
FROM v$rollname a, v$rollstat b, v$transaction c, v$session d
WHERE a.name IN (SELECT segment_name FROM dba_segments WHERE tablespace_name = 'UNDOTBS1')
  AND a.usn = b.usn
  AND a.usn = c.xidusn
  AND c.ses_addr = d.saddr;
```

---

## ORA-30019: Illegal Rollback Segment Operation

```sql
ALTER SYSTEM SET UNDO_SUPPRESS_ERRORS=TRUE SCOPE=both;
```

If TRUE, undo errors are suppressed; some jobs may complete successfully.

---

## Temp Tablespace

### Sort space usage by session

```sql
SET lines 1500 pages 9999
COLUMN sid FORMAT 9999
COLUMN username FORMAT a15
COLUMN SQL_EXEC_START FOR a21
COLUMN sql_text FORMAT a50
COLUMN module FORMAT a35
BREAK ON report
COMPUTE SUM OF MB_USED ON report
SELECT a.inst_id, a.username, a.sid, a.serial#, a.osuser,
       (b.blocks*d.block_size)/1048576 MB_USED, a.sql_id, a.sql_child_number child,
       c.plan_hash_value, TO_CHAR(a.sql_exec_start, 'dd-Mon-yyyy hh24:mi:ss') sql_exec_start,
       c.rows_processed, a.status, SUBSTR(c.sql_text,1,50) sql_text
FROM gv$session a, gv$tempseg_usage b, gv$sqlarea c,
     (SELECT block_size FROM dba_tablespaces WHERE tablespace_name='TEMP') d
WHERE b.tablespace = 'TEMP'
  AND a.saddr = b.session_addr
  AND c.address = a.sql_address
  AND a.inst_id = b.inst_id
ORDER BY 6 DESC;
```

**Alternative (single instance):**

```sql
COL sid_serial FOR a20
SELECT S.sid || ',' || S.serial# sid_serial, S.username, S.osuser, P.spid, S.module,
       S.program, SUM(T.blocks) * TBS.block_size / 1024 / 1024 mb_used, T.tablespace,
       COUNT(*) sort_ops
FROM v$sort_usage T, v$session S, dba_tablespaces TBS, v$process P
WHERE T.session_addr = S.saddr
  AND S.paddr = P.addr
  AND T.tablespace = TBS.tablespace_name
GROUP BY S.sid, S.serial#, S.username, S.osuser, P.spid, S.module,
         S.program, TBS.block_size, T.tablespace
ORDER BY sid_serial;
```

**Alternative (temp MB for specific SID):**

```sql
SELECT ROUND(SUM(tempseg_size)/1048576) temp_mb
FROM gv$sql_workarea_active
WHERE sid = &sid;
```

### History of temp tablespace usage

```sql
SELECT sql_id, SQL_PLAN_HASH_VALUE, MAX(TEMP_SPACE_ALLOCATED)/(1024*1024*1024) gig
FROM DBA_HIST_ACTIVE_SESS_HISTORY
WHERE sample_time > SYSDATE-10
  AND TEMP_SPACE_ALLOCATED > (10*1024*1024*1024)
GROUP BY sql_id, SQL_PLAN_HASH_VALUE
ORDER BY sql_id;
```

### Shrink tempfile

```sql
ALTER TABLESPACE temp SHRINK TEMPFILE '+DATA_KRONIA/p1kronia/tempfile/temp.313.868020865' KEEP 500M;
```

---

## Reference Links

- [vsbabu.org – Undo/Temp section](http://vsbabu.org/oracle/sect07.html)
- [Change or switch undo tablespace](https://community.oracle.com/thread/1098943?tstart=0)
- [ORA-1555 resolution](https://community.oracle.com/community/support/support-blogs/database-support-blog/blog/2015/12/10/ora-1555-do-you-know-how-to-resolve-this-issue)
- [Undo tracker script](http://www.williamrobertson.net/documents/undo_tracker.shtml)

---

Back to [Main Index](README.md)
