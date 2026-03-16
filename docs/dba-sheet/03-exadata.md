---
layout: default
title: "Exadata Commands Reference"
parent: "DBA Reference Sheet"
nav_order: 3
---

# Exadata Commands Reference

Exadata-specific commands for cell management, offloading metrics, resource manager, and DCLI operations.

---

## Exadata Identification

### Identify Exadata box type

```bash
grep -i MACHINETYPES /opt/oracle.SupportTools/onecommand/databasemachine.xml
```

HP = High Performance, HC = High Capacity

### Exadata serial numbers

```bash
/opt/oracle.SupportTools/CheckHWnFWProfile -S
```

Run as root.

---

## Cell Information

### Cell versions and configuration

```sql
set lines 750 pages 9999
col CV_CELLNAME for a30
SELECT
  cellname cv_cellname,
  CAST(extract(xmltype(confval), '/cli-output/cell/releaseVersion/text()') AS VARCHAR2(20)) cv_cellVersion,
  CAST(extract(xmltype(confval), '/cli-output/cell/flashCacheMode/text()') AS VARCHAR2(20)) cv_flashcachemode,
  CAST(extract(xmltype(confval), '/cli-output/cell/cpuCount/text()') AS VARCHAR2(10)) cpu_count,
  CAST(extract(xmltype(confval), '/cli-output/cell/upTime/text()') AS VARCHAR2(20)) uptime,
  CAST(extract(xmltype(confval), '/cli-output/cell/kernelVersion/text()') AS VARCHAR2(30)) kernel_version,
  CAST(extract(xmltype(confval), '/cli-output/cell/makeModel/text()') AS VARCHAR2(50)) make_model
FROM v$cell_config
WHERE conftype = 'CELL'
ORDER BY cv_cellname;
```

---

## Offloading Metrics

### Offloading saved (storage index and smart scan)

```sql
SELECT name, value
FROM v$statname
JOIN v$mystat USING (statistic#)
WHERE name IN (
  'cell physical IO bytes eligible for predicate offload',
  'cell physical IO interconnect bytes returned by smart scan',
  'cell physical IO bytes saved by storage index'
);
```

### % Saved from Exadata storage

```sql
SELECT inst_id, sql_id,
  decode(IO_CELL_OFFLOAD_ELIGIBLE_BYTES,0,'No','Yes') Offloaded,
  decode(IO_CELL_OFFLOAD_ELIGIBLE_BYTES,0,0,
    100*(IO_CELL_OFFLOAD_ELIGIBLE_BYTES - IO_INTERCONNECT_BYTES) / IO_CELL_OFFLOAD_ELIGIBLE_BYTES) "IO_SAVED_%"
FROM gv$sql
WHERE sql_id='&sql_id';
```

---

## Resource Manager

### Exadata resource manager plans

See Oracle Doc ID 1338988.1: Scripts and Tips for Monitoring CPU Resource Manager

### Session waiting for CPU (Resource Manager)

```sql
SELECT s.sid sess_id, g.name consumer_group,
  s.state, s.consumed_cpu_time cpu_time, s.cpu_wait_time, s.queued_time,
  (s.CURRENT_SMALL_READ_MEGABYTES+s.CURRENT_LARGE_READ_MEGABYTES) read_MB,
  (s.CURRENT_SMALL_WRITE_MEGABYTES+s.CURRENT_LARGE_WRITE_MEGABYTES) write_mb
FROM v$rsrc_session_info s, v$rsrc_consumer_group g
WHERE s.current_consumer_group_id = g.id;
```

---

## DCLI

### Execute command on all nodes via DCLI

```bash
dcli -l root -g dbs_group "ls -ltr / | grep -i acfs_u01"
```

---

## References

- Monitoring Exadata Smart Scan: http://www.centroid.com/blog/monitoring-exadata-smart-scan

---

Back to [Main Index](README.md)
