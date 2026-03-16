---
layout: default
title: "ASM Commands Reference"
parent: "DBA Reference Sheet"
nav_order: 4
---

# ASM Commands Reference

ASM disk and diskgroup management, backup, metadata operations, and troubleshooting.

---

## Disk Details

### Disk details

```sql
set lines 250
set pages 9999
column path format a20

SELECT path, group_number group_#, disk_number disk_#, mount_status,
       header_status, state, total_mb, free_mb
FROM v$asm_disk
ORDER BY group_number;
```

### Find available disks to add to diskgroup

```sql
col PATH for a55
SELECT path, group_number group_#, disk_number disk_#, mount_status,
       header_status, state, total_mb, free_mb
FROM v$asm_disk
WHERE header_status='CANDIDATE' OR header_status='FORMER'
ORDER BY 1;
```

### Disk group and disk details

```sql
col PATH for a55
col DG_NAME for a15
col DG_STATE for a10
col FAILGROUP for a20
set lines 750 pages 9999

SELECT dg.name dg_name, dg.state dg_state, dg.type, d.disk_number dsk_no,
       d.path, d.total_mb, d.free_mb, d.mount_status, d.FAILGROUP, d.state
FROM v$asm_diskgroup dg, v$asm_disk d
WHERE dg.group_number = d.group_number
ORDER BY dg_name, dsk_no;
```

### Disk group freespace and mount status

```sql
set pages 40000 lines 120
col NAME for a15
SELECT GROUP_NUMBER DG#, name, ALLOCATION_UNIT_SIZE AU_SZ, STATE,
       TYPE, TOTAL_MB, FREE_MB, OFFLINE_DISKS
FROM v$asm_diskgroup;
```

### Disk group compatibility

```sql
SELECT GROUP_NUMBER, NAME, COMPATIBILITY, DATABASE_COMPATIBILITY
FROM v$asm_diskgroup;
```

### % Used in diskgroups

```sql
COL % FORMAT 99.0
SELECT name, free_mb, total_mb,
       ((total_mb-free_mb)/total_mb)*100 AS "USED %",
       free_mb/total_mb*100 "FREE%"
FROM v$asm_diskgroup
ORDER BY 1;
```

---

## Folder Utilization

### DB folder utilization in MB per diskgroup

```sql
set pagesize 100
column inst format a10 heading 'Inst'
column file_type format a20 heading 'File type'
column mg format 99,999,999
break on inst skip 1
compute sum of mg on inst

SELECT inst, file_type, sum(mg) mg
FROM (
  SELECT substr(full_alias_path,2,instr(full_alias_path,'/',2,1)-2) AS INST,
         file_type, round((szbytes)/1048576) AS MG
  FROM (
    SELECT lower(concat('', sys_connect_by_path(aname, '/'))) full_alias_path,
           system_created, alias_directory, file_type, szbytes
    FROM (
      SELECT b.name gname, a.parent_index pindex, a.name aname,
             a.reference_index rindex, a.system_created, a.alias_directory,
             c.type file_type, c.bytes szbytes
      FROM v$asm_alias a, v$asm_diskgroup b, v$asm_file c
      WHERE a.group_number = b.group_number
        AND a.group_number = c.group_number(+)
        AND a.file_number = c.file_number(+)
        AND a.file_incarnation = c.incarnation(+)
    )
    START WITH (mod(pindex, power(2, 24))) = 0
      AND rindex IN (SELECT a.reference_index FROM v$asm_alias a, v$asm_diskgroup b
                     WHERE a.group_number = b.group_number
                       AND (mod(a.parent_index, power(2, 24))) = 0)
    CONNECT BY prior rindex = pindex
  )
  WHERE szbytes IS NOT NULL
)
GROUP BY inst, file_type;
```

---

## ASM Backup and Metadata

### ASM metadata backup

```bash
asmcmd > md_backup somefilename
```

### Recreate disk SQLs from metadata

```bash
asmcmd > md_backup myasmdiskbackup
asmcmd > md_restore -S mymdscript.sql myasmdiskbackup
```

---

## Create and Drop Diskgroup

### Create diskgroup (single disk)

```sql
CREATE DISKGROUP DATA_TEST EXTERNAL REDUNDANCY
  DISK '/dev/rdsk/c4t60050768018E027998000000000010F6d0s0' SIZE 102400M;
```

### Create diskgroup (multiple disks)

```sql
CREATE DISKGROUP TEST EXTERNAL REDUNDANCY
  DISK '/dev/rdsk/c5t600507680181050F20000000000007D4d0s0' SIZE 102400 M
  DISK '/dev/rdsk/c5t600507680181050F20000000000007D5d0s0' SIZE 102400 M
  DISK '/dev/rdsk/c5t600507680181050F20000000000007D6d0s0' SIZE 102400 M;
```

### Drop diskgroup

```sql
DROP DISKGROUP TEST INCLUDING CONTENTS;
```

---

## Add and Resize Disks

### Add a disk

```sql
ALTER DISKGROUP DATA01 ADD DISK '/dev/oracleasm/disks/ASM_DISK01' SIZE 1000 M REBALANCE POWER 10;
SELECT * FROM v$asm_operation;
```

### Add disks to diskgroup

```sql
ALTER DISKGROUP DATA_DBNAME ADD DISK '/dev/rdsk/c6t600507680180854860000000000000B2d0s0'
  SIZE 102400 M REBALANCE POWER 6;
```

### Resize disks

```sql
ALTER DISKGROUP DATA RESIZE ALL SIZE 1048288 M;
```

**Alternative (single disk):** `ALTER DISKGROUP DATA RESIZE DISK DATA_0004 SIZE 1048288 M;`

---

## File Operations

### Copy ASM file from local to remote ASM instance

```bash
asmcmd
cp +DATA/orcl/datafile/tbsjfv.256.123456789 sys@mydb.+ASM2:+D2/jfv/tbsjfv.dbf
```

### Remove all files for a database in ASM

```sql
SELECT 'ALTER DISKGROUP '||gname||' DROP FILE '''||full_path||''';' gsql
FROM (
  SELECT CONCAT('+'||gname, SYS_CONNECT_BY_PATH(aname,'/')) full_path, gname
  FROM (
    SELECT g.name gname, a.parent_index pindex, a.name aname,
           a.reference_index rindex, a.ALIAS_DIRECTORY adir
    FROM v$asm_alias a, v$asm_diskgroup g
    WHERE a.group_number = g.group_number
  )
  WHERE adir='N'
  START WITH (MOD(pindex, POWER(2, 24))) = 0
  CONNECT BY PRIOR rindex = pindex
)
WHERE full_path LIKE UPPER('%&database%');
```

### Remove all archivelog files from ASM

```sql
SELECT 'alter diskgroup '||dg.name||' drop file ''+'||dg.name||''||SYS_CONNECT_BY_PATH(al.name,'/')||''';'
FROM v$asm_alias al, v$asm_file fi, v$asm_diskgroup dg
WHERE al.file_number = fi.file_number(+)
  AND al.group_number = dg.group_number
  AND fi.type = 'ARCHIVELOG'
START WITH alias_index = 0
CONNECT BY PRIOR al.reference_index = al.parent_index;
```

### Identify files in ASM not known to DB

```sql
SELECT concat('+'||gname, sys_connect_by_path(aname, '/')) full_alias_path,
       system_created, alias_directory, file_type
FROM (
  SELECT b.name gname, a.parent_index pindex, a.name aname,
         a.reference_index rindex, a.system_created, a.alias_directory,
         c.type file_type
  FROM v$asm_alias a, v$asm_diskgroup b, v$asm_file c
  WHERE a.group_number = b.group_number
    AND a.group_number = c.group_number(+)
    AND a.file_number = c.file_number(+)
    AND a.file_incarnation = c.incarnation(+)
)
WHERE alias_directory = 'N'
START WITH (mod(pindex, power(2, 24))) = 0
  AND rindex IN (SELECT a.reference_index FROM v$asm_alias a, v$asm_diskgroup b
                 WHERE a.group_number = b.group_number
                   AND (mod(a.parent_index, power(2, 24))) = 0
                   AND a.name = '&DATABASENAME')
CONNECT BY prior rindex = pindex;
```

---

## Blocking and Connections

### Blocker in ASM

```sql
set linesize 200
set pagesize 1000
SELECT b.inst_id||'/'||b.sid blocker,
       w.inst_id||'/'||w.sid waiter,
       b.type, b.id1, b.id2, b.lmode, w.request
FROM gv$lock b,
     (SELECT inst_id, sid, type, id1, id2, lmode, request
      FROM gv$lock WHERE request > 0) w
WHERE b.lmode > 0
  AND (b.id1 = w.id1 AND b.id2 = w.id2 AND b.type = w.type)
ORDER BY b.inst_id, b.sid;
```

### Kill all local connections to ASM

```bash
ps -eaf | grep ASM | grep beq | awk '{print "kill -9 " $2}'
```

---

## KFED and Low-Level Operations

### Formatting ASM disk header

```bash
DD if=/dev/zero skip=25 bs=4k count=2560 of=$OCRLOC
```

### Detecting disks using kfod

```bash
export ORACLE_HOME=/path/to/oracle/home
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
./kfod.bin verbose=true disks=all status=true op=disks
```

### Check diskgroup using kfod

```bash
kfod ds=true disks=all
```

**Alternative:** `kfod asm_diskstring='ORCL:*' disk=all`

### Disk size (kfed)

```bash
kfed read /dev/rdsk/c0d17s0 | egrep "dskname|dsksize"
```

### Check disk contents

```bash
strings /dev/rdsk/c0d17s0 | head -10
```

### Repair disk corruption

```bash
kfed repair /dev/rdsk/c0d17s0
```

### Check backup block in disk

```bash
kfed read /dev/rdsk/c6t600507680181050F2000000000000A61d0s0 aun=1 blkn=254 | grep KFBTYP
```

### Read disk using od

```bash
od -c -N 128 /dev/rdsk/c6t60050768018E02799800000000000FA2d0s0
```

### Master node in CRS

```bash
cd /opt/oracle/product/crs/log/wpsun564/cssd
cat ocssd.log | grep -i 'master node' | tail -1
```

### Show dismounted diskgroups

```bash
ASMCMD> lsdg --discovery
```

---

## Troubleshooting

### ASM disk I/O timeout issues

```sql
ALTER SYSTEM SET "_asm_hbeatiowait"=200 SCOPE=SPFILE SID='*';
```

Then bounce the ASM instance. See Doc ID 1581684.1.

### Password copy for 12c (Data Guard)

```bash
srvctl config database -db PRODDB | grep -i pass
. oraenv +ASM1
asmcmd --privilege sysdba pwcopy +DATA1/PRODDB/orapwPRODDB /tmp/orapwPRODDB
scp /tmp/orapwPRODDB userid@server.domain.com:/tmp

# On standby:
. oraenv +ASM1
asmcmd --privilege sysdba pwcopy --dbuniquename STBYDB /tmp/orapwPRODDB +DATA1/STBYDB/PASSWORD/orapwSTBYDB
. oraenv STBYDB
srvctl modify database -db STBYDB -pwfile +DATA1/STBYDB/PASSWORD/pwdSTBYDB.3087.999923145
```

### ACFS file system not mounted

```bash
ASMCMD> volinfo --all
/opt/grid/12.1.0.2.PSUJAN2017/bin/acfsload start
```

If kernel fails: stop CRS, run `acfsroot install`, start CRS, then `srvctl start filesystem -d /dev/asm/acf-somename-123`

### ASMCMD privileged logins

```bash
asmcmd --privilege sysdba -p
```

---

## ASMLIB

### ASMLIB device to disk mapping

```bash
#!/bin/bash
for asmlibdisk in `ls /dev/oracleasm/disks/*`
do
  echo "ASMLIB disk name: $asmlibdisk"
  asmdisk=`kfed read $asmlibdisk | grep dskname | tr -s ' ' | cut -f2 -d' '`
  echo "ASM disk name: $asmdisk"
  majorminor=`ls -l $asmlibdisk | tr -s ' ' | cut -f5,6 -d' '`
  device=`ls -l /dev | tr -s ' ' | grep "$majorminor" | cut -f10 -d' '`
  echo "Device path: /dev/$device"
done
```

### OracleASM diskgroup creation

```bash
/etc/init.d/oracleasm createdisk SYSTEMDG2 /dev/mapper/oradisk01p1
/etc/init.d/oracleasm createdisk DISK01 /dev/mapper/oradisk03p1
```

```sql
CREATE DISKGROUP BKUP EXTERNAL REDUNDANCY DISK 'ORCL:DISK01' SIZE 102398 M;
```

### ASM module / package

```bash
lsmod | grep -i oracle
rpm -qa | grep oracleasm
```

### ASM disks list

```bash
ls -ltr /dev/oracleasm/disks | sort
```

### Discovery

```bash
/etc/init.d/oracleasm scandisks
/etc/init.d/oracleasm listdisks
```

### Config files

```bash
cat /etc/sysconfig/oracleasm
```

### Reload ASM (module failure)

```bash
/usr/sbin/oracleasm init
```

**Alternative:** `/etc/init.d/oracleasm disable` then `/etc/init.d/oracleasm enable`

---

Back to [Main Index](README.md)
