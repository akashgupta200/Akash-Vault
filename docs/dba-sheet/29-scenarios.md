---
layout: default
title: "Scenarios"
parent: "DBA Reference Sheet"
nav_order: 29
---

# Scenarios

Common DBA scenarios and reference patterns for RMAN clone, restore, EXP/IMP, EXPDP/IMPDP, and Data Pump features.

---

## Scenario Overview

| Type | Levels | Options |
|------|--------|---------|
| RMAN Clone | Same server | Same diskgroup / Different diskgroup |
| RMAN Clone | Different server | Same diskgroup / Different diskgroup |
| RMAN Restore | Same server | Same diskgroup / Different diskgroup |
| RMAN Restore | Different server | Same diskgroup / Different diskgroup |
| EXP | Table / Schema / Database | Single / Multiple / FULL DB |
| IMP | Table / Schema / Database | Single / Multiple / FULL DB |
| EXPDP | Table / Schema / Database | Single / Multiple / FULL DB |
| IMPDP | Table / Schema / Database | Single / Multiple / FULL DB |

---

## EXPDP

### Table Level — Single Table

```bash
cat > expdp_table_refresh.par <<EOF
userid='/ as sysdba'
directory=DATA_PUMP_DIR
dumpfile=TABLE1_%U.dmp
logfile=TABLE1.log
tables=SCHEMA.TABLE1
parallel=8
EOF
```

```bash
nohup expdp parfile=testfile.par &
tail -200f nohup.out
```

### Table Level — Multiple Tables

```bash
cat > expdp_table_refresh.par <<EOF
userid='/ as sysdba'
directory=DATA_PUMP_DIR
dumpfile=TABLE1_%U.dmp
logfile=TABLE1.log
tables=SCHEMA1.TABLE1,SCHEMA2.TABLE2
parallel=8
EOF
```

### Schema Level — Single Schema

```bash
cat > expdp_schema_refresh.par <<EOF
userid='/ as sysdba'
directory=DATA_PUMP_DIR
dumpfile=SCHEMA_NOBIGTABLES_%U.dmp
logfile=SCHEMA_NOBIGTABLES.log
schemas=SAMPLE_SCHEMA
exclude=table:"IN('TABLE1','TABLE2','TABLE3')"
parallel=8
EOF
```

### Schema Level — Multiple Schemas

```bash
cat > expdp_schema_refresh.par <<EOF
userid='/ as sysdba'
directory=DATA_PUMP_DIR
dumpfile=SCHEMA_NOBIGTABLES_%U.dmp
logfile=SCHEMA_NOBIGTABLES.log
schemas=SAMPLE_SCHEMA1,SAMPLE_SCHEMA2
exclude=table:"IN('TABLE1','TABLE2','TABLE3')"
parallel=8
EOF
```

### Database — FULL DB

```bash
cat > expdp_schema_refresh.par <<EOF
userid='/ as sysdba'
directory=DATA_PUMP_DIR
dumpfile=SCHEMA_NOBIGTABLES_%U.dmp
logfile=SCHEMA_NOBIGTABLES.log
FULL=Y
parallel=8
EOF
```

### EXPDP in background

```bash
nohup expdp parfile=testfile.par &
tail -200f nohup.out
```

---

## IMPDP

IMPDP supports Table Level (Single/Multiple), Schema Level (Single/Multiple), Database (FULL DB), and Network Direct Import.

---

## Data Pump Features

### Exclude Table

```sql
exclude=table:"IN('TABLE1','TABLE2','TABLE3')"
```

**Alternative:**

```sql
exclude=TABLE:"LIKE 'EXAM%'"
```

```sql
exclude=TABLES:">'F'"
```

Reference: [ACE Hints - Data Pump remap_table](http://www.acehints.com/2012/11/data-pump-impdp-remaptable-option-to.html)

### Exclude Common

```sql
exclude=SEQUENCE,PROCEDURES,INDEXES,TABLES:"IN ('EMP1','DEPT')"
```

### Remap Schema

```sql
REMAP_SCHEMA=SCHEMA1:DUPSCHEMA2
TABLE_EXISTS_ACTION=SKIP
```

### Remap Tablespace

```sql
SELECT table_name, tablespace_name FROM dba_tables WHERE owner='HR';
```

```sql
REMAP_TABLESPACE = USERS:EXP_TBS1,USERS2:EXP_TBS2
```

### Remap Table

Same schema:

```sql
remap_table=emp:emp_bkup
```

Different schema:

```sql
REMAP_SCHEMA=SCHEMA1:DUPSCHEMA2
remap_table=emp:emp_bkup
```

### TABLE_EXISTS_ACTION

Options: `APPEND`, `REPLACE`, `SKIP`, `TRUNCATE`

### 12c Transform

```sql
TRANSFORM=DISABLE_ARCHIVE_LOGGING:Y
```

---

## Transfer Utilities

### SCP algorithm

```bash
scp -c arcfour -r myuserid@sourceserver:/explocation/dumpfile.dmp .
```

---

## Space Check

### Check free space on importing tablespace

```sql
SELECT tablespace_name,
SUM (bytes) / (1024 * 1024) "FREE(MB)"
FROM dba_free_space where tablespace_name=UPPER('&TABLESPACE_NAME')
GROUP BY tablespace_name;
```

---

Back to [Main Index](README.md)
