---
layout: default
title: "Data Guard Reference"
parent: "DBA Reference Sheet"
nav_order: 6
---

# Data Guard Reference

Oracle Data Guard: log shipping, gap detection, DGMGRL, switchover, standby database management.

---

## Log Shipping and Errors

### Log shipping status and errors

```sql
SELECT DISTINCT error FROM gv$archive_dest;
```

### How much transferred & applied - GAP detection

**On Primary:**

```sql
SET LINES 160
COL STBY_BEHIND_BY FOR A20
SELECT (SELECT sysdate FROM dual) "TIMESTAMP", s.thread#, s.MAX_SEQ#, a.MAX_APP_SEQ#,
       ' '||(s.MAX_SEQ#-a.MAX_APP_SEQ#)||' sequences' "STBY_BEHIND_BY"
FROM (SELECT thread#, MAX(sequence#) "MAX_SEQ#"
      FROM v$archived_log
      WHERE activation#=(SELECT activation# FROM v$database)
      GROUP BY thread#) s,
     (SELECT thread#, MAX(sequence#) "MAX_APP_SEQ#"
      FROM v$archived_log
      WHERE activation#=(SELECT activation# FROM v$database) AND applied='YES'
      GROUP BY thread#) a
WHERE a.thread#=s.thread#;
```

**Alternative (on Primary):**

```sql
COL STBY_BEHIND_BY FOR A20
SELECT (SELECT sysdate FROM dual) "TIMESTAMP", s.thread#, s.MAX_SEQ#, a.MAX_APP_SEQ#,
       (s.MAX_SEQ#-a.MAX_APP_SEQ#) "Difference"
FROM (SELECT thread#, MAX(sequence#) "MAX_SEQ#" FROM v$archived_log GROUP BY thread#) s,
     (SELECT thread#, MAX(sequence#) "MAX_APP_SEQ#" FROM v$archived_log WHERE applied='YES' GROUP BY thread#) a
WHERE a.thread#=s.thread#
ORDER BY s.thread#;
```

---

## Gaps

### Archive gaps

```sql
SELECT * FROM v$archive_gap;
```

**Alternative:**

```sql
SELECT sequence#, ARCHIVED AS RECEIVED, applied
FROM v$archived_log
WHERE applied='NO'
ORDER BY sequence#;
```

---

## General Health

### General health of Data Guard and errors

**Run at Primary:**

```sql
SET PAGES 1000
SET LINES 120
COLUMN DEST_NAME FORMAT A20
COLUMN DESTINATION FORMAT A35
SELECT DEST_ID, DEST_NAME, DESTINATION, TARGET, STATUS, ERROR
FROM v$archive_dest
WHERE DESTINATION IS NOT NULL;

SELECT ads.dest_id, MAX(sequence#) "Current Sequence", MAX(log_sequence) "Last Archived"
FROM v$archived_log al, v$archive_dest ad, v$archive_dest_status ads
WHERE ad.dest_id=al.dest_id AND al.dest_id=ads.dest_id
GROUP BY ads.dest_id;
```

**Run at Standby:**

```sql
SELECT MAX(al.sequence#) "Last Seq Received" FROM v$archived_log al;
SELECT MAX(al.sequence#) "Last Seq Applied" FROM v$archived_log al WHERE applied='YES';
SELECT process, status, sequence# FROM v$managed_standby;
```

### Standby DB name

```sql
SELECT (SELECT name FROM v$database) PRIMARY, DB_UNIQUE_NAME STANDBY_NAME
FROM v$archive_dest
WHERE TARGET='STANDBY' AND status='VALID';
```

### Last applied archive log

```sql
SELECT thread#, MAX(sequence#) FROM v$log_history GROUP BY thread#;
```

---

## MRP (Media Recovery Process)

### MRP in WAIT_FOR_LOG - recovery steps

```sql
SELECT * FROM gv$managed_standby WHERE process LIKE '%MRP%';
```

```sql
RECOVER MANAGED STANDBY DATABASE CANCEL;
RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT;
```

### Is MRP waiting for something?

```sql
SELECT a.event, a.wait_time, a.seconds_in_wait
FROM gv$session_wait a, gv$session b
WHERE a.sid=b.sid
  AND b.sid=(SELECT SID FROM v$session WHERE PADDR=(SELECT PADDR FROM v$bgprocess WHERE NAME='MRP0'));
```

### MRP apply speed

```sql
SET LINES 1000 PAGES 9999
SELECT TO_CHAR(START_TIME,'DD-MON-YYYY HH24:MI:SS') "Recovery Start Time",
       TO_CHAR(item)||' = '||TO_CHAR(sofar)||' '||TO_CHAR(units) "Progress"
FROM v$recovery_progress
WHERE start_time=(SELECT MAX(start_time) FROM v$recovery_progress);
```

### How to increase MRP apply speed

```sql
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE PARALLEL 8 DISCONNECT FROM SESSION;
```

---

## DGMGRL Commands

### Show configuration

```bash
show configuration
```

### Show database verbose

```bash
show database verbose wdbname
```

### Check database inconsistency

```bash
show database wdbname InconsistentProperties
```

### Equivalent broker commands to ALTER SYSTEM

| SQL | DGMGRL |
|-----|--------|
| `ALTER DATABASE RECOVER MANAGED STANDBY CANCEL` | `edit database 'stby_dbname' set state='LOG-APPLY-OFF'` |
| `ALTER DATABASE RECOVER MANAGED STANDBY DISCONNECT` | `edit database 'stby_dbname' set state='ONLINE'` |
| `ALTER SYSTEM SET log_archive_max_processes=4` | `edit database 'dbname' set property 'LogArchiveMaxProcesses'=4` |

### DGMGRL logging

```bash
dgmgrl -logfile observer.log / "start observer"
```

---

## Configuration

### Database role (primary or standby)

```sql
SELECT database_role FROM v$database;
SELECT controlfile_type FROM v$database;
```

### log_archive_dest_2 configuration

```sql
ALTER SYSTEM SET log_archive_dest_1 = 'location=use_db_recovery_file_dest valid_for=(all_logfiles, all_roles) db_unique_name=primarydbname';
ALTER SYSTEM SET log_archive_dest_2='service="stbydbname",LGWR ASYNC NOAFFIRM delay=0 optional compression=disable max_failure=0 max_connections=1 reopen=300 valid_for=(all_logfiles,primary_role) db_unique_name="stbydbname"';
```

---

## Switchover

### Data Guard switchover

Execute on primary:

```sql
ALTER DATABASE COMMIT TO SWITCHOVER TO PHYSICAL STANDBY WITH SESSION SHUTDOWN;
```

---

## Standby Realtime Apply

### Standby realtime apply enabled or not

Run on standby:

```sql
SELECT DEST_ID, dest_name, status, type, srl, recovery_mode
FROM v$archive_dest_status
WHERE dest_id=1;
```

---

## Validation

### Full details of DG db from DGMGRL

```bash
dgmgrl> validate database verbose db_name
```

---

**Back to [Main Index](README.md)**
