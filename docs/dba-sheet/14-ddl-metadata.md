---
layout: default
title: "DDL & Metadata"
parent: "DBA Reference Sheet"
nav_order: 14
---

# DDL & Metadata

Reference guide for DDL extraction, DBMS_METADATA, users, roles, and privileges.

---

## General DDL

### DDL statement (generic)

```sql
SET LONG 20000 LONGCHUNKSIZE 20000 PAGESIZE 0 LINESIZE 1000
SELECT DBMS_METADATA.GET_DDL('&OBJECT_TYPE','&OBJECT_NAME','&OWNER') AAA FROM DUAL;
```

### Last DDL date

```sql
SELECT TO_CHAR(last_ddl_time,'DD-MON-YYYY HH24:MI:SS')
FROM dba_objects
WHERE owner = '&user'
  AND object_name = '&table_name';
```

### Package DDL

```sql
SET PAGESIZE 0 ECHO OFF TIMING OFF LINESIZE 1000 TRIMSPOOL ON TRIM ON
SET LONG 2000000 LONGCHUNKSIZE 2000000

SELECT DBMS_METADATA.GET_DDL('PACKAGE_SPEC','&OBJECT_NAME','&OWNER') AAA FROM DUAL;
```

### Parallel DDL option

```sql
ALTER SESSION ENABLE PARALLEL DDL;
```

---

## Table DDL

### Generating DDL of table

```sql
SET LONG 1000
SELECT DBMS_METADATA.GET_DDL('TABLE','EMP','SCHEMA')||'/' FROM dual;
```

**Alternative:**

```sql
SET LINES 1000 LONG 2000 PAGES 9999
SELECT DBMS_METADATA.GET_DDL('TABLE', table_name)
FROM user_tables
WHERE table_name IN (...);
```

### Create scripts of all tables in a schema

```sql
SELECT 'SELECT DBMS_METADATA.GET_DDL(''TABLE'', '''||TABLE_NAME||''',''<schema>'') FROM dual;'
FROM DBA_TABLES;
```

### GET_DEPENDENT_DDL (ref constraints)

```sql
SELECT DBMS_METADATA.GET_DEPENDENT_DDL('REF_CONSTRAINT','<table_name>','<schema>')
FROM dual;
```

---

## Database Links & Sequences

### Generate DDL of DB_LINK

```sql
SET LONG 1000
SELECT DBMS_METADATA.GET_DDL('DB_LINK', db.db_link, db.owner)
FROM dba_db_links db;
```

**Alternative:**

```sql
SELECT 'CREATE '||DECODE(U.NAME,'PUBLIC','public ')||'database link '||CHR(10)
  ||DECODE(U.NAME,'PUBLIC',NULL, U.NAME||'.')|| L.NAME||CHR(10)
  ||'connect to ' || L.USERID || ' identified by '||L.PASSWORD||' using ''' || L.host || ''''
  ||CHR(10)||';' TEXT
FROM sys.link$ L, sys.user$ U
WHERE L.OWNER# = U.USER#;
```

### Sequence DDL

```sql
SELECT 'CREATE SEQUENCE '||SEQUENCE_NAME||CHR(10)||
       ' INCREMENT BY '||INCREMENT_BY||CHR(10)||
       ' START WITH '||LAST_NUMBER||CHR(10)||
       ' MINVALUE '||MIN_VALUE||CHR(10)||
       ' MAXVALUE '||MAX_VALUE||CHR(10)||
       DECODE(CYCLE_FLAG,'N',' NOCYCLE','CYCLE')||CHR(10)||
       DECODE(ORDER_FLAG,'N',' NOORDER','ORDER')||CHR(10)||
       ' CACHE '||CACHE_SIZE|| ';'
FROM DBA_SEQUENCES
WHERE SEQUENCE_OWNER = '&OWNER_NAME';
```

---

## Tablespace DDL

### Tablespace DDL

```sql
SELECT 'SELECT DBMS_METADATA.GET_DDL(''TABLESPACE'','''|| tablespace_name || ''') FROM dual;'
FROM dba_tablespaces;

SELECT DBMS_METADATA.GET_DDL('TABLESPACE','&TABLESPACE_NAME') FROM dual;
```

---

## Users DDL

### Users DDL (DBMS_METADATA)

```sql
SET LONG 2000
SELECT (CASE
  WHEN ((SELECT COUNT(*) FROM dba_users WHERE username = 'ODB') > 0)
  THEN DBMS_METADATA.GET_DDL('USER', 'ODB')
  ELSE TO_CLOB('   -- Note: User not found!')
  END) Extracted_DDL FROM dual
UNION ALL
SELECT (CASE
  WHEN ((SELECT COUNT(*) FROM dba_ts_quotas WHERE username = 'ODB') > 0)
  THEN DBMS_METADATA.GET_GRANTED_DDL('TABLESPACE_QUOTA', 'ODB')
  ELSE TO_CLOB('   -- Note: No TS Quotas found!')
  END) FROM dual
UNION ALL
SELECT (CASE
  WHEN ((SELECT COUNT(*) FROM dba_role_privs WHERE grantee = 'ODB') > 0)
  THEN DBMS_METADATA.GET_GRANTED_DDL('ROLE_GRANT', 'ODB')
  ELSE TO_CLOB('   -- Note: No granted Roles found!')
  END) FROM dual
UNION ALL
SELECT (CASE
  WHEN ((SELECT COUNT(*) FROM dba_sys_privs WHERE grantee = 'ODB') > 0)
  THEN DBMS_METADATA.GET_GRANTED_DDL('SYSTEM_GRANT', 'ODB')
  ELSE TO_CLOB('   -- Note: No System Privileges found!')
  END) FROM dual
UNION ALL
SELECT (CASE
  WHEN ((SELECT COUNT(*) FROM dba_tab_privs WHERE grantee = 'ODB') > 0)
  THEN DBMS_METADATA.GET_GRANTED_DDL('OBJECT_GRANT', 'ODB')
  ELSE TO_CLOB('   -- Note: No Object Privileges found!')
  END) FROM dual
/
```

### Roles granted to user, system privs, object privs

```sql
SELECT DBMS_METADATA.GET_DDL('USER', '&USER') || '/' usercreate FROM dual;
SELECT DBMS_METADATA.GET_GRANTED_DDL('ROLE_GRANT','&USER') || '/' FROM DUAL;
SELECT DBMS_METADATA.GET_GRANTED_DDL('SYSTEM_GRANT','&USER') || '/' FROM DUAL;
SELECT DBMS_METADATA.GET_GRANTED_DDL('OBJECT_GRANT','&USER') || '/' FROM DUAL;
SELECT DBMS_METADATA.GET_GRANTED_DDL('TABLESPACE_QUOTA', '&USER') || '/' FROM dual;
```

### DDL of all users (except schema owners)

```sql
SET LONG 20000 LONGCHUNKSIZE 20000 PAGESIZE 0 LINESIZE 1000
SELECT DBMS_METADATA.GET_DDL('USER', username)
FROM dba_users
WHERE username NOT IN (SELECT owner FROM dba_objects)
  AND (username LIKE 'U%' OR username LIKE 'A%');
```

---

## Single Object Grants

### A single object's grants (any type)

```sql
SELECT 'GRANT ' || PRIVILEGE || ' ON ' || OWNER || '.' || TABLE_NAME || ' TO ' || GRANTEE || ';'
FROM dba_tab_privs
WHERE table_name = '&Any_type_of_object_name';
```

---

## Password & Profile

### Password DDL

```sql
SELECT 'ALTER USER ' || NAME ||' IDENTIFIED BY VALUES ''' || password || ''';'
FROM user$;
```

### Cannot reuse password (bypass verification)

```sql
DECLARE
  userNm VARCHAR2(100);
  userpswd VARCHAR2(100);
BEGIN
  userNm := UPPER('&TypeUserNameHere');
  SELECT password INTO userpswd FROM sys.user$ WHERE name = userNm;
  EXECUTE IMMEDIATE 'ALTER PROFILE "FUNCTIONAL_USER" LIMIT PASSWORD_VERIFY_FUNCTION null PASSWORD_LIFE_TIME UNLIMITED PASSWORD_REUSE_TIME UNLIMITED PASSWORD_REUSE_MAX UNLIMITED';
  EXECUTE IMMEDIATE 'ALTER USER '||userNm||' IDENTIFIED BY oct152014oct';
  EXECUTE IMMEDIATE 'ALTER USER '||userNm||' IDENTIFIED BY VALUES '''||userpswd||'''';
  EXECUTE IMMEDIATE 'ALTER PROFILE "FUNCTIONAL_USER" LIMIT PASSWORD_VERIFY_FUNCTION PASSWDCOMPLEXVERIFICATION';
END;
/
```

### ORA-28003: password verification failed

```sql
ALTER PROFILE FUNCTIONAL_USER LIMIT PASSWORD_VERIFY_FUNCTION NULL;
ALTER USER trial IDENTIFIED BY test;
-- conn trial/test;
ALTER PROFILE "FUNCTIONAL_USER" LIMIT PASSWORD_VERIFY_FUNCTION PASSWDCOMPLEXVERIFICATION;
```

### Profile assignment

```sql
SELECT 'ALTER USER '||username||' PROFILE '||PROFILE||';'
FROM dba_users;
```

### When was the password changed for a user

```sql
SELECT NAME, TO_CHAR(ptime, 'DD-MON-YYYY HH24:MI:SS') AS "LAST TIME CHANGED",
       ctime "CREATION TIME", ltime "LOCKED"
FROM USER$
WHERE ptime IS NOT NULL
ORDER BY ptime DESC;
```

---

## Roles DDL

### Roles DDL (DBMS_METADATA)

```sql
SET LONG 20000 LONGCHUNKSIZE 20000 PAGESIZE 0 LINESIZE 1000 FEEDBACK OFF VERIFY OFF TRIMSPOOL ON
COLUMN ddl FORMAT A1000

BEGIN
   DBMS_METADATA.SET_TRANSFORM_PARAM (DBMS_METADATA.SESSION_TRANSFORM, 'SQLTERMINATOR', TRUE);
   DBMS_METADATA.SET_TRANSFORM_PARAM (DBMS_METADATA.SESSION_TRANSFORM, 'PRETTY', TRUE);
END;
/

VARIABLE v_role VARCHAR2(30);
EXEC :v_role := UPPER('&1');

SELECT DBMS_METADATA.GET_DDL('ROLE', r.role) AS ddl
FROM dba_roles r
WHERE r.role = :v_role
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('ROLE_GRANT', rp.grantee) AS ddl
FROM dba_role_privs rp
WHERE rp.grantee = :v_role AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('SYSTEM_GRANT', sp.grantee) AS ddl
FROM dba_sys_privs sp
WHERE sp.grantee = :v_role AND rownum = 1
UNION ALL
SELECT DBMS_METADATA.GET_GRANTED_DDL('OBJECT_GRANT', tp.grantee) AS ddl
FROM dba_tab_privs tp
WHERE tp.grantee = :v_role AND rownum = 1
/

SET LINESIZE 80 PAGESIZE 14 FEEDBACK ON VERIFY ON
```

### Create role and assign privileges

```sql
CREATE ROLE L1_SUPPORT;
CREATE ROLE L2_SUPPORT;
CREATE ROLE L3_SUPPORT;

SET PAGESIZE 0 ECHO OFF TRIMSPOOL ON LINESIZE 120 FEEDBACK OFF
SPOOL grant.sql

SELECT 'GRANT SELECT,INSERT,UPDATE,DELETE ON ' || a.owner || '."' || a.table_name || '"' || CHR(10) ||
       '   TO L2_SUPPORT;' || CHR(10) ||
       'GRANT SELECT ON ' || a.owner || '."' || a.table_name || '"' || CHR(10) ||
       '   TO L1_SUPPORT;' || CHR(10) ||
       'GRANT SELECT ON ' || a.owner || '."' || a.table_name || '"' || CHR(10) ||
       '   TO L3_SUPPORT;'
FROM dba_tables a
WHERE owner IN ('SCHEMA_NAME')
ORDER BY owner, table_name;

SPOOL OFF
SET FEEDBACK ON
```

### Roles comparison between databases

```sql
-- DB link: dev to prod
-- CREATE DATABASE LINK "COMPARE" CONNECT TO DBSNMP IDENTIFIED BY mypwd USING 'destination-db-name';

SELECT *
FROM DBA_TAB_PRIVS@compare T1
WHERE NOT EXISTS (
  SELECT 1 FROM dba_tab_privs T2
  WHERE t1.TABLE_NAME = t2.TABLE_NAME
    AND t1.PRIVILEGE = t2.PRIVILEGE
    AND t1.GRANTEE = t2.GRANTEE
)
AND t1.grantee = '&ROLE_NAME';
```

### Number of users with a particular role

```sql
SELECT GRANTEE AS users
FROM dba_role_privs
WHERE GRANTED_ROLE = UPPER('&GRANTED_ROLE');
```

---

## Privileges

### All privileges for a single user

```sql
SELECT 'GRANT '||granted_role||' TO '||grantee||
       DECODE(ADMIN_OPTION, 'Y', ' WITH ADMIN OPTION;', ';')
FROM dba_role_privs
WHERE grantee LIKE UPPER('%&&uname%')
UNION
SELECT 'GRANT '||privilege||' TO '||grantee||
       DECODE(ADMIN_OPTION, 'Y', ' WITH ADMIN OPTION;', ';')
FROM dba_sys_privs
WHERE grantee LIKE UPPER('%&&uname%')
UNION
SELECT 'GRANT ' || PRIVILEGE || ' ON ' || OWNER || '.' || TABLE_NAME || ' TO ' || GRANTEE || ';'
FROM dba_tab_privs
WHERE grantee LIKE UPPER('%&&uname%');
```

### List of users/roles having privilege on table

```sql
SELECT grantee || ' Through role ' || granted_role ge, 'SELECT' priv
FROM dba_role_privs
START WITH granted_role IN (SELECT grantee FROM dba_tab_privs WHERE PRIVILEGE = 'SELECT')
CONNECT BY PRIOR grantee = granted_role
UNION
SELECT grantee || ' Through role ' || granted_role ge, 'UPDATE' priv
FROM dba_role_privs
START WITH granted_role IN (SELECT grantee FROM dba_tab_privs WHERE PRIVILEGE = 'UPDATE')
CONNECT BY PRIOR grantee = granted_role
UNION
SELECT grantee || ' Through role ' || granted_role ge, 'INSERT' priv
FROM dba_role_privs
START WITH granted_role IN (SELECT grantee FROM dba_tab_privs WHERE PRIVILEGE = 'INSERT')
CONNECT BY PRIOR grantee = granted_role
UNION
SELECT grantee || ' Through role ' || granted_role ge, 'DELETE' priv
FROM dba_role_privs
START WITH granted_role IN (SELECT grantee FROM dba_tab_privs WHERE PRIVILEGE = 'DELETE')
CONNECT BY PRIOR grantee = granted_role
UNION
SELECT grantee || ' Direct' ge, PRIVILEGE priv
FROM SYS.dba_tab_privs
WHERE table_name = UPPER('&TABLE_NAME')
ORDER BY 1, 2;
```

### Privileges granted by you to others

```sql
SELECT * FROM USER_TAB_PRIVS_MADE;   -- table privileges granted by you
SELECT * FROM USER_TAB_PRIVS_RECD;   -- table privileges granted to you
SELECT * FROM USER_COL_PRIVS_MADE;   -- column privileges granted by you
SELECT * FROM USER_COL_PRIVS_RECD;   -- column privileges granted to you
SELECT * FROM USER_ROLE_PRIVS;       -- privileges granted to roles
```

### System privileges to roles and users

```sql
SELECT LPAD(' ', 2*LEVEL) || c "Privilege, Roles and Users"
FROM (
  SELECT NULL p, name c FROM system_privilege_map WHERE name LIKE UPPER('%&enter_privliege%')
  UNION
  SELECT granted_role p, grantee c FROM dba_role_privs
  UNION
  SELECT privilege p, grantee c FROM dba_sys_privs
)
START WITH p IS NULL
CONNECT BY p = PRIOR c;
```

### Roles & privs for a user

```sql
-- Granted Roles
SELECT * FROM DBA_ROLE_PRIVS WHERE GRANTEE = 'USER';

-- Privileges Granted Directly To User
SELECT * FROM DBA_TAB_PRIVS WHERE GRANTEE = 'USER';

-- Privileges Granted to Role Granted to User
SELECT * FROM DBA_TAB_PRIVS
WHERE GRANTEE IN (SELECT granted_role FROM DBA_ROLE_PRIVS WHERE GRANTEE = 'USER');

-- Granted System Privileges
SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE = 'USER';
```

---

## User Queries

### Non-default database users

```sql
SELECT username
FROM dba_users
WHERE username NOT IN (
  'ANONYMOUS','AURORA','$JIS','$UTILITY','$AURORA','$ORB','$UNAUTHENTICATED',
  'CTXSYS','DBSNMP','DMSYS','DSSYS','EXFSYS','MDSYS','ODM','ODM_MTR','OLAPSYS',
  'ORDPLUGINS','ORDSYS','OSE$HTTP$ADMIN','OUTLN','PERFSTAT','PUBLIC','REPADMIN',
  'SYS','SYSMAN','SYSTEM','TRACESVR','WKPROXY','WKSYS','WMSYS','XDB','APEX_PUBLIC_USER',
  'APEX_030200','APPQOSSYS','BI','DIP','FLOWS_XXXXXX','HR','IX','LBACSYS','MDDATA',
  'MGMT_VIEW','OE','ORACLE_OCM','ORDDATA','PM','SCOTT','SH','SI_INFORMTN_SCHEMA',
  'SPATIAL_CSW_ADMIN_USR','SPATIAL_WFS_ADMIN_USR','MTSSYS','OASPUBLIC','OLAPSYS',
  'OWBSYS','OWBSYS_AUDIT','WEBSYS','WK_PROXY','WK_TEST','AURORA$JIS$UTILITY$',
  'SECURE','AURORA$ORB$UNAUTHENTICATED','XS$NULL','FLOWS_FILES'
);
```

---

## Object Queries

### DDL of V$ tables

```sql
SET LONG 10000
SELECT VIEW_DEFINITION
FROM V$FIXED_VIEW_DEFINITION
WHERE view_name = 'GV$SQL_MONITOR';
```

### Source code of all invalid objects

```sql
SELECT DBMS_METADATA.GET_DDL(db.OBJECT_TYPE, db.OBJECT_NAME, db.OWNER)|| '/' 
FROM dba_objects db 
WHERE status = 'INVALID';

SELECT DBMS_METADATA.GET_DDL('PACKAGE_BODY', db.OBJECT_NAME, db.owner)|| '/' 
FROM dba_objects db 
WHERE status = 'INVALID' AND object_type = 'PACKAGE BODY';

SELECT DBMS_METADATA.GET_DDL(db.OBJECT_TYPE, db.OBJECT_NAME, db.OWNER)|| '/' 
FROM dba_objects db 
WHERE status = 'INVALID' AND object_type NOT LIKE '%PACKAGE%';
```

---

## Reference Links

- Reverse Engineering: http://www.orafaq.com/node/807
- Ask Tom: http://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:494205100346718343
- DBMS_METADATA GET_DDL: http://amit7oracledba.blogspot.com/2013/02/dbmsmetadatagetddl-package-how-to-get.html
- Create scripts via DBMS_METADATA: http://tech.padipa.net/generating-create-scripts-through-dbms_metadata-package

---

Back to [Main Index](README.md)
