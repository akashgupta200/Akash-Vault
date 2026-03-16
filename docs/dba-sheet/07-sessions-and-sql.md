---
layout: default
title: "Sessions and SQL Monitoring Reference"
parent: "DBA Reference Sheet"
nav_order: 7
---

# Sessions and SQL Monitoring Reference

Sessions, SQL monitoring, wait events, kill sessions, SQL profiles and baselines.

---

## Sessions

### Total user count on database

```sql
SET LINES 750 PAGES 9999
BREAK ON REPORT
COMPUTE SUM OF tot ON REPORT
COMPUTE SUM OF active ON REPORT
COMPUTE SUM OF inactive ON REPORT
COL username FOR A50

SELECT DECODE(username,NULL,'INTERNAL',USERNAME) Username,
       COUNT(*) TOT,
       COUNT(DECODE(status,'ACTIVE',STATUS)) ACTIVE,
       COUNT(DECODE(status,'INACTIVE',STATUS)) INACTIVE
FROM gv$session
WHERE status IN ('ACTIVE','INACTIVE')
GROUP BY username;
```

### User session details (RAC)

```sql
SET LINESIZE 750 PAGES 9999
COLUMN box FORMAT A30
COLUMN spid FORMAT A10
COLUMN username FORMAT A30
COLUMN program FORMAT A30
COLUMN os_user FORMAT A20
COL LOGON_TIME FOR A20

SELECT b.inst_id, b.sid, b.serial#, a.spid, SUBSTR(b.machine,1,30) box,
       TO_CHAR(b.logon_time, 'dd-mon-yyyy hh24:mi:ss') logon_time,
       SUBSTR(b.username,1,30) username, SUBSTR(b.osuser,1,20) os_user,
       SUBSTR(b.program,1,30) program, status, b.last_call_et AS last_call_et_secs, b.sql_id
FROM gv$session b, gv$process a
WHERE b.paddr = a.addr AND a.inst_id = b.inst_id AND type='USER'
ORDER BY b.inst_id, b.sid;
```

**Alternative (standalone):**

```sql
SELECT b.sid, b.serial#, a.spid, SUBSTR(b.machine,1,30) box, b.logon_time logon_date,
       TO_CHAR(b.logon_time, 'hh24:mi:ss') logon_time,
       SUBSTR(b.username,1,30) username, SUBSTR(b.osuser,1,20) os_user,
       SUBSTR(b.program,1,30) program, status, b.last_call_et AS last_call_et_secs, b.sql_id
FROM v$session b, v$process a
WHERE b.paddr = a.addr AND type='USER'
ORDER BY b.sid;
```

---

## SQL Monitoring

### v$sql_monitor (executing only)

```sql
SET LINES 1000 PAGES 9999
COLUMN sid FORMAT 9999
COLUMN serial FORMAT 999999
COLUMN status FORMAT A15
COLUMN username FORMAT A10
COLUMN sql_text FORMAT A80

SELECT status, inst_id, sid, SESSION_SERIAL# AS Serial, username, sql_id, SQL_PLAN_HASH_VALUE,
       MODULE, program, TO_CHAR(sql_exec_start,'dd-mon-yyyy hh24:mi:ss') AS sql_exec_start,
       ROUND(elapsed_time/1000000) AS "Elapsed (s)", ROUND(cpu_time/1000000) AS "CPU (s)",
       SUBSTR(sql_text,1,30) sql_text
FROM gv$sql_monitor
WHERE status='EXECUTING' AND module NOT LIKE '%emagent%'
ORDER BY sql_exec_start DESC;
```

### SQL Monitor report for sql_id (like OEM)

```sql
SET PAGESIZE 0 ECHO OFF TIMING OFF LINESIZE 1000 TRIMSPOOL ON TRIM ON LONG 2000000 LONGCHUNKSIZE 2000000
SELECT DBMS_SQLTUNE.REPORT_SQL_MONITOR(sql_id=>'&sql_id', report_level=>'ALL', type=>'TEXT')
FROM dual;
```

**Missing statements in SQL Monitoring:** Increase limits:

```sql
ALTER SESSION SET "_SQLMON_MAX_PLAN"=4020;
ALTER SESSION SET "_SQLMON_MAX_PLANLINES"=4000;
```

---

## Long Running Queries

### Session longops

```sql
SELECT a.sid, RPAD(a.opname,30), a.sofar, a.totalwork, a.ELAPSED_SECONDS,
       ROUND(((a.sofar)*100)/a.totalwork,3) "%_COMPLETED",
       RPAD(a.username,10) username, a.SQL_HASH_VALUE, B.STATUS
FROM gv$session_longops a, gv$session b
WHERE a.sid=b.sid AND a.sid=&sid AND a.sofar <> a.totalwork;
```

**Alternative (all longops):**

```sql
SELECT inst_id, sid, serial#, sql_id, opname, username, target, sofar, totalwork,
       start_time, last_update_time, ROUND(time_remaining/60,2) "REMAIN MINS",
       ROUND(elapsed_seconds/60,2) "ELAPSED MINS",
       ROUND(SOFAR/TOTALWORK*100,2) "%_COMPLETE", message
FROM gv$session_longops
WHERE OPNAME NOT LIKE 'RMAN%' AND OPNAME NOT LIKE '%aggregate%'
  AND TOTALWORK != 0 AND sofar<>totalwork AND time_remaining > 0;
```

---

## Wait Events

### Wait event for a SID (currently waiting)

```sql
COL WAIT_CLASS FOR A10
SELECT sw.inst_id, NVL(s.username, '(oracle)') AS username, s.sid, s.serial#,
       sw.event, sw.wait_class, sw.wait_time, sw.seconds_in_wait, sw.state
FROM gv$session_wait sw, gv$session s
WHERE s.sid = sw.sid AND s.inst_id=sw.inst_id AND s.sid=&sid
ORDER BY sw.seconds_in_wait DESC;
```

### Overall waits for SID

```sql
SELECT NVL(s.username, '(oracle)') AS username, s.sid, s.serial#, se.event,
       se.total_waits, se.total_timeouts, se.time_waited, se.average_wait, se.max_wait
FROM v$session_event se, v$session s
WHERE s.sid = se.sid AND s.sid = &Session_ID
ORDER BY se.time_waited DESC;
```

### Time model for SID

```sql
SELECT stat_name, value FROM V$SESS_TIME_MODEL WHERE sid = &sid ORDER BY value DESC;
```

### Database level wait ratios/events

```sql
SELECT METRIC_NAME, VALUE
FROM V$SYSMETRIC
WHERE METRIC_NAME IN ('Database CPU Time Ratio', 'Database Wait Time Ratio')
  AND INTSIZE_CSEC = (SELECT MAX(INTSIZE_CSEC) FROM V$SYSMETRIC);
```

### Wait percentage (with CPU)

```sql
SELECT wait_class time_cat, ROUND(time_secs, 2) time_secs,
       ROUND(time_secs * 100 / SUM(time_secs) OVER (), 2) pct
FROM (SELECT wait_class, SUM(time_waited_micro) / 1000000 time_secs
      FROM gv$system_event
      WHERE wait_class <> 'Idle' AND time_waited > 0
      GROUP BY wait_class
      UNION
      SELECT 'CPU', ROUND(SUM(VALUE)/1000000, 2) time_secs
      FROM gv$sys_time_model
      WHERE stat_name IN ('background cpu time', 'DB CPU'))
ORDER BY time_secs DESC;
```

---

## Currently Running SQL

### v$sqlarea/v$sql (all sessions)

```sql
SELECT a.inst_id, a.sid, a.username, b.PARSING_SCHEMA_NAME, a.module, a.sql_id, a.sql_child_number child,
       b.plan_hash_value, TO_CHAR(a.sql_exec_start, 'dd-Mon-yyyy hh24:mi:ss') sql_exec_start,
       (sysdate-sql_exec_start)*24*60*60 SECS, b.rows_processed, a.status, SUBSTR(b.sql_text,1,50) sql_text
FROM gv$session a, gv$sqlarea b
WHERE a.sql_hash_value = b.hash_value AND a.sql_address = b.address
  AND a.module NOT LIKE '%emagent%' AND a.module NOT LIKE '%oraagent.bin%'
  AND a.username IS NOT NULL
ORDER BY a.status;
```

### v$sqlarea for sql_id

```sql
SELECT h.inst_id, h.sql_id, h.plan_hash_value, h.executions, h.rows_processed,
       TO_CHAR(ROUND(h.elapsed_time/1e6, 3), '999,990.000') et_secs,
       TO_CHAR(ROUND(h.cpu_time/1e6, 3), '999,990.000') cpu_secs,
       TO_CHAR(ROUND(h.USER_IO_WAIT_TIME/1e6, 3), '999,990.000') io_secs
FROM gv$sqlarea h
WHERE h.sql_id = '&sql_id'
ORDER BY source, snap_id, snap_time;
```

---

## Process and SID Lookup

### Finding SID for a PID

```sql
SELECT spid, p.pid, s.sid, s.serial#, p.program
FROM v$session s, v$process p
WHERE paddr=addr AND p.pid=30849
ORDER BY p.pid;
```

### Finding OS PID for a DB SID

```sql
SELECT spid "host-pid", p.pid, s.sid, s.serial#, p.program, s.machine
FROM gv$session s, gv$process p
WHERE paddr=addr AND s.sid=&sid
ORDER BY p.pid;
```

### Finding DB SID from OS SPID

```sql
SELECT s.sid, s.serial#, s.username, TO_CHAR(s.logon_time,'DD-MON HH24:MI:SS') logon_time,
       p.pid oraclepid, p.spid "ServerPID", s.process "ClientPID",
       s.program clientprogram, s.module, s.machine, s.osuser, s.status, s.last_call_et
FROM gv$session s, gv$process p
WHERE p.spid=NVL('&unix_process',' ') AND s.paddr=p.addr
ORDER BY s.sid;
```

### Finding own SID and serial

```sql
SELECT sys_context('USERENV', 'SID') OwnSID FROM dual;
SELECT DISTINCT sid OwnSID FROM v$mystat;
```

---

## Killing Sessions

### Kill a SID

```sql
SELECT 'alter system kill session ' || '''' || sid || ',' || serial# ||',@'|| inst_id || '''' || ' immediate;'
FROM gv$session
WHERE sid='&sid';
```

### Killing old sessions (1 day or 4 hours inactive)

```sql
-- 1 day old
SELECT ' alter system kill session '''||sid||''','''||serial#||''',''@'||inst_id||''' immediate; '
FROM gv$session
WHERE username IN ('SCHEMA1','SCHEMA2') AND logon_time < sysdate-1 AND status='INACTIVE';

-- 4 hours inactive
SELECT ' alter system kill session '''||sid||','||serial#||',@'||inst_id||''' immediate; '
FROM gv$session
WHERE username IN ('SCHEMA1','SCHEMA2','SCHEMA3') AND status='INACTIVE' AND last_call_et > 4*60*60;
```

### Determine if killed session is rolling back

```sql
SELECT a.sid, a.username, b.xidusn rollback_seg_no, b.used_urec undo_records, b.used_ublk undo_blocks
FROM gv$session a, gv$transaction b
WHERE a.saddr = b.ses_addr;
```

---

## SQL Profiles & Baselines

### Check SQL profiles for sql_id

```sql
SELECT NAME, SIGNATURE, STATUS, FORCE_MATCHING FROM dba_sql_profiles;
```

### Enable/Disable/Drop SQL profile

```sql
EXEC DBMS_SQLTUNE.ALTER_SQL_PROFILE('coe_5273fz2cqkk80_3455548535','STATUS','DISABLED');
EXEC DBMS_SQLTUNE.DROP_SQL_PROFILE('coe_5273fz2cqkk80_3455548535');
```

### SQL baselines - check

```sql
SELECT SQL_HANDLE, PLAN_NAME, ENABLED, ACCEPTED, FIXED, sql_text
FROM dba_sql_plan_baselines;
```

### SQL baselines - drop

```sql
SET SERVEROUTPUT ON
DECLARE
  i NATURAL;
BEGIN
  i := dbms_spm.drop_sql_plan_baseline('SQL_b3d69637aa86a8ca');
  dbms_output.put_line(i);
END;
/
```

### Fix baseline - load from cursor

```sql
VARIABLE sqlid NUMBER;
EXECUTE :sqlid := DBMS_SPM.LOAD_PLANS_FROM_CURSOR_CACHE(sql_id=>'5qbbnv0abm2vx', PLAN_HASH_VALUE=>4197102931, SQL_HANDLE=>'SQL_d3318f33dfac7bc2');
```

### Fix PLAN_HASH_VALUE

```sql
VARIABLE x NUMBER;
BEGIN
  :x := dbms_spm.load_plans_from_cursor_cache(sql_id=>'&sql_id', plan_hash_value=>&plan_hash, fixed=>'YES');
END;
/
```

### Purge/flush old plan hash from memory

```sql
BEGIN
  FOR i IN (SELECT address, hash_value FROM gv$sqlarea WHERE sql_id = '&sql_id.')
  LOOP
    SYS.DBMS_SHARED_POOL.PURGE(i.address||','||i.hash_value, 'C');
  END LOOP;
END;
/
```

---

## Explain Plan

### Explain plan for sql_id (shared pool)

```sql
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR('&sql_id', &childnumber, 'ALLSTATS LAST +PEEKED_BINDS +PROJECTION +ALIAS +OUTLINE +PREDICATE +COST +BYTES'));
```

### Explain plan from AWR

```sql
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_AWR('&sql_id', NULL, NULL, 'ALLSTATS LAST'));
```

---

## Identify SQL

### Find sql_id by text

```sql
SELECT sql_id, TO_CHAR(LAST_LOAD_TIME, 'dd-mon-yyyy hh24:mi:ss') LAST_LOAD_TIME,
       exact_matching_signature, SQL_TEXT
FROM v$sqlarea
WHERE UPPER(sql_text) LIKE '%DUMMY%'
ORDER BY UPPER(sql_text);
```

### Get sql_text for sql_id

```sql
SET LONG 20000
SET LINES 750 PAGES 9999
SELECT sql_text FROM dba_hist_sqltext WHERE sql_id = '&SQL_ID';
SELECT sql_text FROM gv$sqlarea WHERE sql_id = '&SQL_ID';
```

### Bind variables for sql_id

```sql
COL VALUE_STRING FOR A50
SELECT NAME, POSITION, DATATYPE_STRING, VALUE_STRING
FROM gv$sql_bind_capture
WHERE sql_id='&sql_id';
```

---

## DML Progress

### Check progress of DML statements

```sql
COL sql_text FOR A60
SELECT rows_processed "Total Rows Processed",
       ROUND((SYSDATE - TO_DATE(first_load_time, 'yyyy-mm-dd hh24:mi:ss')) * 24 * 60, 1) "Total Time (Min)",
       TRUNC(rows_processed/((SYSDATE - TO_DATE(first_load_time, 'yyyy-mm-dd hh24:mi:ss')) * 24 * 60)) "Rows/Min",
       SUBSTR(sql_text, 1, 60) sql_text
FROM gv$sqlarea
WHERE SQL_ID='&SQL_ID' AND open_versions > 0 AND rows_processed > 0;
```

---

## Parallelism

### Enable parallelism at session level

```sql
ALTER SESSION FORCE PARALLEL QUERY;
ALTER SESSION ENABLE PARALLEL DML;
```

### Parallelism hints

```sql
/*+ PARALLEL */
/*+ PARALLEL 8 */
/*+ NOPARALLEL */
```

---

**Back to [Main Index](README.md)**
