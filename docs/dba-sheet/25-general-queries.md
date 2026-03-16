---
layout: default
title: "General DBA Queries Reference"
parent: "DBA Reference Sheet"
nav_order: 25
---

# General DBA Queries Reference

Hidden parameters, internal tables, X$ views, and miscellaneous Oracle DBA queries.

---

## X$ Views and Hidden Parameters

### List Hidden Parameters in Oracle Database

```sql
set lines 750 pages 9999
column KSPPINM format a50
column KSPPSTVL format a50
select a.ksppinm, b.ksppstvl  FROM x$ksppi a, x$ksppcv b  WHERE a.indx=b.indx;
```

**Alternative:**

```sql
select name, value from sys.V$PARAMETER where name like '\_%' escape '\'  and ISDEFAULT='FALSE';
```

### Undocumented Init Parameters

```sql
SELECT * FROM   SYS.X$KSPPI WHERE  SUBSTR(KSPPINM,1,1) = '_';
```

### Hidden Parameter Default Value

```sql
select ksppstvl from x$ksppi join x$ksppcv using (indx) where ksppinm='_high_priority_processes';
```

**Alternative:**

```sql
SELECT
a.ksppinm Param ,
b.ksppstvl SessionVal ,
c.ksppstvl InstanceVal,
a.ksppdesc Descr
FROM
x$ksppi a ,
x$ksppcv b ,
x$ksppsv c
WHERE
a.indx = b.indx AND
a.indx = c.indx AND
a.ksppinm LIKE '/_adg%' escape '/'
ORDER BY
1
/
```

**Alternative:** Create view for all parameters:

```sql
create or replace view all_parameters
as
select x.ksppinm name, y.ksppstvl value
from x$ksppi x , x$ksppcv y
where x.indx = y.indx
order by x.ksppinm
/
grant select on all_parameters to public;
create public synonym all_parameters for all_parameters;
```

### Unset Hidden Parameter

```sql
ALTER SYSTEM RESET "_some_hidden_parameter" scope = spfile;
```

---

## Internal Tables

### Find All Internal Tables (e.g., user$)

```sql
select object_name from dba_objects where object_name like '%$' and object_name not like 'SYS%'
```

### Find All X$ Views

```sql
select distinct table_name from V$INDEXED_FIXED_COLUMN where table_name like 'X$%';
```

---

## Database Information

### Info.sql — Database Summary

```sql
set long 20000 longchunksize 20000 pagesize 9999 linesize 1000
col PDB_NAME for a15
col HOST_NAME for a35
select db_unique_name CDB_NAME,(select name from v$pdbs where name not like '%SEED') PDB_NAME,open_mode,database_role,(SELECT to_char(sysdate,'DD-MON-YYYY HH24:MI:SS') FROM dual) "Current_time_db" ,(select INSTANCE_NAME from v$instance) INSTANCE_NAME,(select HOST_NAME from v$instance ) HOST_NAME from v$database;
select INST_ID,INSTANCE_NAME,HOST_NAME,status,logins,VERSION from gv$instance order by 1;
select con_id,NAME,OPEN_MODE,RESTRICTED,OPEN_TIME from gv$pdbs order by 1;
col WRL_PARAMETER for a30
select INST_ID,STATUS,WALLET_TYPE,WRL_TYPE,WRL_PARAMETER,WALLET_ORDER  from  gv$encryption_wallet;
```

### SP Parameter Difference Between RAC Instances

```sql
SELECT p1.name, p1.value, p2.value FROM gv$parameter p1
  JOIN gv$parameter p2 ON p1.name = p2.name
  WHERE p1.inst_id = 1
    AND p2.inst_id = 2
    AND p1.value != p2.value
    AND p1.name NOT IN ('instance_number', 'instance_name', 'local_listener');
```

### Database Startup Time

```sql
SELECT inst_id,to_char(startup_time,'DD-MON-YYYY HH24:MI:SS') "DB Startup Time" ,host_name FROM   sys.gv_$instance order by 1;
```

### Database Current Time

```sql
SELECT to_char(sysdate,'DD-MON-YYYY HH24:MI:SS') "sysdate" FROM   dual;
```

### Finding DBNAME as Normal User

```sql
select ora_database_name from dual
```

**Alternative:**

```sql
select global_name from global_name;
```

### Oracle Enterprise Edition Check

```sql
select * from product_component_version;
```

### Show Current User

```sql
select sys_context( 'userenv', 'current_schema' ) from dual;
```

**Alternative:**

```sql
SELECT sys_context('USERENV', 'HOST') FROM dual;
SELECT sys_context('USERENV', 'INSTANCE') FROM dual;
```

Reference: [sys_context (morganslibrary.org)](https://www.morganslibrary.org/reference/sys_context.html)

### Current Session SID and SPID

```sql
select
        s.sid, p.spid, substr(s.username,1,20) username, s.terminal, p.Program
  from
       v$session s, v$process p
 where
       s.paddr = p.addr
   and
       s.sid = (select sid from v$mystat where rownum=1)
;
```

### Service Name

```sql
select value from v$parameter where name='service_names';
```

### Database Character Set

```sql
SELECT value$ FROM sys.props$ WHERE name = 'NLS_CHARACTERSET' ;
```

**Alternative:**

```sql
select * from database_properties where property_name like '%CHARACTERSET';
SELECT * FROM NLS_DATABASE_PARAMETERS;
```

### Database 32-bit or 64-bit

```sql
select
   length(addr)*4 || '-bits' word_length
from
   v$process
where
   ROWNUM =1;
```

**Alternative (OS):**

- Solaris: `/usr/bin/isainfo -kv`
- Linux: `uname -m`

**Alternative (Oracle binary):**

```bash
cd $ORACLE_HOME/bin
file oracl*
```

### Time Zone of Database

```sql
SELECT EXTRACT(TIMEZONE_HOUR FROM SYSTIMESTAMP)||':'||
       EXTRACT(TIMEZONE_MINUTE FROM SYSTIMESTAMP)
FROM dual;
```

**Alternative:**

```sql
select current_timestamp from dual;
select dbtimezone from dual;
```

---

## Alert Log

### Reading Alert Log

```sql
select message_text from X$DBGALERTEXT where rownum <= 20;
```

### Create Alert Log Table (External Table)

```sql
create directory BDUMP as '/u01/app/oracle/admin/mysid/bdump';

create table
   alert_log ( msg varchar2(80) )
organization external (
   type oracle_loader
   default directory BDUMP
   access parameters (
      records delimited by newline
   )
   location('alrt_mysid.log')
)
reject limit 1000;
```

### Query Alert Log for Startup

```sql
col ORIGINATING_TIMESTAMP for a40
col MESSAGE_TEXT for a80
set linesize 500
SELECT
originating_timestamp,
message_text
FROM
sys.x$dbgalertext
WHERE
message_text LIKE '%Starting up%';
```

### Query Alert Log for Shutdown

```sql
SELECT
originating_timestamp,
message_text
FROM
sys.x$dbgalertext
WHERE
message_text LIKE '%Instance shutdown complete%';
```

### Write Own Message to Alert Log

```sql
begin
  sys.dbms_system.ksdwrt(2, 'My own message');
end;
/
```

---

## Miscellaneous Queries

### Row Limiting Clause (FETCH FIRST)

```sql
SELECT val
FROM   rownum_order_test
ORDER BY val DESC
FETCH FIRST 5 ROWS ONLY;
```

### DNS Lookup in Oracle

```sql
SELECT utl_inaddr.get_host_name('68.180.206.184') from dual;
```

### Generate Random Password

```sql
select DBMS_RANDOM.string('x',10) PASSWD from dual
```

### Tablespace List (Internal)

```sql
select name from ts$;
```

### General Database Comparison (Session Diff via DB Link)

```sql
SELECT t1.* FROM v$session t1 WHERE NOT EXISTS
(SELECT 1 FROM V$session@dblink_prod t2 WHERE t1.sid = t2.sid and t1.serial#=t2.serial# and t1.AUTH_TYPE_ID=t2.AUTH_TYPE_ID
and t1.APP_VERSION=t2.APP_VERSION and t1.CREATE_TIME=t2.CREATE_TIME )
```

### Case Insensitive Search in SQL

```sql
alter session set nls_comp=linguistic;
alter session set nls_sort=BINARY_CI;
select distinct metric_name from DBA_HIST_SYSMETRIC_SUMMARY where metric_name like '%memory%';
```

**Alternative:** Use exact case in LIKE:

```sql
select distinct metric_name from DBA_HIST_SYSMETRIC_SUMMARY where metric_name like '%Memory%';
```

### Case Insensitive Search (REGEXP_LIKE)

```sql
SELECT * FROM TABLE WHERE REGEXP_LIKE (TABLE.NAME,'IgNoReCaSe','i');
```

### Active Services Running

```sql
col NAME for a20
col SERVICENAME_AVAIL for a20
select
                    b.name , a.inst_id Inst_id_avail , a.name servicename_avail
from
                    gv$active_services a , dba_services b
where
                    b.name = a.name(+) and
                    (a.name not like '%XDB%' AND  a.NAME NOT LIKE '%SYS$%' and a.name not like '%DGB%') and
                    (b.name not like '%XDB%' AND  b.NAME NOT LIKE '%SYS$%' and b.name not like '%DGB%')
order by 1,2
/
```

### Spool with PDB Name (Proper Output)

```sql
column dbname new_value dbname print
select name dbname from v$pdbs;
column timendate new_value spooltime print
select SYS_CONTEXT('USERENV', 'DB_UNIQUE_NAME')||'_'||SYS_CONTEXT('USERENV', 'SESSION_USER')||'_'||to_char(sysdate,'dd-mon-yyyy-hh24-mi-ss') timeNdate from dual;
spool &dbname-&spooltime..log
```

**Alternative (Compact):**

```sql
column dbname new_value dbname noprint
select name dbname from v$pdbs;
column timendate new_value spooltime noprint
select SYS_CONTEXT('USERENV', 'DB_UNIQUE_NAME')||'_'||SYS_CONTEXT('USERENV', 'SESSION_USER')||'_'||to_char(sysdate,'dd-mon-yyyy-hh24-mi-ss') timeNdate from dual;
spool &dbname-&spooltime..log
```

**Alternative (HTML Export):**

```sql
column dbname new_value dbname noprint
select name dbname from v$pdbs;
SET MARKUP HTML ON
spool C:\Oracle\login-db\folder\&dbname..xls
select * from all_db_links;
spool off
SET MARKUP HTML OFF
```

### Column Separator

```sql
SET COLSEP '|'
```

### Put Lines Inside Oracle Output

```sql
set serveroutput on
set heading off
set feedback off
select sysdate from dual;
exec dbms_output.put_line('------------------------------');
select sysdate from dual;
```

---

## Recovery and Recovery Area

### Flash Recovery Area Full — Change Archive Log Location

```sql
SELECT * FROM V$RECOVERY_FILE_DEST;
```

```sql
alter system set log_archive_dest_1='location=/ora10gsoft/archsam reopen';
alter system switch logfile;
```

### ORA-01139: RESETLOGS Option Only Valid After Incomplete Recovery

```sql
RECOVER DATABASE UNTIL CANCEL
alter database open resetlogs
```

Used when all redo logs are lost.

---

## Recovery Scenarios

### Redo Log File Deleted (Single Member Group)

```sql
alter system checkpoint;
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP 1;
alter database drop logfile group 1;
```

### ORA-01565: Datafile Lost — Recover via /proc (Linux)

If the datafile is deleted but Oracle still has an open file descriptor:

1. Find DB writer PID: `ps -edf | grep dbw`
2. Check opened file descriptors: `ls -l /proc/<pid>/fd | grep <filename>`
3. Create symbolic link: `ln -s /proc/<pid>/fd/<fd_num> /path/to/original/file.dbf`

---

## Undo and Corruption

### ORA-00600 / ORA-08007 — Undo Segment Needs Recovery

```sql
select
   segment_name,
   status
from
   dba_rollback_segs
where
   tablespace_name='undotbs_corrupt'
and
   status = 'NEEDS RECOVERY';
```

**Steps:**
1. Create new undo tablespace: `create undo tablespace undotbs2 datafile '/path/undotbs2.dbf' size 500m;`
2. Switch: `alter system set undo_tablespace=undotbs2 scope=both;`
3. Offline old: `alter tablespace undotbs1 offline;`
4. Drop corrupted segment: `drop rollback segment "_SYSSMU22$";`
5. Drop old tablespace: `alter tablespace undotbs1 offline;` then drop

---

## Database Control

### Restrict Database Logins (Single User Mode)

```sql
startup restrict;
alter system disable restricted session;
alter system enable restricted session;
```

### Quiescing Oracle Database

```sql
SQL> select active_state from v$instance;
-- ACTIVE_STATE: NORMAL

SQL> ALTER SYSTEM QUIESCE RESTRICTED;
SQL> select active_state from v$instance;
-- ACTIVE_STATE: QUIESCED

SQL> select bq.sid, username, osuser, program, machine from v$blocking_quiesce bq, v$session s where bq.sid = s.sid;

SQL> ALTER SYSTEM UNQUIESCE;
```

### Freeze Database (Suspend I/O)

```sql
alter system suspend;
select database_status from v$instance;
-- Database_status: Suspended

alter system resume;
select database_status from v$instance;
-- Database_status: Active
```

### Create AWR Snapshot

```sql
EXEC DBMS_WORKLOAD_REPOSITORY.create_snapshot;
```

---

## DBV (Database Verify)

### DBV — Verify Datafile

```bash
dbv file=/usr/acct/dba/dbs/dbf/data1/smp_data.dbf blocksize=4096 feedback=100
```

### DBV — Verify Specific Segment

```sql
select t.ts#, s.header_file, s.header_block
     from v$tablespace t, dba_segments s
     where s.owner = 'SCOTT'
     and s.segment_name='DEPARTMENT'
     and t.name = s.tablespace_name;
```

```bash
dbv userid=scott/tiger segment_id=15.12.9
```

---

## Service Statistics

### Service Previously Running or Not

```sql
BREAK ON SNAP_ID
select snap_id,instance_number,SERVICE_NAME,value/100000 as DBTIME from DBA_HIST_SERVICE_STAT where STAT_NAME='DB time' order by snap_id desc,instance_number FETCH FIRST 50 ROWS ONLY;
```

---

## External Tables

### External Table for Excel/CSV

```sql
CREATE OR REPLACE DIRECTORY costtoserve_dir AS '/work/oracle/costtoserve';
grant read,write on directory costtoserve_dir to costtoserve;
CREATE TABLE costtoserve.costtoserve
 (
     parent        VARCHAR2(10),
     host_to     VARCHAR2(30),
     child     VARCHAR2(30)
)
ORGANIZATION EXTERNAL
   (
         TYPE oracle_loader
         DEFAULT DIRECTORY costtoserve_dir
         ACCESS PARAMETERS
         (
               RECORDS DELIMITED BY NEWLINE
               badfile costtoserve_dir:'upload_costtoserv_file.bad'
               logfile costtoserve_dir:'upload_costtoserv_file.log'
               FIELDS TERMINATED BY ','
               MISSING FIELD VALUES ARE NULL
               (
                      parent,
                      host_to,
                      child
       ))
             LOCATION ('hosttoserver.csv')
)REJECT LIMIT UNLIMITED;
```

---

## Database Links

### Create and Drop DB Link as Another User

```sql
-- Connect as SYS
CREATE or replace PROCEDURE scott.create_db_link AS
BEGIN
EXECUTE IMMEDIATE 'create database link LINK1 connect to scott identified by tiger using ''testdb''';
END create_db_link;
/

exec scott.create_db_link

-- To drop (must be done by owner or via procedure):
CREATE PROCEDURE scott.drop_db_link AS
BEGIN
EXECUTE IMMEDIATE 'drop database link LINK1';
END drop_db_link;
/

exec scott.drop_db_link
```

---

## Triggers and Security

### Block Development Tools from Production

```sql
CREATE OR REPLACE TRIGGER block_tools_from_prod
AFTER LOGON ON DATABASE
DECLARE
v_program sys.v_$session.program%TYPE;
BEGIN
SELECT program INTO v_program
FROM sys.v_$session
WHERE audsid = USERENV('SESSIONID')
AND audsid != 0
AND ROWNUM = 1;

IF UPPER(v_program ) LIKE '%TOAD%' OR UPPER(v_program) LIKE '%T.O.A.D%' OR
UPPER(v_program ) LIKE '%SQLNAV%' OR
UPPER(v_program ) LIKE '%PLSQLDEV%' OR
UPPER(v_program ) LIKE '%BUSOBJ%' OR
UPPER(v_program) LIKE '%EXCEL%'
THEN
RAISE_APPLICATION_ERROR(-20000, 'Development tools are not allowed here.');
END IF;
END;
/
```

### Connect to Other User Without Password (Proxy User)

```sql
-- Connect as SYSTEM
alter user APEX_030200 grant connect through system;
alter user apex_030200 account unlock;

-- Connect as: system[apex_030200]/systempassword
-- Result: SELECT user FROM dual; returns APEX_030200
```

**Revoke:**

```sql
ALTER USER apex_030200 REVOKE CONNECT THROUGH system;
```

---

## Oracle Home and Options

### Get Oracle Home from SQL*Plus

```sql
CREATE OR REPLACE FUNCTION get_java_system_property (prop IN VARCHAR2) RETURN VARCHAR2 IS LANGUAGE JAVA
name 'java.lang.System.getProperty(java.lang.String) return java.lang.String';
/

CREATE OR REPLACE VIEW v$oracle_home AS
SELECT get_java_system_property('user.dir') AS oracle_home FROM dual;

SELECT * FROM v$oracle_home;

drop function get_java_system_property;
drop view v$oracle_home;
```

### Check Database Options (libknlopt.a)

```bash
cd $ORACLE_HOME/rdbms/lib
ar -tv libknlopt.a | grep -c kkxwtp.o
ar -tv libknlopt.a | grep -c kfoff.o
ar -tv libknlopt.a | grep -c ktd.o
ar -tv libknlopt.a | grep -c kxmwsd.o
ar -tv libknlopt.a | grep -c kciwcx.o
ar -tv libknlopt.a | grep -c sllfls.o
ar -tv libknlopt.a | grep -c kprnts.o
ar -tv libknlopt.a | grep -c xsnoolap.o
ar -tv libknlopt.a | grep -c kdzof.o
ar -tv libknlopt.a | grep -c kecnr.o
ar -tv libknlopt.a | grep -c dmndm.o
ar -tv libknlopt.a | grep -c kkpoban.o
ar -tv libknlopt.a | grep -c kcsm.o
ar -tv libknlopt.a | grep -c jox.o
ar -tv libknlopt.a | grep -c kzlilbac.o
ar -tv libknlopt.a | grep -c kzvidv.o
```

**General:** `ar -tv libknlopt.a`

### Product/Component Short Names (libknlopt.a)

| Product/Component | Short Name | Object |
|-------------------|------------|--------|
| Automated Storage Management | ASM | kfon.o |
| Oracle Data Mining | DM | dmwdm.o |
| Database Vault | DV | kzvidv.o |
| Oracle OLAP | OLAP | xsyeolap.o |
| Oracle Label Security | OLS | kzlilbac.o |
| Oracle Partitioning | PART | kkpoban.o |
| Real Application Cluster | RAC | kcsm.o |
| Real Application Testing | RAT | kecwr.o |

### chopt Options (11g)

| Option | Database Option | ON | OFF |
|--------|-----------------|-----|-----|
| dm | Data Mining | dm_on | dm_off |
| dmse | Data Mining Scoring Engine | dmse_on | dmse_off |
| dv | Database Vault | dv_on | dv_off |
| olap | OLAP | olap_on | olap_off |
| lbac | Label Security | lbac_on | lbac_off |
| partitioning | Partitioning | part_on | part_off |
| rat | Real Application Clusters | rac_on | rac_off |
| sdo | Spatial | sdo_on | sdo_off |
| rat | Real Application Testing | rat_on | rat_off |
| olap | OLAP | olap_on | olap_off |
| asm | Automatic Storage Management | asm_on | asm_off |
| ctx | Context Management Text | ctx_on | ctx_off |

### Enabling/Disabling Options

**11g:** `chopt enable partitioning` / `chopt disable partitioning`

**10g:** `make -f ins_rdbms.mk option_switch ioracle` then `cd $ORACLE_HOME/rdbms/lib` and `make -f ins_rdbms.mk part_off ioracle`

---

## Troubleshooting

### ORA-28547: Connection to Server Failed — Oracle Net Admin Error

Comment out the PROGRAM line from `listener.ora` SID_LIST section, then restart the listener.

### ORA-0750 / SP2-0750: Message file sp1<lang>.msb not found

```bash
cd $ORACLE_HOME/install
./changePerm.sh
```

Reference: [ChangePerm.sh (orafaq.com)](http://www.orafaq.com/wiki/ChangePerm_sh)

### DBMS Insufficient Privileges

```sql
GRANT CREATE JOB, MANAGE SCHEDULER, MANAGE ANY QUEUE TO USER1;
```

---

## Other Utilities

### IPv6 Connectivity

```bash
connect arup/arup@[fe80::219:21ff:febb:9aa5]/D112D1
```

JDBC: `jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=[fe80::219:21ff:febb:9aa5]) (PORT=1526))(CONNECT_DATA=(SERVICE_NAME=D112D1)))`

### Create Big Table for Testing

```sql
create table bigtab
as
select rownum id, a.*
  from all_objects a
 where 1=0;
alter table bigtab nologging;

declare
    l_cnt  number;
    l_rows number := 1000000;
begin
    insert /*+ append */
    into bigtab
    select rownum, a.*
      from all_objects a;
    l_cnt := sql%rowcount;
    commit;

    while (l_cnt < l_rows)
    loop
        insert /*+ APPEND */ into bigtab
        select rownum+l_cnt,
               OWNER, OBJECT_NAME, SUBOBJECT_NAME,
               OBJECT_ID, DATA_OBJECT_ID,
               OBJECT_TYPE, CREATED, LAST_DDL_TIME,
               TIMESTAMP, STATUS, TEMPORARY,
               GENERATED, SECONDARY
          from bigtab
         where rownum <= l_rows-l_cnt;
        l_cnt := l_cnt + sql%rowcount;
        commit;
    end loop;
end;
/

alter table bigtab add constraint bigtab_pk primary key(id);
```

### License Usage Report

```sql
select
   samp.dbid,
   fu.name,
   samp.version,
   detected_usages,
   total_samples,
    decode(to_char(last_usage_date, 'MM/DD/YYYY, HH:MI:SS'),
         NULL, 'FALSE',
         to_char(last_sample_date, 'MM/DD/YYYY, HH:MI:SS'), 'TRUE',
         'FALSE')
    currently_used,
   first_usage_date,
   last_usage_date,
   aux_count,
    feature_info,
   last_sample_date,
   last_sample_period,
   sample_interval,
   mt.description
 from
   wri$_dbu_usage_sample     samp,
   wri$_dbu_feature_usage    fu,
   wri$_dbu_feature_metadata mt
 where
  samp.dbid    = fu.dbid and
  samp.version = fu.version and
  fu.name      = mt.name and
  fu.name not like '_DBFUS_TEST%' and
  bitand(mt.usg_det_method, 4) != 4;
```

### Hiding User IDs

```sql
update sys.user$ set name='NEW' where user#=N and name='OLD';
```

You can connect as user OLD, but queries will show NEW. You cannot connect as NEW.

### Magical Faster Secret Query (Hint)

```sql
select /*+ richs_secret_hint */ ename, job
```

### Get SQL Prompt with User and Database Name

```sql
set termout off
define gname=idle
column global_name new_value gname
select lower(user)||'@' ||substr(global_name,1,decode(dot,0,length(global_name),dot-1)) global_name from (select global_name, instr(global_name,'.') dot from global_name);
set sqlprompt '&gname>'
set termout on
```

---

## References

- [Dissassembling the data block (orafaq.com)](http://www.orafaq.com/papers/dissassembling_the_data_block.pdf)
- [Database home options check](http://m.blog.itpub.net/17252115/viewspace-1160554/)
- MOSC Note: 948061.1
- [External tables (oracle.com)](http://www.oracle.com/technetwork/articles/saternos-tables-090560.html)
- [Create/drop DB link from another user](http://dbaoracletips.blogspot.com/2011/11/how-to-dropcreate-database-link-from.html)
- [SYS/SYSTEM and ORA-01031 (gokhanatil.com)](http://www.gokhanatil.com/2011/02/syssystem-users-and-ora-01031-prior-to-oracle-9-2.html)

---

[Back to Main Index](README.md)
