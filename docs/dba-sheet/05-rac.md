---
layout: default
title: "RAC Commands Reference"
parent: "DBA Reference Sheet"
nav_order: 5
---

# RAC Commands Reference

RAC commands, diagnostics, clusterware management, and network verification.

---

## RAC Binary Configuration

### Check if RAC is enabled in ORACLE_HOME

```bash
/usr/ccs/bin/ar -t $ORACLE_HOME/rdbms/lib/libknlopt.a | grep kcsm.o
```

Result returns 1 = RAC enabled; 0 = not enabled.

### Disable RAC on Oracle home

```bash
cd $ORACLE_HOME/rdbms/lib
make -f ins_rdbms.mk rac_off    # Disabling RAC binaries
make -f ins_rdbms.mk ioracle    # Linking to Oracle binaries
```

---

## Resource Management

### Capture resource definition

```bash
for res in `$ORA_CRS_HOME/bin/crs_stat -p | grep "^NAME=" | cut -d= -f2`; do
  $ORA_CRS_HOME/bin/crs_stat -p $res >/opt/oracle/resources/$res.cap
done
```

### CRS resource script

```bash
for res in `/opt/crs/12.1.0/grid/bin/crs_stat -p | grep "^NAME=" | cut -d= -f2`; do
  /opt/crs/12.1.0/grid/bin/crs_stat -p $res >/opt/oracle/resources/$res.cap
done
```

---

## Node Eviction and Process Management

### Node eviction demo

```bash
ps -ef | grep cssd | grep -v grep
chrt -p 3595
# As root:
kill -STOP 3595; sleep 27; kill -CONT 3814
```

### Killing clusterware processes

```bash
ps -ef | grep <keyword> | grep -v grep | awk '{print $2}' | xargs kill -9
```

### Kill all CRS processes

```bash
ps -ef | grep <keyword> | grep -v grep | awk '{print $2}' | xargs kill -9
```

---

## Network

### Verify Jumbo frames

```bash
traceroute -F node02-priv 9000
```

**Alternative (ping with MTU):** Use `ping -c 2 -M do -s 8972 node02-priv` (8972 works with 9000 MTU; 8973 fails).

### Verify jumbo frames using MTU 9000

```bash
/usr/sbin/traceroute -s node1-priv node2-priv 9000
```

### Network connectivity check by OSW

```bash
traceroute -r -F 192.168.0.210
```

---

## Tracing and Diagnostics

### Tracing RAC resources

```bash
export SRVM_TRACE=true
# Clear: export SRVM_TRACE=""
```

### Trace troubleshoot RAC CRS error

```bash
export SRVM_TRACE=true
cd $CRS_HOME/bin
./cluvfy comp crs -n
```

Trace file generated in `$CRS_HOME/cv/log`.

### Tracing cluvfy

```bash
rm -rf /tmp/cvutrace
mkdir /tmp/cvutrace
export CV_TRACELOC=/tmp/cvutrace
export SRVM_TRACE=true
export SRVM_TRACE_LEVEL=1
<STAGE_AREA>/runcluvfy.sh stage -pre crsinst -n <node1>,<node2> -verbose
```

See Doc ID 986822.1.

---

## Cluster Configuration

### Modify cluster services (network interface)

```bash
$GRID_HOME/bin/crsctl modify resource ora.net1.network -attr "USR_ORA_IF=vnet0 vnet2"
```

### Formatting OCR/Voting disks

```bash
$DD if=/dev/zero skip=25 bs=4k count=2560 of=$OCRLOC
```

---

## Diag Collections

### Collect cluster diagnostics

```bash
cd /tmp/myfolder
export GRID_HOME=/opt/crs/12.1.0/grid
export ORA_CRS_HOME=/opt/crs/12.1.0/grid
perl /opt/crs/12.1.0/grid/bin/diagcollection.pl --collect -all \
  --incidenttime 06/12/201415:00:00 --incidentduration 01:00
chown oracle:dba *
tar -zcvf server1038diag.tar.gz myfolder
```

---

## Patching and Upgrade

### Switching RAC home / patching / upgrade

```bash
$NEW_GRID_HOME/oui/bin/runInstaller -updateNodeList ORACLE_HOME=<$NEW_GRID_HOME> CRS="false" –local
```

Ensure only current GI home has `CRS=TRUE` in inventory on ALL nodes. Then as root:

```bash
cp -pR $OLD_GRID_HOME/cdata $NEW_GRID_HOME
cp -pR $OLD_GRID_HOME/gpnp $NEW_GRID_HOME
cp -p $OLD_GRID_HOME/crs/install/s_crsconfig_<hostname>_env.txt $NEW_GRID_HOME/crs/install
$NEW_GRID_HOME/crs/install/rootcrs.pl –verbose -patch -destcrsHome $NEW_GRID_HOME
```

Monitor: `tail $NEW_GRID_HOME/cfgtoollogs/crsconfig/crspatch_<hostname>.log`

### Skip MGMTDB on 12.1.0.2

Add `-J-Doracle.install.mgmtDB=false` to the installer command line.

---

## Database Status

### Database running on all RAC nodes

```bash
export ORACLE_HOME=`cat /etc/oratab | grep -i +ASM | cut -d: -f2`
export ORACLE_SID=`cat /etc/oratab | grep -i +ASM | cut -d: -f1`
export PATH=$ORACLE_HOME/bin:$PATH
for i in `olsnodes`; do
  scp -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -q chek.sh $i:/tmp
done
for i in `olsnodes`; do
  ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -q $i "/tmp/chek.sh" > /tmp/tmp.log_$i
done
paste /tmp/tmp.log* | sed -e 's/\t/ \t/g' | column -t -s$'\t'
```

### Alternative to crs_stat (Linux)

```bash
crsctl status res | grep -v "^$" | awk -F "=" 'BEGIN {print " "} {printf("%s",NR%4 ? $2"|" : $2"\n")}' | \
  sed -e 's/  *, /,/g' -e 's/, /,/g' | \
  awk -F "|" 'BEGIN { printf "%-40s%-35s%-20s%-50s\n","Resource Name","Resource Type","Target ","State" }
  { split ($3,trg,",") split ($4,st,",") }
  { for (i in trg) {printf "%-40s%-35s%-20s%-50s\n",$1,$2,trg[i],st[i]}}'
```

### Grid all databases list

```bash
for i in `olsnodes`; do
  ssh -q $i "ps -aef | grep pmon | awk '{print $8}' | egrep -i -v 'grep' | cut -d'_' -f3" | sort > /tmp/tmp.log_$i
done
paste /tmp/tmp.log* | column -t
```

### Grid all databases list with nodename

```bash
for i in `olsnodes`; do
  echo $i > /tmp/tmp.log_$i
  ssh -q $i "echo '================'; ps -aef | grep pmon | awk '{print $8}' | egrep -i -v 'grep' | cut -d'_' -f3" | sort >> /tmp/tmp.log_$i
done
paste /tmp/tmp.log* | sed -e 's/\t/ \t/g' | column -t -s$'\t'
rm -rf /tmp/tmp.log*
```

### Service name for databases

```bash
awk -F ":" '{if ($1) print "srvctl status service -d "$1""}' /var/opt/oracle/oratab | grep -v '#\|+' | xargs -0 bash -c | grep -v 'PRC'
```

---

## ACFS

### Start ACFS filesystem

```bash
# As root:
/opt/grid/12.1.0.2/bin/acfsload start

# As oracle:
srvctl start filesystem -d /dev/asm/acf-somename-123
srvctl start filesystem -d /dev/asm/acf_ggtrlv1-108
srvctl start filesystem -d /dev/asm/acf_repl1-108
```

---

## Troubleshooting

### If root.sh fails (11gR2)

```bash
/usr/bin/chuser capabilities=CAP_NUMA_ATTACH,CAP_BYPASS_RAC_VMM,CAP_PROPAGATE grid
```

---

## Load Balancing

### RAC DB load balancing across instances

```sql
WITH sys_time AS (
  SELECT inst_id,
         SUM(CASE stat_name WHEN 'DB time' THEN VALUE END) db_time,
         SUM(CASE WHEN stat_name IN ('DB CPU', 'background cpu time') THEN VALUE END) cpu_time
  FROM gv$sys_time_model
  GROUP BY inst_id
)
SELECT instance_name,
       ROUND(db_time/1000000,2) db_time_secs,
       ROUND(db_time*100/SUM(db_time) OVER(),2) db_time_pct,
       ROUND(cpu_time/1000000,2) cpu_time_secs,
       ROUND(cpu_time*100/SUM(cpu_time) OVER(),2) cpu_time_pct
FROM sys_time
JOIN gv$instance USING (inst_id);
```

---

## Utilities

### Ignore error till empty line (olsnodes)

```bash
olsnodes | sed -n '/^$/,/^$/p' | awk '{if (NR==1 && NF==0) next};1'
```

Useful when executing ASM commands as non-oracle user.

### Print output till empty line

```bash
olsnodes | sed -n '/^$/,/^$/!p'
```

### Role-based service check

```bash
crsctl stat res ora.dbname.service_name.svc -p | egrep '(USR_ORA_OPEN_MODE|ROLE)'
```

Primary = primary role; physical_standby = standby role.

---

## References

- Validate network and name resolution: Doc ID 1054902.1
- Changing hostnames in Oracle RAC: http://www.pythian.com/blog/changing-hostnames-in-oracle-rac/
- Export/import CRS resources: http://dbaregistry.blogspot.com/search/label/How%20to%20export%20and%20import%20crs%20resources
- Jumbo frames: http://www.vmcd.org/2012/02/recommendation-for-the-real-application-cluster-interconnect-and-jumbo-frames/

---

Back to [Main Index](README.md)
