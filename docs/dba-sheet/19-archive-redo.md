---
layout: default
title: "Archive & Redo Logs"
parent: "DBA Reference Sheet"
nav_order: 19
---

# Archive & Redo Logs

Archive log management and redo log operations for Oracle DBAs.

---

## Archive Logs

### Archive log size

```sql
SET lines 1000 pages 9999
COL name FOR a100
SELECT thread#, sequence#, name, ROUND(blocks*block_size/1024/1024) MBytes
FROM v$archived_log
ORDER BY thread#, sequence#;
```

### Archive log creation time

```sql
SELECT sequence#, SUBSTR(name,1,96), creator,
       TO_CHAR(first_time,'DD-MON HH24:MI'),
       TO_CHAR(completion_time,'DD-MON HH24:MI')
FROM v$archived_log
WHERE first_time > SYSDATE-5
ORDER BY 1;
```

**Alternative (last 24 hours):**

```sql
SELECT sequence#, name, creator,
       TO_CHAR(first_time,'DD-MON HH24:MI'),
       TO_CHAR(completion_time,'DD-MON HH24:MI')
FROM v$archived_log
WHERE first_time > SYSDATE-1 AND name IS NOT NULL
ORDER BY 1;
```

### SCN to archivelog file

```sql
SELECT name, thread#, sequence#, first_time, next_time, first_change#, next_change#
FROM v$archived_log
WHERE 9113281811148 BETWEEN first_change# AND next_change#;
```

### Archive generated per month (hourly distribution)

```sql
SET lines 750 pages 9999
SELECT TO_CHAR(first_time,'YYYY-MON-DD') day,
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'00',1,0)),'99') "00",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'01',1,0)),'99') "01",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'02',1,0)),'99') "02",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'03',1,0)),'99') "03",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'04',1,0)),'99') "04",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'05',1,0)),'99') "05",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'06',1,0)),'99') "06",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'07',1,0)),'99') "07",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'08',1,0)),'99') "08",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'09',1,0)),'99') "09",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'10',1,0)),'99') "10",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'11',1,0)),'99') "11",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'12',1,0)),'99') "12",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'13',1,0)),'99') "13",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'14',1,0)),'99') "14",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'15',1,0)),'99') "15",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'16',1,0)),'99') "16",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'17',1,0)),'99') "17",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'18',1,0)),'99') "18",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'19',1,0)),'99') "19",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'20',1,0)),'99') "20",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'21',1,0)),'99') "21",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'22',1,0)),'99') "22",
  TO_CHAR(SUM(DECODE(TO_CHAR(first_time,'HH24'),'23',1,0)),'99') "23"
FROM v$log_history
GROUP BY TO_CHAR(first_time,'YYYY-MON-DD')
ORDER BY 1;
```

### Finding busy archive process

```sql
SELECT * FROM v$archive_processes;
```

You can kill that session; nothing will happen to the DB.

---

## Redo Logs

### Redo logfile info

```sql
COL group# FORMAT 999
COL thread# FORMAT 999
COL member FORMAT a70 wrap
COL status FORMAT a10
COL archived FORMAT a10
COL fsize FORMAT 999 HEADING "Size (MB)"
SELECT l.group#, l.thread#, f.member, l.archived, l.status, (bytes/1024/1024) fsize
FROM v$log l, v$logfile f
WHERE f.group# = l.group#
ORDER BY 1, 2;
```

### Redo log files (summary)

```sql
SELECT group#, thread#, bytes/1024/1024 FROM v$log;
```

**Alternative:**

```sql
SELECT * FROM v$logfile;
```

### Standby redo log files

```sql
SELECT group#, dbid, sequence#, status FROM v$standby_log;
```

### Redo generated per day

```sql
SELECT TRUNC(completion_time) rundate,
       COUNT(*) logswitch,
       ROUND((SUM(blocks*block_size)/1024/1024)) "REDO PER DAY (MB)"
FROM v$archived_log
GROUP BY TRUNC(completion_time)
ORDER BY 1;
```

### How full is the current redo log file?

```sql
SELECT le.leseq "Current log sequence No",
       100*cp.cpodr_bno/le.lesiz "Percent Full",
       cp.cpodr_bno "Current Block No",
       le.lesiz "Size of Log in Blocks"
FROM x$kcccp cp, x$kccle le
WHERE le.leseq = cp.cpodr_seq
  AND bitand(le.leflg,24) = 8;
```

---

## Adding Redo Logs

### Add new redo logfile group

```sql
ALTER DATABASE ADD LOGFILE GROUP 3 '/u01/oracle/ica/log3.ora' SIZE 10M;
```

**Alternative (Windows, multiplexed):**

```sql
ALTER DATABASE ADD LOGFILE GROUP 10
  ('c:\oracle\oradata\whs\redo\redo_05_01.log',
   'd:\oracle\oradata\whs\redo\redo_05_02.log')
  SIZE 100M;
```

### Add members to existing group

```sql
ALTER DATABASE ADD LOGFILE MEMBER '/u01/oracle/ica/log11.ora' TO GROUP 1;
```

**Alternative (multiple groups):**

```sql
ALTER DATABASE ADD LOGFILE MEMBER '/DISK2/log1b.rdo' TO GROUP 1,
  '/DISK2/log2b.rdo' TO GROUP 2;
```

**Alternative (RAC – ASM):**

```sql
ALTER DATABASE ADD LOGFILE MEMBER '+DATAC1' TO GROUP 1;
```

---

## Dropping Redo Logs

### Drop logfile member

```sql
ALTER DATABASE DROP LOGFILE MEMBER '/u01/oracle/ica/log11.ora';
```

### Drop logfile group

```sql
ALTER DATABASE DROP LOGFILE GROUP 3;
```

In disaster scenarios (redo log file deleted from OS), clear the unarchived logfile first, then drop. Check status with `SELECT * FROM v$log;` — if UNUSED or CLEARING, you can drop.

### ORA-01624: log needed for crash recovery

```sql
ALTER SYSTEM CHECKPOINT GLOBAL;
ALTER DATABASE DROP LOGFILE GROUP 1;
```

### ORA-01623: log is current for another instance (RAC)

```sql
SELECT * FROM gv$thread;
ALTER DATABASE DISABLE THREAD 2;
ALTER DATABASE DROP LOGFILE GROUP 3;
ALTER DATABASE ENABLE PUBLIC THREAD 2;
SELECT * FROM gv$thread;
```

---

## Renaming or Relocating Logfiles

```sql
ALTER DATABASE RENAME FILE '/u01/oracle/ica/log1.ora' TO '/u02/oracle/ica/log2.ora';
```

---

## Clearing Redo Logfiles

```sql
ALTER DATABASE CLEAR LOGFILE GROUP 3;
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP 3;
ALTER DATABASE CLEAR UNARCHIVED LOGFILE '/ora10gsoft/10.2.0/oradata/dosa/redo1.log';
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP 1 UNRECOVERABLE DATAFILE;
```

In disaster scenarios (redo log file deleted from OS), clear the unarchived logfile. Check status with `SELECT * FROM v$log;` — if UNUSED or CLEARING, you can drop.

---

Back to [Main Index](README.md)
