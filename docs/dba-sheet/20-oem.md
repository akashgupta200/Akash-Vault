---
layout: default
title: "OEM (Enterprise Manager)"
parent: "DBA Reference Sheet"
nav_order: 20
---

# OEM (Enterprise Manager)

Oracle Enterprise Manager repository, targets, blackouts, credentials, and emctl commands.

---

## OEM Repository

### OEM repository details

```bash
ps -ef | grep oms
./emctl config oms -list_repos_details
```

---

## Targets

### Target names

```sql
SELECT * FROM sysman.mgmt$db_dbninstanceinfo;
```

### List hostnames

```sql
SELECT DISTINCT(mgmt$target.host_name)
FROM mgmt$target, mgmt$target_properties
WHERE mgmt$target.target_name = mgmt$target_properties.target_name
  AND mgmt$target.target_type = mgmt$target_properties.target_type
  AND mgmt$target_properties.property_name IN ('CPUCount','DBVersion')
GROUP BY mgmt$target.host_name, mgmt$target_properties.property_name,
         mgmt$target_properties.property_value
ORDER BY mgmt$target.host_name;
```

### Standalone databases (not part of RAC)

```sql
SELECT * FROM sysman.mgmt$db_dbninstanceinfo
WHERE target_type = 'oracle_database'
  AND database_name IN (
    SELECT database_name FROM sysman.mgmt$db_dbninstanceinfo
    WHERE target_type = 'oracle_database'
    HAVING database_name NOT IN (
      SELECT database_name FROM sysman.mgmt$db_dbninstanceinfo
      WHERE target_type = 'rac_database'
    )
    GROUP BY database_name
  )
ORDER BY database_name;
```

### Get target names (RAC)

```sql
SELECT target_name FROM sysman.mgmt$db_dbninstanceinfo
WHERE target_type = 'rac_database';
```

### Get target names (standalone)

```sql
SELECT target_name FROM sysman.mgmt$db_dbninstanceinfo
WHERE target_type = 'oracle_database'
  AND database_name IN (
    SELECT database_name FROM sysman.mgmt$db_dbninstanceinfo
    WHERE target_type = 'oracle_database'
    HAVING database_name NOT IN (
      SELECT database_name FROM sysman.mgmt$db_dbninstanceinfo
      WHERE target_type = 'rac_database'
    )
    GROUP BY database_name
  )
ORDER BY database_name;
```

### Find CLUSTERNAME from hostname

```sql
COL target_name FOR a30
COL host_name FOR a60
COL property_value FOR a30
SELECT a.target_name, a.host_name, b.property_value
FROM sysman.mgmt$target a, sysman.mgmt$target_properties b
WHERE a.target_name = b.target_name
  AND a.target_type = 'rac_database'
  AND b.property_name IN ('ClusterName')
  AND a.host_name LIKE lower('%&HOST_NAME%')
ORDER BY a.host_name;
```

---

## Alerts

### Clear OEM alerts

```sql
SELECT t.target_name, t.target_type, collection_timestamp, message,
  'exec sysman.em_severity.delete_current_severity(''' || t.target_guid || ''',''' ||
  metric_guid || ''',''' || key_value || ''')' em_severity
FROM sysman.mgmt_targets t
INNER JOIN sysman.mgmt_current_severity s ON t.target_guid = s.target_guid;

COMMIT;
```

---

## Credentials

### Changing DBSNMP password for targets

```bash
emcli argfile Chng_DB_mon_passwd1
```

### Update password (RAC)

```bash
emcli update_password -target_type=rac_database -target_name=d1dbname -credential_type=DBCreds -key_column=DBUserName:dbsnmp -non_key_column=DBPassword:p@ssw0rdold:m0nit0r
```

### Update password (non-RAC)

```bash
emcli update_password -target_type=oracle_database -target_name=db_name -credential_type=DBCreds -key_column=DBUserName:dbsnmp -non_key_column=DBPassword:p@ssw0rdold:m0nit0r
```

### Set monitoring credential (RAC and non-RAC)

```bash
emcli set_monitoring_credential -target_type=rac_database -target_name=db_name -set_name="DBCredsMonitoring" -cred_type="DBCreds" -attributes="DBUserName:dbsnmp;DBPassword:m0nit0r;DBRole:NORMAL"
emcli sync
```

---

## Blackouts

### Start blackout

```bash
emctl start blackout test DB1:oracle_database -d 07:00
```

---

## emctl Commands

### List targets

```bash
emctl config agent listtargets
```

### Clearing unknown status of targets

```bash
cd /u01/app/oracle/agent/agent12c/bin
./emctl stop agent
./emctl clearstate agent
./emctl start agent
./emctl upload agent
```

---

## Reference Links

- [Querying Grid Repository tables](http://www.oracledbasupport.co.uk/querying-grid-repository-tables/)
- [Retrieving SID/port from Grid Control repository](http://askdba.org/weblog/2011/01/retrieving-database-sidport-information-from-grid-control-repository/)
- [emctl commands](http://satya-dba.blogspot.com/2010/01/emctl-commands.html)
- [EMCLI command OEM 12c](http://dbaclass.com/article/emcli-command-oem-12c/)
- [Querying the Oracle Management Repository](https://blog.dbi-services.com/querying-the-oracle-management-repository/)

---

Back to [Main Index](README.md)
