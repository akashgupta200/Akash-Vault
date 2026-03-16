---
layout: default
title: "OS Commands Reference"
parent: "DBA Reference Sheet"
nav_order: 1
---

# OS Commands Reference

OS-level commands for hardware, disk, CPU, memory, network, and utilities across AIX, Linux, and Solaris platforms.

---

## Hardware

### Hardware type (AIX)

```bash
uname -M  /   prtconf
```

### Hardware type (Linux)

```bash
cat /sys/devices/virtual/dmi/id/*
```

### Hardware type (Solaris)

```bash
uname -a
```

---

## OS

### OS version (AIX)

```bash
oslevel -r
```

### OS version (Solaris)

```bash
cat /etc/release
```

### OS version (Linux)

```bash
cat /etc/redhat-release
```

### OS 32-bit or 64-bit

```bash
uname -m
# x86_64 ==> 64-bit kernel
# i686   ==> 32-bit kernel
```

### Who command alternative

```bash
pinky
```

### Server reboot time

```bash
who -b
```

---

## Tracing

### Tracing system calls (Solaris, AIX)

```bash
truss -p <spid>
# or
truss -aedfo sqlplus.trc sqlplus /nolog
```

**Alternative:** Solaris: `truss -d -E -p 1454`

### Tracing Unix system calls (Linux)

```bash
strace -p <spid> -o output.txt
```

**Alternative:** `strace -ttT -p 5164`

### Tracing system calls (Tru64, HP-UX)

```bash
trace -p <spid>
# HP-UX: tusc
```

### Identify zombie process

```bash
ps -ecl | grep "Z"
```

### Full details about a PID

```bash
pargs 11637
```

---

## Memory

### RAM size (Solaris)

```bash
/usr/sbin/prtconf | grep Mem
```

**Alternative:** `prstat -a` or `prstat -Z`

### RAM size (HP/UX)

```bash
dmesg
vmstat 5 5
swapinfo -tam
glance and m
```

### RAM size (AIX)

```bash
lsdev -C | grep mem
lsattr -El mem0
```

### RAM size (Linux)

```bash
free -h
cat /proc/meminfo
dmesg | grep Memory
```

### RAM size (DEC-UNIX)

```bash
uerf -r 300 | grep -I mem
```

### Real memory used by Oracle (Solaris)

```bash
prstat -a
```

### Virtual memory usage

```bash
vmstat 1 5
```

### Top memory users

```bash
/usr/ucb/ps vax | head
```

**Alternative:** `svmon`

### Clear cache memory in Linux

```bash
sync; echo 1 > /proc/sys/vm/drop_caches
```

**Alternative:** To free pagecache: `echo 1 > /proc/sys/vm/drop_caches`; To free dentries and inodes: `echo 2 > /proc/sys/vm/drop_caches`; To free all: `echo 3 > /proc/sys/vm/drop_caches`

---

## Disks

### Sort by file size

```bash
du -sh * | sort -h
```

### List disks

```bash
iostat -IEn
# or
echo | format
```

### Disk size (Linux)

```bash
fdisk -l | egrep 'Disk.*bytes' | awk '{ sub(/,/,""); sum +=$3; print $2" "$3" "$4 } END { print "—————–"; print "total: " sum " GB"}'
```

### Disk cloning between two servers

See: http://serverfault.com/questions/4906/using-dd-for-disk-cloning

### Create a dummy disk in Solaris (1GB)

```bash
dd if=/dev/zero of=test.img bs=1024k count=1000
```

### Copying disks

```bash
dd if=test.img of=test.img.bkp bs=1024000k
```

### Disk hardware errors

```bash
iostat -IEn | grep <device>
# Check Soft Errors, Hard Errors, Transport Errors in output
```

### OS space not released after deletion

```bash
/usr/sbin/lsof | grep deleted
file /proc/<pid>/fd/<fd>
echo > /proc/<pid>/fd/<fd>
df -h .
```

### Device statistics (Solaris)

```bash
iostat -xnp
```

**Alternative:** `getdev`, `cfgadm -la`, `prtconf`

### Freespace in zpool

```bash
zpool list
```

### OS used disks

```bash
zpool status
```

---

## CPU

### Number of CPUs (Solaris virtual CPU)

```bash
psrinfo | wc -l
```

### Number of CPUs (Linux)

```bash
lscpu
# or
cat /proc/cpuinfo | grep processor | wc -l
```

**Alternative:** `echo Cores = $(( $(lscpu | awk '/Socket/{ print $2 }') * $(lscpu | awk '/Core/{ print $4 }') ))`

### Number of CPUs (Solaris physical)

```bash
psrinfo -v | grep "Status of processor" | wc -l
```

### Number of CPUs (AIX)

```bash
lsdev -C | grep Process | wc -l
```

### Number of CPUs (HP/UX)

```bash
ioscan -C processor | grep processor | wc -l
```

### Real CPU in Solaris

```bash
kstat | grep -i core_id | uniq
psrinfo -p
psrinfo | wc -l
```

### CPU usage

```bash
sar -u 1 10
```

**Alternative:** `ps -e -o pcpu -o pid -o user -o args`

### Top CPU users

```bash
/usr/ucb/ps uax | head
```

**Alternative:** `topas`

### Top 10 CPU consuming process

```bash
date ; ps auwx | sort -r +2 | head -10
```

### CPU & memory used by a PID

```bash
/usr/ucb/ps auxvv 11637
```

### Top CPU usage

```bash
ps -eo pcpu,pid,user,args | sort -k 1 -r | head -10
```

**Alternative:** `ps auxw | sort -r +2 | head -10`

---

## SAR

### CPU

```bash
sar [-u] [interval [count]]
sar -q [interval [count]]   # Load Average
```

### Memory

```bash
sar -B [interval [count]]   # Kernel Paging
sar -r [interval [count]]   # Unused Memory
sar -S [interval [count]]   # Swap Space
```

### Disk

```bash
sar -b [interval [count]]   # Average Disk I/O
sar -dp [interval [count]] # Disk I/O
```

### Network

```bash
sar -n DEV [interval [count]]   # Network
sar -n EDEV [interval [count]]  # Network Errors
```

---

## ZIP and Compression

### Zipping a folder in Solaris

```bash
zip -r archive_name.zip folder
```

### Compress old files

```bash
gzip logfile.log
```

### Compressing a folder

```bash
tar -zcvf archive.tar.gz directory/
```

### Reading files under .gz without uncompress

```bash
zcat filename
# or
zhead filename
# or
zless filename
```

---

## Find / Replace

### Display above and below 2 lines on grep

```bash
cat a | grep 5 -C 2
```

### Finding a file

```bash
find /opt/tivoli/tsm/client -exec grep -l "servername" {} \;
```

### Find file

```bash
find / -print | grep -i dbmspool.sql
```

### Remove old files

```bash
find /opt/oracle/admin//cdmp* -mtime +1 -exec rm {} \;
```

**Alternative:** `find /backup/logs/ -name daily_backup* -mtime +21 -exec rm -f {} \;`

### Replace a word in all files under a directory

```bash
find oraInventory -type f -exec perl -pi -e 's#/home/oracle/oraInventory#/u01/app/oracle/oraInventory#g' {} \;
```

### Find all files containing specific text on Linux

```bash
find . -type f | xargs grep dc7h92zrnpbnq
find . -type f -exec grep -l "dc7h92zrnpbnq" {} \;
```

**Alternative:** `grep -rnw /location -e "dc7h92zrnpbnq"`

---

## Mail

### Sending a file through mail

```bash
uuencode file.txt file.txt | mailx testmail@domain.com
uuencode file.txt file.txt | mailx -s "this mail has attachments" testmail@domain.com
mailx -a textfile.txt testmailreceiver@domain.com
```

### Sending mail from a server

```bash
mailx -s "Test email `hostname`" testmail@domain.com < /dev/null
```

---

## Network

### Number of open connections to DB from a server

```bash
lsof -i:1521 | grep ESTABLISHED | grep db_name | wc -l
```

### Unplumb the network interface (Solaris)

```bash
ifconfig nxge2:2 unplumb
```

### MTU value (ping with MTU)

```bash
/usr/sbin/ping -s nodename mtu 2
# Example for three-node environment:
/usr/sbin/ping -s node1-pub 1500 2
/usr/sbin/ping -s node2-pub 1500 2
/usr/sbin/ping -s node3-pub 1500 2
/usr/sbin/ping -s node1-priv 9000 2
/usr/sbin/ping -s node2-priv 9000 2
/usr/sbin/ping -s node3-priv 9000 2
```

### Enable remote desktop in Linux

```bash
gconftool-2 -s -t bool /desktop/gnome/remote_access/enabled true
gconftool-2 -s -t bool /desktop/gnome/remote_access/prompt_enabled false
# Disable:
gconftool-2 -s -t bool /desktop/gnome/remote_access/enabled false
```

### VNC server

```bash
export PATH=/:/bin:/usr/local/bin:/usr/ucb:/etc:/usr/sbin:/sbin:/usr/X11/bin:/usr/openwin/bin:/opt/sfw/bin:/bin:/OPatch:$PATH
vncserver
ps -ef | grep vnc
vncserver -kill :4
```

### SCP in background

```bash
nohup scp -P 223 /tmp/myfile.dat user@server:/tmp &
```

### Rsync files between servers

```bash
rsync -avz --progress username@sourceserver:/path/to/file.dmp .
```

### SCP faster file transfer (SSH pwd required)

```bash
scp -c arcfour -r username@sourceserver:/path/to/file.dmp .
```

### File copy to remote server without password

```bash
# SOURCE:
cd /location
tar -cf - expdpmyfile.dmp | gzip -1 | nc -l 9999

# TARGET:
cd /thelocation/u/need
nc Sourceserver 9999 | gzip -d | tar xf - -C .
```

### Identify who occupied the port

```bash
netstat -anlp | grep 1587
```

### Check physical status of NIC card

```bash
/sbin/ip link show
```

---

## Others

### prstat top 5 by CPU

```bash
prstat -n 5 -s cpu
```

### iostat

```bash
iostat -x 1 3
```

### Error log locations

- Linux: `/var/log/messages`
- Solaris, HP Tru64: `/var/adm/messages`
- HP-UX: `/var/adm/syslog/syslog.log`
- AIX: `/bin/errpt`

### ^M Error (remove CTRL-M)

```bash
dos2unix filename filename
```

### Tree command in Solaris

```bash
ls -R | grep ":$" | sed -e 's/:$//' -e 's/[^-][^\/]*\//--/g' -e 's/^/   /' -e 's/-/|/'
```

### History (ksh)

```bash
fc -l -99
```

### History command (bash)

```bash
HISTTIMEFORMAT="%d/%m/%y %T "
history
```

**Alternative:** `history -200`

### Who logged in

```bash
last | grep root | more
```

### Timing at Unix prompt

```bash
PS1="[\d \t \u@\h:\w ] $ "
```

### Time taken for command execution

```bash
time dd if=test.img of=test.img.bkp bs=1M
```

### Check package installed (Solaris)

```bash
pkginfo | grep xvnc
```

### Installation of RPM without dependency

```bash
rpm -Uvh /opt/oracle/admin/redhat-release-6Server-1.noarch.rpm --nodeps
```

### Check server is global or local zone

```bash
zonename
# global = global zone; server name = local zone (virtual machine)
```

### Check HBA ports online or offline

```bash
luxadm -e port
```

### "Tmp file too large" error in vi

```bash
stty columns 120
vi file.log
```

### C program to execute shell script

```c
#include <stdio.h>
#include <stdlib.h>
#define SHELLSCRIPT "ssh mainserver '/work/oracle/dba/bin/mygrid gridname'"
int main(void) {
  system(SHELLSCRIPT);
  return 0;
}
```

```bash
cc a.c
./a.out
```

### Display two files in parallel

```bash
paste c.txt d.txt | awk -F'\t' '{
    if (length($1)>max1) {max1=length($1)};
    col1[NR] = $1; col2[NR] = $2 }
    END {for (i = 1; i<=NR; i++) {printf ("%-*s     %s\n", max1, col1[i], col2[i])}
}'
```

### Difference between two folders

```bash
diff -r folder1 folder2
```

**Alternative:** `diff --brief -r folder1 folder2`

### Difference of two files (side-by-side)

```bash
diff -y -W 170 c.txt d.txt
vimdiff file1 file2
```

**Alternative (Solaris):** `sdiff -w 250 presnap.txt postsnap.txt | more`

### Find file containing particular string

```bash
find . -type f -exec grep -l "DBMS_STATS: GATHER_STATS_JOB: Stopped by Scheduler." {} \;
```

### Screen command

```bash
screen -S Patching                    # Start new session with name
screen -ls                           # List existing sessions
screen -r -d screen_name             # Attach existing screen
screen -r                            # Attach if only one session
screen -x Screen_name                 # Attach multiple display
# CTRL+a d                           # Detach without terminating
```

### Password encryption

```bash
echo -e mypwd | base64
```

### Grep display lines above/below

```bash
grep -A 2 FAILED file_name   # Display below 2 lines
grep -B 2 FAILED file_name   # Display above 2 lines
grep -C 2 FAILED file_name   # Display above and below 2 lines
```

### Folder size including hidden folders

```bash
du -sch .[!.]* * | sort -h
```

### File occupied by whom

```bash
/usr/sbin/lsof +L1 /opt/oracle
```

---

Back to [Main Index](README.md)
