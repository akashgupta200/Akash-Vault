---
layout: default
title: "Performance Tuning Reference"
parent: "DBA Reference Sheet"
nav_order: 8
---

# Performance Tuning Reference

Performance tuning, capacity planning, SGA/PGA, buffer cache, I/O, indexes.

---

## Capacity Planning

### Tablespace growth rate (weekly)

```sql
WITH a AS (
  SELECT name, ts#, block_size FROM v$tablespace, dba_tablespaces WHERE name = tablespace_name
),
c AS (
  SELECT a.name, MIN(snap_id) Begin_snap_ID, MAX(snap_id) End_Snap_ID,
         MIN(TRUNC(TO_DATE(rtime,'MM/DD/YYYY HH24:MI:SS'))) begin_time,
         MAX(TRUNC(TO_DATE(rtime,'MM/DD/YYYY HH24:MI:SS'))) End_time
  FROM dba_hist_tbspc_space_usage, a
  WHERE tablespace_id = a.ts#
  GROUP BY a.name
),
d AS (
  SELECT a.name, ROUND((dh.tablespace_size*A.BLOCK_SIZE)/1024/1024,2) begin_allocated_space,
         ROUND((dh.tablespace_usedsize*A.BLOCK_SIZE)/1024/1024,2) begin_Used_space
  FROM dba_hist_tbspc_space_usage dh, c, a
  WHERE dh.snap_id = c.Begin_snap_ID AND a.ts# = dh.tablespace_id AND a.name = c.name
),
e AS (
  SELECT a.name, ROUND((tablespace_size*a.block_size)/1024/1024,2) End_allocated_space,
         ROUND((tablespace_usedsize*a.block_size)/1024/1024,2) End_Used_space
  FROM dba_hist_tbspc_space_usage c, a
  WHERE snap_id = c.End_Snap_ID AND a.ts# = dba_hist_tbspc_space_usage.tablespace_id AND a.name = c.name
)
SELECT e.name, TO_CHAR(c.begin_time,'DD-MON-YYYY') Begin_time,
       d.begin_allocated_space "Begin_allocated_space(MB)", d.begin_Used_space "Begin_Used_space(MB)",
       TO_CHAR(c.End_time,'DD-MON-YYYY') End_Time,
       e.End_allocated_space "End_allocated_space(MB)", e.End_Used_space "End_Used_space(MB)",
       (e.End_Used_space - d.begin_Used_space) "Total Growth(MB)",
       (c.End_time - c.begin_time) "No.of days",
       ROUND(((e.End_Used_space - d.begin_Used_space)/(c.End_time - c.begin_time))*30,2) "Growth(MB)_in_next30_days"
FROM e, d, c
WHERE e.name = d.name AND e.name = c.name AND (e.End_Used_space - d.begin_Used_space) > 0
ORDER BY 1;
```

### Tablespace growth rate (hourly)

```sql
SET LINESIZE 750 PAGES 9999
COL NAME FOR A30

SELECT DISTINCT DHSS.SNAP_ID, VTS.NAME,
       TO_CHAR(DHSS.END_INTERVAL_TIME, 'DD-MM HH:MI') AS SNAP_Time,
       ROUND((DHTS.TABLESPACE_USEDSIZE*8192)/1024/1024) AS USED_MB,
       ROUND((DHTS.TABLESPACE_SIZE*8192)/1024/1024) AS SIZE_MB
FROM DBA_HIST_TBSPC_SPACE_USAGE DHTS, V$TABLESPACE VTS, DBA_HIST_SNAPSHOT DHSS
WHERE VTS.TS#=DHTS.TABLESPACE_ID AND DHTS.SNAP_ID=DHSS.SNAP_ID
  AND DHSS.INSTANCE_NUMBER=1 AND NAME='&TABLESPACE_NAME'
ORDER BY 1;
```

### SGA allocation guideline

- **45% of RAM** for SGA+PGA+background processes
- **Fixed background processes** ≈ 40 MB
- **SGA** ≈ 160 MB
- **PGA** ≈ 16 MB

### DOP for parallel session

```
PARALLEL_THREADS_PER_CPU * CPU_COUNT (number of CPU cores) * ACTIVE_INSTANCE_COUNT (number of active instances)
```

### Transaction per second

```sql
WITH hist_snaps AS (
  SELECT instance_number, snap_id, ROUND(begin_interval_time,'MI') datetime,
         (begin_interval_time + 0 - LAG(begin_interval_time + 0) OVER (PARTITION BY dbid, instance_number ORDER BY snap_id)) * 86400 diff_time
  FROM dba_hist_snapshot
),
hist_stats AS (
  SELECT dbid, instance_number, snap_id, stat_name,
         VALUE - LAG(VALUE) OVER (PARTITION BY dbid, instance_number, stat_name ORDER BY snap_id) delta_value
  FROM dba_hist_sysstat
  WHERE stat_name IN ('user commits', 'user rollbacks')
)
SELECT datetime, ROUND(SUM(delta_value) / 3600, 2) "Transactions/s"
FROM hist_snaps sn, hist_stats st
WHERE st.instance_number = sn.instance_number AND st.snap_id = sn.snap_id AND diff_time IS NOT NULL
GROUP BY datetime
ORDER BY 1 DESC;
```

---

## Tuning

### CPU wait time

```sql
SELECT metric_name, ROUND(value,2)
FROM v$sysmetric
WHERE metric_name IN ('Database CPU Time Ratio', 'Database Wait Time Ratio')
  AND intsize_csec = (SELECT MAX(INTSIZE_CSEC) FROM V$SYSMETRIC);
```

### Format explain plan

```sql
COLUMN QUERY_PLAN FORMAT A60
SELECT LPAD(' ', 2 * LEVEL) || operation || ' ' || options || ' ' || object_name query_plan
FROM plan_table
CONNECT BY PRIOR id = parent_id
START WITH id = 1
ORDER BY id;
```

### Finding current trace file

```sql
SET LINESIZE 100
COLUMN trace_file FORMAT A60

SELECT s.sid, s.serial#,
       pa.value || '/' || LOWER(SYS_CONTEXT('userenv','instance_name')) || '_ora_' || p.spid || '.trc' AS trace_file
FROM v$session s, v$process p, v$parameter pa
WHERE pa.name = 'user_dump_dest' AND s.paddr = p.addr
  AND s.audsid = SYS_CONTEXT('USERENV', 'SESSIONID');
```

---

## Shared Pool

### Size of shared pool

Default: 8,388,608 bytes (8 MB). Set via `SHARED_POOL_SIZE initialization parameter`.

### Library cache hit ratio

```sql
SELECT SUM(pins) "Executions", SUM(reloads) "Cache Misses", SUM(reloads)/SUM(pins)
FROM v$librarycache;
```

Reloads should be less than 1% of pins. If greater, increase the value of `SHARED_POOL_SIZE`.

### Keeping large objects in SGA

```sql
SELECT * FROM v$db_object_cache
WHERE sharable_mem > 10000
  AND (type='PACKAGE' OR type='PACKAGE BODY' OR type='FUNCTION' OR type='PROCEDURE')
  AND KEPT='NO';

EXECUTE dbms_shared_pool.keep('package_name');
```

### Tuning data dictionary cache

Keep the ratio of GETMISSES to GETS less than 15%:

```sql
SELECT parameter, gets, getmisses FROM v$rowcache;
```

---

## Buffer Cache

### DB cache advice

```sql
SHOW PARAMETER DB_CACHE_ADVICE;
```

```sql
SELECT size_for_estimate, buffers_for_estimate, estd_physical_read_factor, estd_physical_reads
FROM V$DB_CACHE_ADVICE
WHERE name = 'DEFAULT'
  AND block_size = (SELECT value FROM v$parameter WHERE name = 'db_block_size')
  AND advice_status = 'ON';
```

### Calculating hit ratio for multiple pools

```sql
SELECT name, 1 - (physical_reads / (db_block_gets + consistent_gets)) "HIT_RATIO"
FROM sys.v$buffer_pool_statistics
WHERE db_block_gets + consistent_gets > 0;
```

### How many blocks used by each object

```sql
SELECT owner#, name, COUNT(*) blocks
FROM v$cache
GROUP BY owner#, name;
```

---

## Redo Log Buffer

### Log buffer space wait

```sql
SELECT sid, event, seconds_in_wait, state
FROM v$session_wait
WHERE event = 'log buffer space%';
```

### Redo buffer allocation retries

```sql
SELECT name, value FROM v$sysstat
WHERE name IN ('redo buffer allocation retries', 'redo entries');
```

Redo Buffer Allocation Retries should be near 0; less than 1% of redo entries.

---

## I/O Tuning

### I/O by datafile

```sql
COL name FOR A70
SELECT phyrds, phywrts, d.name
FROM v$datafile d, v$filestat f
WHERE d.file#=f.file#
ORDER BY d.name;
```

### DB sequential read / contention

For table fragmentation:

```sql
ALTER TABLE table_name MOVE;
```

---

## Sorting

### Disk vs memory sorts

```sql
SELECT disk.value "Disk", mem.value "Mem", (disk.value/mem.value)*100 "Ratio"
FROM v$sysstat mem, v$sysstat disk
WHERE mem.name = 'sorts (memory)' AND disk.name = 'sorts (disk)';
```

---

## Undo

### Space requirement for undo retention

```
Undo Space = (UNDO_RETENTION * (Undo Blocks Per Second * DB_BLOCK_SIZE)) + DB_BLOCK_SIZE
```

### Size undo tablespace

```sql
SELECT (RD * (UPS * OVERHEAD) + OVERHEAD) AS "Bytes"
FROM (SELECT value AS RD FROM v$parameter WHERE name = 'undo_retention'),
     (SELECT (SUM(undoblks) / SUM(((end_time - begin_time) * 86400))) AS UPS FROM v$undostat),
     (SELECT value AS Overhead FROM v$parameter WHERE name = 'db_block_size');
```

---

## Indexes

### Monitoring index space (rebuilding)

```sql
ANALYZE INDEX EMP_NAME_IX VALIDATE STRUCTURE;

SELECT name, (DEL_LF_ROWS_LEN/LF_ROWS_LEN) * 100 AS wastage
FROM index_stats;

ALTER INDEX EMP_NAME_IX REBUILD;
-- Or: ALTER INDEX EMP_NAME_IX COALESCE;
```

Typically rebuild if 15% of index data is deleted.

### Identifying unused indexes

```sql
ALTER INDEX HR.EMP_NAME_IX MONITORING USAGE;
ALTER INDEX HR.EMP_NAME_IX NOMONITORING USAGE;
SELECT INDEX_NAME, USED FROM V$OBJECT_USAGE;
```

---

## Hot Segments

### Top 10 hot segments

```sql
SELECT * FROM (
  SELECT ob.owner, ob.object_name, SUM(b.tch) Touchs
  FROM x$bh b, dba_objects ob
  WHERE b.obj = ob.data_object_id AND b.ts#> 0
  GROUP BY ob.owner, ob.object_name
  ORDER BY SUM(tch) DESC
) WHERE ROWNUM <= 10;
```

### See hot data files (single block read time)

```sql
SELECT t.file_name, t.tablespace_name,
       ROUND(s.singleblkrdtim/s.singleblkrds, 2) AS CS,
       s.READTIM, s.WRITETIM
FROM v$filestat s, dba_data_files t
WHERE s.file# = t.file_id AND ROWNUM <= 10
ORDER BY cs DESC;
```

---

## PGA

### View PGA proposed

```sql
SELECT (SELECT ROUND(value/1024/1024, 0) FROM v$parameter WHERE name = 'pga_aggregate_target') "Current Mb",
       ROUND(pga_target_for_estimate/1024/1024, 0) "Projected Mb",
       ROUND(estd_pga_cache_hit_percentage) "%"
FROM v$pga_target_advice
ORDER BY 2;
```

---

## Lock Contention

### enq: TX - row lock contention

```sql
SELECT sid, sql_text
FROM v$session s, v$sql q
WHERE sid IN (SELECT sid FROM v$session WHERE state = 'WAITING' AND wait_class != 'Idle'
              AND event='enq: TX - row lock contention'
              AND (q.sql_id = s.sql_id OR q.sql_id = s.prev_sql_id));

SELECT blocking_session, sid, serial#, wait_class, seconds_in_wait
FROM v$session
WHERE blocking_session IS NOT NULL
ORDER BY blocking_session;
```

---

## AWR

### Create manual AWR snapshot

```sql
BEGIN
  DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT();
END;
/
```

### I/O latency check

```sql
SELECT Inst_Id, EVENT, TOTAL_WAITS, TIME_WAITED,
       ROUND(100*Total_Waits/Time_Waited) Rate,
       ROUND(10* Time_Waited/Total_Waits,1) Latency
FROM Gv$System_Event
WHERE Event LIKE '%db file sequential read%'
ORDER BY 1;
```

Latency should be within 5-10 ms.

### Single block read time

```sql
SELECT Inst_Id, EVENT, TOTAL_WAITS, TIME_WAITED,
       ROUND(10* Time_Waited/Total_Waits,1) Latency
FROM GV$System_Event
WHERE Event LIKE '%db file sequential read%'
ORDER BY 1;
```

Should be less than 8 ms.

---

## Top Consumers

### Top 10 CPU users

```sql
SELECT ROWNUM AS rank, a.*
FROM (SELECT v.sid, program, v.value / (100 * 60) CPUMins
      FROM v$statname s, v$sesstat v, v$session sess
      WHERE s.name = 'CPU used by this session'
        AND sess.sid = v.sid AND v.statistic#=s.statistic# AND v.value>0
      ORDER BY v.value DESC) a
WHERE ROWNUM < 11;
```

### Top 10 CPU consumers in last 5 minutes

```sql
SELECT * FROM (
  SELECT session_id, session_serial#, COUNT(*)
  FROM v$active_session_history
  WHERE session_state= 'ON CPU' AND sample_time > SYSDATE - INTERVAL '5' MINUTE
  GROUP BY session_id, session_serial#
  ORDER BY COUNT(*) DESC
) WHERE ROWNUM <= 10;
```

### Top 10 waiting sessions in last 5 minutes

```sql
SELECT * FROM (
  SELECT session_id, session_serial#, COUNT(*)
  FROM v$active_session_history
  WHERE session_state='WAITING' AND sample_time > SYSDATE - INTERVAL '5' MINUTE
  GROUP BY session_id, session_serial#
  ORDER BY COUNT(*) DESC
) WHERE ROWNUM <= 10;
```

---

## SQL Trace

```sql
ALTER SESSION SET SQL_TRACE = TRUE;
-- or
EXECUTE DBMS_SESSION.SET_SQL_TRACE(TRUE);
-- or for another session
EXECUTE DBMS_SYSTEM.SET_SQL_TRACE_IN_SESSION(session_id, serial_id, TRUE);
```

---

## Locking Memory

Set `LOCK_SGA=TRUE` to lock SGA into real memory (use only when sufficient memory available).

---

**Back to [Main Index](README.md)**
