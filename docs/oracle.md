---
layout: default
title: Oracle Queries
nav_order: 2
has_children: true
has_toc: true
---
# Oracle Database — Complete DBA Reference

A comprehensive reference guide for Oracle Database administrators, covering connection, monitoring, performance tuning, backup, user management, tablespace management, and essential DBA queries.

- - -

## 1. ORACLE CONNECTION SQLPLUS COMMAND

Connect to Oracle database using SQL*Plus with a full connection string.

```sql
sqlplus 'username/password@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=your-rds-endpoint.region.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=ORCL)))'
```

- - -

## 2. ORACLE DATABASE STARTUP PROCESS

Oracle database startup follows the sequence: NOMOUNT → MOUNT → OPEN.

```sql
shutdown immediate
startup mount;
alter database open;
```

- - -

## 3. SPFILE AND PFILE

Static and dynamic parameter changes in Oracle. Use `scope` to control where changes apply: `memory`, `both`, or `spfile`.

```sql
alter system set process=400 scope=memory, both, spfile;
```

- - -

## 4. MONITOR ORACLE DATABASE

### DB health check

Check if the database instance is running and verify database/instance status.

```bash
ps -ef|grep pmon
```

```sql
select * from V$database, V$instance
```

### Show parameter db_file_multi

Shows how many blocks can be scanned at once in datafile and pushed to database buffer cache.

```sql
show parameter db_file_multi
```

### Identify alert log location

```sql
select * from v$diag_info;
```

### Check last ORA- error in alert log

```bash
cd /opt/oracle/base/diag/rdbms/ce/ce/trace/
grep 'ORA-' alert_ce.log|tail
grep -A5 'ORA-' alert_ce.log | tail
grep -A5 'ORA-' alert.log | tail
grep -A5 'MV_CM2_PRODUCT_BASE' alert_ce.log | tail
```

### V$session, V$sql, V$sqlstats

```sql
sqlplus / as sysdba
set lines 200 pages 1000
select SID, USERNAME, MACHINE, PROGRAM , SQL_ID, MODULE, ACTION  from V$session ;
```

```sql
select sql_text from V$sql where sql_id='';        --real time query
select sql_text from v$sqlstats where sql_id='';   -- it will contain for longer period
```

```sql
select username, status, sql_id from V$session where status='ACTIVE';
select a.sid, a.serial#,b.sql_text from v$session a, v$sql b  where a.sql_id=b.sql_id and a.sql_id='';
```

### To check active running session in DB

```sql
col username for A30
set lines 200
select status, sql_id, username, count(sql_id), count(username), count(status) from gv$session  group by status, sql_id, username;
```

- - -

## 5. LONG RUNNING SESSION

### LONGOPS

```sql
col RPAD(a.opname,30) for a20
col username for a20
set lines 200 pages 200
SELECT a.sid,RPAD(a.opname,30),b.sql_id,a.sofar,a.totalwork,a.ELAPSED_SECONDS,ROUND(((a.sofar)*100)/a.totalwork,3) "%_COMPLETED",
RPAD(a.username,10) username,a.SQL_HASH_VALUE,B.STATUS
FROM V$SESSION_LONGOPS a, v$session b
WHERE a.sid=b.sid
AND b.status='ACTIVE'
AND a.sofar<> a.totalwork;

select sql_text from v$sql where sql_id='&sql_id';
```

### Set NLS format

```sql
col SID FOR 999
alter session set nls_date_format = 'DD-MON-YYYY HH24:MI:SS';
```

### V$SESSION_LONGOPS

```sql
col opname for a20
col username for a12
select SID, SERIAL#, OPNAME, START_TIME, LAST_UPDATE_TIME, TIME_REMAINING, ELAPSED_SECONDS, SQL_ID, USERNAME,
100 * (SOFAR/TOTALWORK) PCT FROM V$session_longops where sofar > 0 and totalwork >0 ;
```

### Query percentage completion

```sql
set lines 200
col opname for a20
col username for a12
SELECT SID, SERIAL#, SQL_ID, USERNAME, CONTEXT, SOFAR, TOTALWORK,ROUND(SOFAR/TOTALWORK*100,2) "%_COMPLETE"FROM V$SESSION_LONGOPS WHERE TOTALWORK != 0 AND SOFAR <> TOTALWORK;
```

```sql
COL opname FORMAT A30
COL username FORMAT A15
COL target FORMAT A30
COL elapsed_time FORMAT 999,999,999
COL time_remaining FORMAT 999,999,999

SELECT sid,
       serial#,
       username,
       opname,
       target,
       TO_CHAR(start_time,'DD-MON-YYYY HH24:MI:SS') AS start_time,
       sofar,
       totalwork,
       ROUND(sofar/totalwork*100,2) AS pct_complete,
       elapsed_seconds,
       time_remaining
FROM   v$session_longops
WHERE  sofar <> totalwork
ORDER BY start_time DESC;
```

### Import/EXPORT SOFAR

```sql
SELECT sl.sid, sl.serial#, sl.sofar, sl.totalwork, dp.owner_name, dp.state, dp.job_mode
     FROM v$session_longops sl, v$datapump_job dp
     WHERE sl.opname = dp.job_name
     AND sl.sofar != sl.totalwork;
```

```sql
select  unique DATAOBJ_NUM , object_type
from admin.SYS_IMPORT_FULL_02
where process_order > 0 AND processing_state = 'R'
and processing_status = 'C';
```

```sql
select object_schema, object_name
from admin.SYS_IMPORT_FULL_02
where process_order > 0 and processing_state = 'R' and processing_status = 'C' and DATAOBJ_NUM = 14207613;
```

```sql
select sum(dump_orig_length), processing_state
from  admin.SYS_IMPORT_FULL_02
where process_order > 0 and duplicate = 0 and object_type = 'TABLE_DATA'
group by processing_state;
```

### When to look at V$SESSION_LONGOPS?

1. RMAN Backup and Restore
2. Parallel Query (Large chunks)
3. Recovery (Crash and Media)
4. Large Full Table scans
5. Sorting
6. Stats job
7. If the table scan exceeds 10,000 formatted blocks
8. Operation is greater than 6 seconds

**Exception where view does not record:** Index long operations (long time)

- - -

## 6. TO SEE QUERY PLAN FOR A SQL ID

```sql
select * from table(dbms_xplan.display_cursor('&SQL_ID',null));
```

- - -

## 7. DIRECTLY DISPLAY THE PLAN IN SESSION LEVEL

```sql
select * from table(dbms_xplan.display_cursor(null,null));
```

```sql
select /*+parallel(16)*/ name from table;
```

- - -

## 8. GATHER STATS

```sql
exec DBMS_STATS.GATHER_TABLE_STATS('KISH','XTB');
```

- - -

## 9. KILL SESSION

### Kill session in AWS Oracle RDS

```sql
select begin rdsadmin.rdsadmin_util.disconnect(sid => 3790, serial => 38549); end;
```

### Query to kill multiple session in AWS RDS

```sql
SET LINESIZE 1000;
COL Query FOR a60;
SET PAGESIZE 10000;
COL username FOR a20;

SELECT
       'begin'
       || CHR(13) || CHR(10) || 'rdsadmin.rdsadmin_util.kill('
       || 'sid => ' || s.sid
       || ', serial => ' || s.serial#
       || ');'
       || CHR(13) || CHR(10) || 'end;'
       || CHR(10) || '/' AS Query
FROM v$session s, v$process p
WHERE s.paddr = p.addr and 
s.status='INACTIVE'
ORDER BY 1 DESC;
```

### Kill session in Oracle

```sql
alter system kill session 'SID, SERIAL#' immediate;
select 'alter system kill session '''||sid||','||serial#||',@'||inst_id||''' immediate;' from gv$session where SID=8862;
```

Scripts to kill sessions based on various criteria:

```sql
-- Kill all sessions
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session;

-- Kill sessions from username TEST
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE USERNAME LIKE '%CE_SYSTEM%' and sql_id='dpgzymcxvwmba';

-- Kill INACTIVE sessions for user TEST
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE USERNAME LIKE '%TEST%' AND STATUS='INACTIVE';

-- Kill INACTIVE sessions running for more than 10000 seconds
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE USERNAME LIKE '%TEST%' AND STATUS='INACTIVE' and LAST_CALL_ET > 10000;

-- Kill INACTIVE sessions for user TEST, running > 10000 seconds, logged on in last 24 hours
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE USERNAME LIKE '%TEST%' AND STATUS='INACTIVE' and LAST_CALL_ET > 10000 and LOGON_TIME > sysdate - 1 ;

-- Kill specific session by SID
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE SID=7604;

-- Kill blocking sessions
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE BLOCKING_SESSION is NOT NULL;

-- Kill RMAN jobs
SELECT 'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''' IMMEDIATE;' FROM v$session WHERE CLIENT_INFO LIKE '%rman%';
```

### Kill query from OS

```sql
select spid from v$process where addr in (select paddr from v$session where status = 'KILLED');
```

```sql
select paddr,SID  from v$session where status = 'KILLED';
```

```bash
ps -ef|grep 1005529
kill -9 1005529
```

### Kill Export/import jobs in OAD

```sql
SELECT j.owner_name, j.job_name, 
            j.job_mode, j.state, s.session_type, s.saddr
     FROM dba_datapump_jobs j,dba_datapump_sessions s
     WHERE UPPER(j.job_name) = UPPER(s.job_name);
```

```bash
expdp admin/'o9vC4J97fq$SDWT!a$J7wy1L8LgcFc'@t0151pplat1ceoad_high attach=SYS_EXPORT_SCHEMA_01
kill_job
```

- - -

## 10. TO MAP OS PROCESS WITH DB

Pass the `ps -ef` process id to find the corresponding database session.

```sql
select se.sid,se.serial#,se.sql_id,se.machine,se.terminal,se.module,se.action,pr.spid from v$session se
inner join v$process pr on (pr.addr = se.paddr) where pr.spid ='984859';
```

- - -

## 11. SHELL SCRIPT TO SELECT DATA

```bash
#!/bin/bash

export ORACLE_SID=db9zx
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
while true
do
$ORACLE_HOME/bin/sqlplus -S '/ as sysdba' <<EOF

select * from kish.xtbl;

exit;
EOF
done
```

- - -

## 12. BLOCKING SESSION

```sql
select blocking_session from V$session where blocking_session is not null group by blocking_session;
```

```sql
select a.SID "Blocking Session", b.SID "Blocked Session"  
from v$lock a, v$lock b 
where a.SID != b.SID and a.ID1 = b.ID1  and a.ID2 = b.ID2 and 
b.request > 0 and a.block = 1;
```

```sql
col blocking_status for a120;
select s1.username || '@' || s1.machine
 || ' ( SID=' || s1.sid || ' ) is blocking '
 || s2.username || '@' || s2.machine 
 || ' ( SID=' || s2.sid || ' ) ' AS blocking_status
 from v$lock l1, v$session s1, v$lock l2, v$session s2
 where s1.sid=l1.sid and s2.sid=l2.sid
 and l1.BLOCK=1 and l2.request > 0
 and l1.id1 = l2.id1
 and l2.id2 = l2.id2 ;
```

```sql
select    'SID ' || l1.sid ||' is blocking the session ' || l2.sid blocking
from    gv$lock l1, gv$lock l2
where    l1.block =1 and l2.request > 0
and    l1.id1=l2.id1
and    l1.id2=l2.id2;
```

### Lock on table

```sql
set lines 1000
col OBJECT_NAME for a30
select object_name, s.sid, s.serial#, p.spid, s.sql_id
from v$locked_object l, dba_objects o, v$session s, v$process p
where l.object_id = o.object_id and l.session_id = s.sid and s.paddr = p.addr;
```

```sql
select a.sid, a.serial# from v$session a, v$locked_object b, dba_objects c  where b.object_id = c.object_id and a.sid =b.session_id and OBJECT_NAME='IRS1095_TRUNCATE_EXTRACT_FILES';
```

### Query to kill blocking session

```sql
SELECT    'ALTER SYSTEM KILL SESSION '''|| SESLCK.SID|| ','|| SESLCK.SERIAL#|| ',@'|| SESLCK.INST_ID|| ''' IMMEDIATE;'
           BLOCKING_SESSION_KILL_COMMAND
  FROM GV$LOCK     WT,
       GV$LOCK     LCKR,
       GV$SESSION  SESLCK,
       GV$SESSION  SEWT
 WHERE     LCKR.ID1 = WT.ID1
       AND LCKR.SID = SESLCK.SID
       AND LCKR.INST_ID = SESLCK.INST_ID
       AND WT.SID = SEWT.SID
       AND WT.INST_ID = SEWT.INST_ID
       AND LCKR.ID2 = WT.ID2
       AND LCKR.REQUEST = 0
       AND WT.LMODE = 0;
```

### Displays information on database sessions with username as hierarchy if locks are present

```sql
SET LINESIZE 500
SET PAGESIZE 1000

COLUMN username FORMAT A30
COLUMN osuser FORMAT A10
COLUMN machine FORMAT A25
COLUMN logon_time FORMAT A20

SELECT level,
       LPAD(' ', (level-1)*2, ' ') || NVL(s.username, '(oracle)') AS username,
       s.osuser,
       s.sid,
       s.serial#,
       s.lockwait,
       s.status,
       s.module,
       s.machine,
       s.program,
       TO_CHAR(s.logon_Time,'DD-MON-YYYY HH24:MI:SS') AS logon_time
FROM   v$session s
WHERE  level > 1
OR     EXISTS (SELECT 1
               FROM   v$session
               WHERE  blocking_session = s.sid)
CONNECT BY PRIOR s.sid = s.blocking_session
START WITH s.blocking_session IS NULL;

ALTER SYSTEM KILL SESSION '3790,38549' IMMEDIATE;
```

### Script to kill all blocking session

```sql
DECLARE
CURSOR c IS
SELECT c.owner,
      c.object_name,
      c.object_type,
      b.SID,
      b.serial#,
      b.status,
      b.osuser,
      b.machine
 FROM v$locked_object a, v$session b, dba_objects c
 WHERE b.SID = a.session_id AND a.object_id = c.object_id
 and c.object_name in ('JSW_CRM_C_HR_COIL_INFO','T_SPCL_BARCODE_CRS2');
c_row c%ROWTYPE;
l_sql VARCHAR2(100);
BEGIN
OPEN c;
LOOP
FETCH c INTO c_row;
EXIT WHEN c%NOTFOUND;
l_sql := 'alter system kill session '''||c_row.SID||','||c_row.serial#||'''';
EXECUTE IMMEDIATE l_sql;
END LOOP;
CLOSE c;
END;
/
```

```sql
SELECT    'ALTER SYSTEM KILL SESSION '''
       || s.sid
       || ','
       || s.serial#
       || ''' IMMEDIATE;'
          AS ddl
  FROM v$session s
 WHERE s.blocking_session IS NOT NULL AND s.username LIKE 'SCOTT';
```

### Blocking lock wait time

```sql
SELECT 
  blocking_session "BLOCKING_SESSION",
  sid "BLOCKED_SESSION",
  serial# "BLOCKED_SERIAL#", 
  seconds_in_wait/60 "WAIT_TIME(MINUTES)"
FROM v$session
WHERE blocking_session is not NULL
ORDER BY blocking_session;
```

### Find blocked query

```sql
SELECT SES.SID, SES.SERIAL# SER#, SES.PROCESS OS_ID, SES.STATUS, SQL.SQL_FULLTEXT
FROM V$SESSION SES, V$SQL SQL, V$PROCESS PRC
WHERE
   SES.SQL_ID=SQL.SQL_ID AND
   SES.SQL_HASH_VALUE=SQL.HASH_VALUE AND 
   SES.PADDR=PRC.ADDR AND
   SES.SID=&Enter_blocked_session_SID;
```

### Find blocking query

```sql
SELECT *
    FROM GV$OPEN_CURSOR OC
   WHERE     OC.INST_ID = :TYPE_BLOCKING_INSTANCE_ID
         AND OC.SID = :TYPE_BLOCKING_SID
         AND (   OC.SQL_TEXT LIKE 'INSERT%'
              OR OC.SQL_TEXT LIKE 'UPDATE%'
              OR OC.SQL_TEXT LIKE 'DELETE%')
         AND OC.CURSOR_TYPE = 'OPEN'
ORDER BY OC.LAST_SQL_ACTIVE_TIME;
```

### Locked table

```sql
col session_id head 'Sid' form 9999
col object_name head "Table|Locked" form a30
col oracle_username head "Oracle|Username" form a10 truncate 
col os_user_name head "OS|Username" form a10 truncate 
col process head "Client|Process|ID" form 99999999
col owner head "Table|Owner" form a10
col mode_held form a15
select lo.session_id,lo.oracle_username,lo.os_user_name,
lo.process,do.object_name,do.owner,
decode(lo.locked_mode,0, 'None',1, 'Null',2, 'Row Share (SS)',
3, 'Row Excl (SX)',4, 'Share',5, 'Share Row Excl (SSX)',6, 'Exclusive',
to_char(lo.locked_mode)) mode_held
from gv$locked_object lo, dba_objects do
where lo.object_id = do.object_id
order by 5;
```

### Query to check a particular type of SQL in Oracle DB

```sql
set lines 200 
set pages 100
select s.sid ,
       s.serial#, 
       s.sql_id,
       st.command_name,
       st.seconds_in_wait,
       s.status from v$session s 
Inner join v$sqlcommand st on (s.command=st.command_type)
and sql_id is not null;
```

```sql
set lines 200 
set pages 100
select s.sid ,
       s.serial#, 
       s.sql_id,
       sq.sql_text,
       st.command_name,
       st.seconds_in_wait,
       s.status from v$session s 
Inner Join v$sql sq on (sq.sql_id=s.sql_id)
Inner join v$sqlcommand st on (s.command=st.command_type)
and s.sql_id is not null;
```

### Queries access a particular object

```sql
SELECT a.sid,a.owner,a.type,s.username,s.program,s.module,s.sql_id,sq.sql_text from 
v$access a,v$session s,v$sql sq where a.sid=s.sid and sq.sql_id = s.sql_id and a.object='CM2_CMI_MATCHES_NPD';
```

- - -

## 13. MONITOR IMPORT PROGRESS

```sql
SET LINESIZE 200
SET PAGESIZE 100
SET TRIMSPOOL ON
SET WRAP OFF

COLUMN username        FORMAT A20          HEADING "Username"
COLUMN sid             FORMAT 99999        HEADING "SID"
COLUMN opname          FORMAT A30          HEADING "Operation"
COLUMN target          FORMAT A40          HEADING "Target Object"
COLUMN "%DONE"         FORMAT A6           HEADING "%Done"
COLUMN time_remaining  FORMAT 999999       HEADING "Time|Remaining(s)"
COLUMN start_time      FORMAT A20          HEADING "Start Time"

SELECT b.username, a.sid, b.opname, b.target,
            round(b.SOFAR*100/b.TOTALWORK,0) || '%' as "%DONE", b.TIME_REMAINING,
            to_char(b.start_time,'YYYY/MM/DD HH24:MI:SS') start_time
     FROM v$session_longops b, v$session a
     WHERE a.sid = b.sid      and b.opname like '%IMPORT%' ORDER BY 6;
```

```sql
-- Query to monitor Data Pump export progress
SELECT 
    b.username, 
    a.sid, 
    b.opname, 
    b.target,
    ROUND(b.sofar * 100 / b.totalwork, 0) || '%' AS "%DONE",
    b.time_remaining,
    TO_CHAR(b.start_time, 'YYYY/MM/DD HH24:MI:SS') AS start_time
FROM 
    v$session_longops b,
    v$session a
WHERE 
    a.sid = b.sid
    AND b.opname LIKE '%EXPORT%'
    AND b.totalwork > 0
ORDER BY b.time_remaining;
```

```sql
SELECT sl.sid, sl.serial#, sl.sofar, sl.MESSAGE,sl.totalwork, dp.owner_name, dp.state, dp.job_mode
FROM v$session_longops sl, v$datapump_job dp
WHERE sl.opname = dp.job_name;
```

- - -

## 14. BRIGHT DBA TUNING

### Check the ALL Active/Inactive session

```sql
set linesize 750 pages 9999
column box format a30
column spid format a10
column username format a30 
column program format a30
column os_user format a20
col LOGON_TIME for a20  

select b.inst_id,b.sid,b.serial#,a.spid, substr(b.machine,1,30) box,to_char (b.logon_time, 'dd-mon-yyyy hh24:mi:ss') logon_time,
substr(b.username,1,30) username,
substr(b.osuser,1,20) os_user,
substr(b.program,1,30) program,status,b.last_call_et AS last_call_et_secs,b.sql_id ,c.sql_text 
from gv$session b,gv$process a , v$sql c
where b.paddr = a.addr 
and a.inst_id = b.inst_id  
and type='USER'
and b.status='ACTIVE'
AND b.sql_id= c.sql_id
order by logon_time;
```

```sql
alter database default temporary tablespace temp;
drop tablespace temp3 including contents and datafiles;
create bigfile temporary tablespace temp3 tempfile '/mnt/temp03' size 1G autoextend on next 1G maxsize 90G;
alter database default temporary tablespace temp3;
```

```bash
ssh -Y -i ~/.ssh/private_key_t0122.pem oracle@10.232.36.140
```

### Check the all Active session

```sql
set linesize 750 pages 9999
column box format a30
column spid format a10
column username format a11 
column program format a30
column os_user format a20
col LOGON_TIME for a20  

select b.inst_id,b.sid,b.serial#,a.spid,to_char (b.logon_time, 'dd-mon-yyyy hh24:mi:ss') logon_time,
substr(b.username,1,30) username,
substr(b.osuser,1,20) os_user,status,b.last_call_et AS last_call_et_secs,b.sql_id 
from gv$session b,gv$process a 
where b.paddr = a.addr 
and a.inst_id = b.inst_id  
and type='USER' and b.status='ACTIVE'
order by logon_time;
```

### Check the ALL Active/Inactive sessions by SID

```sql
set linesize 750 pages 9999
column box format a30
column spid format a10
column username format a11 
column program format a30
column os_user format a20
col LOGON_TIME for a20  

select b.inst_id,b.sid,b.serial#,a.spid,to_char (b.logon_time, 'dd-mon-yyyy hh24:mi:ss') logon_time,
substr(b.username,1,30) username,
substr(b.osuser,1,20) os_user,
substr(b.program,1,30) program,status,b.last_call_et AS last_call_et_secs 
from gv$session b,gv$process a 
where b.paddr = a.addr 
and a.inst_id = b.inst_id  
and type='USER' and b.SID='&SID'
order by logon_time;
```

### Check the ALL Active/Inactive sessions by Username

```sql
set linesize 750 pages 9999
column box format a30
column spid format a10
column username format a30 
column program format a30
column os_user format a20
col LOGON_TIME for a20  

select b.inst_id,b.sid,b.serial#,a.spid, substr(b.machine,1,30) box,to_char (b.logon_time, 'dd-mon-yyyy hh24:mi:ss') logon_time,
substr(b.username,1,30) username,
substr(b.osuser,1,20) os_user,
substr(b.program,1,30) program,status,b.last_call_et AS last_call_et_secs,b.sql_id 
from gv$session b,gv$process a 
where b.paddr = a.addr 
and a.inst_id = b.inst_id  
and type='USER' and b.username='&username'
order by logon_time;
```

### SQL Monitor

```sql
set lines 1000 pages 9999 
column sid format 9999 
column serial for 999999
column status format a15
column username format a10 
column sql_text format a80
column module format a30
col program for a30
col SQL_EXEC_START for a20

SELECT * FROM
       (SELECT status,inst_id,sid,SESSION_SERIAL# as Serial,username,sql_id,SQL_PLAN_HASH_VALUE,
     MODULE,program,
         TO_CHAR(sql_exec_start,'dd-mon-yyyy hh24:mi:ss') AS sql_exec_start,
         ROUND(elapsed_time/1000000)                      AS "Elapsed (s)",
         ROUND(cpu_time    /1000000)                      AS "CPU (s)",
         substr(sql_text,1,30) sql_text
       FROM gv$sql_monitor where status='EXECUTING' and module not like '%emagent%' 
       ORDER BY sql_exec_start  desc
       );
```

### Sql-Monitor report for a sql_id (Like OEM report)

```sql
column text_line format a254
set lines 750 pages 9999
set long 20000 longchunksize 20000
select 
 dbms_sqltune.report_sql_monitor_list() text_line 
from dual;

select 
 dbms_sqltune.report_sql_monitor() text_line 
from dual;
```

### Blocking sessions

```sql
set lines 750 pages 9999
col blocking_status for a100 
select s1.inst_id,s2.inst_id,s1.username || '@' || s1.machine
|| ' ( SID=' || s1.sid || ' )  is blocking '
|| s2.username || '@' || s2.machine || ' ( SID=' || s2.sid || ' ) ' AS blocking_status
 from gv$lock l1, gv$session s1, gv$lock l2, gv$session s2
  where s1.sid=l1.sid and s2.sid=l2.sid and s1.inst_id=l1.inst_id and s2.inst_id=l2.inst_id
  and l1.BLOCK=1 and l2.request > 0
  and l1.id1 = l2.id1
  and l2.id2 = l2.id2
order by s1.inst_id;
```

```sql
SELECT DECODE(request,0,'Holder: ','Waiter: ') || gv$lock.sid sess, machine, do.object_name as locked_object,id1, id2, lmode, request, gv$lock.type
FROM gv$lock join gv$session on gv$lock.sid=gv$session.sid and gv$lock.inst_id=gv$session.inst_id
join gv$locked_object lo on gv$lock.SID = lo.SESSION_ID and gv$lock.inst_id=lo.inst_id
join dba_objects do on lo.OBJECT_ID = do.OBJECT_ID 
WHERE (id1, id2, gv$lock.type) IN (
  SELECT id1, id2, type FROM gv$lock WHERE request>0)
ORDER BY id1, request;
```

### Kill Sessions

```sql
select 'alter system kill session ' || '''' || sid || ',' || serial# ||',@'|| inst_id || '''' || ' immediate;' from gv$session where osuser='Sikta_Sharma';
```

### SQL History

```sql
set lines 1000 pages 9999
COL instance_number FOR 9999 HEA 'Inst';
COL end_time HEA 'End Time';
COL plan_hash_value HEA 'Plan|Hash Value';
COL executions_total FOR 999,999 HEA 'Execs|Total';
COL rows_per_exec HEA 'Rows Per Exec';
COL et_secs_per_exec HEA 'Elap Secs|Per Exec';
COL cpu_secs_per_exec HEA 'CPU Secs|Per Exec';
COL io_secs_per_exec HEA 'IO Secs|Per Exec';
COL cl_secs_per_exec HEA 'Clus Secs|Per Exec';
COL ap_secs_per_exec HEA 'App Secs|Per Exec';
COL cc_secs_per_exec HEA 'Conc Secs|Per Exec';
COL pl_secs_per_exec HEA 'PLSQL Secs|Per Exec';
COL ja_secs_per_exec HEA 'Java Secs|Per Exec';
SELECT 'gv$dba_hist_sqlstat' source,h.instance_number,
       TO_CHAR(CAST(s.begin_interval_time AS DATE), 'DD-MM-YYYY HH24:MI') snap_time,
       TO_CHAR(CAST(s.end_interval_time AS DATE), 'DD-MM-YYYY HH24:MI') end_time,
       h.sql_id,
       h.plan_hash_value, 
       h.executions_total,
       TO_CHAR(ROUND(h.rows_processed_total / h.executions_total), '999,999,999,999') rows_per_exec,
       TO_CHAR(ROUND(h.elapsed_time_total / h.executions_total / 1e6, 3), '999,990.000') et_secs_per_exec,
       TO_CHAR(ROUND(h.cpu_time_total / h.executions_total / 1e6, 3), '999,990.000') cpu_secs_per_exec,
       TO_CHAR(ROUND(h.iowait_total / h.executions_total / 1e6, 3), '999,990.000') io_secs_per_exec,
       TO_CHAR(ROUND(h.clwait_total / h.executions_total / 1e6, 3), '999,990.000') cl_secs_per_exec,
       TO_CHAR(ROUND(h.apwait_total / h.executions_total / 1e6, 3), '999,990.000') ap_secs_per_exec,
       TO_CHAR(ROUND(h.ccwait_total / h.executions_total / 1e6, 3), '999,990.000') cc_secs_per_exec,
       TO_CHAR(ROUND(h.plsexec_time_total / h.executions_total / 1e6, 3), '999,990.000') pl_secs_per_exec,
       TO_CHAR(ROUND(h.javexec_time_total / h.executions_total / 1e6, 3), '999,990.000') ja_secs_per_exec
  FROM dba_hist_sqlstat h, 
       dba_hist_snapshot s
 WHERE h.sql_id = '&sql_id'
   AND h.executions_total > 0 
   AND s.snap_id = h.snap_id
   AND s.dbid = h.dbid
   AND s.instance_number = h.instance_number
UNION ALL  
SELECT 'gv$sqlarea_plan_hash' source,h.inst_id, 
       TO_CHAR(sysdate, 'DD-MM-YYYY HH24:MI') snap_time,
       TO_CHAR(sysdate, 'DD-MM-YYYY HH24:MI') end_time,
       h.sql_id,
       h.plan_hash_value, 
       h.executions,
       TO_CHAR(ROUND(h.rows_processed / h.executions), '999,999,999,999') rows_per_exec,
       TO_CHAR(ROUND(h.elapsed_time / h.executions / 1e6, 3), '999,990.000') et_secs_per_exec,
       TO_CHAR(ROUND(h.cpu_time / h.executions / 1e6, 3), '999,990.000') cpu_secs_per_exec,
       TO_CHAR(ROUND(h.USER_IO_WAIT_TIME / h.executions / 1e6, 3), '999,990.000') io_secs_per_exec,
       TO_CHAR(ROUND(h.CLUSTER_WAIT_TIME / h.executions / 1e6, 3), '999,990.000') cl_secs_per_exec,
       TO_CHAR(ROUND(h.APPLICATION_WAIT_TIME / h.executions / 1e6, 3), '999,990.000') ap_secs_per_exec,
       TO_CHAR(ROUND(h.CLUSTER_WAIT_TIME / h.executions / 1e6, 3), '999,990.000') cc_secs_per_exec,
       TO_CHAR(ROUND(h.PLSQL_EXEC_TIME / h.executions / 1e6, 3), '999,990.000') pl_secs_per_exec,
       TO_CHAR(ROUND(h.JAVA_EXEC_TIME / h.executions / 1e6, 3), '999,990.000') ja_secs_per_exec
  FROM gv$sqlarea_plan_hash h 
 WHERE h.sql_id = '&sql_id'
   AND h.executions > 0 
order by source ;
```

### Find Force Matching Signature

```sql
col exact_matching_signature for 99999999999999999999999999
col sql_text for a50
set long 20000
set  lines 750 pages 9999
select sql_id, exact_matching_signature, force_matching_signature, SQL_TEXT from v$sqlarea where sql_id='&sql_id'
UNION ALL
select sql_id,force_matching_signature,SQL_TEXT from dba_hist_sqltext where sql_id='&sql_id';
```

### If you want to find Bind variable for SQL_ID

```sql
col VALUE_STRING for a50  
SELECT NAME,POSITION,DATATYPE_STRING,VALUE_STRING FROM gv$sql_bind_capture WHERE sql_id='&sql_id';
```

- - -

## 15. FRA FULL — Flash Recovery Area

FRA can fill due to: archivelog, backup files, redo log files.

Check FRA location:

```sql
show parameter db_recovery
desc v$recovery_file_dest
desc V$FLASH_RECOVERY_AREA_USAGE
```

### FRA usage

```sql
SELECT
  ROUND((A.SPACE_LIMIT / 1024 / 1024 / 1024), 2) AS FLASH_IN_GB,
  ROUND((A.SPACE_USED / 1024 / 1024 / 1024), 2) AS FLASH_USED_IN_GB,
  ROUND((A.SPACE_RECLAIMABLE / 1024 / 1024 / 1024), 2) AS FLASH_RECLAIMABLE_GB,
  SUM(B.PERCENT_SPACE_USED)  AS PERCENT_OF_SPACE_USED
FROM
  V$RECOVERY_FILE_DEST A,
  V$FLASH_RECOVERY_AREA_USAGE B
GROUP BY
  SPACE_LIMIT,
  SPACE_USED ,
  SPACE_RECLAIMABLE ;
```

### Clear archive log

```bash
rman target /
crosscheck archivelog all;
list backup;
delete obsolete;
backup archivelog all delete input;
delete archivelog all;
DELETE noprompt archivelog until time 'SYSDATE-5';
```

```sql
set lines 120
break on report
compute sum of percent_space_used on report
compute sum of percent_space_reclaimable on report

select file_type
,      percent_space_used
,      percent_space_reclaimable
,      number_of_files
,      con_id from   v$recovery_area_usage
order by 1;
```

**PROCESS, SESSIONS, TRANSACTIONS:** `sessions= (1.5* processes)+22`, `opt transactions = 1.1*sessions`. Use `desc v$resource_limit` for real-time utilization. To change processes parameter, reboot the database (5-10 min downtime). Use `parallel_max_servers` to limit parallel process.

- - -

## 16. Important Queries

### Inactive session

```sql
select username, status, machine , sql_id , count(*) from v$session  where status='INACTIVE' group by username,machine,status,sql_id  order by count(*);
```

### Kill inactive session

```sql
select 'alter system kill session '''||sid||','||serial#||',@'||inst_id||''' immediate;' from gv$session where sql_id='47j2vxa67zj39';
```

### Setting NLS format

```sql
ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD,HH24:MI:SS';
```

### XPLAN_CACHE

```sql
select * from table(dbms_xplan.display);
```

```sql
insert /*+ parallel(32) */ into CALHEERS_CALHEERS_DB.DR$BG_TS_IX$I@ALM_TO_NON_ALM select * from CALHEERS_CALHEERS_DB.DR$BG_TS_IX$I;
```

### Database info

```sql
select name, open_mode, database_role , a.host_name from v$database, v$instance a;
```

```sql
col "Database Size" format a20
col "Free space" format a20
col "Used space" format a20
select round(sum(used.bytes) / 1024 / 1024 / 1024 ) || ' GB' "Database Size"
, round(sum(used.bytes) / 1024 / 1024 / 1024 ) -
round(free.p / 1024 / 1024 / 1024) || ' GB' "Used space"
, round(free.p / 1024 / 1024 / 1024) || ' GB' "Free space"
from (select bytes from v$datafile
union all select bytes from v$tempfile
union all select bytes from v$log) used
, (select sum(bytes) as p from dba_free_space) free
group by free.p;
```

```sql
Select Sum (Bytes) / 1024 / 1024 / 1024 /1024 As TB From Dba_Data_Files;
```

### Index DDL

```sql
SELECT DBMS_METADATA.get_ddl ('INDEX', index_name, owner)
FROM   all_indexes
WHERE  owner = UPPER('&1');
```

### MRP related

```sql
select dest_id, status, destination, error from v$archive_dest where dest_id<=2;
alter database recover managed standby database disconnect;
alter database recover managed standby database cancel;
select process, status, sequence# from v$managed_standby;
archive log list;
```

```sql
column name format a50;
column "Size (MB)" format 9999.999;
select sequence#, name, blocks*block_size/1024/1024 "Size (MB)" from v$archived_log where status = 'A' and standby_dest = 'NO' and completion_time > sysdate-1 order by name;
```

```sql
select A.*,B.Applied "Last Standby Seq Applied" , A.Received - B.Applied "Gap" from
(select thread#, max(sequence#) Received
from v$archived_log val, v$database vdb
where val.resetlogs_change# = vdb.resetlogs_change#
group by thread#) A,
(select thread#, max(sequence#) Applied
from v$archived_log val, v$database vdb
where val.resetlogs_change# = vdb.resetlogs_change#
and val.applied='YES'
group by thread# ) B
where A.thread#=B.thread#
order by 1;
```

### To calculate the whole size of a table

```sql
SELECT SEGMENT_NAME,SUM(BYTES)/1024/1024/1024 GB FROM DBA_SEGMENTS WHERE SEGMENT_NAME='DM_SUBMIT_INDV_F' GROUP BY SEGMENT_NAME;
```

### DB size

```sql
select
"Reserved_Space(GB)", "Reserved_Space(GB)" - "Free_Space(GB)" "Used_Space(GB)",
"Free_Space(GB)"
from(
select 
(select sum(bytes/(1014*1024*1024)) from dba_data_files) "Reserved_Space(GB)",
(select sum(bytes/(1024*1024*1024)) from dba_free_space) "Free_Space(GB)"
from dual
);
```

### Top 10 table

```sql
SELECT * FROM (select owner, SEGMENT_NAME, SEGMENT_TYPE, BYTES/1024/1024/1024 GB, TABLESPACE_NAME from dba_segments order by 4 desc ) WHERE ROWNUM <= 15;
```

### Restore point

```sql
create restore point BEFORE_DROPPING_OWNER_SCHEMA guarantee flashback database;
```

```sql
COLUMN scn FOR 999,999,999,999,999
COLUMN Incar FOR 99
COLUMN name FOR A25
COLUMN time for a39
SET LINES 400
COLUMN storage_size FOR 999,999,999,999
COLUMN guarantee_flashback_database FOR A3

SELECT
      database_incarnation# as Incar,
      scn,
      name,
      time,
      storage_size/1024/1024  "Occupied size (MB)",
      guarantee_flashback_database
FROM
      v$restore_point;
```

### SQL session details

```sql
col event format a26
col machine for a15
col logon format a16
col program format a15
col username format a10
col terminal format a10
col fg format 99999
set pages 50000 lines 200

select a.inst_id,a.sid, a.serial#, a.username, a.terminal,a.program,
a.event, a.last_call_et elapsed,
a.blocking_session,
a.sql_id,
a.machine,
a.osuser
from gv$session a, gv$process p
where
a.status = 'ACTIVE'
and a.username is not null
and a.paddr = p.addr
order by elapsed;
```

```sql
Set long 20000000
Select sql_fulltext from gv$sql where sql_id='5qdb6yr6xzywr';
```

### ASM usage

```sql
SET LINESIZE 145
SET PAGESIZE 9999
SET VERIFY off
COLUMN group_name FORMAT a20 HEAD 'Disk Group|Name'
COLUMN sector_size FORMAT 99,999 HEAD 'Sector|Size'
COLUMN block_size FORMAT 99,999 HEAD 'Block|Size'
COLUMN allocation_unit_size FORMAT 999,999,999 HEAD 'Allocation|Unit Size'
COLUMN state FORMAT a11 HEAD 'State'
COLUMN type FORMAT a6 HEAD 'Type'
COLUMN total_mb FORMAT 999,999,999 HEAD 'Total Size (MB)'
COLUMN used_mb FORMAT 999,999,999 HEAD 'Used Size (MB)'
COLUMN pct_used FORMAT 999.99 HEAD 'Pct. Used'
break on report on disk_group_name skip 1
compute sum label "Grand Total: " of total_mb used_mb on report
SELECT
name group_name
, sector_size sector_size
, block_size block_size
, allocation_unit_size allocation_unit_size
, state state
, type type
, total_mb total_mb
, (total_mb - free_mb) used_mb
, ROUND((1- (free_mb / total_mb))*100, 2) pct_used
FROM
v$asm_diskgroup where state='CONNECTED'
ORDER BY
name ;
```

```sql
set echo off feed off
set lines 200 pages 200
col FREE_MB for a10
col Total_MB for a20
col instance_name for a30
col host_name for a30
SELECT
    a.name,a.Free_mb/1024 "Free_GB", a.ToTal_Mb/1024 "Total_GB",b.instance_name,b.host_name
FROM
    v$asm_diskgroup a,v$instance b
WHERE
    round((1 -(free_mb / total_mb)) * 100, 2) > 90
ORDER BY
    name;
```

### Standby sync

```sql
SELECT ARCH.THREAD# "Thread", ARCH.SEQUENCE# "Last Sequence Received", APPL.SEQUENCE# "Last Sequence Applied", (ARCH.SEQUENCE# - APPL.SEQUENCE#) "Difference"
FROM
 (SELECT THREAD# ,SEQUENCE# FROM V$ARCHIVED_LOG WHERE (THREAD#,FIRST_TIME ) IN (SELECT THREAD#,MAX(FIRST_TIME) FROM V$ARCHIVED_LOG GROUP BY THREAD#)) ARCH,
(SELECT THREAD# ,SEQUENCE# FROM V$LOG_HISTORY WHERE (THREAD#,FIRST_TIME ) IN (SELECT THREAD#,MAX(FIRST_TIME) FROM V$LOG_HISTORY GROUP BY THREAD#)) APPL
 WHERE
  ARCH.THREAD# = APPL.THREAD#
 ORDER BY 1;
```

### Database size

```sql
select a.data_size+b.temp_size+c.redo_size+d.controlfile_size "total_size in MB" from ( select sum(bytes)/1024/1024 data_size
from dba_data_files) a,( select nvl(sum(bytes),0)/1024/1024 temp_size from dba_temp_files ) b,( select sum(bytes)/1024/1024 redo_size from sys.v_$log ) c,
( select sum(BLOCK_SIZE*FILE_SIZE_BLKS)/1024/1024 controlfile_size from v$controlfile) d;
```

### Constraint DDL

```sql
select dbms_metadata.get_ddl('CONSTRAINT','CODE_DETL_PK','HBX_CODE_DETL') from dual;
select * from dba_constraints  where owner = 'AEDBADMIN'  and constraint_type in ('P');
```

### LOB size

```sql
SELECT OWNER,SEGMENT_NAME,ROUND(SUM(BYTES)/1024/1024/1024) "LOB size (GB)" 
FROM DBA_SEGMENTS
WHERE SEGMENT_NAME IN 
    (
    SELECT SEGMENT_NAME 
    FROM DBA_LOBS WHERE TABLE_NAME  in ('REPOSITORY_ITEM_STATES','REPOSITORY_CONTENT_STORAGE','SCM_CONTENT','SCM_HISTORIC_ENTRY')
    AND OWNER IN ('CCM')
    )
GROUP BY OWNER,SEGMENT_NAME ;
```

### LOB details

```sql
select /*+ parallel(32) */ ((max(length(ST_DESCRIPTION)))/(1024)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select /*+ parallel(32) */ ((max(length(ST_EXPECTED)))/(1024)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select  /*+ parallel(32) */  ((max(length(ST_ACTUAL)))/(1024)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select /*+ parallel(32) */  ((max(length(ST_COMPONENT_DATA)))/(1024)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select /*+ parallel(32) */  ((max(length(ST_BPTA_CONDITION)))/(1024)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select /*+ parallel(32) */  ((max(length(ST_BPT_PATH)))/(1024)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select /*+ parallel(32) */  ((max(length(SP_PARAM_ACTUAL_VALUE)))/(1024))  from CALHEERS_CALHEERS_ATS_DB.STEP_PARAMS;

select sum(dbms_lob.getlength(ST_DESCRIPTION)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select sum(dbms_lob.getlength(ST_EXPECTED)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select sum(dbms_lob.getlength(ST_ACTUAL)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select sum(dbms_lob.getlength(ST_COMPONENT_DATA)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select sum(dbms_lob.getlength(ST_BPTA_CONDITION)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select sum(dbms_lob.getlength(ST_BPT_PATH)) from CALHEERS_CALHEERS_ATS_DB.STEP;
select sum(dbms_lob.getlength(SP_PARAM_ACTUAL_VALUE)) from CALHEERS_CALHEERS_ATS_DB.STEP_PARAMS;
```

### Table size

```sql
set linesize 999
COLUMN TABLE_NAME FORMAT A32
COLUMN OBJECT_NAME FORMAT A32
COLUMN OWNER FORMAT A50
SELECT
   owner, table_name, TRUNC(sum(bytes)/1024/1024/1024) GB
FROM
(SELECT segment_name table_name, owner, bytes
FROM dba_segments
WHERE segment_type = 'TABLE'
UNION ALL
SELECT i.table_name, i.owner, s.bytes
FROM dba_indexes i, dba_segments s
WHERE s.segment_name = i.index_name
AND   s.owner = i.owner
AND   s.segment_type = 'INDEX'
UNION ALL
SELECT l.table_name, l.owner, s.bytes
FROM dba_lobs l, dba_segments s
WHERE s.segment_name = l.segment_name
AND   s.owner = l.owner
AND   s.segment_type = 'LOBSEGMENT'
UNION ALL
SELECT l.table_name, l.owner, s.bytes
FROM dba_lobs l, dba_segments s
WHERE s.segment_name = l.index_name
AND   s.owner = l.owner
AND   s.segment_type = 'LOBINDEX')
GROUP BY table_name, owner
HAVING SUM(bytes)/1024/1024 > 10
ORDER BY SUM(bytes) desc;
```

### Tablespace size

```sql
set colsep |
set linesize 100 pages 100 trimspool on numwidth 14 
col name format a25
col owner format a15 
col "Used (GB)" format a15
col "Free (GB)" format a15 
col "(Used) %" format a15 
col "Size (M)" format a15 
SELECT d.status "Status", d.tablespace_name "Name", 
 TO_CHAR(NVL(a.bytes / 1024 / 1024 /1024, 0),'99,999,990.90') "Size (GB)", 
 TO_CHAR(NVL(a.bytes - NVL(f.bytes, 0), 0)/1024/1024 /1024,'99999999.99') "Used (GB)", 
 TO_CHAR(NVL(f.bytes / 1024 / 1024 /1024, 0),'99,999,990.90') "Free (GB)", 
 TO_CHAR(NVL((a.bytes - NVL(f.bytes, 0)) / a.bytes * 100, 0), '990.00') "(Used) %"
 FROM sys.dba_tablespaces d, 
 (select tablespace_name, sum(bytes) bytes from dba_data_files group by tablespace_name) a, 
 (select tablespace_name, sum(bytes) bytes from dba_free_space group by tablespace_name) f WHERE 
 d.tablespace_name = a.tablespace_name(+) AND d.tablespace_name = f.tablespace_name(+) AND NOT 
 (d.extent_management like 'LOCAL' AND d.contents like 'TEMPORARY') 
UNION ALL 
SELECT d.status 
 "Status", d.tablespace_name "Name", 
 TO_CHAR(NVL(a.bytes / 1024 / 1024 /1024, 0),'99,999,990.90') "Size (GB)", 
 TO_CHAR(NVL(t.bytes,0)/1024/1024 /1024,'99999999.99') "Used (GB)",
 TO_CHAR(NVL((a.bytes -NVL(t.bytes, 0)) / 1024 / 1024 /1024, 0),'99,999,990.90') "Free (GB)", 
 TO_CHAR(NVL(t.bytes / a.bytes * 100, 0), '990.00') "(Used) %" 
 FROM sys.dba_tablespaces d, 
 (select tablespace_name, sum(bytes) bytes from dba_temp_files group by tablespace_name) a, 
 (select tablespace_name, sum(bytes_cached) bytes from v$temp_extent_pool group by tablespace_name) t 
 WHERE d.tablespace_name = a.tablespace_name(+) AND d.tablespace_name = t.tablespace_name(+) AND 
 d.extent_management like 'LOCAL' AND d.contents like 'TEMPORARY';
```

### Tablespace all details

```sql
SELECT
    FILE_ID,
    TABLESPACE_NAME,
    FILE_NAME,
    BYTES / 1024 / 1024 / 1024 AS SIZE_GB,
    MAXBYTES / 1024 / 1024 / 1024 AS MAX_SIZE_GB,
    AUTOEXTENSIBLE,
    INCREMENT_BY * (SELECT VALUE FROM V$PARAMETER WHERE NAME = 'db_block_size') / 1024 / 1024 AS INCREMENT_MB,
    STATUS,
    ONLINE_STATUS,
    LOST_WRITE_PROTECT
FROM
    DBA_DATA_FILES
ORDER BY
    SIZE_GB DESC;
```

### Temp size

```sql
set pages 999
set lines 400
col TABLESPACE_NAME for  a10
col FILE_NAME format a75
select tablespace_name,file_name,bytes/1024/1024/1024 as "GB", maxbytes/1024/1024/1024 "GB" from dba_temp_files where tablespace_name like 'TEMP%';
```

```sql
SELECT property_name, property_value 
FROM database_properties 
WHERE property_name = 'DEFAULT_TEMP_TABLESPACE';

create bigfile temporary tablespace temp3 tempfile '/mnt/temp03/temp03.dbf'  size 1G autoextend on next 1G maxsize 1024G;
create bigfile temporary tablespace temp3 tempfile '/opt/data/CE/datafile/temp03' size 1G autoextend on next 1G maxsize 1024G;
```

### Query info

```sql
WITH t AS (
SELECT
dt,
username,
sql_id,
sql_profile,
plan_hash_value,
executions,
rows_p,
exec_time / executions AS per_time,
disk_read / executions AS per_disk_read,
cpu_time / executions AS per_cpu_time,
io_time / executions AS per_io_time,
rd_bytes / executions AS per_rd_bytes,
rd_buffer / executions AS per_rd_buffer
FROM
(
SELECT
to_char(s.begin_interval_time, 'mm-dd-yyyy hh24:mi') AS dt,
u.username,
t.sql_id,
t.sql_profile,
t.plan_hash_value,
SUM(executions_delta) AS executions,
SUM(rows_processed_delta) AS rows_p,
SUM(elapsed_time_delta) / 1000000 AS exec_time,
SUM(disk_reads_delta) AS disk_read,
SUM(cpu_time_delta) AS cpu_time,
SUM(iowait_delta) AS io_time,
SUM(physical_read_bytes_delta) AS rd_bytes,
SUM(buffer_gets_delta) AS rd_buffer
FROM
dba_hist_sqlstat t,
dba_hist_snapshot s,
dba_users u
WHERE
s.begin_interval_time between sysdate -120 and sysdate
AND t.snap_id = s.snap_id
AND t.dbid = s.dbid
AND t.instance_number = s.instance_number
AND t.parsing_schema_id = u.user_id
GROUP BY
to_char(s.begin_interval_time, 'mm-dd-yyyy hh24:mi'),
u.username,
t.sql_id,
t.sql_profile,
t.plan_hash_value
)
WHERE
executions > 0
)
SELECT
sys_context('USERENV', 'DB_UNIQUE_NAME') AS dbname,
dt,
t.username,
t.sql_id,
executions,
round(per_time, 6) AS per_execution_time_sec,
plan_hash_value,
s.sql_text,
round(per_disk_read),
round(per_cpu_time),
round(per_io_time),
round(per_rd_bytes),
round(per_rd_buffer)
FROM
t,
dba_hist_sqltext s
WHERE
t.sql_id = s.sql_id
and t.sql_id='3mf2nnpb85gaq'
ORDER BY
2 DESC;
```

### Blocking report

```sql
SET TERMOUT OFF;
COLUMN current_instance NEW_VALUE current_instance NOPRINT;
SELECT rpad(instance_name, 17) current_instance FROM v$instance;
SET TERMOUT ON;

SET ECHO OFF
SET FEEDBACK 6
SET HEADING ON
SET LINESIZE 256
SET PAGESIZE 50000
SET TERMOUT ON
SET TIMING OFF
SET TRIMOUT ON
SET TRIMSPOOL ON
SET VERIFY OFF

CLEAR COLUMNS
CLEAR BREAKS
CLEAR COMPUTES

COLUMN instance_name FORMAT a9 HEADING 'Instance'
COLUMN locking_oracle_user FORMAT a20 HEADING 'Locking Oracle User'
COLUMN sid_serial FORMAT a15 HEADING 'SID / Serial#'
COLUMN mode_held FORMAT a15 HEADING 'Mode Held'
COLUMN mode_requested FORMAT a15 HEADING 'Mode Requested'
COLUMN lock_type FORMAT a15 HEADING 'Lock Type'
COLUMN object FORMAT a42 HEADING 'Object'
COLUMN program FORMAT a20 HEADING 'Program'
COLUMN lock_time_min FORMAT 999,999 HEADING 'Lock Time (min)'

select distinct * from (SELECT
i.instance_name instance_name
, l.sid || ' / ' || s.serial# sid_serial
, s.username locking_oracle_user
, DECODE( l.lmode
, 1, NULL
, 2, 'Row Share'
, 3, 'Row Exclusive'
, 4, 'Share'
, 5, 'Share Row Exclusive'
, 6, 'Exclusive'
, 'None') mode_held
, DECODE( l.request
, 1, NULL
, 2, 'Row Share'
, 3, 'Row Exclusive'
, 4, 'Share'
, 5, 'Share Row Exclusive'
, 6, 'Exclusive'
, 'None') mode_requested
, DECODE ( l.type
, 'CF', 'Control File'
, 'DX', 'Distributed Transaction'
, 'FS', 'File Set'
, 'IR', 'Instance Recovery'
, 'IS', 'Instance State'
, 'IV', 'Libcache Invalidation'
, 'LS', 'Log Start or Log Switch'
, 'MR', 'Media Recovery'
, 'RT', 'Redo Thread'
, 'RW', 'Row Wait'
, 'SQ', 'Sequence Number'
, 'ST', 'Diskspace Transaction'
, 'TE', 'Extend Table'
, 'TT', 'Temp Table'
, 'TX', 'Transaction'
, 'TM', 'DML'
, 'UL', 'PLSQL User_lock'
, 'UN', 'User Name'
, 'Nothing'
) lock_type
, o.owner || '.' || o.object_name object
, ROUND(l.ctime/60, 2) lock_time_min
FROM
gv$instance i
, gv$session s
, gv$lock l
, dba_objects o
, dba_tables t
WHERE
l.id1 = o.object_id
AND s.sid = l.sid
AND o.owner = t.owner
AND o.object_name = t.table_name
AND o.owner <> 'SYS'
AND l.type = 'TM'
AND i.inst_id=s.inst_id
ORDER BY
i.instance_name
, l.sid);
```

### Query to find the full tablescan SQLs

```sql
select *
 from
 (select distinct sql_id, object_name , num_rows,
 (select sum(executions)
from gv$sqlarea
where sql_id = p.sql_id ) executions
from dba_hist_sql_plan p, dba_tables t
 where options = 'FULL' 
 and operation = 'TABLE ACCESS'
 and object_name = table_name
 and owner = 'PRD_HBX_OWNER'
and num_rows > 10000
)
where executions > 100
order by 4 DESC;
```

```sql
create tablespace DUMMYWALLET  datafile size 1m autoextend on encryption using 'AES256' default storage (encrypt);
```

- - -

## 17. BACKUP JOB

### Backup job details summary

```sql
set lines 999 pages 999
col time_taken_display format a30
select
decode(to_char(j.start_time, 'd'), 1, 'Sunday', 2, 'Monday',
3, 'Tuesday', 4, 'Wednesday',
5, 'Thursday', 6, 'Friday',
7, 'Saturday') "START DAY"
,to_char(j.start_time, 'yyyy-mm-dd hh24:mi:ss') "START TIME"
,decode(j.status,'RUNNING',null,to_char(j.end_time, 'yyyy-mm-dd hh24:mi:ss')) "END TIME"
,x.output_bytes OUTPUT_MBYTES
,decode(j.status,'FAILED','** FAILED **',
'COMPLETED WITH ERRORS','** COMPLETED WITH ERRORS **',
'RUNNING','** RUNNING **',j.status) STATUS
,decode(x.itype,0,'** LEVEL 0 **',1,'LEVEL 1',decode(x.output_bytes,null,' ',j.input_type)) "BACKUP TYPE"
,nvl(j.time_taken_display,'00:00:00') TIME_TAKEN_DISPLAY
from
V$RMAN_BACKUP_JOB_DETAILS j
left outer join (select
d.session_recid, d.session_stamp
,max(d.incremental_level) ITYPE
,sum(output_bytes)/1024/1024 OUTPUT_BYTES
from
V$BACKUP_SET_DETAILS d
join V$BACKUP_SET s on s.set_stamp = d.set_stamp
and s.set_count = d.set_count
where s.input_file_scan_only = 'NO'
group by d.session_recid, d.session_stamp) x
on x.session_recid = j.session_recid
and x.session_stamp = j.session_stamp
where j.start_time > trunc(sysdate)-40
order by j.start_time;
```

### Backup detail with size

```sql
set lines 999
alter session set nls_date_format = 'DD-MON-YYYY HH24:MI:SS';
SELECT SESSION_KEY,INPUT_TYPE,STATUS,START_TIME,END_TIME,ELAPSED_SECONDS/3600 hrs
FROM V$RMAN_BACKUP_JOB_DETAILS;
```

```sql
col STATUS format a30
col hrs format 999.99
set linesize 200
col start_time format a20
col end_time format a20
select
SESSION_KEY, INPUT_TYPE, STATUS,
to_char(START_TIME,'mm/dd/yy hh24:mi') start_time,
to_char(END_TIME,'mm/dd/yy hh24:mi') end_time,
elapsed_seconds/3600 hrs,
OUTPUT_BYTES/1024/1024/1024 OUTPUT_BYTES_GB
from V$RMAN_BACKUP_JOB_DETAILS
order by session_key;
```

```sql
select to_char(COMPLETION_TIME,'DD/MON/YYYY') Day,
sum(blocks*block_size)/1048576/1024 "Size(GB)",
count(sequence#) "Total Archives"
from (select distinct sequence#,
thread#,
COMPLETION_TIME,
blocks,
block_size
from v$archived_log
where completion_time>=sysdate-30)
group by to_char(COMPLETION_TIME,'DD/MON/YYYY')
order by 1;
```

### In case of archivelog disk filled 100% and archivelog gets deleted

**Never delete archivelog directly from disk without taking backup — instead trigger backup.**

Error: `RMAN-03002: failure of backup command`, `RMAN-06059: expected archived log not found`, `ORA-19625: error identifying file`

Solution:

```bash
rman target /
change archivelog all crosscheck;
trigger backup again
```

### Compression options

Basic, low, medium, high.

```bash
rman target /
CONFIGURE COMPRESSION ALGORITHM 'HIGH';
CONFIGURE COMPRESSION ALGORITHM 'BASIC';
CONFIGURE COMPRESSION ALGORITHM 'HIGH' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE;
```

### Delete archivelog sysdate

```sql
delete archivelog all backed up 1 time to device_type DISK until time 'SYSDATE-4';
DELETE noprompt archivelog until time 'SYSDATE-0';
```

### Kill the RMAN backup session in Oracle

```sql
col sid for 9999
col serial# for 9999999
col spid for a5
col client_info for a20
select b.sid, b.serial#, a.spid, b.client_info
from v$process a, v$session b
where a.addr=b.paddr and client_info like 'rman%';
```

### Kill all the session with commands from SQLPLUS

```sql
select 'Alter system kill session '''||b.sid||','||b.serial#||''' immediate;'
from v$process a, v$session b
where a.addr=b.paddr and client_info like 'rman%';
```

```bash
SQL> host orakill xe 6916
```

### List available backup

```bash
RMAN> list backup of database;
RMAN> list backup of archivelog all;
RMAN> list backup of controlfile;
```

### Use recover database until cancel

Lets you manually cancel recovery when it asks for an archive log you don't have.

```bash
recover database until cancel;
```

```sql
SELECT TO_CHAR(MAX(completion_time), 'YYYY-MM-DD HH24:MI:SS') AS backup_completion_time
FROM v$backup_set;
```

### Check using backup end time and get the timestamp used for recover the database

```sql
run {
  set until time "to_date('2025-05-23 23:48:30', 'YYYY-MM-DD HH24:MI:SS')";
  startup mount;
  restore database;
  recover database;
  alter database open resetlogs;
}
```

- - -

## 18. FORMATTING IN SQLPLUS

```sql
Show pagesize
Show linesize
Set pagesize 300
Set linesize 132
ttitle 'TOP TITLE'
btitle 'BOTTOM TITLE'
col col_name format A30;
Spool filename;
Show feedback 
Set feedback on
```

Container and pluggable:

```sql
Show con_name;
V$pdbs;
```

- - -

## 19. MANAGING INSTANCE

### Parameter file

* **Server parameter file (SPFILE):** Binary file. Use `alter system` to change parameters. Cannot write manually. `Spfile<SID>.ora`
* **Parameter file (PFILE):** Key-value pair file. Can write manually. After changing, restart instance to refresh. `init.ora`. Location: `$ORACLE_HOME/dbs`

### When instance get started

* `Startup;` — Searches for spfile&lt;sid&gt;.ora, then spfile.ora, then init&lt;sid&gt;.ora
* `Startup pfile=filename;` — If pfile name is not init&lt;sid&gt;.ora

### Parameters

```sql
V$parameter;
V$parameter2;
V$PARAMETER2;
V$parameter; (in effect only for this session)
-- ISSES_MODIFIABLE: is session modifiable
-- ISSYS_MODIFIABLE: is system modifiable
-- ISPDB_MODIFIABLE: is pdb modifiable

Show parameter parameter_name;
Show parameter block;
```

**Basic parameter:** ~30 in count (e.g., DB_BLOCK_SIZE, SGA_TARGET)
**Advanced parameter:** e.g., DB_CACHE_SIZE
**Derived parameter:** Calculated from other parameters. Sessions=(1.5*PROCESS)+22

### Change the parameter

* **MEMORY** — Change happens immediately but is lost after shutdown
* **SPFILE** — Change visible only after shutdown and restart
* **BOTH** — Both memory and spfile
* **DEFERRED** — Change visible only for future sessions

```sql
Alter session set parameter_name='value';
Alter session set current_schema=ENV_HBX_OWNER;
ALTER SESSION SET NLS_DATE_FORMAT='dd-mm-yyyy';
```

```sql
Create pfile='initorcl.ora' from spfile;
Show parameter spfile;
Startup pfile='pfile_name';
```

### SCOPE

```sql
Alter system set parameter=value scope=memory;
Alter system set parameter=value scope=spfile;
Alter system set parameter=value scope=both;
Alter system reset parameter scope=both;
Alter system set parameter=value container= CURRENT | ALL;
```

### Default scope in alter statement

If spfile was used to start the database, then `scope=both` is default. If pfile was used, then `scope=memory` is default.

- - -

## 20. ADR (AUTOMATIC DIAGNOSTIC REPOSITORY)

```bash
echo $ORACLE_BASE
```

```sql
Show parameter diagnostic_dest;
```

ADR structure: `<ADR_BASE>/diag/rdbms/databasename/SID/alert,trace,cdump,incident,others`

```bash
adrci
show alert
exit
```

- - -

## 21. User Management in Oracle

### Create a user

```sql
create user DEV_CLASS identified by DEV_CLASS#1234
PROFILE DEFAULT
DEFAULT TABLESPACE USERS
TEMPORARY TABLESPACE TEMP;
grant create session to DEV_CLASS;
```

### Change password of a user

```sql
alter user DEV_CLASS identified by DEV_CLASS#91234;
```

### Lock/unlock a user

```sql
alter user dev_class account lock;
alter user dev_class account unlock;
```

### Make a user password expiry

```sql
alter user dev_class account expire;
```

### Changing default tablespace of a user

```sql
select username,default_tablespace from dba_users where username='DEV_CLASS';
alter user DEV_CLASS default tablespace DATATS;
```

### Changing default TEMP tablespace of a user

```sql
select username,TEMPORARY_TABLESPACE from dba_users where username='DEV_CLASS';
alter user DEV_CLASS temporary tablespace TEMP2;
```

### Profile

A profile enforces password security rules and resource usage limits. If no profile is mentioned when creating a user, DEFAULT profile is assigned.

#### Default profile setting

```sql
col limit for a12
col profile for a14
set lines 200
set pagesize 200
select profile,resource_name,RESOURCE_TYPE,limit from dba_profiles where profile='DEFAULT';
```

#### Create a new profile

```sql
CREATE PROFILE "APP_PROFILE"
LIMIT
COMPOSITE_LIMIT UNLIMITED
SESSIONS_PER_USER UNLIMITED
CPU_PER_SESSION UNLIMITED
CPU_PER_CALL UNLIMITED
LOGICAL_READS_PER_SESSION UNLIMITED
LOGICAL_READS_PER_CALL UNLIMITED
IDLE_TIME 90
CONNECT_TIME UNLIMITED
PRIVATE_SGA UNLIMITED
FAILED_LOGIN_ATTEMPTS 10
PASSWORD_LIFE_TIME 180
PASSWORD_REUSE_TIME UNLIMITED
PASSWORD_REUSE_MAX UNLIMITED
PASSWORD_VERIFY_FUNCTION NULL
PASSWORD_LOCK_TIME UNLIMITED
PASSWORD_GRACE_TIME UNLIMITED;
```

#### Alter a profile

```sql
ALTER PROFILE APP_PROFILE LIMIT FAILED_LOGIN_ATTEMPS UNLIMITED;
```

### How to make a user non-expiry

```sql
ALTER PROFILE APP_PROFILE LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```

### Privileges

#### System privilege

```sql
Grant create any table,alter any table to DEV_CLASS;
select privilege,grantee from dba_sys_privs where grantee='DEV_CLASS';
REVOKE create any table from dev_class;
```

#### Object privilege

```sql
grant insert,update,delete on SIEBEL.TEST2 to DEV_CLASS;
grant execute on SIEBLE.DAILYPROC to DEV_CLASS;
select grantee,owner,table_name,privilege from dba_tab_privs where grantee='DEV_CLASS';
revoke update on siebel.test2 from DEV_CLASS;
```

### Create a role

```sql
create role DEV_ROLE;
grant create session to dev_role;
grant select any table to dev_role;
grant insert on siebel.test2 to dev_role;
select role,privilege from role_sys_privs where role='DEV_ROLE';
select role,owner,table_name,privilege from role_tab_privs where role='DEV_ROLE';
grant dev_role to dev_class;
select grantee,GRANTED_ROLE from dba_role_privs where granted_role='DEV_ROLE';
drop user DEV_CLASS cascade;
Drop role DEV_ROLE;
```

### Data dictionary views

* `session_privs`, `role_sys_privs`, `user_sys_privs`
* `user_tab_privs_recd`, `user_tab_privs_made`
* `user_col_privs_recd`, `user_col_privs_made`
* `user_role_privs`, `dba_role_privs`

### Some user administrative account

* **SYS** — Super user
* **SYSTEM** — Other than backup everything
* **SYSDBA** — Super privilege
* **SYSBACKUP** — Taking backup
* **SYSDG** — Dataguard
* **SYSRAC** — RAC
* **SYSKM** — TDE
* **SYSMAN** — OEM administration
* **SYSASM** — ASM

### Oracle supplied roles

DBA, RESOURCE, SELECT_CATALOG_ROLE, SCHEDULER_ADMIN

### User profile

* **RESOURCE type:** RESOURCE_LIMITS \[KERNEL] must be true for IDLE_TIME
* **PASSWORD:** PASSWORD EXPIRE AND AGING

- - -

## 22. TABLESPACE MANAGEMENT

### Logical storage

(ROWPIECE) → DATA BLOCKS (8KB) → EXTENTS → SEGMENTS (table, index) → TABLESPACE (data files) → DATABASE

### Physical storage

DATA_FILES → FILE SYSTEMS (NFS, ASM, NAS)

### Default tablespaces

* **SYSTEM** — Stores data dictionary
* **SYSAUX**
* **UNDO** — Rollback, stores uncommitted changes
* **TEMP** — Sort and join operations (ORDER BY, GROUP BY, SELECT DISTINCT, UNION, CREATE INDEX)
* **USERS** — User tables

### Tablespace creation

Types: PERMANENT, UNDO, TEMP

* **BIGFILE TABLESPACE:** 1 datafile, up to 4 billion blocks
* **SMALLFILE TABLESPACE:** Up to 1022 datafiles

```sql
create tablespace tablespace_name datafile datafile_name size 10m;
create tablespace tablespace_name datafile datafile_name size 10m autoextend on next 512k maxsize 50m;
```

```sql
CREATE TABLESPACE "PRD_IDX" DATAFILE SIZE 100M AUTOEXTEND ON NEXT 10M MAXSIZE 30G 
LOGGING ONLINE PERMANENT BLOCKSIZE 8192 EXTENT MANAGEMENT LOCAL AUTOALLOCATE 
ENCRYPTION USING 'AES256' DEFAULT NOCOMPRESS STORAGE(ENCRYPT) SEGMENT SPACE MANAGEMENT AUTO;
```

```sql
drop tablespace tablespace_name;
drop tablespace tablespace_name including contents;
drop tablespace tablespace_name including contents and datafiles;
alter tablespace tablespace_name offline;
alter tablespace tablespace_name online;
alter tablespace tablespace_name read only;
alter tablespace tablespace_name read write;
```

### To change size of tablespace

```sql
ALTER TABLESPACE tablespace_name ADD DATAFILE datafile_name size 10M;
ALTER DATABASE datafile datafile_name RESIZE 200M;
```

### Should take tablespace offline before moving/renaming datafile

```sql
Alter database move datafile datafile_name_old to datafile_name_new;
Alter database move datafile old_path to new_path;
alter tablespace tbs1 rename to tbs2;
alter database rename file datafile1 to datafile2;
```

### Temporary tablespace

```sql
create temporary tablespace tablespace_name tempfile tempfile_name size 10m autoextend on next 512k maxsize 50m;
select * from database_properties where property_name like '%tablespace%';
alter database default temporary tablespace temp1;
```

### Undo tablespace

```sql
show parameter undo;
show parameter undo_tablespace;
create undo tablespace undotbs2 datafile /u01/app/oracle/dbms/datafile1.dbf size 10M;
alter system set undo_tablespace=undotbs2 scope both;
alter system set undo_retention=2400;
alter tablespace undotbs1 retention guarantee;
alter tablespace undotbs1 retention noguarantee;
```

- - -

## 23. Data Dictionary Views

Provides info on database tables, indexes, tablespaces, users.

```sql
DICTIONARY;
SYS.TAB$;
```

### Static views

* **CDB_** — Container database
* **DBA_** — Entire database (pluggable)
* **ALL_** — Current user schema + objects user has privileges on
* **USER_** — Current user schema only

Examples: DBA_TABLES, DBA_EXTENTS, DBA_FREE_SPACE, DBA_VIEWS, DBA_INDEXES, DBA_TABLESPACES, DBA_DATA_FILES, DBA_SEGMENTS

### Dynamic views

* **V$** — Instance-level
* **GV$** — Global (RAC)

Examples: V$parameter, V$database, V$instance, V$session, V$lock, V$transaction, V$logfile, V$log, V$archived_log

### Miscellaneous views

SESSION_PRIVS, DICTIONARY, DICT_COLUMNS, TABLE_PRIVILEGES

- - -

## 24. UNDO VS REDO

* **UNDO:** Stores uncommitted changes for rollback
* **REDO:** Stores committed changes for database recovery

### Local vs shared undo tablespace

`LOCAL_UNDO_ENABLED=TRUE` in DATABASE_PROPERTIES for undo tablespace in each PDB. When cloning PDB with uncommitted transactions, LOCAL_UNDO_ENABLED must be TRUE or clone fails with ORA-65035.

```sql
shutdown immediate;
startup upgrade;
alter database local undo off/on;
shutdown immediate;
startup;
```

- - -

## 25. Get DDL

```sql
SELECT DBMS_METADATA.GET_DDL('TABLE', 'table_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('VIEW', 'view_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('PROCEDURE', 'procedure_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('FUNCTION', 'function_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('PACKAGE', 'package_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('PACKAGE_BODY', 'package_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('TRIGGER', 'trigger_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('SEQUENCE', 'sequence_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('SYNONYM', 'synonym_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('INDEX', 'index_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('USER', 'user_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('ROLE', 'role_name') FROM DUAL;
SELECT DBMS_METADATA.GET_DDL('TABLESPACE','USERS') FROM dual;
SELECT DBMS_METADATA.GET_DEPENDENT_DDL('REF_CONSTRAINT','<table_name>','<schema>') from dual;
SELECT DBMS_METADATA.GET_GRANTED_DDL('SYSTEM_GRANT','<schema>') from dual;
SELECT DBMS_METADATA.GET_GRANTED_DDL('ROLE_GRANT','<schema>') from dual;
SELECT DBMS_METADATA.GET_GRANTED_DDL('OBJECT_GRANT','<schema>') from dual;
```

- - -

## 26. Statspack

```sql
list snapshot: select name,snap_id,to_char(snap_time,'DD-MON-YYYY:HH24:MI:SS') "Date/Time" from stats$snapshot,v$database;
```

```bash
@$ORACLE_HOME/rdbms/admin/spreport.sql
```

```sql
select SNAP_ID, SNAP_LEVEL from STATS$SNAPSHOT;
exec statspack.modify_statspack_parameter(i_snap_level=>10, i_modify_parameter=>'true');
exec statspack.snap;
@spauto.sql
```

```sql
alter session set nls_date_format = 'DD-MON-YYYY HH24:MI:SS';
set lines 180
col SCHEMA_USER for a20
col INTERVAL for a30
col WHAT for a30
select JOB, SCHEMA_USER, INTERVAL, BROKEN, WHAT from dba_jobs where JOB=1;
```

Use `ORACLE_HOME/rdbms/admin/sppurge.sql` to purge snapshot range. Use `spdrop.sql` to uninstall Statspack.

```sql
select a.instance_number,to_char(a.snap_time,'dd/mon/yyyy hh24:mi') meas_date, b.value
 from stats$snapshot a, stats$sysstat b, v$statname c
 where c.name='sorts (disk)'
 and a.snap_time>sysdate-7
 and c.statistic#=b.statistic#
 and b.snap_id=a.snap_id
 order by a.instance_number,a.snap_time;
```

- - -

## 27. Gather table stats

```sql
EXEC DBMS_STATS.gather_table_stats('MSO_DB','PRODUCT_IMAGE_LIST',cascade=>TRUE);
```

```sql
sqlplus / as sysdba
Select * from V$version;
```

- - -

## 28. Websites

**Reference:**

* Migrating Oracle AWS RDS using RMAN Backup: https://www.amazonaws.cn/en/blog-selection/migrate-oracle-database-workloads-from-amazon-rds-for-oracle-to-amazon-rds-custom-for-oracle/
* AWS RDS Oracle external table: https://www.linkedin.com/pulse/data-pump-external-tables-aws-oracle-rds-sanjay-arya/
* https://www.ludovicocaldara.net/dba/
* https://www.mirsayeedhassan.com/
* https://rupeshanantghubade.blogspot.com/
* SQL tuning scripts: https://github.com/bobbydurrett/OracleDatabaseTuningSQL
* https://support.dbagenesis.com/oracle-database/oracle-19c-installation-on-linux
* https://ahmedfattah.com/?p=1405
* Rollback a patch: https://dohdatabase.com/2025/01/13/how-to-roll-back-after-patching/
* New patch: https://mikedietrichde.com/2022/05/17/simple-database-installation-with-applyru-and-applyoneoffs/
* Create new database: https://paggyru.medium.com/7-steps-to-create-a-new-oracle-database-from-the-command-line-f802938fa1f6
* Export/import RDS Oracle: https://medium.com/@gupta.megha/export-import-rds-oracle-database-using-oracle-data-pump-53c7d60d074e
* Export/import to Azure blob: https://database-heartbeat.com/2022/03/03/azure-blob-to-oracle-database/

- - -

## 29. Performance tuning

* Read "SQL Performance Explained" by Markus Winand: https://use-the-index-luke.com/
* DevGym: https://devgym.oracle.com/pls/apex/f?p=10001:1061::::::
* COE: https://heliosguneserol.com/2020/04/02/what-is-coe-sql-and-how-we-can-set-execution-plan-for-any-sql/
* https://easyoradba.com/2020/09/12/oracle-rds-performance-tuning-queries/
* https://aws.amazon.com/blogs/database/managing-your-sql-plan-in-oracle-se-with-amazon-rds-for-oracle/
* https://www.br8dba.com/tag/manually-run-sql-advisor/
* Non CDB to PDB: https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=103493439082889&parent=EXTERNAL_SEARCH&sourceId=HOWTO&id=2012448.1

```sql
@?/rdbms/admin/sqltrpt.sql
```

- - -

## 30. Delete unwanted file from RDS Oracle datapump directory

```sql
SELECT * FROM TABLE(rdsadmin.rds_file_util.listdir('DATA_PUMP_DIR')) ORDER BY MTIME;
EXEC UTL_FILE.FREMOVE('DATA_PUMP_DIR','LFR_01.dmp');
```

- - -

## 31. Resize undo

Reference: https://db-master.com/oracle/resizing-the-undo-tablespace/

- - -

## 32. Find command

```bash
find . -mtime +0   # find files modified greater than 24 hours ago
find . -mtime 0    # find files modified in past 24 hours only
find . -mtime -1   # find files modified less than 1 day ago (SAME AS -mtime 0)
find . -mtime 1    # find files modified between 24 and 48 hours ago
find . -mtime +1   # find files modified more than 48 hours ago
```

- - -

## 33. SDIFF TUTORIAL

`sdiff` used to compare two files. `>` symbol shown when line is only present in one file.

```bash
sdiff -l file1 file2
sdiff -s file1 file2
sdiff -s -o outputfilename file1 file2
sdiff -l -B -I  -s  txt1.sql txt2.sql
```

- - -

## 34. OSB

```bash
./aws configure --profile db-load
java -jar osbws_install.jar -AWSID YOUR_AWS_ACCESS_KEY_ID -AWSKey YOUR_AWS_SECRET_ACCESS_KEY -walletDir $ORACLE_HOME/dbs/osbws_wallet -libDir $ORACLE_HOME/lib -awsEndPoint s3.us-west-2.amazonaws.com -location us-west-2 -useHttps
```

- - -

## 35. Change archivelog dest in prod

```sql
alter system set log_archive_dest='/opt/archive/archivelogs' scope=both;
alter system set log_archive_dest='/opt/backup/archivelogs' scope=both;
```

- - -

## 36. Fragmentation: re-org

### Re-org table candidate

```sql
SELECT 
  tablespace_name,        
  SUM(bytes) / 1024 / 1024/1024 AS free_space_gb   
FROM dba_free_space  
GROUP BY tablespace_name;
```

### Fragmented space

```sql
set lines 200
col table_name for a25
col total_size for a20
col ACTUAL_SIZE for a20
col FRAGMENTED_SPACE for a20
SELECT      
  table_name,     
  avg_row_len,     
  ROUND(((blocks*16/1024)),2) || 'MB' AS "TOTAL_SIZE",     
  ROUND((num_rows*avg_row_len/1024/1024),2) || 'MB' AS "ACTUAL_SIZE",     
  ROUND(((blocks*16/1024) - (num_rows*avg_row_len/1024/1024)),2) || 'MB' AS "FRAGMENTED_SPACE",     
  (ROUND(((blocks*16/1024) - (num_rows*avg_row_len/1024/1024)),2) / ROUND(((blocks*16/1024)),2)) * 100 AS "PERCENTAGE" 
FROM dba_tables  
WHERE ROUND(((blocks*16/1024)),2) > 0 
and table_name='RETAIL_LOAD_OFFER' and owner='MSO_SRC'
ORDER BY 6 DESC;
```

### For all tables

```sql
set lines 200
col owner for a20
col table_name for a25
col total_size for a20
col ACTUAL_SIZE for a20
col FRAGMENTED_SPACE for a20
SELECT      
  table_name, 
owner,  
  avg_row_len,     
  ROUND(((blocks*16/1024)),2)  AS "TOTAL_SIZE_MB",     
  ROUND((num_rows*avg_row_len/1024/1024),2) AS "ACTUAL_SIZE_MB",     
  ROUND(((blocks*16/1024) - (num_rows*avg_row_len/1024/1024)),2)  AS "FRAGMENTED_SPACE_MB",     
  (ROUND(((blocks*16/1024) - (num_rows*avg_row_len/1024/1024)),2) / ROUND(((blocks*16/1024)),2)) * 100 AS "PERCENTAGE" 
FROM dba_tables  
WHERE ROUND(((blocks*16/1024)),2) > 0 
ORDER BY 4 DESC fetch first 20 rows only;
```

### Top 10 tables

```sql
SELECT * FROM
(select 
 SEGMENT_NAME, 
 SEGMENT_TYPE, 
 BYTES/1024/1024/1024 GB, 
 TABLESPACE_NAME 
from 
 dba_segments
order by 3 desc ) WHERE
ROWNUM <= 10;
```

### High water mark calculation

```sql
set linesize 400
col tablespace_name format a15
col file_size format 99999
col file_name format a50
col hwm format 99999
col can_save format 99999
SELECT tablespace_name, file_name, file_size, hwm, file_size-hwm can_save
FROM (SELECT /*+ RULE */ ddf.tablespace_name, ddf.file_name file_name,
ddf.bytes/1048576 file_size,(ebf.maximum + de.blocks-1)*dbs.db_block_size/1048576 hwm
FROM dba_data_files ddf,(SELECT file_id, MAX(block_id) maximum FROM dba_extents GROUP BY file_id) ebf,dba_extents de,
(SELECT value db_block_size FROM v$parameter WHERE name='db_block_size') dbs
WHERE ddf.file_id = ebf.file_id
AND de.file_id = ebf.file_id
AND de.block_id = ebf.maximum
ORDER BY 1,2);
```

### Resize datafiles to new size (slightly greater than HWM)

```sql
alter database datafile '/opt/data/CE/datafile/client176.dbf' resize 1176G;
```

### Active session

```sql
select b.inst_id,b.sid,b.serial#,a.spid, substr(b.machine,1,30) box,to_char (b.logon_time, 'dd-mon-yyyy hh24:mi:ss') logon_time,
substr(b.username,1,30) username,
substr(b.osuser,1,20) os_user,
substr(b.program,1,30) program,status,b.last_call_et AS last_call_et_secs,b.sql_id ,c.sql_text 
from gv$session b,gv$process a , v$sql c
where b.paddr = a.addr 
and a.inst_id = b.inst_id  
and type='USER'
and b.status='ACTIVE'
AND b.sql_id= c.sql_id
order by logon_time;
```

- - -

## 37. To check dependency of view on tables, functions

```sql
SET VERIFY OFF
SET LINESIZE 255
SET PAGESIZE 1000

COLUMN referenced_object FORMAT A50
COLUMN referenced_type FORMAT A20
COLUMN referenced_link_name FORMAT A20
col owner for a10
col name for a40
SELECT a.owner, a.name , a.type, RPAD('', level*2, ' ') || a.referenced_owner || '.' || a.referenced_name AS referenced_object,
       a.referenced_type,
       a.referenced_link_name
FROM   all_dependencies a
WHERE  a.owner NOT IN ('SYS','SYSTEM','PUBLIC')
AND    a.referenced_owner NOT IN ('SYS','SYSTEM','PUBLIC')
AND    a.referenced_type != 'NON-EXISTENT'
START WITH a.owner = UPPER('CLIENT292ARCHIVE')
AND        a.name  in ('MV_CM2_OFFER_BASE_TEST_021425',
'BCKP_STG_220811_V_PA_OUTPUT_SOURCE_PRE',
'BCKP_STG_220822_PA_OUTPUT_SOURCE_PRE',
'BCKP_STG_221109_PA_OUTPUT_SOURCE_PRE',
'PA_OUTPUT_SOURCE_PRE_TEST_050924')
CONNECT BY a.owner = PRIOR a.referenced_owner
AND        a.name  = PRIOR a.referenced_name
AND        a.type  = PRIOR a.referenced_type;
```

- - -

## 38. Oracle unused space claim article

https://oracle-base.com/articles/misc/reclaiming-unused-space#setup_test_environment

- - -

## 39. Bash profile content

```bash
vi .bash_profile 
export ORACLE_HOSTNAME="`cat /etc/hostname`"
export ORACLE_UNQNAME=ce
export ORACLE_BASE=/opt/oracle/base
export ORACLE_HOME=$ORACLE_BASE/product/19
export ORACLE_SID=ce
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=/usr/sbin:/usr/local/bin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
```

- - -

## 40. LFR logging check

```sql
SELECT * FROM lfr.lfr_statistic;
SELECT * FROM lfr.v_lfr_stat   ORDER BY start_date DESC ;
SELECT * FROM lfr.v_lfr_stat WHERE NAM_PROCEDURE='etl2.sh' ORDER BY start_date DESC FETCH FIRST 50 ROWS only;
```

- - -

## 41. AWS RDS datapump files with size

```sql
SELECT 
  filename,
  type,
  ROUND(filesize / (1024 * 1024 * 1024), 2) AS size_gb,
  mtime
FROM TABLE(
  rdsadmin.rds_file_util.listdir(p_directory => 'DATA_PUMP_DIR')
)
WHERE type = 'file' and filename like '%.dmp'
ORDER BY mtime DESC;
```

- - -

## 42. AWS RDS storage query

```sql
select
'===========================================================' || chr(10) ||
'Total Database Physical Size = ' || round(redolog_size_gib+dbfiles_size_gib+tempfiles_size_gib+ctlfiles_size_gib,2) || ' GiB' || chr(10) ||
'===========================================================' || chr(10) ||
' Redo Logs Size : ' || round(redolog_size_gib,3) || ' GiB' || chr(10) ||
' Data Files Size : ' || round(dbfiles_size_gib,3) || ' GiB' || chr(10) ||
' Temp Files Size : ' || round(tempfiles_size_gib,3) || ' GiB' || chr(10) ||
' Archive Log Size - Approx only : ' || round(archlog_size_gib,3) || ' GiB' || chr(10) ||
' Control Files Size : ' || round(ctlfiles_size_gib,3) || ' GiB' || chr(10) ||
'===========================================================' || chr(10) ||
' Used Database Size : ' || used_db_size_gib || ' GiB' || chr(10) ||
' Free Database Size : ' || free_db_size_gib || ' GiB' ||chr(10) ||
' Data Pump Directory Size : ' || dpump_db_size_gib || ' GiB' || chr(10) ||
' BDUMP Directory Size : ' || bdump_db_size_gib || ' GiB' || chr(10) ||
' ADUMP Directory Size : ' || adump_db_size_gib || ' GiB' || chr(10) ||
'===========================================================' || chr(10) ||
'Total Size (including Dump and Log Files) = ' || round(round(redolog_size_gib,2) +round(dbfiles_size_gib,2)+round(tempfiles_size_gib,2)+round(ctlfiles_size_gib,2) +round(adump_db_size_gib,2) +round(dpump_db_size_gib,2)+round(bdump_db_size_gib,2),2) || ' GiB' || chr(10) ||
'===========================================================' as summary
FROM (SELECT sys_context('USERENV', 'DB_NAME')
db_name,
(SELECT SUM(bytes) / 1024 / 1024 / 1024 redo_size
FROM v$log)
redolog_size_gib,
(SELECT SUM(bytes) / 1024 / 1024 / 1024 data_size
FROM dba_data_files)
dbfiles_size_gib,
(SELECT nvl(SUM(bytes), 0) / 1024 / 1024 / 1024 temp_size
FROM dba_temp_files)
tempfiles_size_gib,
(SELECT SUM(blocks * block_size / 1024 / 1024 / 1024) size_gib
FROM v$archived_log
WHERE first_time >= SYSDATE - (
(SELECT value
FROM rdsadmin.rds_configuration
WHERE name =
'archivelog retention hours') /
24 ))
archlog_size_gib,
(SELECT SUM(block_size * file_size_blks) / 1024 / 1024 / 1024
controlfile_size
FROM v$controlfile)
ctlfiles_size_gib,
round(SUM(used.bytes) / 1024 / 1024 / 1024, 3)
db_size_gib,
round(SUM(used.bytes) / 1024 / 1024 / 1024, 3) - round(
free.f / 1024 / 1024 / 1024)
used_db_size_gib,
round(free.f / 1024 / 1024 / 1024, 3)
free_db_size_gib,
(SELECT round(SUM(filesize) / 1024 / 1024 / 1024, 3)
FROM TABLE(rdsadmin.rds_file_util.listdir('BDUMP')))
bdump_db_size_gib,
(SELECT round(SUM(filesize) / 1024 / 1024 / 1024, 3)
FROM TABLE(rdsadmin.rds_file_util.listdir('ADUMP')))
adump_db_size_gib,
(SELECT round(SUM(filesize) / 1024 / 1024 / 1024, 3)
FROM TABLE(rdsadmin.rds_file_util.listdir('DATA_PUMP_DIR')))
dpump_db_size_gib
FROM (SELECT bytes
FROM v$datafile
UNION ALL
SELECT bytes
FROM v$tempfile) used,
(SELECT SUM(bytes) AS f
FROM dba_free_space) free
GROUP BY free.f);
```

- - -

## 43. Export directly from AWS RDS Oracle using DBMS datapump

```sql
DECLARE
hdnl NUMBER;
BEGIN
hdnl := DBMS_DATAPUMP.OPEN( operation => 'EXPORT', job_mode => 'SCHEMA', job_name=>null);
DBMS_DATAPUMP.ADD_FILE( handle => hdnl, filename => 'LFR.dmp', directory => 'DATA_PUMP_DIR', filetype => dbms_datapump.ku$_file_type_dump_file);
DBMS_DATAPUMP.METADATA_FILTER(hdnl,'SCHEMA_EXPR','IN (''LFR'')');
DBMS_DATAPUMP.START_JOB(hdnl);
END;
```

```sql
select * from table(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) order by mtime;
exec utl_file.fremove('DATA_PUMP_DIR','DEFAULT_TEST_DB.dmp');
```

```sql
begin 
for i in (select filename from table(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) where type='file') 
loop UTL_FILE.FREMOVE ('DATA_PUMP_DIR', i.filename); 
end loop; 
end;
/
```

```sql
SELECT rdsadmin.rdsadmin_s3_tasks.upload_to_s3(
		  p_bucket_name    =>  't0107-preview-mv-1-us-east-1-bucket-mv-ce',
		  p_prefix         =>  'offer_2023_tables',
		  p_s3_prefix      =>  'offer_2022_2023_tables_archive/',
		  p_directory_name =>  'DATA_PUMP_DIR')
	   AS TASK_ID FROM DUAL;

SELECT owner_name, job_name, state from dba_datapump_jobs where state='EXECUTING';
```

```sql
SELECT 
  filename,
  type,
  ROUND(filesize / (1024 * 1024 * 1024 ), 2) AS size_gb,
  mtime
FROM TABLE(
  rdsadmin.rds_file_util.listdir(p_directory => 'DATA_PUMP_DIR')
)
WHERE type = 'file' and filename like '%.dmp'
ORDER BY filename ;
```

- - -

## 44. SGA and PGA usage

```sql
SELECT
   ROUND(SUM(CASE WHEN name != 'free memory' THEN bytes ELSE 0 END) / 1024 / 1024 / 1024, 2) AS used_sga_gb,
   ROUND(SUM(CASE WHEN name = 'free memory' THEN bytes ELSE 0 END) / 1024 / 1024 / 1024, 2) AS free_sga_gb,
   ROUND(SUM(bytes) / 1024 / 1024 / 1024, 2) AS total_sga_gb
   FROM V$SGASTAT;
```

```sql
SELECT
    ROUND(SUM(pga_used_mem) / 1024 / 1024 / 1024, 2) AS pga_used_gb,
    ROUND(SUM(pga_alloc_mem) / 1024 / 1024 / 1024, 2) AS pga_alloc_gb,
    ROUND(SUM(pga_max_mem) / 1024 / 1024 / 1024, 2) AS pga_max_gb
    FROM V$PROCESS;
```

```sql
SELECT VALUE AS ECPU_COUNT FROM   CLOUD$SERVICE_STATS WHERE  NAME = 'ECPU_COUNT';
SHOW PARAMETER SGA;
SHOW PARAMETER PGA;
SHOW PARAMETER MEMORY_TARGET;
```

```sql
SELECT name, ROUND(bytes / 1024 / 1024 / 1024, 2) AS size_gb
FROM V$SGASTAT
WHERE name IN ('buffer_cache', 'shared_pool', 'large_pool', 'java_pool', 'redo log buffer', 'fixed_sga', 'free memory')
ORDER BY bytes DESC;
```

- - -

## 45. Install SQLPLUS/IMPDP/EXPDP utility in a LINUX machine

```bash
mkdir -p ~/oracle_client
cd oracle_client

wget https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-basic-linux.x64-21.13.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-sqlplus-linux.x64-21.13.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/2113000/instantclient-tools-linux.x64-21.13.0.0.0dbru.zip

unzip instantclient-sqlplus-linux.x64-21.13.0.0.0dbru.zip
unzip instantclient-basic-linux.x64-21.13.0.0.0dbru.zip
unzip instantclient-tools-linux.x64-21.13.0.0.0dbru.zip

vi .bash_profile
export TNS_ADMIN=/home/ubuntu/client/wallet
export LD_LIBRARY_PATH=/home/ubuntu/oracle_client/instantclient_21_13:$LD_LIBRARY_PATH
export PATH=/home/ubuntu/oracle_client/instantclient_21_13:$PATH
```

- - -

## 46. Daily archivelog generation query

```sql
SELECT A.*, 
Round(A.Count#*B.AVG#/1024/1024/1024) Daily_Avg_gb 
FROM 
(SELECT 
To_Char(First_Time,'YYYY-MM-DD') DAY, 
Count(1) Count#, 
Min(RECID) Min#, 
Max(RECID) Max# 
FROM v$log_history 
GROUP 
BY To_Char(First_Time,'YYYY-MM-DD') 
ORDER 
BY 1 
) A, 
(SELECT 
Avg(BYTES) AVG#, 
Count(1) Count#, 
Max(BYTES) Max_Bytes, 
Min(BYTES) Min_Bytes 
FROM 
v$log ) B;
```

- - -

## 47. Find child table

```sql
SELECT 
    a.table_name       AS child_table,
    a.column_name      AS child_column,
    c.constraint_name  AS fk_constraint_name,
    c.owner            AS child_owner
FROM 
    all_constraints c
JOIN 
    all_cons_columns a ON a.constraint_name = c.constraint_name AND a.owner = c.owner
WHERE 
    c.constraint_type = 'R'
    AND c.r_constraint_name IN (
        SELECT constraint_name
        FROM all_constraints
        WHERE table_name = 'EC2_INSTANCE_TYPES'
          AND owner = 'LIXTO_LBOMI'
    );
```

- - -

## 48. Undo usage

Reference: https://smarttechways.com/2021/02/09/check-the-undo-tablespace-usage-in-oracle/

```sql
SELECT a.tablespace_name,
SIZEMB,
USAGEMB,
(SIZEMB - USAGEMB) FREEMB
FROM ( SELECT SUM (bytes) / 1024 / 1024 SIZEMB, b.tablespace_name
FROM dba_data_files a, dba_tablespaces b
WHERE a.tablespace_name = b.tablespace_name AND b.contents like 'UNDO'
GROUP BY b.tablespace_name) a,
( SELECT c.tablespace_name, SUM (bytes) / 1024 / 1024 USAGEMB
FROM DBA_UNDO_EXTENTS c
WHERE status <> 'EXPIRED'
GROUP BY c.tablespace_name) b
WHERE a.tablespace_name = b.tablespace_name;
```

```sql
select tablespace_name tablespace, status, sum(bytes)/1024/1024 sum_in_mb, count(*) counts
from dba_undo_extents
group by tablespace_name, status order by 1,2;
```

```sql
select u.tablespace_name tablespace, s.username, u.status, sum(u.bytes)/1024/1024 sum_in_mb, count(u.segment_name) seg_cnts
from dba_undo_extents u, v$transaction t , v$session s
where u.segment_name = '_SYSSMU' || t.xidusn || '$' and t.addr = s.taddr
group by u.tablespace_name, s.username, u.status order by 1,2,3;
```

- - -

## Additional queries

### Temp details

```sql
set pages 999
set lines 400
col TABLESPACE_NAME for  a10
col FILE_NAME format a75
select d.TABLESPACE_NAME, d.FILE_NAME, d.BYTES/1024/1024 SIZE_MB, d.AUTOEXTENSIBLE, d.MAXBYTES/1024/1024 MAXSIZE_MB, d.INCREMENT_BY*(v.BLOCK_SIZE/1024)/1024 INCREMENT_BY_MB
from dba_temp_files d,
 v$tempfile v
where d.FILE_ID = v.FILE#
order by d.TABLESPACE_NAME, d.FILE_NAME;
```

### Add tempfile

```sql
ALTER TABLESPACE TEMP3 ADD TEMPFILE '/opt/data/CE/datafile/temp3' SIZE 1G AUTOEXTEND ON NEXT 1G MAXSIZE 31G;
ALTER TABLESPACE TEMP3 ADD TEMPFILE '/opt/data/CE/datafile/temp3-7' SIZE 1G AUTOEXTEND ON NEXT 1G MAXSIZE 31G;
ALTER TABLESPACE TEMP3 ADD TEMPFILE '/mnt/temp3' SIZE 1G AUTOEXTEND ON NEXT 1G MAXSIZE 31G;
drop tablespace temp including contents and datafiles;
```

### Temp usage

```sql
set lines 200
select TABLESPACE_NAME, sum(BYTES_USED/1024/1024),sum(BYTES_FREE/1024/1024) 
from V$TEMP_SPACE_HEADER group by TABLESPACE_NAME;
```

### Table size (segment-based)

```sql
select sum(bytes)/1024/1024/1024 as gbytes from dba_segments where segment_name=upper('');
SELECT SUM(VSIZE(DOCUMENT)) FROM LIXTO_REPORTING.REPORTS;
select sum(dbms_lob.getlength(DOCUMENT))/1024/1024/1024 from LIXTO_REPORTING.REPORTS;
select sum(bytes)/1024/1024/1024 as gbytes from dba_segments where segment_name=upper('PRODUCT') and owner='MSO_DB';
```

### Query using temp

```sql
SELECT sysdate,a.username, a.sid, a.serial#, a.osuser, 
(b.blocks*d.block_size)/1048576 MB_used, c.sql_text
FROM v$session a, v$tempseg_usage b, v$sqlarea c,
     (select block_size from dba_tablespaces where tablespace_name='TEMP3') d
    WHERE b.tablespace = 'TEMP3'
    and a.saddr = b.session_addr
    AND c.address= a.sql_address
    AND c.hash_value = a.sql_hash_value
    AND (b.blocks*d.block_size)/1048576 > 1024
    ORDER BY b.tablespace, 6 desc;
```

### Kill multiple jobs from OS

```bash
ps aux | grep "/bin/bash /home/ubuntu/qa/automatic_qa/run_qa_procedure.sh QA_PTS_OFFERS" | grep -v grep | awk '{print $2}' | xargs kill
ps aux | grep "sqlplus                                                                            @/home/ubuntu/qa/automatic_qa/proc_automatic_qa.sql QA_PTS_OFFERS /home/ubuntu/qa/automatic_qa/log/automatic_qa_QA_PTS_OFFERS2025-06-13.log" | grep -v grep | awk '{print $2}' | xargs kill
```

### SOFAR for a SQL_ID

```sql
SELECT SQL_ID,
       SUM(SOFAR) AS TOTAL_SOFAR,
       SUM(TOTALWORK) AS TOTAL_WORK,
       ROUND(100 * SUM(SOFAR)/SUM(TOTALWORK), 2) AS PCT_DONE,
       MIN(START_TIME) AS START_TIME,
       MAX(LAST_UPDATE_TIME) AS LAST_UPDATE_TIME
FROM V$SESSION_LONGOPS
WHERE SOFAR > 0
  AND TOTALWORK > 0
  AND SQL_ID = '35jqxma1xunhb'
GROUP BY SQL_ID;
```

### Database backup

```bash
rman target /
backup as compressed backupset database;
delete backup;
```

### desc v$mystat

```sql
desc v$mystat;
select sid from V$mystat where rownum <2 ;
select sid, serial# , sql_id, program, terminal, action, module from v$session where audsid=sys_context('USERENV','SESSIONID');
```

### select with HashJoin hint

```sql
select /*+ HashJoin(a,b) */ count(*)
```

### LFR logging check cron

```bash
0 3 * * * sh /cloud_prod_scripts/cronsetup/prod_scripts/dump_remove.sh > /cloud_prod_scripts/cronsetup/FGA_AUTO/logs/dump_remove_'$(date +"\%Y\%m\%d-\%H\%M\%S")'.log 2>&1
```

### Statspack websites

* https://www.br8dba.com/statspack/
* https://www.akadia.com/services/ora_statspack_survival_guide.html
* https://dbaclass.com/
* https://oracle-base.com/dba/scripts

  ```
  select username from dba_users where oracle_maintained='N';
  ```
