---
layout: default
title: "RMAN Backup & Recovery"
parent: "DBA Reference Sheet"
nav_order: 11
---

# RMAN Backup & Recovery

Reference guide for RMAN backup, restore, duplicate, monitoring, and tuning operations.

---

## RMAN Scripts & Duplication

### Run RMAN script in background

```bash
oracle@hostname$ cat rman_db1_clone.sh
nohup rman target sys/pwd@db1 catalog user/pwd@rcat_db auxiliary / cmdfile='rman_db1_clone.rcv' log='rman_db1_clone.log' &

oracle@hostname$ cat rman_db1_clone.rcv
run{
allocate auxiliary channel aux1 type 'sbt_tape' parms 'ENV=(TDPO_OPTFILE=/usr/tivoli/tsm/client/oracle/bin64/tdpo.opt.host123)';
allocate auxiliary channel aux2 type 'sbt_tape' parms 'ENV=(TDPO_OPTFILE=/usr/tivoli/tsm/client/oracle/bin64/tdpo.opt.host123)';
allocate auxiliary channel aux3 type 'sbt_tape' parms 'ENV=(TDPO_OPTFILE=/usr/tivoli/tsm/client/oracle/bin64/tdpo.opt.host123)';
SET NEWNAME FOR DATAFILE  1 to '/db/db1/d1/DB2SYS.DBF';
SET NEWNAME FOR DATAFILE  2 to '/db/db1/d2/DB2UNDOT1A.DBF';
SET NEWNAME FOR DATAFILE  3 to '/db/db1/d1/DB2TLST1A.DBF';
SET NEWNAME FOR DATAFILE  4 to '/db/db1/d2/DB2USRT1A.DBF';
SET NEWNAME FOR DATAFILE  5 to '/db/db1/d13/DB2RAPT7R.DBF';
SET NEWNAME FOR DATAFILE  6 to '/db/db1/d12/DB2RAPI7O.DBF';
SET NEWNAME FOR DATAFILE  7 to '/db/db1/d14/DB2RAPT7I.DBF';
duplicate target database to db2;
}
```

### Get the latest SCN

```sql
SELECT MAX(first_change#) chng
  FROM v$archived_log
/
```

### Generate SET NEWNAME FOR DATAFILE commands

```sql
SELECT 'SET NEWNAME FOR DATAFILE ' || FILE# || ' TO ''' || '/u03/oradata/&1/' || SUBSTR(name, INSTR(name,'/',-1)+1) || ''';'
  FROM v$datafile;
```

### Generate ALTER DATABASE RENAME FILE commands

```sql
SELECT 'SQL "ALTER DATABASE RENAME FILE '''|| MEMBER ||''' '||CHR(10)||'TO '''|| member || '''" ;'
  FROM v$logfile;
```

---

## Monitoring & Progress

### Clone progress

```sql
COLUMN sid FORMAT 999
COLUMN serial# FORMAT 9999999
COLUMN machine FORMAT A30
COLUMN progress_pct FORMAT 99999999.00
COLUMN elapsed FORMAT A10
COLUMN remaining FORMAT A10
SELECT sl.opname, s.sid, s.serial#, s.machine,
       TRUNC(sl.elapsed_seconds/60) || ':' || MOD(sl.elapsed_seconds,60) elapsed,
       TRUNC(sl.time_remaining/60) || ':' || MOD(sl.time_remaining,60) remaining,
       ROUND(sl.sofar/sl.totalwork*100, 2) progress_pct
FROM   v$session s, v$session_longops sl
WHERE  s.sid = sl.sid AND s.serial# = sl.serial#;
```

### Percent completed (RMAN operations)

```sql
SELECT SID, SERIAL#, CONTEXT, SOFAR, TOTALWORK, opname,
       ROUND(SOFAR/TOTALWORK*100,2) "%_COMPLETE", Time_remaining
FROM V$SESSION_LONGOPS
WHERE OPNAME LIKE 'RMAN%'
  AND OPNAME NOT LIKE '%aggregate%'
  AND TOTALWORK != 0
  AND SOFAR != TOTALWORK;
```

**Alternative:**

```sql
SELECT sl.sid, sl.opname,
       TO_CHAR(100*(sofar/totalwork), '990.9')||'%' pct_done,
       SYSDATE+(TIME_REMAINING/60/60/24) done_by
  FROM v$session_longops sl, v$session s
 WHERE sl.sid = s.sid
   AND sl.serial# = s.serial#
   AND sl.sid IN (SELECT sid FROM v$session WHERE module LIKE 'backup%' OR module LIKE 'restore%' OR module LIKE 'rman%')
   AND sofar != totalwork
   AND totalwork > 0
/
```

### RMAN throughput speed

```sql
SET LINESIZE 126
COLUMN Pct_Complete FORMAT 99.99
COLUMN client_info FORMAT A25
COLUMN sid FORMAT 999
COLUMN MB_PER_S FORMAT 999.99
SELECT s.client_info, l.sid, l.serial#, l.sofar, l.totalwork,
       ROUND(l.sofar / l.totalwork*100,2) "Pct_Complete",
       aio.MB_PER_S, aio.LONG_WAIT_PCT
FROM v$session_longops l, v$session s,
     (SELECT sid, serial,
             100*SUM(long_waits) / SUM(io_count) AS "LONG_WAIT_PCT",
             SUM(effective_bytes_per_second)/1024/1024 AS "MB_PER_S"
      FROM v$backup_async_io
      GROUP BY sid, serial) aio
WHERE aio.sid = s.sid
  AND aio.serial = s.serial#
  AND l.opname LIKE 'RMAN%'
  AND l.opname NOT LIKE '%aggregate%'
  AND l.totalwork != 0
  AND l.sofar <> l.totalwork
  AND s.sid = l.sid
  AND s.serial# = l.serial#
ORDER BY 1;
```

### RMAN backup speed history

```sql
SELECT TO_CHAR(START_TIME,'MM-DD-YYYY HH24:MI:SS') AS START_TIME,
       TO_CHAR(END_TIME,'MM-DD-YYYY HH24:MI:SS') AS END_TIME,
       ELAPSED_SECONDS/60/60,
       INPUT_BYTES/1024/1024/1024/1024 AS INPUT_TB,
       OUTPUT_BYTES/1024/1024/1024/1024 AS OUTPUT_TB,
       INPUT_BYTES_PER_SEC/1024/1024 AS INPUT_SEC_MB,
       OUTPUT_BYTES_PER_SEC/1024/1024 AS OUTPUT_SEC_MB,
       status
FROM V$RMAN_BACKUP_JOB_DETAILS
WHERE INPUT_TYPE LIKE 'DB%INCR%';
```

**Alternative:**

```sql
SET LINES 750 PAGES 9999
COL start_time         HEADING 'Started'      FORMAT A30
COL end_time           HEADING 'End'          FORMAT A30
COL time_taken_display HEADING 'Elapsed|Time' FORMAT A10

SELECT TO_CHAR(start_time, 'Dy MM-DD-YYYY hh24:mi:ss') start_time,
       TO_CHAR(end_time, 'Dy MM-DD-YYYY hh24:mi:ss') end_time,
       time_taken_display,
       INPUT_BYTES_PER_SEC/1024/1024 AS INPUT_SEC_MB,
       OUTPUT_BYTES_PER_SEC/1024/1024 AS OUTPUT_SEC_MB,
       status
FROM V$RMAN_BACKUP_JOB_DETAILS
WHERE INPUT_TYPE LIKE 'DB%INCR%';
```

### Running RMAN info

```sql
SELECT p.SPID, s.sid, s.serial#, sw.EVENT, sw.SECONDS_IN_WAIT AS SEC_WAIT, sw.STATE, CLIENT_INFO
FROM V$SESSION_WAIT sw, V$SESSION s, V$PROCESS p
WHERE s.client_info LIKE 'rman%'
  AND s.SID = sw.SID
  AND s.PADDR = p.ADDR;
```

### RMAN currently running or not

```sql
COLUMN CLIENT_INFO FORMAT A30
COLUMN SID FORMAT 999
COLUMN SPID FORMAT 9999

SELECT s.SID, p.SPID, s.CLIENT_INFO
FROM V$PROCESS p, V$SESSION s
WHERE p.ADDR = s.PADDR
  AND CLIENT_INFO LIKE 'rman%'
/
```

### SBT-RMAN interaction

```sql
COLUMN EVENT FORMAT A10
COLUMN SECONDS_IN_WAIT FORMAT 999
COLUMN STATE FORMAT A20
COLUMN CLIENT_INFO FORMAT A30

SELECT p.SPID, EVENT, SECONDS_IN_WAIT AS SEC_WAIT, sw.STATE, CLIENT_INFO
FROM V$SESSION_WAIT sw, V$SESSION s, V$PROCESS p
WHERE sw.EVENT LIKE 's%bt%'
  AND s.SID = sw.SID
  AND s.PADDR = p.ADDR;
```

### Backup throughput

```sql
SELECT 'BACKUP THROUGHPUT',
       ROUND(SUM(v.value/1024/1024),1) mbytes_sofar,
       ROUND(SUM(v.value/1024/1024)/NVL((SELECT MIN(elapsed_seconds)
         FROM v$session_longops
         WHERE OPNAME LIKE 'RMAN: aggregate output'
           AND SOFAR != TOTALWORK
           AND elapsed_seconds IS NOT NULL), SUM(v.value/1024/1024)),2) mbytes_per_sec,
       n.name
FROM gv$sesstat v, v$statname n, gv$session s
WHERE v.statistic# = n.statistic#
  AND n.name = 'physical read total bytes'
  AND v.sid = s.sid
  AND v.inst_id = s.inst_id
  AND s.program LIKE 'rman@%'
GROUP BY 'BACKUP THROUGHPUT', n.name;
```

### Restore/duplicate throughput

```sql
SELECT 'DUPLICATE/RESTORE THROUGHPUT',
       ROUND(SUM(v.value/1024/1024),1) mbytes_sofar,
       ROUND(SUM(v.value/1024/1024)/NVL((SELECT MIN(elapsed_seconds)
         FROM v$session_longops
         WHERE OPNAME LIKE 'RMAN: aggregate input'
           AND SOFAR != TOTALWORK
           AND elapsed_seconds IS NOT NULL), SUM(v.value/1024/1024)),2) mbytes_per_sec,
       n.name
FROM gv$sesstat v, v$statname n, gv$session s
WHERE v.statistic# = n.statistic#
  AND n.name = 'physical write total bytes'
  AND v.sid = s.sid
  AND v.inst_id = s.inst_id
  AND s.program LIKE 'rman@%'
GROUP BY 'DUPLICATE/RESTORE THROUGHPUT', n.name;
```

---

## Configuration & Reports

### View RMAN configuration in SQL*Plus

```sql
SELECT * FROM v$rman_configuration;
```

### Last 7 days backup report

```sql
COL STATUS FORMAT A25
COL HRS    FORMAT 999.99
COL start_time  FORMAT A15
COL end_time  FORMAT A15
COL in_size  FORMAT A10

SELECT INPUT_TYPE, STATUS,
       TO_CHAR(START_TIME,'mm/dd/yy hh24:mi') start_time,
       TO_CHAR(END_TIME,'mm/dd/yy hh24:mi') end_time,
       INPUT_BYTES_DISPLAY in_size,
       ROUND(ELAPSED_SECONDS/3600,3) HRS
FROM V$RMAN_BACKUP_JOB_DETAILS
WHERE SYSDATE - start_time <= 7;
```

### Short RMAN report

```sql
COL start_time         HEADING 'Started'      FORMAT A13
COL backup_type        HEADING 'Backup Type'  FORMAT A12
COL time_taken_display HEADING 'Elapsed|Time' FORMAT A10
COL elapsed_min        HEADING 'Run|Min'      FORMAT 999
COL output_mbytes      HEADING 'Size MB'      FORMAT 9,999,999
COL backup_status      HEADING 'Status'       FORMAT A10 TRUNC
COL cf                 HEADING 'Ctrl|Files'   FORMAT 9,999
COL dfiles             HEADING 'Data|Files'   FORMAT 9,999
COL l                  HEADING 'Arch|Files'   FORMAT 9,999
COL output_instance    HEADING 'Ran on|Inst'  FORMAT 9

SELECT TO_CHAR(j.start_time, 'Dy hh24:mi:ss') start_time,
       DECODE(j.input_type,'DB INCR',DECODE(i0,0,'Incr Lvl 1','Incr Lvl 0'),INITCAP(j.input_type)) backup_type,
       j.time_taken_display, j.elapsed_seconds/60 elapsed_min,
       (j.output_bytes/1024/1024) output_mbytes, INITCAP(j.status) backup_status,
       x.cf, x.i0 + x.i1 dfiles, x.l, ro.inst_id output_instance
FROM V$RMAN_BACKUP_JOB_DETAILS j
  LEFT OUTER JOIN (SELECT d.session_recid, d.session_stamp,
         SUM(CASE WHEN d.controlfile_included = 'YES' THEN d.pieces ELSE 0 END) CF,
         SUM(CASE WHEN d.controlfile_included = 'NO' AND d.backup_type||d.incremental_level = 'D' THEN d.pieces ELSE 0 END) DF,
         SUM(CASE WHEN d.backup_type||d.incremental_level = 'D0' THEN d.pieces ELSE 0 END) I0,
         SUM(CASE WHEN d.backup_type||d.incremental_level = 'I1' THEN d.pieces ELSE 0 END) I1,
         SUM(CASE WHEN d.backup_type = 'L' THEN d.pieces ELSE 0 END) L
       FROM V$BACKUP_SET_DETAILS d
       JOIN V$BACKUP_SET s ON s.set_stamp = d.set_stamp AND s.set_count = d.set_count
       WHERE s.input_file_scan_only = 'NO'
       GROUP BY d.session_recid, d.session_stamp) x
    ON x.session_recid = j.session_recid AND x.session_stamp = j.session_stamp
  LEFT OUTER JOIN (SELECT o.session_recid, o.session_stamp, MIN(inst_id) inst_id
                   FROM GV$RMAN_OUTPUT o GROUP BY o.session_recid, o.session_stamp) ro
    ON ro.session_recid = j.session_recid AND ro.session_stamp = j.session_stamp
WHERE j.start_time > TRUNC(NEXT_DAY(SYSDATE-6,'SUNDAY'))
ORDER BY j.start_time
/
```

### RMAN backup report (detailed)

```sql
SET LINES 750 PAGES 9999
COL start_time  FOR A30
COL TIME_TAKEN_DISPLAY FOR A10

SELECT j.session_recid, j.session_stamp,
       TO_CHAR(j.start_time, 'yyyy-mm-dd hh24:mi:ss') start_time,
       TO_CHAR(j.end_time, 'yyyy-mm-dd hh24:mi:ss') end_time,
       (j.output_bytes/1024/1024) output_mbytes, j.status, j.input_type,
       DECODE(TO_CHAR(j.start_time, 'd'), 1, 'Sunday', 2, 'Monday', 3, 'Tuesday', 4, 'Wednesday',
             5, 'Thursday', 6, 'Friday', 7, 'Saturday') dow,
       j.elapsed_seconds, j.time_taken_display,
       x.cf, x.df, x.i0, x.i1, x.l, ro.inst_id output_instance
FROM V$RMAN_BACKUP_JOB_DETAILS j
  LEFT OUTER JOIN (SELECT d.session_recid, d.session_stamp,
         SUM(CASE WHEN d.controlfile_included = 'YES' THEN d.pieces ELSE 0 END) CF,
         SUM(CASE WHEN d.controlfile_included = 'NO' AND d.backup_type||d.incremental_level = 'D' THEN d.pieces ELSE 0 END) DF,
         SUM(CASE WHEN d.backup_type||d.incremental_level = 'D0' THEN d.pieces ELSE 0 END) I0,
         SUM(CASE WHEN d.backup_type||d.incremental_level = 'I1' THEN d.pieces ELSE 0 END) I1,
         SUM(CASE WHEN d.backup_type = 'L' THEN d.pieces ELSE 0 END) L
       FROM V$BACKUP_SET_DETAILS d JOIN V$BACKUP_SET s ON s.set_stamp = d.set_stamp AND s.set_count = d.set_count
       WHERE s.input_file_scan_only = 'NO' GROUP BY d.session_recid, d.session_stamp) x
    ON x.session_recid = j.session_recid AND x.session_stamp = j.session_stamp
  LEFT OUTER JOIN (SELECT o.session_recid, o.session_stamp, MIN(inst_id) inst_id FROM GV$RMAN_OUTPUT o GROUP BY o.session_recid, o.session_stamp) ro
    ON ro.session_recid = j.session_recid AND ro.session_stamp = j.session_stamp
ORDER BY j.start_time DESC;
```

### RMAN output log file viewing

```sql
SELECT output
FROM GV$RMAN_OUTPUT
WHERE session_recid = &SESSION_RECID
  AND session_stamp = &SESSION_STAMP
ORDER BY recid;
```

### Yesterday's backup output/status

```sql
SELECT output
FROM v$rman_output
WHERE session_recid = (SELECT MAX(session_recid) FROM v$rman_status)
ORDER BY recid;
```

**Alternative:**

```sql
SELECT object_type, mbytes_processed, start_time, end_time, status
FROM v$rman_status
WHERE session_recid = (SELECT MAX(session_recid) FROM v$rman_status)
  AND operation != 'RMAN'
ORDER BY recid;
```

---

## Backup Availability & Size

### RMAN backup available or not

```sql
SELECT bs.recid bs_key, bp.tag,
       DECODE(backup_type, 'L', 'Archived Redo Logs', 'D', 'Datafile Full Backup', 'I', 'Incremental Backup') backup_type,
       bs.incremental_level,
       DECODE(bp.status, 'A', 'Available', 'D', 'Deleted', 'X', 'Expired') status,
       DECODE(bs.controlfile_included, 'NO', '-', bs.controlfile_included) controlfile_included,
       NVL(sp.spfile_included, '-') spfile_included,
       bs.pieces,
       TO_CHAR(bs.start_time, 'mm/dd/yyyy HH24:MI:SS') start_time,
       TO_CHAR(bs.completion_time, 'mm/dd/yyyy HH24:MI:SS') completion_time,
       bs.elapsed_seconds, bs.block_size, bs.keep,
       NVL(TO_CHAR(bs.keep_until, 'mm/dd/yyyy HH24:MI:SS'),'') keep_until,
       bs.keep_options, device_type
FROM v$backup_set bs,
     (SELECT DISTINCT set_stamp, set_count, tag, device_type, status FROM v$backup_piece WHERE status IN ('A','D', 'X')) bp,
     (SELECT DISTINCT set_stamp, set_count, 'YES' spfile_included FROM v$backup_spfile) sp
WHERE bs.set_stamp = bp.set_stamp
  AND bs.set_count = bp.set_count
  AND bs.set_stamp = sp.set_stamp (+)
  AND bs.set_count = sp.set_count (+)
ORDER BY bs.start_time DESC;
```

### Available backup size

```sql
SELECT ROUND(SUM(bytes/1024/1024/1024),2) bsize
FROM v$backup_piece
WHERE STATUS = 'A';
```

### Obsolete backup size

```sql
SELECT SUM(BYTES/1024/1024/1024)
FROM V$BACKUP_FILES
WHERE OBSOLETE = 'YES'
  AND FNAME IS NOT NULL
  AND ((FILE_TYPE = 'PIECE' AND BACKUP_TYPE = 'BACKUP SET') OR BACKUP_TYPE = 'COPY');
```

### Day-wise backup size

```sql
SELECT ctime "Date",
       DECODE(backup_type, 'L', 'Archive Log', 'D', 'Full', 'Incremental') backup_type,
       bsize "Size MB"
FROM (SELECT TRUNC(bp.completion_time) ctime, backup_type,
             ROUND(SUM(bp.bytes/1024/1024),2) bsize
      FROM v$backup_set bs, v$backup_piece bp
      WHERE bs.set_stamp = bp.set_stamp
        AND bs.set_count = bp.set_count
        AND bp.status = 'A'
      GROUP BY TRUNC(bp.completion_time), backup_type)
ORDER BY 1, 2;
```

### How long we can go back using RMAN backup

```sql
SELECT TO_CHAR(MIN(bs.completion_time),'DD/MM/YYYY HH24:MI') completion_time
FROM v$backup_set bs, v$backup_piece bp
WHERE bs.set_stamp = bp.set_stamp
  AND bs.set_count = bp.set_count
  AND bs.backup_type = 'D'
  AND bp.status = 'A';
```

---

## Recovery & Corruption

### Files needing recovery

```sql
SELECT * FROM V$RECOVER_FILE;
```

### Block recovery

```bash
RMAN> blockrecover datafile 8 block 13;
```

### Object under corrupted block

```sql
SELECT owner, segment_name, segment_type
FROM dba_extents
WHERE file_id = 1
  AND 16516 BETWEEN block_id AND block_id + blocks - 1;
```

### Validate tablespace for corruption

```bash
RMAN> validate tablespace corrupt;
```

### Skip corrupted blocks

```sql
BEGIN
  DBMS_REPAIR.SKIP_CORRUPT_BLOCKS (
    SCHEMA_NAME => 'SCOTT',
    OBJECT_NAME => 'DEPT',
    OBJECT_TYPE => dbms_repair.table_object,
    FLAGS => dbms_repair.skip_flag);
END;
/

SELECT OWNER, TABLE_NAME, SKIP_CORRUPT FROM DBA_TABLES WHERE OWNER = 'SCOTT';
```

### Extract useful data from corrupted table

```sql
EXEC DBMS_REPAIR.SKIP_CORRUPT_BLOCKS('MANA', 'TAXINQUIRY_LOG');

INSERT INTO MANA.TAXINQUIRY_LOG_REPAIR (SELECT * FROM MANA.TAXINQUIRY_LOG);
COMMIT;

DROP TABLE MANA.TAXINQUIRY_LOG;
ALTER TABLE TAXINQUIRY_LOG_REPAIR RENAME TO TAXINQUIRY_LOG;
ALTER SYSTEM FLUSH SHARED_POOL;
```

### Restore missing archive log file

```bash
RMAN> run {
  set archivelog destination to '/ora_backup/rman/arch/';
  restore archivelog from logseq=8619 until logseq=8632 thread=2;
}
```

### ORA-01113: file needs media recovery (using backup controlfile)

```sql
-- Find current redo member
SELECT MEMBER FROM V$LOG G, V$LOGFILE F
WHERE G.GROUP# = F.GROUP# AND G.STATUS = 'CURRENT';

-- Start cancel-based recovery, specify log file when prompted
RECOVER DATABASE USING BACKUP CONTROLFILE UNTIL CANCEL
-- When prompted: /OraRedo/RedoLogFiles/siamst_log01.dbf

ALTER DATABASE OPEN RESETLOGS;
```

---

## RMAN Catalog

### RMAN catalog version

```sql
SELECT * FROM schemaname.rcver;
```

### Unregister from catalog

```bash
RMAN> RUN {
  SET DBID 3668200963;
  UNREGISTER DATABASE DB_NAME NOPROMPT;
}
```

### View DBID from catalog

```sql
SELECT DBID, NAME, STATUS
FROM RC_DATABASE_INCARNATION
WHERE name LIKE '%IKB%';
```

### History of database incarnations

```sql
SELECT LPAD(' ',2*(LEVEL-1)) || TO_CHAR(DBINC_KEY) AS DBINC_KEY,
       db_key, db_name, TO_CHAR(reset_time,'YYYY-MM-DD HH24:MI:SS'), dbinc_status
FROM rman.dbinc
START WITH PARENT_DBINC_KEY IS NULL
CONNECT BY PRIOR DBINC_KEY = PARENT_DBINC_KEY;
```

---

## Flash Recovery Area

### FRA used

```sql
SELECT ROUND((A.SPACE_LIMIT / 1024 / 1024 / 1024), 2) AS FLASH_IN_GB,
       ROUND((A.SPACE_USED / 1024 / 1024 / 1024), 2) AS FLASH_USED_IN_GB,
       ROUND((A.SPACE_RECLAIMABLE / 1024 / 1024 / 1024), 2) AS FLASH_RECLAIMABLE_GB,
       SUM(B.PERCENT_SPACE_USED) AS PERCENT_OF_SPACE_USED
FROM V$RECOVERY_FILE_DEST A, V$FLASH_RECOVERY_AREA_USAGE B
GROUP BY SPACE_LIMIT, SPACE_USED, SPACE_RECLAIMABLE;
```

**Alternative:**

```sql
SELECT * FROM v$flash_recovery_area_usage;
```

---

## Archive Log & Maintenance

### Backup archivelog with skip inaccessible (ORA-19625)

```bash
RMAN> run {
  allocate channel c1 type 'SBT' parms'ENV=(TDPO_OPTFILE=/opt/tivoli/tsm/client/oracle/bin64/tdpo.opt)';
  allocate channel c2 type 'SBT' parms'ENV=(TDPO_OPTFILE=/opt/tivoli/tsm/client/oracle/bin64/tdpo.opt)';
  backup archivelog all skip inaccessible delete input;
  release channel c1;
  release channel c2;
}
```

### Standby archivelog deletion policy

```bash
RMAN> run {
  CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
  crosscheck archivelog all;
  crosscheck backup;
  delete noprompt expired backup;
  delete noprompt expired archivelog all;
  delete noprompt obsolete;
  delete noprompt archivelog all completed before 'sysdate - 1/24';
}
```

### Backup backup sets from ASM to tape

```bash
RMAN> crosscheck backup;
RMAN> BACKUP DEVICE TYPE sbt BACKUPSET ALL DELETE INPUT;
```

---

## Tracing

### Tracing RMAN

```bash
rman rcvcat rman/rman@catalog target user/passwd@target debug trace trace_file_name
```

---

## Reference Links

- RMAN Tuning: http://www.oraclecoursebooks.com/books/oracle10g_rman/12_10g_rman/oracle10g_rman12.HTM
- RMAN Examples: https://web.stanford.edu/dept/itss/docs/oracle/10gR2/backup.102/b14191/rcmdupdb006.htm

---

Back to [Main Index](README.md)
