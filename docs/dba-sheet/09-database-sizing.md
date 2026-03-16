---
layout: default
title: "Database Sizing Reference"
parent: "DBA Reference Sheet"
nav_order: 9
---

# Database Sizing Reference

Database, table, schema, and tablespace sizing queries for Oracle DBAs.

---

## Database Size

### Total database size (CDB)

```sql
SELECT a.data_size+b.temp_size+c.redo_size+d.controlfile_size "total_size in MB"
FROM (SELECT SUM(bytes)/1024/1024 data_size FROM dba_data_files) a,
     (SELECT NVL(SUM(bytes),0)/1024/1024 temp_size FROM dba_temp_files) b,
     (SELECT SUM(bytes)/1024/1024 redo_size FROM sys.v_$log) c,
     (SELECT SUM(BLOCK_SIZE*FILE_SIZE_BLKS)/1024/1024 controlfile_size FROM v$controlfile) d;
```

**Alternative (data + temp only):**

```sql
SELECT a.data_size+b.temp_size "total_size in MB"
FROM (SELECT SUM(bytes)/1024/1024 data_size FROM dba_data_files) a,
     (SELECT NVL(SUM(bytes),0)/1024/1024 temp_size FROM dba_temp_files) b;
```

### CDB with all PDB sizes

```sql
SELECT con_id, name, SUM(SIZE_MB)
FROM (
  SELECT c.con_id, NVL(p.name, 'CDB') name, SUM(bytes)/1024/1024 SIZE_MB
  FROM cdb_data_files c, v$pdbs p
  WHERE c.con_id=p.con_id(+)
  GROUP BY c.con_id, name
  UNION
  SELECT c.con_id, NVL(p.name, 'CDB') name, SUM(bytes)/1024/1024 SIZE_MB
  FROM cdb_temp_files c, v$pdbs p
  WHERE c.con_id=p.con_id(+)
  GROUP BY c.con_id, name
)
GROUP BY con_id, name
ORDER BY con_id;
```

### Segment-based database size

```sql
SELECT SUM(bytes)/1024/1024/1024 "DATABASE SIZE GB" FROM dba_segments;
```

---

## Table Size

### Table size in MB (without LOB)

```sql
SELECT segment_name, SUM(bytes)/(1024*1024) "TABLE_SIZE(MB)"
FROM dba_extents
WHERE segment_type='TABLE' AND owner=UPPER('&OWNER') AND segment_name=UPPER('&SEGMENT_NAME')
GROUP BY segment_name;
```

### Table size with LOB segments

```sql
COLUMN TABLE_NAME FORMAT A50
COLUMN OWNER FORMAT A30

SELECT owner, table_name, TRUNC(SUM(bytes)/1024/1024) Meg,
       ROUND(ratio_to_report(SUM(bytes)) OVER () * 100) Percent
FROM (
  SELECT segment_name table_name, owner, bytes
  FROM dba_segments
  WHERE segment_type IN ('TABLE', 'TABLE PARTITION', 'TABLE SUBPARTITION')
  UNION ALL
  SELECT i.table_name, i.owner, s.bytes
  FROM dba_indexes i, dba_segments s
  WHERE s.segment_name = i.index_name AND s.owner = i.owner
    AND s.segment_type IN ('INDEX', 'INDEX PARTITION', 'INDEX SUBPARTITION')
  UNION ALL
  SELECT l.table_name, l.owner, s.bytes
  FROM dba_lobs l, dba_segments s
  WHERE s.segment_name = l.segment_name AND s.owner = l.owner
    AND s.segment_type IN ('LOBSEGMENT', 'LOB PARTITION')
  UNION ALL
  SELECT l.table_name, l.owner, s.bytes
  FROM dba_lobs l, dba_segments s
  WHERE s.segment_name = l.index_name AND s.owner = l.owner
    AND s.segment_type = 'LOBINDEX'
)
WHERE owner = UPPER('&owner') AND table_name='&TABLE_NAME'
GROUP BY table_name, owner
HAVING SUM(bytes)/1024/1024 > 10
ORDER BY 2;
```

### Each table size under a schema

```sql
SELECT segment_name table_name, SUM(bytes)/(1024*1024) table_size_meg
FROM dba_extents
WHERE segment_type='TABLE' AND owner='GLOGOWNER'
GROUP BY segment_name
ORDER BY 2 DESC;
```

### LOB segment size in a table (column-wise)

```sql
SET LINESIZE 200
COLUMN owner FORMAT A30
COLUMN table_name FORMAT A30
COLUMN column_name FORMAT A30
COLUMN size_mb FORMAT 99999999.00

SELECT *
FROM (SELECT l.owner, l.table_name, l.column_name, l.segment_name, l.tablespace_name,
             ROUND(s.bytes/1024/1024,2) size_mb
      FROM dba_lobs l
      JOIN dba_segments s ON s.owner = l.owner AND s.segment_name = l.segment_name
      WHERE l.owner='&OWNER' AND l.TABLE_NAME='&TABLE_NAME'
      ORDER BY 6 DESC)
WHERE ROWNUM <= 20;
```

### Table partition size

```sql
SELECT owner, segment_name, partition_name, segment_type, bytes/1024/1024 "MB"
FROM dba_segments
WHERE segment_type = 'TABLE PARTITION'
  AND segment_name='&TABLE_NAME' AND partition_name='&SUBPARTITION_NAME' AND owner='&OWNER';
```

### Table subpartition size

```sql
SELECT owner, segment_name, partition_name, segment_type, bytes/1024/1024 "MB"
FROM dba_segments
WHERE segment_type = 'TABLE SUBPARTITION'
  AND segment_name='&TABLE_NAME' AND partition_name='&SUBPARTITION_NAME' AND owner='&OWNER';
```

---

## Schema Size

### Schema size in GB

```sql
SELECT owner, SUM(bytes)/1024/1024/1024 AS total_size_gb
FROM dba_segments
WHERE owner = UPPER('&1')
GROUP BY owner;
```

**Alternative (by tablespace):**

```sql
SELECT owner, tablespace_name, SUM(bytes)/1024/1024/1024 AS total_size_gb
FROM dba_segments
WHERE owner = UPPER('&1')
GROUP BY owner, tablespace_name;
```

### Each schema size in MB

```sql
SELECT owner, SUM(bytes/1024/1024)||'M' FROM dba_segments GROUP BY owner ORDER BY SUM(bytes/1024/1024) DESC;
```

---

## Tablespace

### Tablespace free space (detailed)

```sql
SET PAGES 50000 LINES 32767
COL tablespace_name FORMAT A30

SELECT a.tablespace_name,
       a.alloc_size/1024/1024/1024 Allocated_size,
       a.cur_size/1024/1024/1024 Current_Size,
       (u.used+a.file_count*65536)/1024/1024/1024 Used_size,
       (a.alloc_size-(u.used+a.file_count*65536))/1024/1024/1024 Available_size,
       ((u.used+a.file_count*65536)*100)/a.alloc_size Pct_used
FROM dba_tablespaces t,
     (SELECT t1.tablespace_name, NVL(SUM(s.bytes),0) used
      FROM dba_segments s, dba_tablespaces t1
      WHERE t1.tablespace_name=s.tablespace_name(+)
      GROUP BY t1.tablespace_name) u,
     (SELECT d.tablespace_name, SUM(GREATEST(d.bytes,NVL(d.maxbytes,0))) alloc_size,
             SUM(d.bytes) cur_size, COUNT(*) file_count
      FROM dba_data_files d
      GROUP BY d.tablespace_name) a
WHERE t.tablespace_name=u.tablespace_name
  AND t.tablespace_name=a.tablespace_name
ORDER BY Pct_used DESC;
```

### Tablespace free space (simple)

```sql
SELECT tablespace_name, SUM(bytes)/(1024*1024) "FREE(MB)"
FROM dba_free_space
WHERE tablespace_name=UPPER('&TABLESPACE_NAME')
GROUP BY tablespace_name;
```

### Datafiles under tablespace

```sql
COL file_name FOR A60
SELECT TABLESPACE_NAME, FILE_ID, FILE_NAME, BYTES/1024/1024 "size in MB",
       AUTOEXTENSIBLE, MAXBYTES/1024/1024, ONLINE_STATUS
FROM dba_data_files
WHERE TABLESPACE_NAME='&TABLESPACE_NAME';
```

**Add datafile:**

```sql
ALTER TABLESPACE USERTABLESPACE ADD DATAFILE '+DATAGROUP' SIZE 10G;
```

### Max datafile limit formula

```sql
SELECT 'ALTER TABLESPACE ' || tablespace_name || ' AUTOEXTEND ON MAXSIZE '
       || TO_CHAR(ROUND((bytes * 1.25)/(1024*1024*1024),0),9999) || 'G;'
FROM dba_data_files
WHERE bytes > maxbytes
ORDER BY tablespace_name;
```

---

## Temp Tablespace Usage

```sql
SELECT A.tablespace_name tablespace, D.mb_total,
       SUM(A.used_blocks * D.block_size) / 1024 / 1024 mb_used,
       D.mb_total - SUM(A.used_blocks * D.block_size) / 1024 / 1024 mb_free
FROM gv$sort_segment A,
     (SELECT B.name, C.block_size, SUM(C.bytes) / 1024 / 1024 mb_total
      FROM v$tablespace B, v$tempfile C
      WHERE B.ts#= C.ts#
      GROUP BY B.name, C.block_size) D
WHERE A.tablespace_name = D.name
GROUP BY A.tablespace_name, D.mb_total;
```

---

## Datafile Resize

### DB block size

```sql
COLUMN value NEW_VAL blksize
SELECT value FROM v$parameter WHERE name = 'db_block_size';
```

### Minimum size to reduce a datafile

Use the HWM (high-water mark) query to determine reclaimable space. Resize only if at least 1MB can be reclaimed.

### How much size we can reduce a datafile

```sql
SET VERIFY OFF
COLUMN file_name FORMAT A50 WORD_WRAPPED
COLUMN smallest FORMAT 999,990 HEADING "Smallest|Size|Poss."
COLUMN currsize FORMAT 999,990 HEADING "Current|Size"
COLUMN savings FORMAT 999,990 HEADING "Poss.|Savings"

COLUMN value NEW_VAL blksize
SELECT value FROM v$parameter WHERE name = 'db_block_size';

SELECT file_name,
       CEIL((NVL(hwm,1)*&&blksize)/1024/1024) smallest,
       CEIL(blocks*&&blksize/1024/1024) currsize,
       CEIL(blocks*&&blksize/1024/1024) - CEIL((NVL(hwm,1)*&&blksize)/1024/1024) savings
FROM dba_data_files a,
     (SELECT file_id, MAX(block_id+blocks-1) hwm FROM dba_extents GROUP BY file_id) b
WHERE a.file_id = b.file_id(+);
```

---

## Other Sizing

### Top 10 large tables

```sql
SELECT * FROM (
  SELECT owner, segment_name, bytes/1024/1024 meg
  FROM dba_segments
  WHERE segment_type = 'TABLE'
    AND owner NOT IN ('SYS','SYSTEM','DBSNMP','APEX_040200','MDSYS')
  ORDER BY bytes/1024/1024 DESC
) WHERE ROWNUM <= 10;
```

### Index larger than table (for rebuilding)

```sql
SELECT si.owner, si.tablespace_name, i.index_name, t.table_name,
       ROUND(si.bytes/1024/1024,1) index_mb, ROUND(st.bytes/1024/1024,1) table_mb,
       ROUND(si.bytes/st.bytes*100,1) pct_larger
FROM dba_segments si, dba_indexes i, dba_tables t, dba_segments st
WHERE si.owner = i.owner AND si.segment_name = i.index_name
  AND si.segment_type = 'INDEX'
  AND i.table_owner = t.owner AND i.table_name = t.table_name
  AND t.owner = st.owner AND t.table_name = st.segment_name
  AND st.segment_type = 'TABLE'
  AND si.bytes > st.bytes
  AND si.bytes > 100*1024*1024
ORDER BY pct_larger DESC;
```

### Table fragmentation

```sql
COL table_name FOR A30
SELECT table_name,
       ROUND((blocks*8/1024),2)||'MB' "TOTAL_SIZE",
       ROUND((num_rows*avg_row_len/1024/1024),2)||'Mb' "ACTUAL_SIZE",
       ROUND((blocks*8/1024)-(num_rows*avg_row_len/1024/1024),2)||'MB' "FRAGMENTED_SPACE",
       (ROUND((blocks*8/1024)-(num_rows*avg_row_len/1024/1024),2)/ROUND((blocks*8/1024),2))*100 "percentage"
FROM dba_tables
WHERE owner='&OWNER' AND table_name='&TABLE_NAME';
```

### LOB space allocations - partition

```sql
SELECT l.column_name, l.partition_name, l.lob_name, l.lob_partition_name, s.bytes/1048576 "Size (MB)"
FROM dba_segments s, dba_lob_partitions l
WHERE s.segment_name = l.lob_name
  AND s.owner='&OWNER'
  AND l.table_name='&table_name'
  AND l.lob_partition_name = s.partition_name;
```

---

**Back to [Main Index](README.md)**
