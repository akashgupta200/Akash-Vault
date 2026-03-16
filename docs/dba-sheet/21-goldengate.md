---
layout: default
title: "GoldenGate Reference"
parent: "DBA Reference Sheet"
nav_order: 21
---

# GoldenGate Reference

Oracle GoldenGate monitoring, troubleshooting, agctl commands, lag reporting, and process management for DBAs.

---

## Troubleshooting

### Troubleshooting ABENDED Process

Reference: [Approach to troubleshoot an ABENDED OGG process](http://www.oracle-scn.com/approach-to-troubleshoot-an-abended-ogg-process/)

### Error: No Data Found

Check the table name. Add `HANDLECOLLISIONS` and `NOHANDLECOLLISIONS` before and after the MAP statement in the parameter file and restart the EXTRACT/REPLICAT.

Once the LAG is zero, stop the replicat and remove `HANDLECOLLISIONS` and `NOHANDLECOLLISIONS` from the parameter file. Then start the replicat.

---

## Process Status

### Check if GoldenGate Manager is Running

```bash
ps -ef | grep ./mgr
```

### GGSCI Status Command

```bash
send EWDBNAMEP1, status
```

### View Process Details

```bash
info REPNAME detail
```

---

## agctl Commands

### Locate agctl Binary

```bash
locate agctl
```

### Start GoldenGate via agctl

```bash
/opt/oracle/product/emagent/gg/install/xag/bin/agctl start goldengate ggate_prod --node server2
```

### Check agctl Status

```bash
cd /opt/oracle/product/emagent/gg/install/xag/bin
./agctl status goldengate ggate_<uat/prod/bcp>
```

### Relocate GoldenGate to Another Node

```bash
cd /opt/oracle/product/emagent/gg/install/xag/bin/
./agctl relocate goldengate ggate_prod --node secondnode
```

### agctl Configuration

```bash
agctl config goldengate ggate_uat
```

---

## Lag and Reports

### Get Lag Report (GGSCI)

```
GGSCI (server1) 3> send RW12345PB1, getlag

Sending GETLAG request to REPLICAT RWDBNAME1 ...
Last record lag 1,218 seconds.
```

### Identify Lag from PDB (Target Side)

Execute on the target database:

```sql
select incoming_extract incoming_extract, round(
       (extract (second from a)
     + extract (minute from a) * 60
     + extract (hour   from a) * 60 * 60
     + extract (day    from a) * 60 * 60 * 24)/60,0) as heartbeat_lag_secs
from (
  select remote_database||'_'||incoming_extract incoming_extract
  , (systimestamp - EXTRACT(TIMEZONE_HOUR FROM SYSTIMESTAMP)/24 + EXTRACT(TIMEZONE_MINUTE FROM SYSTIMESTAMP)/60/24
  - heartbeat_timestamp) +
    (systimestamp - EXTRACT(TIMEZONE_HOUR FROM SYSTIMESTAMP)/24 + EXTRACT(TIMEZONE_MINUTE FROM SYSTIMESTAMP)/60/24
  - incoming_replicat_ts) as a
from C##GGADMIN.GG_HEARTBEAT);
```

### View Report (Runtime Statistics)

```
Send <Process_name>, Report
View <process_name>, report
```

### View Process Report

```bash
view report EXT02
```

---

## Parameter Files

### View Manager Parameters

```bash
view params mgr
```

### View Extract Process Parameters

```bash
view params EXTPROCESS
```

---

## Process Control

### Start Extract with Detail

```
START EXTRACT <extract name>, DETAIL
```

---

## Logs and Events

### View Error Log from GGSCI

```
VIEW GGSEVT
```

### Execute Shell Commands in GGSCI

```
ggsci > sh date
```

### History Command

```
h
```

**Alternative:** Execute previous command by number:

```
h 10
```

**Alternative:** Execute command by history number (e.g., `!4` runs command 4):

```
GGSCI (oelr5u7) 30> !
h 29

GGSCI Command History
    2: start mgr
    3: start er *
    4: status all
    ...
   29:  h 29
   30:  h 29

GGSCI (oelr5u7) 31> !4
status all
```

---

## Grid Infrastructure Integration

### Check GoldenGate Resources (crsctl)

```bash
$GRID_HOME/bin/crsctl stat res -w "TYPE = xag.goldengate.type" -p
```

### GoldenGate Resource Status

```bash
crsctl status res xag.ggate_uat.goldengate -f
```

---

[Back to Main Index](README.md)
