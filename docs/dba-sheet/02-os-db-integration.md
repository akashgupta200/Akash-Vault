---
layout: default
title: "OS-Database Integration"
parent: "DBA Reference Sheet"
nav_order: 2
---

# OS-Database Integration

Queries and commands for OS-DB integration: host CPU from database, trace cleanup, ORA errors, and related utilities.

---

## Host CPU from Database

### Host CPU utilization report from database

```sql
col waitday for a6
col "00" for 99.9
col "01" for 99.9
col "02" for 99.9
col "03" for 99.9
col "04" for 99.9
col "05" for 99.9
col "06" for 99.9
col "07" for 99.9
col "08" for 99.9
col "09" for 99.9
col "10" for 99.9
col "11" for 99.9
col "12" for 99.9
col "13" for 99.9
col "14" for 99.9
col "15" for 99.9
col "16" for 99.9
col "17" for 99.9
col "18" for 99.9
col "19" for 99.9
col "20" for 99.9
col "21" for 99.9
col "22" for 99.9
col "23" for 99.9
set linesize 200
set pagesize 50

SELECT to_char(BEGIN_TIME,'DD-Mon') WaitDay,
max(decode(to_char(END_TIME,'HH24'),'00',AVERAGE,0)) "00",
max(decode(to_char(END_TIME,'HH24'),'01',AVERAGE,0)) "01",
max(decode(to_char(END_TIME,'HH24'),'02',AVERAGE,0)) "02",
max(decode(to_char(END_TIME,'HH24'),'03',AVERAGE,0)) "03",
max(decode(to_char(END_TIME,'HH24'),'04',AVERAGE,0)) "04",
max(decode(to_char(END_TIME,'HH24'),'05',AVERAGE,0)) "05",
max(decode(to_char(END_TIME,'HH24'),'06',AVERAGE,0)) "06",
max(decode(to_char(END_TIME,'HH24'),'07',AVERAGE,0)) "07",
max(decode(to_char(END_TIME,'HH24'),'08',AVERAGE,0)) "08",
max(decode(to_char(END_TIME,'HH24'),'09',AVERAGE,0)) "09",
max(decode(to_char(END_TIME,'HH24'),'10',AVERAGE,0)) "10",
max(decode(to_char(END_TIME,'HH24'),'11',AVERAGE,0)) "11",
max(decode(to_char(END_TIME,'HH24'),'12',AVERAGE,0)) "12",
max(decode(to_char(END_TIME,'HH24'),'13',AVERAGE,0)) "13",
max(decode(to_char(END_TIME,'HH24'),'14',AVERAGE,0)) "14",
max(decode(to_char(END_TIME,'HH24'),'15',AVERAGE,0)) "15",
max(decode(to_char(END_TIME,'HH24'),'16',AVERAGE,0)) "16",
max(decode(to_char(END_TIME,'HH24'),'17',AVERAGE,0)) "17",
max(decode(to_char(END_TIME,'HH24'),'18',AVERAGE,0)) "18",
max(decode(to_char(END_TIME,'HH24'),'19',AVERAGE,0)) "19",
max(decode(to_char(END_TIME,'HH24'),'20',AVERAGE,0)) "20",
max(decode(to_char(END_TIME,'HH24'),'21',AVERAGE,0)) "21",
max(decode(to_char(END_TIME,'HH24'),'22',AVERAGE,0)) "22",
max(decode(to_char(END_TIME,'HH24'),'23',AVERAGE,0)) "23"
FROM DBA_HIST_SYSMETRIC_SUMMARY
WHERE METRIC_NAME LIKE 'Host CPU Utilization%' AND INSTANCE_NUMBER = &inst_num
GROUP BY to_char(BEGIN_TIME,'DD-Mon');
```

---

## Trace Cleanup

### Delete old trace files

```bash
find /opt/oracle/diag/rdbms/dbname/instance_name/trace -name "*.trc" -mtime +3 -exec rm {} \;
find /opt/oracle/diag/rdbms/dbname/instance_name/trace -name "*.trm" -mtime +3 -exec rm {} \;
find /opt/oracle/diag/rdbms/dbname/instance_name/trace -name "*.gz" -mtime +3 -exec rm {} \;
find /opt/oracle/diag/tnslsnr/hostname/listener_dbname/alert -name "log*xml" -mtime +3 -exec rm {} \;
```

**Alternative:** `find . -name "*.aud" -mtime +7 -exec rm -f {} \;`

### rm -rf *.aud — arg list too long

```bash
find . -name '*.aud' -exec rm {} +
```

---

## ORA Errors and Issues

### ORA-09925: Unable to create audit trail file (No space left on device)

Check inode usage. If `df -h -i` shows 100% IUse%, delete old `*.aud` files from that location.

```bash
df -kh /u01
df -h -i /u01
```

### Remove MGMT audit files (files starting with hyphen)

```bash
rm -- *.aud
```

### Login to MGMT directory (directory starting with hyphen)

```bash
cd ./-MGMTDB
cd trace
```

### ORA-27102: Out of memory (Linux)

```bash
free -m
```

If cache memory is more than 5GB, ask Unix admin to clear the memory.

### Memory allocated to Oracle user (Solaris)

```bash
prtconf | grep Mem
id -p
prctl -n project.max-shm-memory -i project 100
```

---

## Tracing

### Tracing Oracle process at OS level

```bash
strace -fF -v -p 16311
```

### Tracing SQLPLUS at OS level

```bash
strace /oracle/product/10.2.0.1/bin/sqlplus -V 2>&1 | less
```

---

## Others

### Asking for oratab while logging in

```bash
grep "^[^#]" /var/opt/oracle/oratab | awk -F: '{printf("\t%-6s\t%-30s\n",$1,$2)}'
. oraenv
```

### List databases running on server

```bash
ps -ef | grep pmon | awk '{print $8}' | grep -v "grep" | grep -v "ASM" | cut -d"_" -f3
```

**Alternative:** `ps -eaf | grep _pmon_ | grep -v grep | grep -v + | awk '{print $NF}' | cut -c 10-` or `ps -eo comm | grep -v grep | grep pmon | sed s/ora_pmon_//`

### Run SQL commands in terminal

```bash
echo 'select count(*) from tab;' | sqlplus / as sysdba
```

### Stopping processes for a database

```bash
ps -ef | grep ora_ | grep orcl2 | awk '{print $2}' | while read PID
do
  kill -STOP $PID
done
```

### Killing clusterware processes

```bash
ps -ef | grep <keyword> | grep -v grep | awk '{print $2}' | xargs kill -9
```

### Killable Oracle processes

ARCn, CJQn, Dnnn, DIA0, DIAG, FDBA, Jnnn, PING, Qnnn, QMNC, RECO, Snnn, SMCO, Wnnn

### Instance-critical (non-killable) processes

ACMS, CKPT, DBRM, DBWn, LGWR, LMDn, LMON, MMAN, PMON, PSPn, RMSn, RVWR, SMON, VKTM, MMNL, MMON

### Alias for sqlplus

```bash
alias s="sqlplus / as sysdba"
```

### Freeze process (skill command)

```bash
skill -STOP 1
```

### Continue frozen process

```bash
skill -CONT 16514
```

### Stop all Oracle processes

```bash
skill -STOP oracle
```

### Stop all RMAN processes

```bash
skill -STOP rman
```

---

## Scripts

### Get CPU/Memory free usage from database

```sql
SELECT
  STAT_NAME,
  DECODE(STAT_NAME,'PHYSICAL_MEMORY_BYTES',(ROUND(VALUE/1024/1024/1024,2)) || ' GB',
         'FREE_MEMORY_BYTES',(ROUND(VALUE/1024/1024/1024,2)) || ' GB',VALUE) VALUE
FROM v$osstat
WHERE stat_name IN ('FREE_MEMORY_BYTES', 'LOAD', 'NUM_CPUS', 'NUM_CPU_CORES',
     'NUM_CPU_SOCKETS', 'PHYSICAL_MEMORY_BYTES');
```

### Extract DB alias names from tnsnames.ora

```bash
sort -u $TNS_ADMIN/tnsnames.ora | grep -v "^ " | grep -v "^#" | grep -v "^(" | grep -v "^)" | sed "s/ =//g" | grep -v "^$"
```

### OS-level scripting (loop through databases)

```bash
for i in `ps -ef | grep pmon | awk '{print $8}' | grep -v "grep" | grep -v "ASM" | cut -d"_" -f3`
do
  export ORACLE_SID=$i
  sqlplus -s /nolog <<EOF
  set head off echo off
  conn / as sysdba
  SELECT comp_name, version, status FROM dba_registry WHERE status='INVALID';
  exit
EOF
done
```

### Shell script to execute multiple databases (12c)

```bash
#!/bin/ksh
rm -rf log.txt
for i in `cat a.txt`
do
  echo ' ------'$i'----- ' >> /tmp/log.txt
  sqlplus -L -S << EOF >> /tmp/log.txt
  / @$i
  set feedback off
  set lines 750 pages 0
  col value for a20
  select (select name from v$database) dbname, (select NAME from v$pdbs where con_id=(select con_id from v$pdbs where rownum < 2)) con,
         i.host_name,p.inst_id,p.name,p.value
  from gv$parameter p, gv$instance i
  where p.name='global_names' and p.inst_id=i.inst_id
  order by p.inst_id;
  exit;
EOF
done
```

### CMD script for logging into multiple databases

```batch
echo off
cls
for /f "tokens=1-2 delims= " %%b in (1.txt) do (
   echo.***********************************************
   echo.Connect to %%b
   echo.***********************************************
   sqlplus -L "sys/pwd@%%b as sysdba" @C:\dbname.sql
)
echo.
echo.DONE
echo.Press any key to exit.
pause >nul
```

### Extract CRS_HOME from oratab

```bash
export ORATAB="/var/opt/oracle/oratab"
export ORA_CRS_HOME=`cat /etc/oratab | grep -i asm | awk -F: '{print $2}'`
```

### Extract host/port/servicename from TNSPING

```bash
tnsping gpo_core_u | awk '{FS="[()]+";for(i=1;i<=NF;i++) if($i ~ /(HOST|PORT|SERVICE_NAME)/) print $i}'
```

### Execute script after delay (from SQL*Plus)

```sql
host sleep 4.5h
start run/sqltxtract.sql 51x6yr9ym5hdc sqltxplain
```

### Server maintenance precheck script

Create `osprecheck.sh` and run: `./osprecheck.sh | tee presnap.txt`

### Server maintenance postcheck script

Create `ospostcheck.sh` and run: `./ospostcheck.sh | tee postsnap.txt`

Compare: `diff -y -W 250 presnap.txt postsnap.txt | more`

### Force re-register with remote_listener (workaround for bug 13066936)

```bash
#!/bin/ksh
for INST in `ps -aef | grep 'ora_pmon' | egrep -v '(grep|sed)' | sed 's/^.*ora_pmon_//'`
do
  echo . Reregistering $INST with remote_listener
  export ORAENV_ASK=NO
  . oraenv $INST
  sqlplus -SL "/ as sysdba" <<-EOF >/dev/null
    col remote_val new_value remote_val
    select value remote_val from v$parameter where name='remote_listener';
    alter system set remote_listener='';
    alter system register;
    alter system set remote_listener='&remote_val';
    alter system register;
EOF
done
```

### Profile for OS (sample .profile)

```bash
export ORACLE_HOME=/opt/oracle/product/11.1.0/db7
PATH=/usr/bin:/etc:/usr/sbin:/usr/ucb:$HOME/bin:/usr/bin/X11:/sbin:.:/usr/local/bin:${ORACLE_HOME}:${ORACLE_HOME}/bin:${ORACLE_HOME}/OPatch:/oracle/dba/bin
export PATH
export HISTFILE=$HOME/.histdir/$(tty|sed 's-/-_-g')
export DBA_INIT_INI=/oracle/dba/funcs/dba_init.ini
export NMONAIX=5.2.0.0
export EDITOR=vi
export FPATH=/oracle/dba/funcs
export TNS_ADMIN=/var/opt/oracle
export ORACLE_BASE=/opt/oracle
export ORACLE_ASK=YES
export ORACLE_OWNER=oracle
export NLS_LANG=AMERICAN_AMERICA.UTF8
export NLS_DATE_FORMAT='YYYY/MM/DD HH24:MI'
umask 002
export HOST=$(uname -n)
export PWD=`pwd`
export PS1='$ORACLE_SID@$HOST: $PWD> '
```

---

## Network Wait Event

### Network wait event by database

```sql
col c1 heading 'end|time' format a10
col c2 heading 'wait|class' format a20
col c3 heading 'time|waited' format 999,999,999,999
break on c1 skip 2

SELECT
   trunc(end_interval_time) c1,
   wait_class c2,
   sum(time_waited) c3
FROM dba_hist_service_wait_class
JOIN dba_hist_snapshot USING(snap_id)
WHERE wait_class = 'Network'
GROUP BY trunc(end_interval_time), wait_class
ORDER BY trunc(end_interval_time), c3 DESC;
```

---

Back to [Main Index](README.md)
