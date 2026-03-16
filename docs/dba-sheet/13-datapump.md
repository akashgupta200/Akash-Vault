---
layout: default
title: "Data Pump"
parent: "DBA Reference Sheet"
nav_order: 13
---

# Data Pump

Reference guide for Data Pump export/import, monitoring, network copy, and related operations.

---

## Monitoring & Progress

### How long will expdp take?

```sql
SET LINES 750 PAGES 9999
COL job_name FOR A30
COL STATE FOR A10
COL sql_text FOR A100
COL message FOR A100
COL job_mode FOR A30

SELECT x.job_name, b.state, p.MESSAGE, p.totalwork, p.sofar,
       ROUND((p.sofar / p.totalwork) * 100, 2) done,
       p.time_remaining
FROM dba_datapump_jobs b
     LEFT JOIN dba_datapump_sessions x ON (x.job_name = b.job_name)
     LEFT JOIN v$session y ON (y.saddr = x.saddr)
     LEFT JOIN v$sql z ON (y.sql_id = z.sql_id)
     LEFT JOIN v$session_longops p ON (p.sql_id = y.sql_id)
WHERE y.module = 'Data Pump Worker'
  AND p.time_remaining > 0;
```

**Alternative:**

```sql
COL username FOR A20
COL opname FOR A50
COL message FOR A100
SET LINES 750 PAGES 9999

SELECT username, opname, target_desc, sofar, totalwork, message
FROM V$SESSION_LONGOPS
WHERE message NOT LIKE '%RMAN%'
  AND username = 'SYS';
```

**Alternative:**

```sql
SELECT b.username, a.sid, b.opname, b.target,
       ROUND(b.SOFAR*100/b.TOTALWORK,0) || '%' AS "%DONE",
       b.TIME_REMAINING,
       TO_CHAR(b.start_time,'YYYY/MM/DD HH24:MI:SS') start_time
FROM v$session_longops b, v$session a
WHERE a.sid = b.sid
ORDER BY 6;
```

### What expdp/impdp is actually executing

```sql
SELECT DISTINCT dp.job_name, dp.session_type, s.inst_id, s.SID, s.serial#,
       s.username, s.inst_id, s.event, s.sql_id, q.sql_text,
       dj.operation, dj.state
FROM gv$session s, dba_datapump_sessions dp, dba_datapump_jobs dj, gv$sql q
WHERE s.saddr = dp.saddr
  AND dp.job_name = dj.job_name
  AND s.sql_id = q.sql_id
  AND s.inst_id IN (1, 2, 3)
ORDER BY s.inst_id;
```

### Querying V$SESSION_LONGOPS & V$SESSION

```sql
SELECT b.username, a.sid, b.opname, b.target,
       ROUND(b.SOFAR*100/b.TOTALWORK,0) || '%' AS "%DONE",
       b.TIME_REMAINING,
       TO_CHAR(b.start_time,'YYYY/MM/DD HH24:MI:SS') start_time
FROM v$session_longops b, v$session a
WHERE a.sid = b.sid
ORDER BY 6;
```

### Querying V$SESSION_LONGOPS & V$DATAPUMP_JOB

```sql
SELECT sl.sid, sl.serial#, sl.sofar, sl.totalwork, dp.owner_name, dp.state, dp.job_mode
FROM v$session_longops sl, v$datapump_job dp
WHERE sl.opname = dp.job_name
  AND sl.sofar != sl.totalwork;
```

### Querying all related views with a single query

```sql
SELECT x.job_name, b.state, b.job_mode, b.degree,
       x.owner_name, z.sql_text, p.message,
       p.totalwork, p.sofar,
       ROUND((p.sofar/p.totalwork)*100,2) done,
       p.time_remaining
FROM dba_datapump_jobs b
LEFT JOIN dba_datapump_sessions x ON (x.job_name = b.job_name)
LEFT JOIN v$session y ON (y.saddr = x.saddr)
LEFT JOIN v$sql z ON (y.sql_id = z.sql_id)
LEFT JOIN v$session_longops p ON (p.sql_id = y.sql_id)
WHERE y.module = 'Data Pump Worker'
  AND p.time_remaining > 0;
```

### DBA resumable

```sql
SELECT * FROM dba_resumable;
```

---

## Export Operations

### Run expdp in background

```bash
nohup expdp parfile=/dbexports/EXP/dbname/schemaexpdp.par &
```

### Schema backup (parfile contents)

```text
userid='/ as sysdba'
dumpfile=WRNAME_exp_dbname_SCOTT.dmp
logfile=WRNAME_exp_w665pr_SCOTT.log
schemas=SCOTT
compression=all
directory=DATA_PUMP_DIR
```

### Estimate dumpfile size

```bash
expdp system/******* SCHEMAS=CMX_ORS NOLOGFILE=y ESTIMATE_ONLY=y DIRECTORY=tmp_pump_dir
```

### Scan table using exp without taking dumpfile

```bash
exp tables=<XXX> file=/dev/null log=/tmp/export.log statistics=none volsize=0
```

### Export table from previous time (legacy exp)

```bash
exp system/***** tables=schema.table_name file=exp_ix_str_store.dmp log=exp_ix_str_store.log query=\"where DBTIME<to_date('2014-05-27 00:01:08','yyyy-mm-dd hh24:mi:ss')\"
```

---

## Import & Network Copy

### Copy one table from one DB to another

```sql
SET arraysize 1000
COPY FROM username/passwd@source_connect_string TO username/passwd@target_connect_string
INSERT table_target USING SELECT * FROM table_source;
```

### Copy when table exists (insert with filter)

```sql
COPY FROM username1/passwd1@PROD TO username2/passwd2@SANDBOX
    INSERT TABLE_C (*) USING (SELECT * FROM TABLE_C WHERE COL_A = 4884);
```

**Alternative (with EZConnect):**

```sql
COPY FROM username1/passwd1@//192.168.3.17:1521/PROD_SERVICE TO username2/passwd2@//192.168.4.17:1521/SANDBOX_SERVICE
    INSERT TABLE_C (*) USING (SELECT * FROM TABLE_C WHERE COL_A = 4884);
```

---

## User Privileges for impdp

### Generate user DDL and privileges for impdp

```sql
SET LONG 20000 LONGCHUNKSIZE 20000 PAGESIZE 0 LINESIZE 1000 FEEDBACK OFF VERIFY OFF TRIMSPOOL ON
COLUMN ddl FORMAT A1000

BEGIN
   DBMS_METADATA.SET_TRANSFORM_PARAM (DBMS_METADATA.SESSION_TRANSFORM, 'SQLTERMINATOR', TRUE);
   DBMS_METADATA.SET_TRANSFORM_PARAM (DBMS_METADATA.SESSION_TRANSFORM, 'PRETTY', TRUE);
END;
/

VARIABLE v_username VARCHAR2(30);
EXEC :v_username := UPPER('&1');

SELECT DBMS_METADATA.GET_DDL('USER', u.username) AS ddl
FROM dba_users u
WHERE u.username = :v_username
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('TABLESPACE_QUOTA', tq.username) AS ddl
FROM dba_ts_quotas tq
WHERE tq.username = :v_username AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('ROLE_GRANT', rp.grantee) AS ddl
FROM dba_role_privs rp
WHERE rp.grantee = :v_username AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('SYSTEM_GRANT', sp.grantee) AS ddl
FROM dba_sys_privs sp
WHERE sp.grantee = :v_username AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('OBJECT_GRANT', tp.grantee) AS ddl
FROM dba_tab_privs tp
WHERE tp.grantee = :v_username AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('DEFAULT_ROLE', rp.grantee) AS ddl
FROM dba_role_privs rp
WHERE rp.grantee = :v_username AND rp.default_role = 'YES' AND rownum = 1
UNION ALL
SELECT TO_CLOB('/* Start profile creation script in case they are missing') AS ddl
FROM dba_users u
WHERE u.username = :v_username AND u.profile <> 'DEFAULT' AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_DDL('PROFILE', u.profile) AS ddl
FROM dba_users u
WHERE u.username = :v_username AND u.profile <> 'DEFAULT'
UNION ALL
SELECT TO_CLOB('End profile creation script */') AS ddl
FROM dba_users u
WHERE u.username = :v_username AND u.profile <> 'DEFAULT' AND rownum = 1
/

SET LINESIZE 80 PAGESIZE 14 FEEDBACK ON TRIMSPOOL ON VERIFY ON
```

---

## Troubleshooting

### ORA-02266: unique/primary keys referenced by enabled foreign keys

```sql
ALTER SESSION SET current_schema=CMFP;

SELECT 'ALTER TABLE '||a.owner||'.'||a.table_name||' DISABLE CONSTRAINT '||a.constraint_name||';'
FROM all_constraints a, all_constraints b
WHERE a.constraint_type = 'R'
  AND a.r_constraint_name = b.constraint_name
  AND a.r_owner = b.owner
  AND b.table_name = 'CMFP_HRCHY';
```

### ORA-27054: NFS file system not mounted with correct options

```sql
ALTER SYSTEM SET EVENTS '10298 trace name context forever, level 32';
```

**Alternative (persistent):**

```sql
ALTER SYSTEM SET event='10298 trace name context forever, level 32' SCOPE=spfile;
```

---

Back to [Main Index](README.md)
