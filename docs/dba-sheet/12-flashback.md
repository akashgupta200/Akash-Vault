---
layout: default
title: "Flashback"
parent: "DBA Reference Sheet"
nav_order: 12
---

# Flashback

Reference guide for restore points, flashback database, flashback query, and FRA usage.

---

## Restore Points

### Restore point check

```sql
SELECT name, scn, time, guarantee_flashback_database, DATABASE_INCARNATION#
FROM v$restore_point;
```

### Drop restore point

```sql
DROP RESTORE POINT PSUCHECK;
```

---

## Flashback Database

### Flashback on/off

```sql
SELECT flashback_on FROM v$database;

ALTER DATABASE flashback OFF;
ALTER DATABASE flashback ON;
```

---

## Flash Recovery Area

### FRA usage

```sql
SELECT * FROM v$flash_recovery_area_usage;
```

### FRA size

```sql
SET LINES 100
COL name FORMAT A60

SELECT name,
       FLOOR(space_limit / 1024 / 1024) "Size MB",
       CEIL(space_used / 1024 / 1024) "Used MB"
FROM v$recovery_file_dest
ORDER BY name
/
```

---

## Flashback Query

### Is row movement enabled for a table?

```sql
SELECT table_name, row_movement
FROM dba_tables
WHERE owner = 'ODB'
  AND table_name = 'AC_ACTUAL_FLIGHTS';
```

### Finding dependent constraints (ORA-02266)

```sql
SELECT CONSTRAINT_NAME, OWNER, TABLE_NAME
FROM DBA_CONSTRAINTS
WHERE R_CONSTRAINT_NAME IN (
  SELECT CONSTRAINT_NAME
  FROM dba_constraints
  WHERE OWNER = UPPER('&user_dependent_from')
    AND TABLE_NAME = UPPER('&object_dependent_from')
)
AND STATUS = 'ENABLED';
```

---

## SCN & Timestamp Conversion

### Timestamp to SCN

```sql
SELECT TIMESTAMP_TO_SCN(TO_TIMESTAMP('25/07/2012 16:32:30','DD/MM/YYYY HH24:MI:SS')) AS scn
FROM dual;
```

### SCN to timestamp

```sql
SELECT SCN_TO_TIMESTAMP(176195435) AS timestamp
FROM dual;
```

### Finding current SCN

```sql
SELECT current_scn FROM v$database;
```

---

## FRA Cleanup

### Fix incorrect archivelog count in v$flash_recovery_area_usage

```bash
RMAN> RUN {
  change archivelog all crosscheck;
  report obsolete orphan;
  report obsolete;
  crosscheck backup;
  crosscheck copy;
  crosscheck backup of controlfile;

  delete noprompt expired backup;
  delete noprompt expired archivelog all;
  delete noprompt expired backup of controlfile;
  delete force noprompt expired copy;
  delete force noprompt obsolete orphan;
  delete force noprompt obsolete;
}
```

---

Back to [Main Index](README.md)
