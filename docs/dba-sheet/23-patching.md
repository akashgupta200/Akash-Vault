---
layout: default
title: "Patching Reference"
parent: "DBA Reference Sheet"
nav_order: 23
---

# Patching Reference

OPatch, datapatch, PSU application, PDB violations, and patch validation for Oracle DBAs.

---

## Post-Patching Activity

### Run datapatch After Patching

```bash
cd /opt/oracle/product/12.1.0.2.PSUAPR2017/OPatch
./datapatch -verbose
```

---

## PDB Violations

### Check PDB Plug-In Violations

```sql
col CAUSE for a15
col MESSAGE for a30
select name,cause,message, STATUS from pdb_plug_in_violations;
```

---

## Patch Information

### Applied Patch Versions (registry$sqlpatch)

```sql
select PATCH_ID, ACTION, ACTION_TIME, DESCRIPTION, STATUS, LOGFILE from registry$sqlpatch;
```

### Patch Validation

```bash
datapatch -db db_name -prereq
```

```sql
select PATCH_ID,PATCH_UID,VERSION,ACTION,ACTION_TIME,STATUS,DESCRIPTION,BUNDLE_SERIES,BUNDLE_ID,BUNDLE_DATA from dba_registry_sqlpatch;
```

Status should be SUCCESS.

---

## Patch Check (OPatch)

### Database PSUs

```bash
opatch lsinventory -bugs_fixed | egrep -i 'PSU|DATABASE PATCH SET UPDATE'
```

### CRS PSUs

```bash
opatch lsinventory -bugs_fixed | grep -i 'TRACKING BUG' | grep -i 'PSU'
```

### GI (Grid Infrastructure) PSUs

```bash
opatch lsinventory -bugs_fixed | grep -i 'GRID INFRASTRUCTURE PATCH SET UPDATE'
```

### Enterprise Manager Agent PSUs

```bash
opatch lsinventory -bugs_fixed | grep -i 'ENTERPRISE MANAGER' | grep -i 'AGENT'
```

---

## Component Status

### Database Components Status

```sql
col COMP_ID for a25
col COMP_NAME for a60
set lines 700
select COMP_ID,COMP_NAME,VERSION,STATUS from dba_registry;
```

---

## Invalid Components

### Invalid Components Error Messages

```sql
set lines 750 pages 9999
col text for a80
SELECT e.owner, e.name, TO_CHAR(e.line) || '/' || TO_CHAR(e.position) "POSITION", e.text
   FROM dba_errors e
   ORDER BY e.owner, e.name, e.sequence;
```

### Fix Invalid XDB Components After Patching

```sql
delete from obj$ where name in (select OBJECT_NAME from dba_objects where status ='INVALID' and owner ='XDB');
commit;
set serveroutput on
execute sys.dbms_regxdb.validatexdb;
select comp_name, version, status from dba_registry where comp_id='XDB';
```

---

## Automated Patching

### Auto Patching Script (Datapatch Apply)

```bash
for i in `crsstat.sh |grep .db| grep -v svc |grep -v .vip|grep -v .lsnr |grep -v .mgmt | grep -v OFFLINE | grep "Open,STABLE"|cut -d"." -f2`

do
GRID_ENV=`ps -ef|grep pmon|awk '{print $NF}'|grep asm|cut -d_ -f3`
export GRID_ENV
. oraenv $GRID_ENV
NEW_HOME=`srvctl config database -v|grep $i|awk '{print $2}'`
export NEW_HOME
. /opt/oracle/local/bin/oraenv $i
for j in `srvctl status database -d $i | grep "running" | grep -v "not running" |awk '{print $2}' | awk 'NR==1'`
do
node=`srvctl status database -d $i |grep $j| awk '{print $NF}'`
connection="nohup `ssh -q $node "$(typeset -f Datapatch_apply); Datapatch_apply $j "` &"
done
done
```

---

[Back to Main Index](README.md)
