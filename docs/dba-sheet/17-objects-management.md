---
layout: default
title: "Objects Management"
parent: "DBA Reference Sheet"
nav_order: 17
---

# Objects Management

Invalid objects, recompilation, indexes, fragmentation, constraints, and database links.

---

## Invalid Objects

### List invalid objects

```sql
SELECT owner, object_type, SUBSTR(object_name,1,30) object_name, status
FROM dba_objects
WHERE status='INVALID'
ORDER BY object_type;
```

**Alternative (generate select statements):**

```sql
SET lines 750 pages 9999
COL owner FOR a30
COL object_name FOR a50
COL object_type FOR a30
SELECT 'select owner,object_name,object_type from dba_objects where object_name='''|| object_name ||''' and owner='''|| owner ||''';'
FROM dba_objects WHERE status='INVALID';
```

### Details of an object

```sql
SET lines 1000 pages 9999
COL object_name FOR a50
COL owner FOR a30
COL object_type FOR a30
COL STATUS FOR a30
SELECT owner, object_name, object_type, CREATED, STATUS,
       TO_CHAR(LAST_DDL_TIME,'dd-mon-yyyy hh24:mi:ss') AS LAST_DDL_TIME
FROM dba_objects
WHERE object_name IN ('&OBJECT_NAME1','&OBJECT_NAME2','&OBJECT_NAME3');
```

### Source code of all invalid objects

```sql
SELECT DBMS_METADATA.GET_DDL(db.OBJECT_TYPE, db.OBJECT_NAME, db.OWNER)|| '/' 
FROM dba_objects db WHERE status='INVALID';
```

---

## Recompilation

### Generate recompile statements (non-PDB)

```sql
SELECT 'alter '||object_type||' '||owner||'."'||object_name||'" compile;'
FROM dba_objects
WHERE status<>'VALID' AND object_type NOT IN ('PACKAGE BODY','TYPE BODY','UNDEFINED','JAVA CLASS','SYNONYM')
UNION
SELECT 'alter package '||owner||'.'||object_name||' compile body;'
FROM dba_objects WHERE status<>'VALID' AND object_type='PACKAGE BODY'
UNION
SELECT 'alter type '||owner||'.'||object_name||' compile body;'
FROM dba_objects WHERE status<>'VALID' AND object_type='TYPE BODY'
UNION
SELECT 'alter materialized view '||owner||'.'||object_name||' compile;'
FROM dba_objects WHERE status<>'VALID' AND object_type='UNDEFINED'
UNION
SELECT 'alter java class '||owner||'."'||object_name||'" resolve;'
FROM dba_objects WHERE status<>'VALID' AND object_type='JAVA CLASS'
UNION
SELECT 'alter synonym '||owner||'.'||object_name||' compile;'
FROM dba_objects WHERE status<>'VALID' AND object_type='SYNONYM' AND owner<>'PUBLIC'
UNION
SELECT 'alter public synonym '||object_name||' compile;'
FROM dba_objects WHERE status<>'VALID' AND object_type='SYNONYM' AND owner='PUBLIC';
```

For PDB, set before running:

```sql
ALTER SESSION SET "_ORACLE_SCRIPT"=true;
```

**Alternative (PDB):**

```sql
EXEC dbms_pdb.exec_as_oracle_script('alter package SYS.DBMS_METADATA compile body;');
```

### Script to recompile invalid objects

```sql
SET serveroutput ON SIZE 100000;
DECLARE
  sqlstring VARCHAR2(2000);
  rec_count INTEGER;
BEGIN
  dbms_utility.compile_schema('SCHEMA1');
  dbms_utility.compile_schema('SCHEMA2');
  SELECT COUNT(1) INTO rec_count FROM sys.all_objects
  WHERE owner IN ('SCHEMA1','SCHEMA2')
    AND object_type IN ('FUNCTION','VIEW','PROCEDURE','TRIGGER') AND status = 'INVALID';
  IF (rec_count > 0) THEN
    FOR recs IN (SELECT owner, object_type, object_name FROM sys.dba_objects
                 WHERE owner IN ('SCHEMA1','SCHEMA2')
                   AND object_type IN ('FUNCTION','VIEW','PROCEDURE','TRIGGER')
                   AND status = 'INVALID') LOOP
      BEGIN
        sqlstring := 'ALTER ' || recs.object_type || ' ' || recs.owner || '.' || recs.object_name || ' COMPILE';
        EXECUTE IMMEDIATE sqlstring;
      EXCEPTION
        WHEN OTHERS THEN
          dbms_output.put_line(recs.object_type || ' ' || recs.owner || '.' || recs.object_name || ' failed: ' || sqlerrm);
      END;
    END LOOP;
  END IF;
END;
/
```

### Compile package with reuse settings

```sql
ALTER PACKAGE pkg1 COMPILE REUSE SETTINGS;
```

---

## Compile Errors

### View compilation errors

```sql
SELECT text FROM dba_errors WHERE name='VIEW1';
```

**Alternative (all errors):**

```sql
SET lines 750 pages 9999
COL text FOR a150
COL owner FOR a15
COL name FOR a50
COL position FOR a10
SELECT e.owner, e.name, TO_CHAR(e.line) || '/' || TO_CHAR(e.position) "POSITION", e.text
FROM dba_errors e
ORDER BY e.owner, e.name, e.sequence;
```

---

## Dropping Objects

### Drop all objects under a schema

```sql
ALTER SESSION SET current_schema=&SCHEMA;

DECLARE
BEGIN
  FOR r1 IN (
    SELECT 'DROP ' || object_type || ' ' || owner || '."'|| object_name || '"' ||
           DECODE(object_type, 'TABLE', ' CASCADE CONSTRAINTS PURGE') AS v_sql
    FROM dba_objects
    WHERE owner IN ('&SCHEMA')
      AND object_type IN ('TABLE','VIEW','PACKAGE','TYPE','PROCEDURE','FUNCTION','TRIGGER','SEQUENCE','MATERIALIZED VIEW','JAVA CLASS')
    ORDER BY object_type, object_name
  ) LOOP
    EXECUTE IMMEDIATE r1.v_sql;
  END LOOP;
END;
/
PURGE dba_recyclebin;
```

---

## Object Counts

### Objects count by owner and type

```sql
BREAK ON owner SKIP 1 ON object_type
COMPUTE SUM OF cnt ON owner
COL owner FOR a30
SELECT DISTINCT owner, object_type, COUNT(*) cnt
FROM dba_objects
GROUP BY object_type, owner
ORDER BY owner;
CLEAR BREAK
CLEAR COMPUTES
```

### Row count of all tables in a schema

```sql
SELECT table_name,
  TO_NUMBER(EXTRACTVALUE(XMLTYPE(dbms_xmlgen.getxml('select count(*) c from '||table_name)), '/ROWSET/ROW/C')) count
FROM user_tables
ORDER BY table_name;
```

---

## Indexes

### All indexes for a table

```sql
COL owner FOR a30
COL index_name FOR a30
COL table_name FOR a30
COL column_name FOR a30
SELECT owner, b.uniqueness, a.index_name, a.table_name, a.column_name
FROM dba_ind_columns a, dba_indexes b
WHERE a.index_name = b.index_name
  AND b.owner = '&OWNER'
  AND a.table_name = UPPER('&TABLE_NAME')
ORDER BY a.table_name, a.index_name, a.column_position;
```

### Find table name from index name

```sql
COL owner FOR a30
COL index_name FOR a30
COL table_name FOR a30
COL column_name FOR a30
SELECT owner, b.uniqueness, a.index_name, a.table_name, a.column_name
FROM dba_ind_columns a, dba_indexes b
WHERE a.index_name = b.index_name
  AND b.owner = '&OWNER'
  AND b.index_name = UPPER('&INDEX_NAME')
ORDER BY a.table_name, a.index_name, a.column_position;
```

### Unused indexes

```sql
SELECT table_name, index_name
FROM user_indexes i
WHERE uniqueness <> 'UNIQUE'
  AND index_name NOT IN (
    SELECT DISTINCT object_name FROM v$sql_plan
    WHERE operation LIKE '%INDEX%' AND object_owner='&OWNER'
  );
```

---

## Triggers and Constraints

### Disable trigger

```sql
SELECT 'ALTER TRIGGER '||OWNER||'.'||TRIGGER_NAME||' DISABLE;'
FROM dba_triggers
WHERE table_name IN ('&TABLE_NAME1') AND owner='&OWNER';
```

### Disable constraint

```sql
SELECT 'ALTER TABLE '||OWNER||'.'||TABLE_NAME||' DISABLE CONSTRAINT '||CONSTRAINT_NAME||';'
FROM dba_constraints
WHERE table_name IN ('&TABLE1') AND owner='&OWNER';
```

### Disable foreign key constraints for schema

```sql
SELECT 'ALTER TABLE '||OWNER||'.'||TABLE_NAME||' '||var_action||' CONSTRAINT '||CONSTRAINT_NAME AS sql_string,
       CONSTRAINT_NAME
FROM ALL_CONSTRAINTS
WHERE CONSTRAINT_TYPE='R' AND OWNER=Target_Schema_Name;
```

### ORA-02292: integrity constraint violated – child record found

```sql
SELECT owner, constraint_name, constraint_type, table_name, r_owner, r_constraint_name
FROM all_constraints
WHERE owner='&OWNER' AND constraint_name='&CONSTRAINT_NAME';
```

---

## Dependencies

### Objects depending on a given object

```sql
SELECT name, type, owner
FROM all_dependencies
WHERE referenced_owner = 'USER_NAME' AND referenced_name = 'OBJECT_NAME';
```

### Dependent objects (detailed)

```sql
SELECT TYPE || ' ' || OWNER || '.' || NAME || ' references ' || REFERENCED_TYPE || ' ' ||
       REFERENCED_OWNER || '.' || REFERENCED_NAME AS DEPENDENCIES
FROM dba_dependencies
WHERE referenced_name = UPPER(LTRIM(RTRIM('&ls_REF_name')))
   OR name = UPPER(LTRIM(RTRIM('&ls_REF_name')))
  AND (REFERENCED_OWNER NOT IN ('SYS','SYSTEM','PUBLIC') AND OWNER NOT IN ('SYS','SYSTEM','PUBLIC'))
ORDER BY 1;
```

---

## LOB Objects

### BLOB/CLOB tables in database

```sql
SET lines 750 pages 999
COL owner FOR a30
COL table_name FOR a50
COL column_name FOR a50
COL data_type FOR a30
SELECT OWNER, TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM dba_tab_columns
WHERE DATA_TYPE IN ('CLOB','BLOB','LOB','BFILE','NCLOB')
  AND owner NOT IN ('SYS','SYSTEM','CTXSYS','WMSYS','XDB','OUTLN')
ORDER BY 1, 2;
```

---

## Column Analysis

### Count of columns per table in schema

```sql
COL owner FOR a30
COL table_name FOR a50
SELECT t.owner, t.table_name, t.num_rows, COUNT(*)
FROM dba_tables t
LEFT JOIN dba_tab_columns c ON t.table_name = c.table_name
WHERE t.owner='&OWNER_NAME' AND num_rows IS NOT NULL
GROUP BY t.owner, t.table_name, t.num_rows
ORDER BY COUNT(*) DESC;
```

### Find duplicate records in a column

```sql
SELECT column_name, COUNT(column_name)
FROM table
GROUP BY column_name
HAVING COUNT(column_name) > 1;
```

---

## Database Links

### List database links

```sql
COL owner FOR a30
COL db_link FOR a50
COL username FOR a30
COL host FOR a100
SELECT * FROM dba_db_links;
```

### Drop private database links for schema

```sql
SET serveroutput ON
DECLARE
  l_sql CLOB := 'CREATE PROCEDURE <OWNER>.drop_db_links_prc IS BEGIN
    FOR i IN (SELECT * FROM user_db_links) LOOP
      dbms_output.put_line(''to drop ''||i.db_link);
      EXECUTE IMMEDIATE ''DROP DATABASE LINK ''||i.db_link;
    END LOOP;
  END;';
  l_sql1 CLOB;
BEGIN
  FOR i IN (SELECT DISTINCT owner FROM dba_objects
            WHERE owner IN ('TOMASZ') AND object_type='DATABASE LINK') LOOP
    l_sql1 := REPLACE(l_sql, '<OWNER>', i.owner);
    EXECUTE IMMEDIATE l_sql1;
    l_sql1 := 'BEGIN '||i.owner||'.drop_db_links_prc; END;';
    EXECUTE IMMEDIATE l_sql1;
    l_sql1 := 'DROP PROCEDURE '||i.owner||'.drop_db_links_prc';
    EXECUTE IMMEDIATE l_sql1;
  END LOOP;
END;
/
```

---

## Index Rebuild and Fragmentation

### Indexes that may need rebuild

```sql
SELECT a.*, ROUND(index_leaf_estimate_if_rebuilt/current_leaf_blocks*100) percent,
       CASE WHEN index_leaf_estimate_if_rebuilt/current_leaf_blocks < 0.5 THEN 'candidate for rebuild' END status
FROM (
  SELECT table_name, index_name, current_leaf_blocks,
    ROUND(100/90*(ind_num_rows*(rowid_length+uniq_ind+4)+SUM((avg_col_len)*(tab_num_rows)))/(8192-192)) AS index_leaf_estimate_if_rebuilt
  FROM (
    SELECT tab.table_name, tab.num_rows tab_num_rows, DECODE(tab.partitioned,'YES',10,6) rowid_length,
           ind.index_name, ind.index_type, ind.num_rows ind_num_rows, ind.leaf_blocks AS current_leaf_blocks,
           DECODE(uniqueness,'UNIQUE',0,1) uniq_ind, ic.column_name AS ind_column_name, tc.column_name, tc.avg_col_len
    FROM dba_tables tab
    JOIN dba_indexes ind ON ind.owner=tab.owner AND ind.table_name=tab.table_name
    JOIN dba_ind_columns ic ON ic.table_owner=tab.owner AND ic.table_name=tab.table_name AND ic.index_owner=tab.owner AND ic.index_name=ind.index_name
    JOIN dba_tab_columns tc ON tc.owner=tab.owner AND tc.table_name=tab.table_name AND tc.column_name=ic.column_name
    WHERE tab.owner='&OWNER' AND ind.leaf_blocks IS NOT NULL AND ind.leaf_blocks > 1000
  ) GROUP BY table_name, index_name, current_leaf_blocks, ind_num_rows, uniq_ind, rowid_length
) a
WHERE index_leaf_estimate_if_rebuilt/current_leaf_blocks < 0.5
ORDER BY index_leaf_estimate_if_rebuilt/current_leaf_blocks;
```

### Generate rebuild index statements

```sql
SELECT 'alter index ' || owner || '.' || index_name || ' rebuild;'
FROM dba_indexes
WHERE owner NOT IN ('SYS','SYSTEM');
```

### Index fragmentation (ANALYZE INDEX VALIDATE STRUCTURE)

```sql
ANALYZE INDEX emp_id_idx VALIDATE STRUCTURE;
SELECT name, del_lf_rows, lf_rows - del_lf_rows lf_rows_used,
       TO_CHAR(del_lf_rows/(DECODE(lf_rows,0,0.01,lf_rows))*100,'999.99999') ibadness
FROM index_stats;
```

---

## Table Fragmentation

### Table size vs actual data

```sql
SELECT owner, table_name, ROUND((blocks*8),2)||' kb' "TABLE SIZE",
       ROUND((num_rows*avg_row_len/1024),2)||' kb' "ACTUAL DATA"
FROM dba_tables
WHERE table_name='&Table_name';
```

### Table fragmentation – all tables in schema

```sql
SELECT table_name, ROUND((blocks*8),2) "size (kb)",
       ROUND((num_rows*avg_row_len/1024),2) "actual_data (kb)",
       (ROUND((blocks*8),2) - ROUND((num_rows*avg_row_len/1024),2)) "wasted_space (kb)"
FROM dba_tables
WHERE owner='&OWNER' AND table_name='&TABLE_NAME'
  AND (ROUND((blocks*8),2) > ROUND((num_rows*avg_row_len/1024),2))
ORDER BY 4 DESC;
```

**Alternative (with MB):**

```sql
SELECT owner, table_name, blocks, num_rows, avg_row_len,
       ROUND((blocks*8/1024),2) "TOTAL_SIZE(MB)",
       ROUND((num_rows*avg_row_len/1024/1024),2) "ACTUAL_SIZE(MB)",
       ROUND((blocks*8/1024)-(num_rows*avg_row_len/1024/1024),2) "FRAGMENTED_SPACE(MB)"
FROM dba_tables
WHERE owner='&owner' AND table_name='&table_name'
  AND ROUND((blocks*8/1024)-(num_rows*avg_row_len/1024/1024),2) > 1
ORDER BY 8 DESC;
```

---

## Reference Links

- [Scripts to find object dependencies](https://rodgersnotes.wordpress.com/2011/12/27/scripts-to-find-object-dependencies/)
- [Debugging ORA-02292](https://bommaritollc.com/2012/01/22/debugging-ora-02292-integrity-constraint-owner-constraint-violated-child-record-found/)
- [How I measure Oracle index fragmentation](https://blog.dbi-services.com/how-i-measure-oracle-index-fragmentation/)

---

Back to [Main Index](README.md)
