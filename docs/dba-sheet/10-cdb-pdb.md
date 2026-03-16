---
layout: default
title: "Container and Pluggable Database Reference"
parent: "DBA Reference Sheet"
nav_order: 10
---

# Container and Pluggable Database Reference

CDB and PDB management for Oracle Multitenant architecture.

---

## Services

### Find services belonging to PDB

```sql
COLUMN name FORMAT A30
SELECT name, pdb FROM v$services ORDER BY name;
```

### PDB name for a service

```bash
export CHANGE_ME=abcd_srv
crsctl status resource -t|grep -iw $CHANGE_ME |grep ora.*.svc|cut -f2 -d. |
while read line ; do
  echo -e "\033[0;31mDatabase Name: \033[0;0m"$line
  echo -e "\033[0;31mNodes: \033[0;0m"`crsctl stat res ora.$line.db -p | grep "GEN_USR_ORA_INST_NAME@SERVERNAME" | cut -f 2 -d'@'`
  ORACLE_HOME=`crsctl stat res ora.$line.db -p | grep -iw ORACLE_HOME | cut -d'=' -f2`
  export ORACLE_HOME
  echo -e "\033[0;31mPDB: \033[0;0m"`$ORACLE_HOME/bin/srvctl config service -d $line -s $CHANGE_ME | grep -i "Pluggable database name" | cut -f2 -d:`
done
unset CHANGE_ME
```

### Find CDB name from PDB name

```bash
ps -ef | grep tns | grep SCAN | awk {'print $9'} |
while read line ; do
  lsnrctl status $line | grep -i pdb_name -A 2
done
```

---

## PDB Operations

### Close and open PDBs

```sql
ALTER PLUGGABLE DATABASE <PDB_NAME> CLOSE IMMEDIATE INSTANCES=ALL;
ALTER PLUGGABLE DATABASE <PDB_NAME> OPEN INSTANCES=ALL;
ALTER PLUGGABLE DATABASE <PDB_NAME> CLOSE INSTANCES=('INST1');
```

### Connect to PDB directly

```bash
# Set SID or use . oraenv to CDB, then:
export TWO_TASK=PDB1
```

---

## Service Registration

### Registering service on CRS

```bash
srvctl add service -d RAC -pdb PDB -s svctest -r RAC1 -a RAC2 -P BASIC
srvctl start service -d RAC -s svctest
```

**Alternative (with TAF):**

```bash
srvctl add service -db RAC12C -service TAFSRV -preferred RAC12C1 -available RAC12C2 \
  -tafpolicy BASIC -policy AUTOMATIC -failovertype SELECT -failovermethod BASIC \
  -failoverretry 5 -pdb DEMOPDB -verbose
srvctl start service -d RAC12C -s TAFSRV
```

### Relocate services

```bash
srvctl status service -d <db_name> -s <service>
srvctl relocate service -d RAC12C -s <service> -oldinst <INST1> -newinst <INST2>
```

---

## MAX_PDB_STORAGE

### Check MAX_PDB_STORAGE property

```sql
SELECT PROPERTY_NAME, PROPERTY_VALUE, DESCRIPTION, CON_ID
FROM cdb_properties
WHERE property_name = 'MAX_PDB_STORAGE';
```

**Alternative (detailed with sizes):**

```sql
COL DB_UNIQUE_NAME FORMAT A10
SET LINES 5000
COL BYTES FORMAT 999,999,999,999,999
COL pdb_name FORMAT A15

SELECT *
FROM (
  SELECT c.db_unique_name, b.name pdb_name,
         TO_NUMBER(a.property_value)/1024/1024/1024 zmax_pdb_storage_GB,
         c.database_role, c.open_mode
  FROM database_properties a, v$pdbs b, v$database c
  WHERE a.property_name ='MAX_PDB_STORAGE'
) pdb_info,
(
  SELECT a.data_size_GB+b.temp_size_GB "dbfile_total_size_GB",
         a.max_size_GB+b.max_size_GB AS dbfile_max_size_GB,
         data_size_GB, temp_size_GB
  FROM (SELECT SUM(bytes)/1024/1024/1024 data_size_GB, SUM(maxbytes)/1024/1024/1024 max_size_GB
        FROM dba_data_files) a,
       (SELECT NVL(SUM(bytes)/1024/1024/1024,0) temp_size_GB, SUM(maxbytes)/1024/1024/1024 max_size_GB
        FROM dba_temp_files) b
) sdf;
```

---

## PDB & CDB Name

### CDB and PDB identification

```sql
SELECT name CDB_NAME,
       (SELECT name FROM v$pdbs) PDB_NAME,
       open_mode,
       database_role,
       (SELECT TO_CHAR(sysdate,'DD-MON-YYYY HH24:MI:SS') FROM dual) "Current_time_db",
       (SELECT INSTANCE_NAME FROM v$instance) INSTANCE_NAME,
       (SELECT HOST_NAME FROM v$instance) HOST_NAME
FROM v$database;
```

---

**Back to [Main Index](README.md)**
