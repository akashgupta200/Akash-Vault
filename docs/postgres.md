---
layout: default
title: PostgreSQL Queries
nav_order: 3
has_children: true
has_toc: true
---

# PostgreSQL — Complete DBA Reference

This document is a comprehensive DBA reference for PostgreSQL, covering architecture, installation, configuration, user management, backup/restore, replication, maintenance, and version-specific features.

---

## 1. POSTGRES DATABASE INTRODUCTION

Database : it is a storage space or device which is used to manage or maintain only summarized data.


summarized data:

->data which will gives proper meaning


->data will be used for proper purpose


->data will be having proper format.


RAW data:

->data will not give proper meaning


->not usefull for proper purpose


->not contains proper format.


default RAW data will be there, every one will get the summarized data from RAW data.


### 01-03-2021

while deciding centralized database we have lot of alternatives.


1.oracle


2.ms sql


3.mysql


4.postgresql


5.mongodb


6.teradata


7.greenplum


8.redis


9.memsql


10.couchbase


11.aerospike


12.dynamodb


13.db2


almost 30+ techonolgoies in the market.


### why we need to choose postgresql

->oracle or ms sql or all other database support & licencing cost will be very high.


->not that much userfriendly


->these database licensing cost will be depend up on number of CPU's in the server.


->if data size growing , defenetely system perfromance will be reduced, to increase the perfromance we will increase the number of cpus and RAM.


->but thing is if number of cpu's increasing , perfromance will be good, cost also will increase.


so most of the people started searching for alternative, people are expecting database with below features.


1.database s/w should be either free of cost or cheap price.


2.it should support data in relational(table) format.


3.it shold provide high level security.


4.it should support High Avaiblity. : if server went down, immediately another server will plays its role.


5.it should support DR (disaster recovery): if server completely crashed, immeditely another server plays its reole, it will be permenent


6.it should support point intime recovery: eventhough we dont have enough backup, still we should be capable to restore upto specific point


of time.


7.if my DBA not able to do the things, in that case some one should be there to provide support.


8.my clinet is asking certification for the product, we need to certification for the product as well.


->with the same alternatives one product came into market, i.e postgresql.


### History behind postgresql

in 1980's 2 phd stundents from barcleys universitry built one database, they named it as ingress. after thrier project done they left this.


after some time one corporate company started funding for this product furhter development. after development, they renamed it as PostgreSQL in 1989.


but one thing is as this product developed by funding, product should not be sold as commercial.


community version(open source/avaible for any one), postgresql team started community website they kep their software and doucmentation.


anyone can download & use.


starting version is 1.0. 1.6, later directly jumped to version 6, current postgresql version 14.2


from version 6 to 14, almost postgresql team released 160+ major and minor versions.


6  to 14.2 (160+ major and minor versions)


7

7.1.0

7.1.11

7.1.12


first digit we will call it as version, next 2 digits we will call it as release.


->as it is open source product, no one is there to give the support if required, but by taking this as advantage some of the companies started


services.


ex:


->openscg


->2nd quadrant


->fujitsu


->cybertec


->dbi


->dbops


->migops


->percona


->pgexperts


->serverlines


->edb


if required someone should be there to provide certification for the product, by taking this as advantage one more company came into picutre


i.e EDB (enterprisedb).


there is an agreement b/w open source and edb team, EDB team will give funding to open source team for the new features/product development.


EDB team should not make any changes to product developed by open source team.


EDB just renaming the open source postgres product and releasing into market.


EDB giving few user friendly tools for DBA team. [EFM, PEM, BART]


->2017,2019,2020 best growing RDBS of the year award comes to postgresql.


->2018 award goes to mysql due to oracle.


## 2. POSTGRES ARCHITECTURE

how to prepare a tea?


milk,sugar,water, bowl,stove ,fire -->boil mil, pour tea powder, mix sugar, then bil, then filter, then serving


if you are a developer , you no need to aware all the internals of postgresql database.


if you are DBA, you should aware all the things, or else it will be very difficlut.


->user will execute query and will get the output .


->from starting point of query execution to until get the result set complete backend flow we will call it as architecture.


->based on the process and work, architecture devided into 3 layers.


1.genral architecture


2.memory based architecture


3.process based architecture


## 3. GENERAL ARCHITECTURE

->from starting point of query execution to until get the result set complete backend flow we will call it as general architecture.


->running query we will call it as raising request.


[user: ram  dbname: postgres  host: 192.168.10.11 query : select * from emp clint ip: 192.168.11.30]


->end user will raise the request from clinet interface, that request will reaches to postgresql server operating system.


->normally , people who is doing work we will call him as worker.


->but in operating system we call worker as process.


->on top of operating system there is a process called postmaster process.


->this process will stop and start postgresql instance services.


->postmaster will perfrom hostbased authntication for the new request came from userr.


## 4. HOSTBASED AUTHENTICATION

->in postgresql there is a configuration file calld pg_hba.conf file.


->it will contains all the databaes and username and clinet ip addresses details.


->this file located under database server, in this server there is a directory called DATA directory, under DATA directory this file exist.


->pg_hba.conf file will be used to prevent or manage remote/local conenctions.


->postmaster process will read pg_hba.conf file and check whether username and database name and clinet ip details or there in pg_hba.conf file


or not, if no entries in file, it will through error, if entries are there it will treat like a connection authorized.


->if connection is authorized , postmaster will create new postgres process id for the request.


->that process id will be responsbile for complete process..


->it will perfrom some generic steps, we will call it as plastic steps.


a.syntaxer : whether user running query with correct syntax or nor.


b.security check : whether user ram has select permission on emp table not.


c.resolver : whether same user, same query already executed or not, if executed that query result set existed in cache or nor.


->if query already exicuted, there will be benfit for perfromance. as there is no need to go to disk to get the data, directly we can get the


result set from cache.


d.optimizer : it will generate best least cost execution plan.


e.dispatcher: according to the plan , dispatcher will execute the steps.


optimzer working like an engineer, dispatcher working like a labour.


### brief view

end users[product users/developers/application]


->they will raise request from Clinet interface[GUI/CLI/App servers]


->request will reaches to postgresql server operating system


->on top OS there is a process called postmaster


->hostbased authentication by reading pg_hba.conf file


->if entries are there, it will treats connection is authorized


->it will create a postgres process id


->complete responsibility with postgres process id


->it will perfrom generic(syntaxer,security check,resolver,optimizer,dispatcher)


## 5. MEMORY BASED ARCHITECTURE

->postgresql completely operating system based architecture.


->if you run any query in the database, that query will appers on operating system.


moble1 : 2gb ram windows os -10000 rs

mobile2 : 2gb ram, andoriod os --10000rs


first priority for operating system, we will go with linux operating system.


moble1 : 2gb ram andorid os -10000 rs

mobile2 : 4gb ram, andoriod os --11000 rs


if RAM is more, system perfromance will be good.


RAM means memory, we are accepting memory will play key role in perfromance.


->coming to postgresql, based on the work type memory catagorized into few components.


1. shared_buffers


2. work_mem


3. temp_buffers


4.maintanance_Work_mem


5.autovacuum_work_mem


6.effective_Cache_size


system RAM : 16GB


### shared_buffers

->this is a piece of RAM , which is usefull for postgresql internal operations.


internal operations: getting query, connection authorization checking, sytnax checking, process flow generation, queries infromation recording,


if database crashed, perform recovery..


->recomended value is 25% of total RAM.


shared_buffers= 4gb


->if we give bigger vaulue, perfromance will be good, but other operation will face the problem due to lack of RAM.


sometimes there might be chances for server crash as well.


->in postgresql server, under DATA directory there is configuration file called postgresql.conf file, this is the main file, it will decide


```bash
which parameter requires how much value.

```

->if you change this parameter value, we need to restart postgresql services.


->shared_buffers value instance level.


->if allocated shared_buffers not enough for operation, it will fial with out of memory error, it will never cross the assigned limit.


->default value of shared_buffers 128MB.


->minimum shared_buffers 128kb we can configure.


### work_mem

this is piece of remaining RAM, this will be utilized for postgresql extenral operations.


but condition is that external operation should perfrom etiher hashing or sorting.


->restarting services will not clear the cache.


->until ur reasult set in the cache, optimizer will not come into the pciture.


->if optimizer comes into the picture then only hashing/sorting concept will come.


->this is operation specific, that means each query this work_mem will be allocated.


->remaining RAM : 12GB


->if i mention work_mem value as 1GB, later 20 connections connected at a time , total memory required 20*1GB..


->but we have only 12GB RAM, it will through out of memory error.


->thats is the reason always we will keep work_mem value very less.


->we will assign this value by considering mx_connections value.


max_connections: at a time how many connections can connect to database, default value is 100.


->if my work_mem 10MB, in worest case if all 100 connections connected to database, 100* 10MB -1gb RAM required.


->default value of work_mem is 4MB.


->plan like max_connections* work_mem value always 50% of reamaining RAM.


->some times we will go with 8mb/16mb/32mb/64mb/128mb/256mb


->until query execution in progress it will be hold, once execution done this memory automatically released.


> **NOTE: we will drop cache during non bussiness hours.**


### 3. temp_buffers

->if any query generating temp result sets


->while creating temprory tables


this temp_buffers will be utilized.


this is also piece of RAM.


> **note: temp tables: tables will be avaible until u r in the session, once u came out from the session table will be gone.**


->sub queries/nested joins will generate more temp result sets.


```sql
select * from emp where eno in(select eno from dept));

```

->first part call it as parent query


->second part call it as child query


->always child query will execute first and that output will be passed to parent query as a input.


->there is minimum gap b/w child query and parent query execcution, so in the meantime child query result set will be stored in temp_buffers.


->default value is 8MB


### 4.maintanace_work_mem

->this is also piece of remaining RAM


->for databases to keep it safe and secure and to improve the perfromance we need to perform maintiance actvities like


```sql
VACUUM
ANALYZE
VACUUM FULL
REINDEX.

```

->while running maintance activties this memory will be consumed. only consumed during manual run.


->while running ALTER TABLE STATEMENTS also this will be consumed.


->default value is 64MB.


### 5.autovacuum_work_mem

->this is also piece of RAM.


->postgresql given option to automate the maitanance activties like vacuum & analyze operations only, at this time this memory will be consumed.


->Default value is -1


->that means it will utilize the memory equal to maintanance_work_mem.


### 6.effective_cache_size

->this is a dummy value.


->we can assign 85% of total RAM as this parameter.(12gb of 16GB)


->it is only improving perfromance of queries which includes indexes.


->to improve the indexes effective ness this parameter will be used.


->default value is 128MB.


->all the above components/parameters values we will configure in postgresql.conf file.


->static parameters : which requires postgresql servieces restart


->dynamic parameters: no need any downtime, just reload is enough.


## 6. TUNE MEMORY RELATED PARAMETER

1. If we increase the shared buffer, transaction per second will decrease only. --Shared buffer tunning.

2. If you are using "order by" clause in query , basically any sorting operation we can increase work mem for that particular session

user to speed up the query execution. --- Work Mem tunning

3. If you are creating index on a table and it is slow then we can increase maintanace_work_mem temporarly on session level to speed up

the index creation query. --- Maintenance_work_mem

4. If effective_Cache_size memory is less , then optimizer will not use index even index is helpful in that case we can try to increase this.

5. checkpoint_timeout, default value is 5 min, if we increase it to 20 min so that performance will increase as it dont have to flush

dirty buffer to disk in every 5 min which is slowing down the system whereever disk comes into picture. but setting it to 20 min will

have impact if server goes crash as we might loose that 20 min of data which is there in memory and not flushed to disk, additionally

shutdown of the system will be slow.


## 7. PROCESS BASED ARCHITECTURE

->postgresql purely operating system based architecture.


->memory will play key role in postgresql architecture.


->outside we will call labour as worker, but in operating system we call worker as Process.


->based on work or profession naming will be coming, in the same manner db side/os side.


->bassed on the work or task, we have few process in postgresql.


->we have mandatory process & optional processes.


mandatory process: we can't enable or disable, by default those process will be up and running.


optional process: we can enable or disable based on our requirement.


->by default in postgresql we will start only process i.e Postmaster


### immedietely it will start mandatory child process.

1a. postmaster : to start /stop.restart/reload postgresql services


1b. writr process (wal writer) : all committed transactions will be writing to wal file from wal buffer area.


1c. bg writer process: for every 200 ms, it will flushes all dirty buffers to disk.


1d. checkpointer process : for every 5 minutes, if any gaps with bgwriter, it will flushes.


1e. stats collector process: it will keep on update the statistics.


1f. autovacuum launcher process : this process will take care about automatic vacuum and analyze.


1g. logical replication launcher process : as part of logical replication, this process will come.


### otpinal process

1h. archiver process : to take wal files backup to archive folder , we need to enable archive_mode, once enabled this proces will start


copying wal files from pg_Wal to archive directory.


1i. logger process: to track each and every activity, we will enable audit log, this process will collect the log.


1k. wal sender process : it will send data to standby


1l. wal receiver process:  it will receive data from master


->transaction is different , actual data is different.


emp (no int,name varchar)


```sql
update emp set no=100 where no=1;  --> transaction

```

emp(1,ram),(2,sam),(3,bheem),(4,jan),(5,raj)  --> actual data


->each and every transaction happening at database levl, will be recorded in one file.


->we call that file as WAL file


->in postgresql only committed (completed) transactions information will be recorded here.


->all uncommited transactions will be located in wal_buffer aera( this is i piece of shared_buffers)


->for every 16MB size of transactions new wal file will be created.


->these WAL files will be created under pg_wal directory, this directory under DATA directory.


->until postgresql 9.6 version, pg_wal directory name is pg_xlog.


->postgresql by default autocommit, that means if u run any query immedieatly it will save the changes to disk.


->once saed means not possible for rollback eventhough required.


->default size of pg_wal directory is 1GB..that means at a time 64 wal files will be there in pg_wal.


->if 65th wal file comes, first wal file be cleared autoamatically.


->if files removed from the pg_wal no impact to the services, as already thos transactions completed.


->if by any chance to perfrom point intime recovery, we need all the old wal files.


->eventhough wal file size not reached 16MB, still we can switch the wal. select pg_switch_wal();


->yes, we can increase the wal file size from 16mb to bigger value to reduce the impact on disk.


->this is possible only one time, that is during database initialization we can use segment-size option.


->we can increase pg_wal directory capacity as well, by changing wal_keep_size parameter in postgresql.


->this value in MB's, that much size of wal files will be managed.


->until postgresql 12, wal_keep_size parameter is max_wal_keep_segments, at that this is number of files.


->safe side always we will take WAL files backup by enabling archive_mode.


## 9. LINUX SETUP

->postgresql purelu operating system based architecture.


->it is open source product.


->we need high level security, security bit difficult with windows server, so 99.9% people will prefer linux only.


->for 1 or 2 hrs purpose , completely changing operating system from windows to linux is not good.


->to get the linux machine whenever we need, once done with our work simply can close it, with this kind of featur VMWARE player avaible in


the marker.


## 10. VMWARE INSTALLATION

download vmware player.


- https://www.vmware.com/asean/products/workstation-player/workstation-player-evaluation.html

->click on download player for windows.


->go to downloads and double click on vmware player file-->click -next -next-finish.


->we will get one icon on top of desktop.


->download linux operating system file.


- http://mirror.aminidc.com/redhat/RHEL_7.2/Server/rhel-server-7.2-x86_64-dvd.iso

->open you vmware work station player


->click on create new virtual machine


->after everything done, it will ask logins, enter logins.


->once logged in to move cursos ctrl+alt press togehter and move mouse pointer.


->run ifconfig command to get the ip address of newly created server.


192.168.38.136  [server ip]


->to work with linux server, to make more easier we have one tool called putty.


->download putty


- https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.76-installer.msi

->goto downloads, double click on putty -->next ->next ->finish


->goto start menu and open putty->add hostname ->clic on open-->putty will be opned.


FYI: first need to login to vm ware player.


### linux basic commands

```sql
with which user conenct to server?

[ram@localhost ~]$ whoami
```

ram

[ram@localhost ~]$


```bash
which server we have connected?

[ram@localhost ~]$ hostname
```

localhost


server ip address?


```bash
[ram@localhost ~]$ hostname -i
```

192.168.38.136

[ram@localhost ~]$


in linux machine we call drives as mount points.


get the list of mount points in linux server?


```bash
[ram@localhost ~]$ df -h
```

Filesystem      Size  Used Avail Use% Mounted on

/dev/sda3        48G   16G   33G  33% /

devtmpfs        1.9G     0  1.9G   0% /dev

tmpfs           1.9G     0  1.9G   0% /dev/shm

tmpfs           1.9G  8.7M  1.9G   1% /run

tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup

/dev/sda1       297M  113M  185M  38% /boot

tmpfs           378M     0  378M   0% /run/user/1000


root mount point is the critcial in linux.  /  is the root mount point.


to work with more advanced operations normal user not good, we need super user/sudo user.


switch from one user to another user:


su - username


ex: su - root


```bash
[ram@localhost ~]$ su - root
```

Password:

Last login: Sun Feb 27 02:57:22 PST 2022 on pts/0

```bash
[root@localhost ~]# whoami
```

root

```bash
[root@localhost ~]#


```

get the current path/present working directory:


```bash
[root@localhost ~]# whoami
```

root

```bash
[root@localhost ~]# pwd
```

/root

```bash
[root@localhost ~]#


```

how to jump from one folder to another folder:


```bash
[root@localhost ~]# cd / [change directory]
[root@localhost /]# pwd
```

/


list out sub folders and files under any mountpoint:


```bash
[root@localhost /]# du -sh *
```

23M     etc

152K    home

557M    opt

2.9G    tmp

1.4G    usr

1.3G    var


just list out files and folders:


```bash
[root@localhost /]# ls -ltr
```

total 36

drwxr-xr-x.   2 root   root    6 May 25  2015 srv

drwxr-xr-x.   2 root   root    6 May 25  2015 mnt

drwxr-xr-x.   2 root   root    6 May 25  2015 media

lrwxrwxrwx.   1 root   root    7 Dec 19  2020 bin -> usr/bin

lrwxrwxrwx.   1 root   root    8 Dec 19  2020 sbin -> usr/sbin

lrwxrwxrwx.   1 root   root    9 Dec 19  2020 lib64 -> usr/lib64

lrwxrwxrwx.   1 root   root    7 Dec 19  2020 lib -> usr/lib

dr-xr-xr-x.   3 root   root 4096 Dec 19  2020 boot

drwxr-xr-x.   2 root   root 4096 Dec 29  2020 applications-merged

drwxr-xr-x.  17 root   root 4096 Jul  2  2021 usr

dr-xr-x---.   7 root   root 4096 Aug 14  2021 root

drwxr-xr-x.   6 root   root   59 Feb 25 23:45 home

drwxr-xr-x.   6 root   root   62 Feb 26 04:07 opt

drwxr-xr-x.   3 oracle dba    16 Feb 26 07:13 u01

dr-xr-xr-x  427 root   root    0 Mar  3 17:18 proc

dr-xr-xr-x   13 root   root    0 Mar  3 17:18 sys

drwxr-xr-x.  21 root   root 4096 Mar  3 17:18 var

drwxrwxrwt.  11 root   root 4096 Mar  3 17:18 tmp

drwxr-xr-x   19 root   root 3160 Mar  3 17:18 dev

drwxr-xr-x.  94 root   root 8192 Mar  3 17:18 etc

drwxr-xr-x   28 root   root  820 Mar  3 17:18 run


starting with dr means directory, remaining all are files.


```bash
[root@localhost /]# cd var

[root@localhost var]# cd lib
[root@localhost lib]# pwd
```

/var/lib


```bash
[root@localhost ~]# cd /var/lib/
[root@localhost lib]# pwd
```

/var/lib

```bash
[root@localhost lib]#


create a directory:
```

```bash
[root@localhost pgsql_feb22]# pwd
```

/var/lib/pgsql_feb22

```bash
[root@localhost pgsql_feb22]# pwd
```

/var/lib/pgsql_feb22

```bash
[root@localhost pgsql_feb22]#

create file:
```

```bash
vi filename

```

ESC i [it will takes u to write mode]


write something


ESC : wq enter


```bash
[root@localhost pgsql_feb22]# ls -ltr
```

total 4

-rw-r--r-- 1 root root 19 Mar  4 16:44 test

```bash
[root@localhost pgsql_feb22]#


first root : owner

```

second root : group


change ownership of file.

```bash
[root@localhost pgsql_feb22]# ls -ltr
```

total 4

-rw-r--r-- 1 ram ram 19 Mar  4 16:44 test

```bash
[root@localhost pgsql_feb22]#

```

changing file permissions:

0 : no permissions


r = 4

w = 2

x = 1

rw: 6

rwe:7


for every file or folder 3 options will be there like below.


-rw-                          r--                               r--


permissions of owner          permissions of group              permissions of outside users.


example : user ram need read,write,exectue on one file, remanining groups, outside users should not a  ccess.


```bash
chmod 700 test

[root@localhost pgsql_feb22]# chmod 700 test
[root@localhost pgsql_feb22]# ls -ltr
```

total 4

-rwx------ 1 ram ram 19 Mar  4 16:44 test

```bash
[root@localhost pgsql_feb22]#


ex2: give read permission for all (owner,group,outside users)

[root@localhost pgsql_feb22]# chmod 444 test
[root@localhost pgsql_feb22]# ls -ltr
```

total 4

-r--r--r-- 1 ram ram 19 Mar  4 16:44 test

```bash
[root@localhost pgsql_feb22]#


```

rename the file:

```bash
[root@localhost pgsql_feb22]# mv test ajinkya
[root@localhost pgsql_feb22]# ls -ltr
```

total 4

-r--r--r-- 1 ram ram 19 Mar  4 16:44 ajinkya

```bash
[root@localhost pgsql_feb22]#

```

make a copy of file:

```bash
[root@localhost pgsql_feb22]# cp ajinkya sachin
[root@localhost pgsql_feb22]# ls -ltr
```

total 8

-r--r--r-- 1 ram  ram  19 Mar  4 16:44 ajinkya

-r--r--r-- 1 root root 19 Mar  4 16:55 sachin

```bash
[root@localhost pgsql_feb22]#

```

read the file:

```bash
[root@localhost pgsql_feb22]# cat ajinkya
```

hi ram how are you


edit the file:

```bash
vi ajinkya

```

esc i


write something


esc : wq (for save)


esc : q!(for not save)


check the difference b/w 2 files:

```bash
[root@localhost pgsql_feb22]# diff ajinkya sachin
```

2,3d1

<

< hi im fine


go onne step back from current path:

```bash
[root@localhost pgsql_feb22]# pwd
```

/var/lib/pgsql_feb22

```bash
[root@localhost pgsql_feb22]# cd ..
[root@localhost lib]# pwd
```

/var/lib

```bash
[root@localhost lib]#

```

display list of users:

```bash
[root@localhost lib]# cat /etc/passwd|grep /bin/bash
```

root:x:0:0:root:/root:/bin/bash

ram:x:1000:1000:redhat:/home/ram:/bin/bash

enterprisedb:x:1002:1002:EDB Postgres Advanced Server:/opt/edb/as10:/bin/bash

efm:x:1003:1003::/var/efm:/bin/bash

santhosh:x:1004:1004::/home/santhosh:/bin/bash

postgres:x:1005:1005::/home/postgres:/bin/bash

oracle:x:1006:1006::/home/oracle:/bin/bash


```sql
create user:
```

```bash
[root@localhost lib]# useradd sachin
[root@localhost lib]#
[root@localhost lib]# cat /etc/passwd|grep sachin
```

sachin:x:1007:1008::/home/sachin:/bin/bash

```bash
[root@localhost lib]#

```

make this user as sudo user:

## Allow root to run any commands anywhere


```bash
vi /etc/sudoers

```

esc i


root    ALL=(ALL)       ALL

santhosh ALL=(ALL)      ALL

postgres ALL=(ALL)      ALL

sachin   ALL=(ALL)      ALL (write this user)


esc : wq


postgres user add and make it as sudo user:

```bash
[root@localhost lib]# useradd postgres

vi /etc/sudoers

```

esc i


root    ALL=(ALL)       ALL

santhosh ALL=(ALL)      ALL

postgres ALL=(ALL)      ALL (make this entry)


esc : wq


chnage owner ship f directory:

```bash
[root@localhost lib]# chown -R postgres:postgres pgsql_feb22

[root@localhost lib]# ls -ltr pgsql_feb22
```

total 8

-r--r--r-- 1 postgres postgres 19 Mar  4 16:55 sachin

-rw----rw- 1 postgres postgres 31 Mar  4 16:58 ajinkya

```bash
[root@localhost lib]#

```

to remove file:

```bash
[root@localhost pgsql_feb22]# rm sachin ajinkya


```

number of process running on operating system:

```bash
[root@localhost lib]# ps -ef
```

UID         PID   PPID  C STIME TTY          TIME CMD

root          1      0  0 16:41 ?        00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 21

root          2      0  0 16:41 ?        00:00:00 [kthreadd]

root          3      2  0 16:41 ?        00:00:00 [ksoftirqd/0]

root          7      2  0 16:41 ?        00:00:00 [migration/0]

root          8      2  0 16:41 ?        00:00:00 [rcu_bh]

root          9      2  0 16:41 ?        00:00:00 [rcuob/0]

root         10      2  0 16:41 ?        00:00:00 [rcuob/1]


UID: username


PID : process id


PPID : for all process ids parent process id will be there.


STIME : start time


TIME : from how long that session running


CMD : what command they are running


filter specific user processes:

```bash
[root@localhost lib]# ps -ef|grep postgres
```

root       2653   2387  0 17:10 pts/0    00:00:00 grep --color=auto postgres

```bash
[root@localhost lib]#

```

count of porcesses :

```bash
[root@localhost lib]# ps -ef|wc -l
```

417

```bash
[root@localhost lib]#


```

current cpu usage:

```bash
[root@localhost lib]#
[root@localhost lib]# top
top - 17:16:08 up 35 min,  2 users,  load average: 0.00, 0.01, 0.05
```

Tasks: 415 total,   2 running, 413 sleeping,   0 stopped,   0 zombie

%Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st


```bash
id : avaible cpu

for exacmple if idle is 15, used cpu 85%.


if idle is 40, that means 60% used.

then start moiotring and take action immedieltey.

```

server uptime:

```bash
[root@localhost lib]# uptime
```

17:18:47 up 37 min,  2 users,  load average: 0.00, 0.01, 0.05

```bash
[root@localhost lib]#

```

memory usage:

```bash
[root@localhost lib]# free -m
```

total        used        free      shared  buff/cache   available

Mem:           3776         138        3476           8         161        3456

Swap:          2048           0        2048

```bash
[root@localhost lib]#

```

number of cpu/cores in the server:

```bash
[root@localhost lib]# lscpu
```

Architecture:          x86_64

CPU op-mode(s):        32-bit, 64-bit

Byte Order:            Little Endian

CPU(s):                1

On-line CPU(s) list:   0

Thread(s) per core:    1


## 11. POSTGRESQL INSTALLATION

we have 2 types of installations.


1.source code installation  :

1a.everything manual

1b.we need to install manually

1c.we need to configure

1d.we need to enable autostartup

1e.we need configure audit log

1f.good thing is in the same server n number of instances we can run, same version of s/w we can install in 2 folders.


2.binaries installation:

everything automated.

```bash
service auto startup default enabled.
```

audit log default enabled.

same version s/w can't install 2 times, second one will override first one.


in realtime 99% using binaries installation, easy for installation.


binaries installation : it will connect to internet/repo and download required rpm files and install.


so some people will call it as yum installation/ rpm installation/binaries installation.


## 12. SOURCE CODE INSTALLATION

->go to folder:  https://www.postgresql.org/ftp/source/


->select version


->download etiher bz or gz big file, once downloaded file will be in ur local downloads.


->downloaded file in my local downloads, but we need this file in linux server.


->to copy files from local server to linux server we have s/w called winscp.


->download winscp and istall.


- https://winscp.net/eng/download.php

once downloaded, double click -next next -finish


copied s/w to /tmp/ using winscp.


```bash
[root@localhost lib]# cd /tmp/
[root@localhost tmp]# ls -ltr
```

total 3016120

-rw-rw-r-- 1 ram    ram      3059705302 Feb 26 00:52 LINUX.X64_193000_db_home.zip

drwxr-x--- 3 oracle Oinstall       4096 Feb 26 07:47 CVU_19.0.0.0.0_oracle

drwxr-xr-x 2 oracle Oinstall          6 Feb 26 07:58 hsperfdata_oracle

drwx------ 3 root   root             16 Mar  4 16:41 systemd-private-b8015f4355eb40d6a6ef19cb082b604f-vmtoolsd.service-TPiYIb

-rw-rw-r-- 1 ram    ram        28794023 Mar  4 17:38 postgresql-14.2.tar.gz


```bash
[root@localhost tmp]# mv postgresql-14.2.tar.gz /var/lib/pgsql_feb22/

[root@localhost tmp]# cd /var/lib/pgsql_feb22/
[root@localhost pgsql_feb22]# ls -ltt
```

total 28128

-rw-rw-r-- 1 ram      ram      28794023 Mar  4 17:38 postgresql-14.2.tar.gz

-rw----rw- 1 postgres postgres       31 Mar  4 16:58 ajinkya

-r--r--r-- 1 postgres postgres       19 Mar  4 16:55 sachin

```bash
[root@localhost pgsql_feb22]#

unzip and untar the file at a time:
```

```bash
[root@localhost pgsql_feb22]# tar -xvf postgresql-14.2.tar.gz

[root@localhost pgsql_feb22]# ls -ltr
```

total 28124

drwxrwxrwx 6 1107 1107     4096 Feb  7 13:25 postgresql-14.2

-rw-rw-r-- 1 ram  ram  28794023 Mar  4 17:38 postgresql-14.2.tar.gz

```bash
[root@localhost pgsql_feb22]#

[root@localhost pgsql_feb22]# rm -rf postgresql-14.2.tar.gz

[root@localhost pgsql_feb22]# cd postgresql-14.2
[root@localhost postgresql-14.2]# pwd
```

/var/lib/pgsql_feb22/postgresql-14.2

```bash
[root@localhost postgresql-14.2]# ls -ltr
```

total 764

-rw-r--r--  1 1107 1107   1213 Feb  7 13:15 README

-rw-r--r--  1 1107 1107   1665 Feb  7 13:15 Makefile

-rw-r--r--  1 1107 1107    277 Feb  7 13:15 HISTORY

-rw-r--r--  1 1107 1107   4259 Feb  7 13:15 GNUmakefile.in

-rw-r--r--  1 1107 1107   1192 Feb  7 13:15 COPYRIGHT

-rw-r--r--  1 1107 1107  85061 Feb  7 13:15 configure.ac

-rwxr-xr-x  1 1107 1107 586668 Feb  7 13:15 configure

-rw-r--r--  1 1107 1107    445 Feb  7 13:15 aclocal.m4

drwxrwxrwx 58 1107 1107   4096 Feb  7 13:24 contrib

drwxrwxrwx  2 1107 1107   4096 Feb  7 13:24 config

drwxrwxrwx  3 1107 1107     82 Feb  7 13:24 doc

-rw-r--r--  1 1107 1107  63953 Feb  7 13:25 INSTALL

drwxrwxrwx 16 1107 1107   4096 Feb  7 13:25 src

```bash
[root@localhost postgresql-14.2]#


```

before installation we need to make sure no one is installed earlier:

by default source code installation location is /usr/local/pgsql.


make sure no pgsql folder exist, that means no one installed earlier.


```bash
[root@localhost postgresql-14.2]# ls -ltr /usr/local/pgsql/
```

ls: cannot access /usr/local/pgsql/: No such file or directory


proceed with installation:

```bash
[root@localhost local]# cd /var/lib/pgsql_feb22/postgresql-14.2
[root@localhost postgresql-14.2]# ls -ltr
```

total 764

-rw-r--r--  1 1107 1107   1213 Feb  7 13:15 README

-rw-r--r--  1 1107 1107   1665 Feb  7 13:15 Makefile

-rw-r--r--  1 1107 1107    277 Feb  7 13:15 HISTORY

-rw-r--r--  1 1107 1107   4259 Feb  7 13:15 GNUmakefile.in

-rw-r--r--  1 1107 1107   1192 Feb  7 13:15 COPYRIGHT

-rw-r--r--  1 1107 1107  85061 Feb  7 13:15 configure.ac

-rwxr-xr-x  1 1107 1107 586668 Feb  7 13:15 configure

-rw-r--r--  1 1107 1107    445 Feb  7 13:15 aclocal.m4

drwxrwxrwx 58 1107 1107   4096 Feb  7 13:24 contrib

drwxrwxrwx  2 1107 1107   4096 Feb  7 13:24 config

drwxrwxrwx  3 1107 1107     82 Feb  7 13:24 doc

-rw-r--r--  1 1107 1107  63953 Feb  7 13:25 INSTALL

drwxrwxrwx 16 1107 1107   4096 Feb  7 13:25 src

```bash
[root@localhost postgresql-14.2]#


as per source code installation instruction to install we need to perfrom 3 activties.


```

configure : this one will check whether all the required libraries avaible in the current linux server or not.

make : all the avaible libraries, paths it will export and it will make s/w ready for installation

make install : direct installation.


->in linux to execute any script ./script name is the syntax.


```bash
[root@localhost postgresql-14.2]# ./configure --without-readline --without-zlib (my demo linux dont have readline and zlib libraries)

in realtime running below command is enoguh:
```


step1: [root@localhost postgresql-14.2]# ./configure


make sure confgiure script not thowing any errors.


below file exit status 0 means success full.


```bash
[root@localhost postgresql-14.2]# cat config.status


```

step2 :[root@localhost postgresql-14.2]# make


step3: [root@localhost postgresql-14.2]# make install


postgresql in addition to main s/w, given few extra features, to get those features we need to install contrib module.


that extra features we will call it as extensions.


step4:  [root@localhost postgresql-14.2]# cd contrib


```bash
[root@localhost contrib]# pwd
```

/var/lib/pgsql_feb22/postgresql-14.2/contrib

```bash
[root@localhost contrib]#

[root@localhost contrib]# make
[root@localhost contrib]# make install

```

Step5:  s/w validation:

```bash
[root@localhost contrib]# cd /usr/local/pgsql/
[root@localhost pgsql]# ls -ltr
```

total 16

drwxr-xr-x 6 root root 4096 Mar  4 18:04 include

drwxr-xr-x 7 root root 4096 Mar  4 18:06 share

drwxr-xr-x 4 root root 4096 Mar  4 18:06 lib

drwxr-xr-x 2 root root 4096 Mar  4 18:06 bin

```bash
[root@localhost pgsql]# cd bin/
[root@localhost bin]# pwd
```

/usr/local/pgsql/bin

```bash
[root@localhost bin]# ls -ltr
```

total 13328

-rwxr-xr-x 1 root root 8737963 Mar  4 18:04 postgres

lrwxrwxrwx 1 root root       8 Mar  4 18:04 postmaster -> postgres

-rwxr-xr-x 1 root root 1009296 Mar  4 18:04 ecpg

-rwxr-xr-x 1 root root  148321 Mar  4 18:04 initdb

-rwxr-xr-x 1 root root  111071 Mar  4 18:04 pg_amcheck

-rwxr-xr-x 1 root root   48771 Mar  4 18:04 pg_archivecleanup

-rwxr-xr-x 1 root root  135995 Mar  4 18:04 pg_basebackup

-rwxr-xr-x 1 root root   94173 Mar  4 18:04 pg_receivewal

-rwxr-xr-x 1 root root   98859 Mar  4 18:04 pg_recvlogical

-rwxr-xr-x 1 root root   67440 Mar  4 18:04 pg_checksums

-rwxr-xr-x 1 root root   43159 Mar  4 18:04 pg_config

-rwxr-xr-x 1 root root   61769 Mar  4 18:04 pg_controldata

-rwxr-xr-x 1 root root   77366 Mar  4 18:04 pg_ctl

-rwxr-xr-x 1 root root  428574 Mar  4 18:04 pg_dump

-rwxr-xr-x 1 root root  193121 Mar  4 18:04 pg_restore

-rwxr-xr-x 1 root root  119893 Mar  4 18:04 pg_dumpall

-rwxr-xr-x 1 root root   71937 Mar  4 18:04 pg_resetwal

-rwxr-xr-x 1 root root  147773 Mar  4 18:04 pg_rewind

-rwxr-xr-x 1 root root   49564 Mar  4 18:04 pg_test_fsync

-rwxr-xr-x 1 root root   43527 Mar  4 18:04 pg_test_timing

-rwxr-xr-x 1 root root  158772 Mar  4 18:04 pg_upgrade

-rwxr-xr-x 1 root root  119653 Mar  4 18:04 pg_verifybackup

-rwxr-xr-x 1 root root  109010 Mar  4 18:04 pg_waldump

-rwxr-xr-x 1 root root  198238 Mar  4 18:04 pgbench

-rwxr-xr-x 1 root root  515805 Mar  4 18:04 psql

-rwxr-xr-x 1 root root   88122 Mar  4 18:04 createdb

-rwxr-xr-x 1 root root   79063 Mar  4 18:04 dropdb

-rwxr-xr-x 1 root root   84292 Mar  4 18:04 createuser

-rwxr-xr-x 1 root root   79001 Mar  4 18:04 dropuser

-rwxr-xr-x 1 root root   79795 Mar  4 18:04 clusterdb

-rwxr-xr-x 1 root root   93329 Mar  4 18:04 vacuumdb

-rwxr-xr-x 1 root root   93107 Mar  4 18:04 reindexdb

-rwxr-xr-x 1 root root   74539 Mar  4 18:04 pg_isready

-rwxr-xr-x 1 root root   54000 Mar  4 18:06 oid2name

-rwxr-xr-x 1 root root   53721 Mar  4 18:06 vacuumlo

```bash
[root@localhost bin]#

[root@localhost bin]# /usr/local/pgsql/bin/pg_config
```

BINDIR = /usr/local/pgsql/bin

DOCDIR = /usr/local/pgsql/share/doc

HTMLDIR = /usr/local/pgsql/share/doc

INCLUDEDIR = /usr/local/pgsql/include

PKGINCLUDEDIR = /usr/local/pgsql/include

INCLUDEDIR-SERVER = /usr/local/pgsql/include/server

LIBDIR = /usr/local/pgsql/lib

PKGLIBDIR = /usr/local/pgsql/lib

LOCALEDIR = /usr/local/pgsql/share/locale

MANDIR = /usr/local/pgsql/share/man

SHAREDIR = /usr/local/pgsql/share

SYSCONFDIR = /usr/local/pgsql/etc

PGXS = /usr/local/pgsql/lib/pgxs/src/makefiles/pgxs.mk

CONFIGURE =  '--without-readline' '--without-zlib'

CC = gcc -std=gnu99

CPPFLAGS = -D_GNU_SOURCE

CFLAGS = -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -O2

CFLAGS_SL = -fPIC

LDFLAGS = -Wl,--as-needed -Wl,-rpath,'/usr/local/pgsql/lib',--enable-new-dtags

LDFLAGS_EX =

LDFLAGS_SL =

LIBS = -lpgcommon -lpgport -lpthread -lrt -ldl -lm

VERSION = PostgreSQL 14.2


to install s/w in a different folder instead of /usr/local/pgsql folder.


```bash
mkdir /opt/pgsql

from the s/w folder:

./configure --prefix=/opt/pgsql/
```

make

make install

```bash
cd contrib
```

make

make install


step6: only installation done, no instance, no services up and running.


to bring up the services and to create instance we need default data files and configuration files.


to keep those files we need one directory, we call that directory as DATA directory.


first create DATA directory where ever u want.


```bash
[root@localhost bin]# mkdir /var/lib/pgsql_feb22/DATA

[root@localhost bin]# cd /var/lib/pgsql_feb22/DATA
[root@localhost DATA]# ls -ltr
```

total 0

```bash
[root@localhost DATA]#

and one more thing to work with postgresql, all folder related postgresql should be owned by postgres user.

[root@localhost DATA]# chown -R postgres:postgres /usr/local/pgsql/
[root@localhost DATA]# chown -R postgres:postgres /var/lib/pgsql_feb22/DATA/


```

to work with postgresql need to switch to postgres user.


```bash
[root@localhost DATA]# su - postgres
```

Last login: Fri Mar  4 16:41:16 PST 2022

```bash
[postgres@localhost ~]$


```

we need to generate default data files and configuration files, we call this concept as database initialization.


to perfom database initialization we have an utility called initdb.


```bash
[postgres@localhost bin]$ /usr/local/pgsql/bin/initdb -D /var/lib/pgsql_feb22/DATA/
```

The files belonging to this database system will be owned by user "postgres".

This user must also own the server process.


The database cluster will be initialized with locale "en_US.UTF-8".

The default database encoding has accordingly been set to "UTF8".

The default text search configuration will be set to "english".


Data page checksums are disabled.


fixing permissions on existing directory /var/lib/pgsql_feb22/DATA ... ok

creating subdirectories ... ok

selecting dynamic shared memory implementation ... posix

selecting default max_connections ... 100

selecting default shared_buffers ... 128MB

selecting default time zone ... America/Los_Angeles

creating configuration files ... ok

running bootstrap script ... ok

performing post-bootstrap initialization ... ok

syncing data to disk ... ok


initdb: warning: enabling "trust" authentication for local connections

You can change this by editing pg_hba.conf or using the option -A, or

--auth-local and --auth-host, the next time you run initdb.


Success. You can now start the database server using:


/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ -l logfile start


```bash
[postgres@localhost bin]$


[postgres@localhost bin]$ cd /var/lib/pgsql_feb22/DATA
[postgres@localhost DATA]$ ls -ltr
```

total 56

-rw------- 1 postgres postgres     3 Mar  4 18:19 PG_VERSION

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_twophase

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_tblspc

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_stat_tmp

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_stat

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_snapshots

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_serial

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_replslot

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_notify

drwx------ 4 postgres postgres    34 Mar  4 18:19 pg_multixact

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_dynshmem

drwx------ 2 postgres postgres     6 Mar  4 18:19 pg_commit_ts

-rw------- 1 postgres postgres 28785 Mar  4 18:19 postgresql.conf

-rw------- 1 postgres postgres    88 Mar  4 18:19 postgresql.auto.conf

-rw------- 1 postgres postgres  1636 Mar  4 18:19 pg_ident.conf

-rw------- 1 postgres postgres  4789 Mar  4 18:19 pg_hba.conf

drwx------ 2 postgres postgres    17 Mar  4 18:19 pg_xact

drwx------ 3 postgres postgres    58 Mar  4 18:19 pg_wal

drwx------ 2 postgres postgres    17 Mar  4 18:19 pg_subtrans

drwx------ 2 postgres postgres  4096 Mar  4 18:19 global

drwx------ 5 postgres postgres    38 Mar  4 18:19 base

drwx------ 4 postgres postgres    65 Mar  4 18:19 pg_logical


to start/stop postgresql services we have an utility called pg_ctl.


```bash
[postgres@localhost DATA]$ /usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ status
```

pg_ctl: no server running

```bash
[postgres@localhost DATA]$

```

/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ start

/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ stop

/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ restart

/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ reload [no downtime, any time can run, just refresh the configuration files]


->by default we are starting postmaster process, it will start child processes.


```bash
[postgres@localhost ~]$ ps -ef|grep postgres
```

postgres  15843      1  0 18:26 ?        00:00:00 /usr/local/pgsql/bin/postgres -D /var/lib/pgsql_feb22/DATA

postgres  15845  15843  0 18:26 ?        00:00:00 postgres: checkpointer

postgres  15846  15843  0 18:26 ?        00:00:00 postgres: background writer

postgres  15847  15843  0 18:26 ?        00:00:00 postgres: walwriter

postgres  15848  15843  0 18:26 ?        00:00:00 postgres: autovacuum launcher

postgres  15849  15843  0 18:26 ?        00:00:00 postgres: stats collector

postgres  15850  15843  0 18:26 ?        00:00:00 postgres: logical replication launcher


## 8. SETUP ENVIRONMENT VARIABLES

```bash
[postgres@localhost ~]$ vi .bash_profile
export PATH=$PATH:/usr/local/pgsql/bin/
export PGDATA=/var/lib/pgsql_feb22/DATA/

[postgres@localhost ~]$ exit
```

logout

```bash
[root@localhost DATA]# su - postgres
```

Last login: Fri Mar  4 18:25:12 PST 2022 on pts/0

-bash-4.2$


```bash
-bash-4.2$ pg_ctl status
pg_ctl: server is running (PID: 15843)
```

/usr/local/pgsql/bin/postgres "-D" "/var/lib/pgsql_feb22/DATA"


## 15. DATA DIRECTORY EXPLANATION

```bash
[root@localhost ~]# cd /etc/rc.d/
[root@localhost rc.d]#

[root@localhost rc.d]# vi rc.local

```

su - postgres -c 'pg_ctl -D $PGDATA start'


```bash
[root@localhost rc.d]# chmod +x /etc/rc.d/rc.local

```

## 14. UNINSTALLATION

->stop services


```bash
[root@localhost ~]# ps -ef|grep postgres
```

postgres   1229      1  0 16:39 ?        00:00:00 /usr/local/pgsql/bin/postgres -D /var/lib/pgsql_feb22/DATA

postgres   1299   1229  0 16:39 ?        00:00:00 postgres: checkpointer

postgres   1300   1229  0 16:39 ?        00:00:00 postgres: background writer

postgres   1301   1229  0 16:39 ?        00:00:00 postgres: walwriter

postgres   1302   1229  0 16:39 ?        00:00:00 postgres: autovacuum launcher

postgres   1303   1229  0 16:39 ?        00:00:00 postgres: stats collector

postgres   1304   1229  0 16:39 ?        00:00:00 postgres: logical replication launcher

root       2426   2396  0 16:40 pts/0    00:00:00 grep --color=auto postgres

```bash
[root@localhost ~]# su - postgres
```

Last login: Sat Mar  5 16:39:17 PST 2022

```bash
-bash-4.2$ pg_ctl stop
```

waiting for server to shut down.... done

server stopped


->[root@localhost ~]# cd /var/lib/pgsql_feb22/


```bash
[root@localhost pgsql_feb22]# ls -ltr
```

total 8

drwxrwxrwx  6     1107     1107 4096 Mar  4 17:54 postgresql-14.2

drwx------ 19 postgres postgres 4096 Mar  5 16:40 DATA

```bash
[root@localhost pgsql_feb22]# cd postgresql-14.2/
[root@localhost postgresql-14.2]# pwd
```

/var/lib/pgsql_feb22/postgresql-14.2

```bash
[root@localhost postgresql-14.2]# ls -ltr
```

total 1224

-rw-r--r--  1 1107 1107   1213 Feb  7 13:15 README

-rw-r--r--  1 1107 1107   1665 Feb  7 13:15 Makefile

-rw-r--r--  1 1107 1107    277 Feb  7 13:15 HISTORY

-rw-r--r--  1 1107 1107   4259 Feb  7 13:15 GNUmakefile.in

-rw-r--r--  1 1107 1107   1192 Feb  7 13:15 COPYRIGHT

-rw-r--r--  1 1107 1107  85061 Feb  7 13:15 configure.ac

-rwxr-xr-x  1 1107 1107 586668 Feb  7 13:15 configure

-rw-r--r--  1 1107 1107    445 Feb  7 13:15 aclocal.m4

drwxrwxrwx 58 1107 1107   4096 Feb  7 13:24 contrib

drwxrwxrwx  2 1107 1107   4096 Feb  7 13:24 config

drwxrwxrwx  3 1107 1107     82 Feb  7 13:24 doc

-rw-r--r--  1 1107 1107  63953 Feb  7 13:25 INSTALL

-rwxr-xr-x  1 root root  39607 Mar  4 17:54 config.status

-rw-r--r--  1 root root   4259 Mar  4 17:54 GNUmakefile

drwxrwxrwx 16 1107 1107   4096 Mar  4 17:54 src

-rw-r--r--  1 root root 418922 Mar  4 17:54 config.log

```bash
[root@localhost postgresql-14.2]#

[root@localhost postgresql-14.2]# make clean
[root@localhost postgresql-14.2]# make uninstall
[root@localhost postgresql-14.2]# cd contrib/
[root@localhost contrib]# make clean
[root@localhost contrib]# make uninstall

```

check all ur utilities will be gone after uninstallation. data directory still remain.


```bash
[root@localhost contrib]# cd /usr/local/pgsql/bin/
[root@localhost bin]# ls -ltr
```

total 0

```bash
[root@localhost bin]#

[root@localhost ~]# rm -rf /usr/local/pgsql/
[root@localhost ~]# rm -rf /var/lib/pgsql_feb22/DATA/

```

## 13. BINARIES INSTALLATION

->check whether already some one installed same s/w or not.


```bash
[root@mariadb1 ~]# rpm -qa|grep postgres
[root@mariadb1 ~]#

```

->https://www.postgresql.org/download/linux/redhat/


repository: its folder , where all the s/w and dependent s/w files are there.


->create/install repository.


```bash
[root@mariadb1 ~]# wget https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
--2022-03-05 19:58:12--  https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Resolving download.postgresql.org (download.postgresql.org)... 72.32.157.246, 217.196.149.55, 147.75.85.69, ...

Connecting to download.postgresql.org (download.postgresql.org)|72.32.157.246|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 8260 (8.1K) [application/x-redhat-package-manager]

Saving to: ‘pgdg-redhat-repo-latest.noarch.rpm’


100%[===================================================================================================================>] 8,260       --.-K/s   in 0.001s


2022-03-05 19:58:13 (7.03 MB/s) - ‘pgdg-redhat-repo-latest.noarch.rpm’ saved [8260/8260]


```bash
[root@mariadb1 ~]#


[root@mariadb1 ~]# rpm -ivh pgdg-redhat-repo-latest.noarch.rpm
```

Preparing...                          ################################# [100%]

Updating / installing...

1:pgdg-redhat-repo-42.0-23         ################################# [100%]

```bash
[root@mariadb1 ~]#


```

->check whether repo created or not


```bash
[root@mariadb1 ~]# cd /etc/yum.repos.d/
[root@mariadb1 yum.repos.d]# ls -ltr
```

total 80


```bash
[root@mariadb1 yum.repos.d]# cat pgdg-redhat-all.repo

[root@mariadb1 yum.repos.d]# yum install -y postgresql14-server postgresql14-contrib

```

Resolving Dependencies

--> Running transaction check

---> Package postgresql14-contrib.x86_64 0:14.2-1PGDG.rhel7 will be installed

--> Processing Dependency: postgresql14-libs(x86-64) = 14.2-1PGDG.rhel7 for package: postgresql14-contrib-14.2-1PGDG.rhel7.x86_64

--> Processing Dependency: postgresql14(x86-64) = 14.2-1PGDG.rhel7 for package: postgresql14-contrib-14.2-1PGDG.rhel7.x86_64

---> Package postgresql14-server.x86_64 0:14.2-1PGDG.rhel7 will be installed

--> Running transaction check

---> Package postgresql14.x86_64 0:14.2-1PGDG.rhel7 will be installed

---> Package postgresql14-libs.x86_64 0:14.2-1PGDG.rhel7 will be installed

--> Finished Dependency Resolution


Dependencies Resolved


=============================================================================================================================================================

Package                                      Arch                           Version                                    Repository                      Size

=============================================================================================================================================================

Installing:

postgresql14-contrib                         x86_64                         14.2-1PGDG.rhel7                           pgdg14                         682 k

postgresql14-server                          x86_64                         14.2-1PGDG.rhel7                           pgdg14                         5.5 M

Installing for dependencies:

postgresql14                                 x86_64                         14.2-1PGDG.rhel7                           pgdg14                         1.5 M

postgresql14-libs                            x86_64                         14.2-1PGDG.rhel7                           pgdg14                         267 k


Transaction Summary

=============================================================================================================================================================

Install  2 Packages (+2 Dependent packages)


Total download size: 7.9 M

Installed size: 33 M

Downloading packages:

(1/4): postgresql14-contrib-14.2-1PGDG.rhel7.x86_64.rpm                                                                               | 682 kB  00:00:02

(2/4): postgresql14-libs-14.2-1PGDG.rhel7.x86_64.rpm                                                                                  | 267 kB  00:00:00

(3/4): postgresql14-14.2-1PGDG.rhel7.x86_64.rpm                                                                                       | 1.5 MB  00:00:04

(4/4): postgresql14-server-14.2-1PGDG.rhel7.x86_64.rpm                                                                                | 5.5 MB  00:00:10

-------------------------------------------------------------------------------------------------------------------------------------------------------------

Total                                                                                                                        602 kB/s | 7.9 MB  00:00:13

Running transaction check

Running transaction test

Transaction test succeeded

Running transaction

Warning: RPMDB altered outside of yum.

** Found 2 pre-existing rpmdb problem(s), 'yum check' output follows:

pg-auto-failover16_14-1.6.3-1.el7.x86_64 has missing requires of postgresql14-contrib

pg-auto-failover16_14-1.6.3-1.el7.x86_64 has missing requires of postgresql14-server

Installing : postgresql14-libs-14.2-1PGDG.rhel7.x86_64                                                                                                 1/4

Installing : postgresql14-14.2-1PGDG.rhel7.x86_64                                                                                                      2/4

Installing : postgresql14-server-14.2-1PGDG.rhel7.x86_64                                                                                               3/4

Installing : postgresql14-contrib-14.2-1PGDG.rhel7.x86_64                                                                                              4/4

Verifying  : postgresql14-contrib-14.2-1PGDG.rhel7.x86_64                                                                                              1/4

Verifying  : postgresql14-libs-14.2-1PGDG.rhel7.x86_64                                                                                                 2/4

Verifying  : postgresql14-server-14.2-1PGDG.rhel7.x86_64                                                                                               3/4

Verifying  : postgresql14-14.2-1PGDG.rhel7.x86_64                                                                                                      4/4


Installed:

postgresql14-contrib.x86_64 0:14.2-1PGDG.rhel7                                postgresql14-server.x86_64 0:14.2-1PGDG.rhel7


Dependency Installed:

postgresql14.x86_64 0:14.2-1PGDG.rhel7                                     postgresql14-libs.x86_64 0:14.2-1PGDG.rhel7


Complete!


for manual approach :

download from the below link.


- https://download.postgresql.org/pub/repos/yum/14/redhat/rhel-7.2-x86_64/

postgresql14-contrib-14.2-1PGDG.rhel7.x86_64.rpm

postgresql14-libs-14.2-1PGDG.rhel7.x86_64.rpm

postgresql14-14.2-1PGDG.rhel7.x86_64.rpm

postgresql14-server-14.2-1PGDG.rhel7.x86_64.rpm


install in order:

```bash
 rpm -ivh postgresql14-libs-14.2-1PGDG.rhel7.x86_64     
 rpm -ivh postgresql14-14.2-1PGDG.rhel7.x86_64          
 rpm -ivh postgresql14-server-14.2-1PGDG.rhel7.x86_64   
 rpm -ivh postgresql14-contrib-14.2-1PGDG.rhel7.x86_64  


```

validation:

```bash
[root@mariadb1 yum.repos.d]# rpm -qa|grep postgres
```

postgresql14-14.2-1PGDG.rhel7.x86_64

postgresql14-contrib-14.2-1PGDG.rhel7.x86_64

postgresql14-server-14.2-1PGDG.rhel7.x86_64

postgresql14-libs-14.2-1PGDG.rhel7.x86_64


->default installation location is


```bash
[root@mariadb1 yum.repos.d]# cd /usr/pgsql-14/bin/
[root@mariadb1 bin]# ls -ltr
```

total 12432

-rwxr-xr-x 1 root root  774888 Nov 11 09:47 pg_autoctl

-rwxr-xr-x 1 root root    9655 Feb  9 03:12 postgresql-14-setup

-rwxr-xr-x 1 root root    2172 Feb  9 03:12 postgresql-14-check-db-dir

-rwxr-xr-x 1 root root   46024 Feb  9 03:12 vacuumlo

-rwxr-xr-x 1 root root   84728 Feb  9 03:12 vacuumdb

-rwxr-xr-x 1 root root   80384 Feb  9 03:12 reindexdb

-rwxr-xr-x 1 root root  669024 Feb  9 03:12 psql

-rwxr-xr-x 1 root root 8183432 Feb  9 03:12 postgres

-rwxr-xr-x 1 root root  100640 Feb  9 03:12 pg_waldump

-rwxr-xr-x 1 root root   96712 Feb  9 03:12 pg_verifybackup

-rwxr-xr-x 1 root root  151048 Feb  9 03:12 pg_upgrade

-rwxr-xr-x 1 root root   37296 Feb  9 03:12 pg_test_timing

-rwxr-xr-x 1 root root   45832 Feb  9 03:12 pg_test_fsync

-rwxr-xr-x 1 root root  134592 Feb  9 03:12 pg_rewind

-rwxr-xr-x 1 root root  180656 Feb  9 03:12 pg_restore

-rwxr-xr-x 1 root root   66816 Feb  9 03:12 pg_resetwal

-rwxr-xr-x 1 root root   88928 Feb  9 03:12 pg_recvlogical

-rwxr-xr-x 1 root root   84736 Feb  9 03:12 pg_receivewal

-rwxr-xr-x 1 root root   67296 Feb  9 03:12 pg_isready

-rwxr-xr-x 1 root root  109976 Feb  9 03:12 pg_dumpall

-rwxr-xr-x 1 root root  413848 Feb  9 03:12 pg_dump

-rwxr-xr-x 1 root root   71088 Feb  9 03:12 pg_ctl

-rwxr-xr-x 1 root root   58056 Feb  9 03:12 pg_controldata

-rwxr-xr-x 1 root root   41248 Feb  9 03:12 pg_config

-rwxr-xr-x 1 root root   62576 Feb  9 03:12 pg_checksums

-rwxr-xr-x 1 root root  181032 Feb  9 03:12 pgbench

-rwxr-xr-x 1 root root  126944 Feb  9 03:12 pg_basebackup

-rwxr-xr-x 1 root root   41464 Feb  9 03:12 pg_archivecleanup

-rwxr-xr-x 1 root root  101632 Feb  9 03:12 pg_amcheck

-rwxr-xr-x 1 root root   46240 Feb  9 03:12 oid2name

-rwxr-xr-x 1 root root  134832 Feb  9 03:12 initdb

-rwxr-xr-x 1 root root   67424 Feb  9 03:12 dropuser

-rwxr-xr-x 1 root root   67488 Feb  9 03:12 dropdb

-rwxr-xr-x 1 root root   76240 Feb  9 03:12 createuser

-rwxr-xr-x 1 root root   75896 Feb  9 03:12 createdb

-rwxr-xr-x 1 root root   71784 Feb  9 03:12 clusterdb

lrwxrwxrwx 1 root root       8 Mar  5 20:05 postmaster -> postgres


perfrom database initialization:

```bash
[root@mariadb1 bin]# /usr/pgsql-14/bin/postgresql-14-setup initdb
```

Initializing database ... OK


```bash
[root@mariadb1 bin]# cd /var/lib/pgsql/14/data/
[root@mariadb1 data]# ls -ltr
```

total 56

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_twophase

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_tblspc

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_stat_tmp

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_stat

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_snapshots

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_serial

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_replslot

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_notify

drwx------ 4 postgres postgres    34 Mar  5 20:13 pg_multixact

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_dynshmem

drwx------ 2 postgres postgres     6 Mar  5 20:13 pg_commit_ts

-rw------- 1 postgres postgres     3 Mar  5 20:13 PG_VERSION

-rw------- 1 postgres postgres 28776 Mar  5 20:13 postgresql.conf

-rw------- 1 postgres postgres    88 Mar  5 20:13 postgresql.auto.conf

-rw------- 1 postgres postgres  1636 Mar  5 20:13 pg_ident.conf

-rw------- 1 postgres postgres  4577 Mar  5 20:13 pg_hba.conf

drwx------ 2 postgres postgres    17 Mar  5 20:13 pg_xact

drwx------ 3 postgres postgres    58 Mar  5 20:13 pg_wal

drwx------ 2 postgres postgres    17 Mar  5 20:13 pg_subtrans

drwx------ 2 postgres postgres  4096 Mar  5 20:13 global

drwx------ 5 postgres postgres    38 Mar  5 20:13 base

drwx------ 4 postgres postgres    65 Mar  5 20:13 pg_logical

drwx------ 2 postgres postgres     6 Mar  5 20:13 log


->to stop /start postgresql services we can use either pg_ctl or system created service as well.


```bash
[root@mariadb1 data]# systemctl list-unit-files|grep postgres
```

postgresql-14.service                                                   disabled

```bash
[root@mariadb1 data]#
[root@mariadb1 data]# systemctl start postgresql-14

[root@mariadb1 data]# systemctl status postgresql-14
```

● postgresql-14.service - PostgreSQL 14 database server

Loaded: loaded (/usr/lib/systemd/system/postgresql-14.service; disabled; vendor preset: disabled)

Active: active (running) since Sat 2022-03-05 20:15:35 EST; 6s ago

Docs: https://www.postgresql.org/docs/14/static/

Process: 10510 ExecStartPre=/usr/pgsql-14/bin/postgresql-14-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)

Main PID: 10515 (postmaster)

CGroup: /system.slice/postgresql-14.service

├─10515 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/

├─10517 postgres: logger

├─10519 postgres: checkpointer

├─10520 postgres: background writer

├─10521 postgres: walwriter

├─10522 postgres: autovacuum launcher

├─10523 postgres: stats collector

└─10524 postgres: logical replication launcher


Mar 05 20:15:35 mariadb1 systemd[1]: Starting PostgreSQL 14 database server...

Mar 05 20:15:35 mariadb1 postmaster[10515]: 2022-03-05 20:15:35.605 EST [10515] LOG:  redirecting log output to logging collector process

Mar 05 20:15:35 mariadb1 postmaster[10515]: 2022-03-05 20:15:35.605 EST [10515] HINT:  Future log output will appear in directory "log".

Mar 05 20:15:35 mariadb1 systemd[1]: Started PostgreSQL 14 database server.


### binaries uninstallation

```bash
[root@mariadb1 data]# systemctl stop postgresql-14
[root@mariadb1 data]# yum remove -y postgresql14-server postgresql14-contrib

[root@mariadb1 data]# rpm -qa|grep postgres
```

postgresql14-14.2-1PGDG.rhel7.x86_64

postgresql14-libs-14.2-1PGDG.rhel7.x86_64


```bash
[root@mariadb1 data]# rpm -e postgresql14-14.2-1PGDG.rhel7.x86_64 postgresql14-libs-14.2-1PGDG.rhel7.x86_64

[root@mariadb1 data]# rpm -qa|grep postgres
[root@mariadb1 data]#

[root@mariadb1 data]# rm -rf /var/lib/pgsql/14/data/

```

## 16. USER MANAGEMENT

amazon EC2 instance creation:


->create login for aws console


->vpc (network ip range for the client /company) [192.0.0.0]


->number of subnet groups (ip address range) [192.168.0.0 for backup, 192.167.0.0 for network team, 192.166.0.0 for db team]


->any server ip address 4 digits like below


1.1.1.1


[0-255].[0-255].[0-255].[0-255]


### AWS RDS PostgreSQL

```bash
-bash-4.2$ psql -h pgsqlfeb22.crfk9g7requz.us-east-1.rds.amazonaws.com -U postgres -d postgres
```

Password for user postgres:

```bash
psql (14.2, server 14.1)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
```

Type "help" for help.


```sql
postgres=>


```

->uinstallation

->DATA directory explanation


```bash
-bash-4.2$ cd /var/lib/pgsql/14/data/

```

pg_twophase : potgresql by default auto commit, commit means completed. eventhough autom committed still we can break the rule by running


queries in BTET mode. in this mode DATA will be managed to acording 2 phases.


pg_snapshots : in postgresql we have concept called snapshots, this is equal to AWR in oracle.


pg_serial : to manange all serial transactions information to handle deadlocks.


pg_notify : this one contains all the notifications, errors, warnings related messages.


pg_dynshmem : this will track run time shared_buffers utilization.


pg_commit_ts : all committed transactions information will be tracked here.


PG_VERSION : current database version


```bash
-bash-4.2$ cat PG_VERSION
```

14


pg_tblspc : only user created tablespaces symbolic/soft links will be managed here.


pg_stat : all the objects statistics will be managed here.


pg_replslot : to manage all the transactions until copied into the standby server.


postgresql.conf : this is the main configuration file, to manage resources. it is just like a pfile in oracle/my.conf in mysql.


postgresql.auto.conf :normally we will configure all resources in postgresql.conf file, but at the same time we have one more option that is


```sql
alter system set command from the database level, parameters which are changed with alter system set command, will be sotred in this file.

```

pg_hba.conf : to manage or prevent remote/local connections. to perfrom hostbased authentication.


pg_ident.conf : some times os level user and db level user will be same, in that case we need to make user entry in this file.


especially for LDAP users.


pg_wal : this folder contains all the wal files.


pg_subtrans : all the sub queries information will be managed here.


global : it will contains global objects data (users, user access rights, tablespaces)


base : it will contains actual data, inside the base folder seperate folder for each database, under those folders data files for objects.


pg_logical : logical replication related data will be managed here.


log : as it is binaries installation, by default audit log will be enabled, log files will be stored here.


current_logfiles : it will display current log files.


```bash
-bash-4.2$ cat current_logfiles
```

stderr log/postgresql-Sun.log


postmaster.opts : how we have started services, it will give that option.


```bash
-bash-4.2$ cat postmaster.opts
```

/usr/pgsql-14/bin/postgres "-D" "/var/lib/pgsql/14/data/"


postmaster.pid: postmaster details


```bash
-bash-4.2$ cat postmaster.pid
```

4712

/var/lib/pgsql/14/data

1646613754 (timestamp: postmaster startup)

5432 : postgresql running port

/var/run/postgresql (socket file location)

localhost : local host it is running

103668571: kernel related memory parameter

ready: db system is ready for connectivity


pg_stat_tmp : all the temp stats will be managed here.


## 20. METADATA TABLES/VIEWS

one postgresql instance means one DATA directory & one port.


under instance


->multiple databases


->under each database multiple schemas (schema different,user different)


->under each schema multiple objects (tables,functions,procedures,views...)


### Database Connecitivity

->to connect database we will use psql utility.


```bash
-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=#


```

->check the connectivity information.


```sql
postgres=# \c
```

You are now connected to database "postgres" as user "postgres".

```sql
postgres=#

```

->list of databases:


```sql
postgres=# \l
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges

-----------+----------+----------+-------------+-------------+-----------------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

(3 rows)


owner: user who created/who can has all permissions on the object.


->list of users:


```sql
postgres=# \du
```

List of roles

Role name |                         Attributes                         | Member of

-----------+------------------------------------------------------------+-----------

postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}


```sql
postgres=#

```

we will assign attributes to users to get additional privileges or permissions.


SUPERUSER (he can do anything)| NOSUPERUSER  (default nosuper user)

CREATEDB(user can create database) | NOCREATEDB

CREATEROLE (user can create another user)| NOCREATEROLE

INHERIT | NOINHERIT

LOGIN | NOLOGIN

REPLICATION | NOREPLICATION

BYPASSRLS | NOBYPASSRLS

CONNECTION LIMIT connlimit

[ ENCRYPTED ] PASSWORD 'password' | PASSWORD NULL

VALID UNTIL 'timestamp'


### object management

switch database:


```sql
postgres=# \c template1
```

You are now connected to database "template1" as user "postgres".

```sql
template1=# \c template0
```

connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  database "template0" is not currently accepting connections

Previous connection kept

```sql
template1=#


```

templat0 we will call it as dumb database, it will not allow any connections.


```sql
template1=# create database rajdb;
CREATE DATABASE
template1=# \l
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges

-----------+----------+----------+-------------+-------------+-----------------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

rajdb     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

schema : namespace : just for naming only.


list of schemas:


rajdb=# \dn

List of schemas

Name  |  Owner

--------+----------

public | postgres

(1 row)


by default under every database public schema will be there.


```sql
create schema:
```

rajdb=# create schema ram_schema;

```sql
CREATE SCHEMA

```

rajdb=# \dn

List of schemas

Name    |  Owner

------------+----------

public     | postgres

ram_schema | postgres

(2 rows)


check list of tables:


if u not mention schema name , it will check in public schema.


rajdb=# \dt

Did not find any relations.

rajdb=#


rajdb=# \dt ram_schema.*;

Did not find any relation named "ram_schema.*".


rajdb=# create table ram_schema.emp(eno int,ename varchar);

```sql
CREATE TABLE
rajdb=# \dt ram_schema.*;
```

List of relations

Schema   | Name | Type  |  Owner

------------+------+-------+----------

ram_schema | emp  | table | postgres

(1 row)


rajdb=#


rajdb=# insert into ram_schema.emp values(1,'ram'),(2,'sam'),(3,'jan'),(4,'bheem');

```sql
INSERT 0 4
rajdb=# create view ram_schema.emm_v as select * from ram_schema.emp;
CREATE VIEW
```

rajdb=#


list of views:


rajdb=# \dv ram_schema.*;

List of relations

Schema   | Name  | Type |  Owner

------------+-------+------+----------

ram_schema | emm_v | view | postgres

(1 row)


list of functions: \df


list of sequences : \ds


list of tablespaces : \db


### create user

rajdb=# create user rambabu;

```sql
CREATE ROLE
```

rajdb=# \du

List of roles

Role name |                         Attributes                         | Member of

-----------+------------------------------------------------------------+-----------

postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

rambabu   |                                                            | {}


reset password for user:


rajdb=# alter user rambabu with password 'rambabu';

```sql
ALTER ROLE


```

make user as superuser:


rajdb=# alter user rambabu superuser;

```sql
ALTER ROLE
```

rajdb=# \du rambabu

List of roles

Role name | Attributes | Member of

-----------+------------+-----------

rambabu   | Superuser  | {}


if u gratn super user, no need to give any other permission.


make  no superuser:

rajdb=# alter user rambabu nosuperuser;

```sql
ALTER ROLE
```

rajdb=# \du rambabu

List of roles

Role name | Attributes | Member of

-----------+------------+-----------

rambabu   |            | {}


give createdb attribute:

rajdb=# alter user rambabu createdb;

```sql
ALTER ROLE
```

rajdb=# \du rambabu

List of roles

Role name | Attributes | Member of

-----------+------------+-----------

rambabu   | Create DB  | {}


```sql
set password expiration:
```

rajdb=# alter user rambabu valid until '06-08-2022';

```sql
ALTER ROLE
```

rajdb=# \du rambabu

List of roles

Role name |                 Attributes                  | Member of

-----------+---------------------------------------------+-----------

rambabu   | Create DB                                  +| {}

| Password valid until 2022-06-08 00:00:00-04 |


```sql
set connection limit for user:
```

rajdb=# alter user rambabu connection limit 10;

```sql
ALTER ROLE
```

rajdb=# \du rambabu

List of roles

Role name |                 Attributes                  | Member of

-----------+---------------------------------------------+-----------

rambabu   | Create DB                                  +| {}

| 10 connections                             +|

| Password valid until 2022-06-08 00:00:00-04 |


chnage ownership of schema:


rajdb=# alter schema ram_schema owner to rambabu;

```sql
ALTER SCHEMA
```

rajdb=# \dn

List of schemas

Name    |  Owner

------------+----------

public     | postgres

ram_schema | rambabu


rename schema:


rajdb=# alter schema ram_schema rename to raj_schema;

```sql
ALTER SCHEMA
```

rajdb=# \dn

List of schemas

Name    |  Owner

------------+----------

public     | postgres

raj_schema | rambabu

(2 rows)


db owner change:


rajdb=# alter database rajdb owner to rambabu;

```sql
ALTER DATABASE
```

rajdb=# \l+ rajdb

List of databases

Name  |  Owner  | Encoding |   Collate   |    Ctype    | Access privileges |  Size   | Tablespace | Description

-------+---------+----------+-------------+-------------+-------------------+---------+------------+-------------

rajdb | rambabu | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                   | 8793 kB | pg_default |

(1 row)


## 17. AUDITLOG MANAGEMENT

to track each and every operation happening at db level, we need to enable audit log.


by default in source code installation it is in disable state.


by default in binaries installation it is in enabled state.


to enable audit log make below changes in postgresql.conf file:


log_destination = 'stderr' (all meesages in the text format)


csvlog: log will be in excel sheet format

syslog: only system related errors

eventlog: only events will be recorded.


logging_collector = on (logger process will up and running and collect metrics)


log_directory = 'pg_log' (with the same name directory will be created under DATA directory)


log_filename = ''postgresql-%Y-%m-%d_%H%M%S.log'


log_rotation_age = 1d


log_rotation_size = 10MB


log_min_duration_statement = 0 (it will capture all the statements in log)


-it is in milli seconds, if i mention 1000 , queries execution time more than 1 second, will be captured.


->to record queries taking more than 1 minute, 60000 ms-->1 minute


log_connections = on

log_disconnections = on

log_line_prefix = '%a %u %d %h %p'

log_statement = 'all'


```bash
[root@mariadb1 ~]# systemctl restart postgresql-14

```

login to database with specific user:

```bash
-bash-4.2$ psql -U postgres -d rajdb


```

housekeeping of audit logs:

->more than 30 days files will be removed


->more than 3 days files compressing


->now we have only 3 days files in normal mode.


->linux given good command zcat to read the zip file.


->user and role both are same, but by default user can login to database, role can have only set of access rights, and no login.


## 18. HOSTBASED AUTHENTICATION

->to prevent or manage remote/local connections, we will use hostbased authentication.


->by making entries in pg_hba.conf file


->if we make change to this file, we need to reload services to reflect.


->by default postgresql will nt allow any remote connections/local connections except postgres user local connection.


->if any new database created, new user created, that user need to be keep in pg_hba.conf file, then only that user can access database.


->so if any db /user created, immedieltey deveopers/application team will request for pg_hba.conf file entry.


->before making entries in pg_hba.conf file proper approvals mandatory.


below is the header part of file:


# TYPE  DATABASE        USER            ADDRESS                 METHOD


TYPE : always 2 values will come 1.local, 2.host


local : if conenction is local to db server, type will be local.


host  : if connection remote to db server, type will be host.


DATABASE: which database user need access/user want to connect


USER: which user want to acces database


ADDRESS: if type is local , address can be empty, if type is host, remote server/cleinet ip need to mention


METHOD :

trust: that specific connection will not require any password to connect database


scram-sha256 : to make any conenction , with mandatory password, we need to meentions scram-sha256(it is password encryption)


reject: blindly if u want to reject any connections.


peer : this is only if database level user & os level user having same name, in that case connection will allow.


1.# "local" is for Unix domain socket connections only

local   all             all                                     peer  (postgres)


2.# IPv4 local connections:


host    all             all             127.0.0.1/32            scram-sha-256 [this is also local conenctions]


3. user rambabu, dbname: rajdb (only rambabu can connect rajdb without password)


local   rajdb           rambabu                                 trust


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/ reload
```

server signaled


if no entry for any user , below errror will come.


```bash
-bash-4.2$ psql
```

psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  no pg_hba.conf entry for host "[local]", user "postgres", database "postgres", no encryption


```bash
-bash-4.2$ psql -d rajdb -U rambabu
psql (14.2)
```

Type "help" for help.


rajdb=> \c

You are now connected to database "rajdb" as user "rambabu".

rajdb=>


4.rambabu user need to enter password:


local   rajdb           rambabu                                 scram-sha-256


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/ reload
```

server signaled

-bash-4.2$


if u give wrong password below error message will come.


```bash
-bash-4.2$ psql -d rajdb -U rambabu
```

Password for user rambabu:

psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  password authentication failed for user "rambabu"


5.blindly want to reject connections from tambabu user.


local   rajdb           rambabu                                 reject


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/ reload
```

server signaled


```bash
-bash-4.2$ psql -d rajdb -U rambabu
```

psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  pg_hba.conf rejects connection for host "[local]", user "rambabu", database "rajdb", no encryption

-bash-4.2$


6.if u need to allow multiple/unlimite ip address we will follow cidr range.


normally ip addresses cidr will be like below design.


[0-255].[0-255].[0-255].[0-255]


255.255.255.255/32 [exact ip address need to match]


255.255.255.0/24 [if first 3 postions matching, last one any number ok upto 0 to 255]


255.255.0.0/16 [first 2 positions should match, next 2 positions anything ok]


255.0.0.0/8 [first 1 position need to match, next 3 anything fine]


0.0.0.0/0 [anything ok ]


host    all             all             0.0.0.0/0              scram-sha-256


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/ reload
```

server signaled


7]all local connections should allow without password.


local   all             all                                     trust


8]what ever the permission applied to rambabu, same thing need to apply for 1000 users.


first grant rambabu to other users.


```sql
postgres=# grant rambabu to shiva;
GRANT ROLE
postgres=# grant rambabu esw

postgres=# grant rambabu to eswar;
GRANT ROLE
postgres=# grant rambabu to tarun;
GRANT ROLE
postgres=# \du
```

List of roles

Role name |                         Attributes                         | Member of

-----------+------------------------------------------------------------+-----------

eswar     |                                                            | {rambabu}

postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

rambabu   | Create DB                                                 +| {}

| 10 connections                                            +|

| Password valid until 2022-06-08 00:00:00-04                |

shiva     |                                                            | {rambabu}

tarun     |                                                            | {rambabu}


then make below enttry pg_hba.conf file.


local   rajdb           +rambabu                                 reject[from local]


host    rajdb           +rambabu     192.168.10.11/32            reject[from host]


## 19. USER ACCESS MANAGEMENT

in postgresql security handling in 3 phases.


->db level security : using pg_hba.conf file


->schema level : which user can access which schema , we can configure here.[USAGE : only select,insert,delete,update permissions allowed for


users & ALL : create,drop permissions also allowed.]


->object level : under which schema, which object requires select/insert/update/..etc permissions.


[select,insert,delete,updte can grant on objects if USAGE is there on the schema level].


```sql
postgres=# create schema rams;
CREATE SCHEMA
postgres=# create table rams.emp(no int,name varchar);
CREATE TABLE
postgres=# \dt rams.emp
```

List of relations

Schema | Name | Type  |  Owner

--------+------+-------+----------

rams   | emp  | table | postgres

(1 row)


```sql
postgres=#


-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> \dt rams.*
```

List of relations

Schema | Name | Type  |  Owner

--------+------+-------+----------

rams   | emp  | table | postgres

(1 row)


```sql
postgres=> select * from rams.emp;
```

ERROR:  permission denied for schema rams

LINE 1: select * from rams.emp;

^

```sql
postgres=>


grant schema level permissions first :
```

```sql
postgres=# grant USAGE on SCHEMA rams to eswar;
```

GRANT

```sql
postgres=# \dn+ rams
```

List of schemas

Name |  Owner   |  Access privileges   | Description

------+----------+----------------------+-------------

rams | postgres | postgres=UC/postgres+|

|          | eswar=U/postgres     |

(1 row)


```sql
revoke permission: postgres=# revoke USAGE on SCHEMA rams from eswar;
```

REVOKE


```bash
-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> select * from rams.emp;
```

ERROR:  permission denied for table emp

```sql
postgres=>


grant permissions on object level as well:
```

```sql
postgres=# grant SELECT,INSERT,DELETE,UPDATE on rams.emp to eswar;
```

GRANT

```sql
postgres=#

-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> select * from rams.emp;
```

no | name

----+------

(0 rows)


```sql
postgres=> insert into rams.emp values(1,'raj');
INSERT 0 1
postgres=> delete from rams.emp;
DELETE 1
postgres=>

if in my schema lot of tables are there:
```

```sql
postgres=# grant select,insert,update,delete on ALL tables in schema rams to eswar;
```

GRANT


```bash
-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> create table rams.dept(no int);
```

ERROR:  permission denied for schema rams

LINE 1: create table rams.dept(no int);

^


ALL Permission on schema:

```sql
postgres=# grant ALL on SCHEMA rams to eswar;
```

GRANT

```sql
postgres=# \dn+ rams
```

List of schemas

Name |  Owner   |  Access privileges   | Description

------+----------+----------------------+-------------

rams | postgres | postgres=UC/postgres+|

|          | eswar=UC/postgres    |

(1 row)


```sql
postgres=#

-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> create table rams.dept(no int);
CREATE TABLE
postgres=>

```

modify existing table with eswar:

```sql
postgres=> alter table rams.emp drop column eno;
```

ERROR:  must be owner of table emp

```sql
postgres=>

```

> **note: only table owner/super user can modify the table structure.**


```sql
postgres=> alter schema rams rename to sams;
```

ERROR:  must be owner of schema rams

```sql
postgres=>


```

eventhough user has select,insert on all tables in schema, still new tables will not be accesible.


```bash
-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> select * from rams.news;
```

ERROR:  permission denied for table news

```sql
postgres=>


```

### alter default previleges

```sql
postgres=# alter default privileges in schema rams grant select,insert,delete,update on tables to eswar;
ALTER DEFAULT PRIVILEGES

```

after alter default privileges, whatever the objects creating default will get those permissions for the all newly created tables.


```bash
-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# create table rams.t1(no int);
CREATE TABLE
postgres=# create table rams.t2(no int);
CREATE TABLE
postgres=# \q
```

-bash-4.2$

```bash
-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> select * from rams.t1;
```

no

----

(0 rows)


```sql
postgres=> select * from rams.t2;
```

no

----

(0 rows)


```sql
postgres=>


```

access rights through roles:

same as eswar , need permissions for shiva.


```sql
postgres=# grant eswar to shiva;
GRANT ROLE

postgres=# \du shiva
```

List of roles

Role name | Attributes | Member of

-----------+------------+-----------

shiva     |            | {eswar}


```bash
-bash-4.2$ psql -U shiva -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> select * from rams.t2;
```

no

----

(0 rows)


```sql
postgres=>


```

user eswar requires schema creation permission:

```sql
postgres=# grant CREATE on DATABASE postgres to eswar;
```

GRANT

```sql
postgres=# \l+ postgres
```

List of databases

Name   |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description

----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------

postgres | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +| 8825 kB | pg_default | default administrative connection database

|          |          |             |             | postgres=CTc/postgres+|         |            |

|          |          |             |             | eswar=C/postgres      |         |            |

(1 row)


```bash
-bash-4.2$ psql -U eswar -d postgres
psql (14.2)
```

Type "help" for help.


```sql
postgres=> create schema newsence;
CREATE SCHEMA
postgres=>

```

### Metadata /dictinory tables in PostgreSQL

metadata means data about data.


->in postgresql each and every object will be represented by object id, this is a unique number.


->under each and every database, 2 hidden schema's are there to manage all metadata tables and views.


1.pg_catalog


2.information_schema


tables under pg_catalog schema:

```sql
postgres=# \dt pg_catalog.*
```

List of relations

Schema   |          Name

------------+-------------------------

pg_catalog | pg_auth_members : it will display list of users and assigned roles.


```sql
postgres=# select * from pg_auth_members;
```

roleid | member | grantor | admin_option

--------+--------+---------+--------------

3374 |   3373 |      10 | f

3375 |   3373 |      10 | f

3377 |   3373 |      10 | f

16395 |  24588 |      10 | f

16395 |  24590 |      10 | f

24588 |  24589 |      10 | f


pg_catalog | pg_authid   : list of users and system created roles with oid.


```sql
postgres=# select oid,rolname,rolsuper,rolcreatedb from pg_authid;
```

oid  |          rolname          | rolsuper | rolcreatedb

-------+---------------------------+----------+-------------

6171 | pg_database_owner         | f        | f

6181 | pg_read_all_data          | f        | f

6182 | pg_write_all_data         | f        | f

3373 | pg_monitor                | f        | f

3374 | pg_read_all_settings      | f        | f

3375 | pg_read_all_stats         | f        | f

3377 | pg_stat_scan_tables       | f        | f

4569 | pg_read_server_files      | f        | f

4570 | pg_write_server_files     | f        | f

4571 | pg_execute_server_program | f        | f

4200 | pg_signal_backend         | f        | f

10 | postgres                  | t        | t

16395 | rambabu                   | f        | t

24588 | eswar                     | f        | f

24589 | shiva                     | f        | f

24590 | tarun                     | f        | f

24592 | rams                      | f        | f


from the above list few of them are system created roles:


pg_database_owner : normal user also can act as database owner.

pg_read_all_data : those users can select all tables data

pg_write_all_data    : users can INSERT, UPDATE, and DELETE data into all tables.


```sql
postgres=# grant pg_write_all_data to eswar; (only if u grant usage on the schema)
GRANT ROLE
postgres=# \du eswar
```

List of roles

Role name | Attributes |          Member of

-----------+------------+-----------------------------

eswar     |            | {pg_write_all_data,rambabu}


```sql
postgres=#


```

pg_monitor : that user cann monitor whole instances

pg_read_all_settings : in postgresql instance almost we have 355+ parameters, but for normal users those parameters not visible, if we grant


this role to normal user, he can able to see all parameters.


pg_read_all_stats : normally super user can see all pg_stat* views, but if i grant this role to normal user they can also see all views.

pg_stat_scan_tables  : normal users can execute monitoring functions

pg_read_server_files : import data to table from file

pg_write_server_files    : export data from table to file

pg_execute_server_program: to run copy command , normal users will be allowed

pg_signal_backend   : to close/cancel sessions we will use one function, normal users can't execute, but if i grant this role normal users also

can.


pg_catalog | pg_class  : list of objects, and type of objects, how many pages of data.


```sql
postgres=# select oid,relname,relnamespace,reltype,relowner,relpages,relkind from pg_class limit 10;
```

oid  |       relname        | relnamespace | reltype | relowner | relpages | relkind

-------+----------------------+--------------+---------+----------+----------+---------

32784 | pg_toast_32781       |           99 |       0 |       10 |        0 | t

32785 | pg_toast_32781_index |           99 |       0 |       10 |        1 | i

32781 | emp                  |        32780 |   32783 |       10 |        0 | r

2619 | pg_statistic         |           11 |   12029 |       10 |       19 | r

1247 | pg_type              |           11 |      71 |       10 |       14 | r

32786 | dept                 |        32780 |   32788 |    24588 |        0 | r

2836 | pg_toast_1255        |           99 |       0 |       10 |        1 | t

2837 | pg_toast_1255_index  |           99 |       0 |       10 |        1 | i

4171 | pg_toast_1247        |           99 |       0 |       10 |        0 | t

4172 | pg_toast_1247_index  |           99 |       0 |       10 |        1 | i


pg_catalog | pg_database : list of databases


```sql
postgres=# select oid,datname,datallowconn,datconnlimit,dattablespace from pg_database;
```

oid  |  datname  | datallowconn | datconnlimit | dattablespace

-------+-----------+--------------+--------------+---------------

1 | template1 | t            |           -1 |          1663

14485 | template0 | f            |           -1 |          1663

16384 | rajdb     | t            |           -1 |          1663

14486 | postgres  | t            |           -1 |          1663


pg_catalog | pg_db_role_setting    : user level settings will be displayed.


```sql
postgres=# alter user eswar set work_mem to "10MB";
ALTER ROLE
postgres=# select * from pg_db_role_setting;
```

setdatabase | setrole |    setconfig

-------------+---------+-----------------

0 |   24588 | {work_mem=10MB}


pg_catalog | pg_depend: what are the dependent objects

pg_catalog | pg_event_trigger: list of events part of trigger.

pg_catalog | pg_extension :[extension means extra features which is not part of the postgresql orignal s/w]

above one will display existing extension.


```sql
postgres=# select * from pg_extension;
```

oid  | extname | extowner | extnamespace | extrelocatable | extversion | extconfig | extcondition

-------+---------+----------+--------------+----------------+------------+-----------+--------------

14472 | plpgsql |       10 |           11 | f              | 1.0        |           |

(1 row)


it is equal to plsql in oracle.


pg_catalog | pg_foreign_data_wrapper

pg_catalog | pg_foreign_server

pg_catalog | pg_foreign_table

pg_catalog | pg_user_mapping


these above 4 tables related to dblink.


pg_catalog | pg_index  : list of indexes


```sql
postgres=# select indexrelid,indrelid,indisvalid,indisready,indislive from pg_index limit 10;
```

indexrelid | indrelid | indisvalid | indisready | indislive

------------+----------+------------+------------+-----------

2837 |     2836 | t          | t          | t

4172 |     4171 | t          | t          | t

2831 |     2830 | t          | t          | t

2833 |     2832 | t          | t          | t

4158 |     4157 | t          | t          | t

4160 |     4159 | t          | t          | t

2841 |     2840 | t          | t          | t

3440 |     3439 | t          | t          | t

3431 |     3430 | t          | t          | t

2839 |     2838 | t          | t          | t

(10 rows)


pg_catalog | pg_inherits  : it will display list of tables inherits

pg_catalog | pg_largeobject : list of large objects

pg_catalog | pg_largeobject_metadata : lob objects metadata

pg_catalog | pg_namespace: list of schemas


```sql
postgres=# select * from pg_namespace;
```

oid  |      nspname       | nspowner |                  nspacl

-------+--------------------+----------+------------------------------------------

99 | pg_toast           |       10 |

11 | pg_catalog         |       10 | {postgres=UC/postgres,=U/postgres}

2200 | public             |       10 | {postgres=UC/postgres,=UC/postgres}

14120 | information_schema |       10 | {postgres=UC/postgres,=U/postgres}

32780 | rams               |       10 | {postgres=UC/postgres,eswar=UC/postgres}

32799 | newsence           |    24588 |

(6 rows)


pg_catalog | pg_partitioned_table  : list of partioned tables will be listed.

pg_catalog | pg_policy : to achive row level securtiy we will create policies, it will display list of policies.

pg_catalog | pg_proc  : list of store procedures and functions


pg_catalog | pg_publication  : list of publications

pg_catalog | pg_publication_rel : tables part of publication

pg_catalog | pg_subscription_rel : tables part of subscription

pg_catalog | pg_subscription: list of subscription


above 4 tables related to logical replication.


pg_catalog | pg_sequence : list of sequences


```sql
postgres=# create sequence ram_s start with 1 increment by 1 maxvalue 100;
CREATE SEQUENCE
postgres=# \ds
```

List of relations

Schema | Name  |   Type   |  Owner

--------+-------+----------+----------

public | ram_s | sequence | postgres

(1 row)


```sql
postgres=#

```

pg_catalog | pg_tablespace   : list of tablesapces


```sql
postgres=# select * from pg_tablespace;
```

oid  |  spcname   | spcowner | spcacl | spcoptions

------+------------+----------+--------+------------

1663 | pg_default |       10 |        |

1664 | pg_global  |       10 |        |

(2 rows)


pg_catalog | pg_trigger  : list of triggers


### pg_catalog views

```sql
postgres=# \dv pg_catalog.*;
```

List of relations

Schema   |              Name

------------+---------------------------------

pg_catalog | pg_available_extension_versions : extensions with available versions.


```sql
postgres=# select name,version,comment from pg_available_extension_versions;
```

name        | version |                                comment

--------------------+---------+------------------------------------------------------------------------

hstore_plperlu     | 1.0     | transform between hstore and plperlu

plpgsql            | 1.0     | PL/pgSQL procedural language

dblink             | 1.2     | connect to other PostgreSQL databases from within a database

dict_int           | 1.0     | text search dictionary template for integers

insert_username    | 1.0     | functions for tracking who changed a table

adminpack          | 1.0     | administrative functions for PostgreSQL

adminpack          | 1.1     | administrative functions for PostgreSQL

adminpack          | 2.0     | administrative functions for PostgreSQL

adminpack          | 2.1     | administrative functions for PostgreSQL

dict_xsyn          | 1.0     | text search dictionary template for extended synonym processing

amcheck            | 1.0     | functions for verifying relation integrity

amcheck            | 1.1     | functions for verifying relation integrity

amcheck            | 1.2     | functions for verifying relation integrity

amcheck            | 1.3     | functions for verifying relation integrity

intagg             | 1.1     | integer aggregator and enumerator (obsolete)


pg_catalog | pg_available_extensions    : all the extensions with latest version.


```sql
postgres=# select * from pg_available_extensions limit 10;
```

name       | default_version | installed_version |                             comment

-----------------+-----------------+-------------------+-----------------------------------------------------------------

hstore_plperlu  | 1.0             |                   | transform between hstore and plperlu

plpgsql         | 1.0             | 1.0               | PL/pgSQL procedural language

dblink          | 1.2             |                   | connect to other PostgreSQL databases from within a database

dict_int        | 1.0             |                   | text search dictionary template for integers

insert_username | 1.0             |                   | functions for tracking who changed a table

adminpack       | 2.1             |                   | administrative functions for PostgreSQL

dict_xsyn       | 1.0             |                   | text search dictionary template for extended synonym processing

amcheck         | 1.3             |                   | functions for verifying relation integrity

intagg          | 1.1             |                   | integer aggregator and enumerator (obsolete)

autoinc         | 1.0             |                   | functions for autoincrementing fields


pg_catalog | pg_config : it will display installation folders.


BINDIR            | /usr/pgsql-14/bin

DOCDIR            | /usr/pgsql-14/doc

HTMLDIR           | /usr/pgsql-14/doc/html

INCLUDEDIR        | /usr/pgsql-14/include

PKGINCLUDEDIR     | /usr/pgsql-14/include

INCLUDEDIR-SERVER | /usr/pgsql-14/include/server

LIBDIR            | /usr/pgsql-14/lib

PKGLIBDIR         | /usr/pgsql-14/lib

LOCALEDIR         | /usr/pgsql-14/share/locale

MANDIR            | /usr/pgsql-14/share/man

SHAREDIR          | /usr/pgsql-14/share

SYSCONFDIR        | /etc/sysconfig/pgsql

PGXS              | /usr/pgsql-14/lib/pgxs/src/makefiles/pgxs.mk

CONFIGURE         |  '--enable-rpath' '--prefix=/usr/pgsql-14' '--includedir=/usr/pgsql-14/include' '--mandir=/usr/pgsql-14/share/man' '--datadir=/usr/pgsql

-14/share' '--libdir=/usr/pgsql-14/lib' '--with-lz4' '--with-icu' '--with-llvm' '--with-perl' '--with-python' '--with-tcl' '--with-tclconfig=/usr/lib64' '--w

ith-openssl' '--with-pam' '--with-gssapi' '--with-includes=/usr/include' '--with-libraries=/usr/lib64' '--enable-nls' '--enable-dtrace' '--with-uuid=e2fs' '-

-with-libxml' '--with-libxslt' '--with-ldap' '--with-selinux' '--with-systemd' '--with-system-tzdata=/usr/share/zoneinfo' '--sysconfdir=/etc/sysconfig/pgsql'

'--docdir=/usr/pgsql-14/doc' '--htmldir=/usr/pgsql-14/doc/html' 'CFLAGS=-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --p

aram=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic' 'LLVM_CONFIG=/usr/lib64/llvm5.0/bin/llvm-config' 'CLANG=/opt/rh/llvm-toolset-7/root/usr/b

in/clang' 'PKG_CONFIG_PATH=:/usr/lib64/pkgconfig:/usr/share/pkgconfig' 'PYTHON=/usr/bin/python3'

CC                | gcc -std=gnu99

CPPFLAGS          | -D_GNU_SOURCE -I/usr/include/libxml2 -I/usr/include

CFLAGS            | -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wformat-

security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ss

p-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic

CFLAGS_SL         | -fPIC

LDFLAGS           | -L/usr/lib64/llvm5.0/lib -L/usr/lib64 -Wl,--as-needed -Wl,-rpath,'/usr/pgsql-14/lib',--enable-new-dtags

LDFLAGS_EX        |

LDFLAGS_SL        |

LIBS              | -lpgcommon -lpgport -lselinux -llz4 -lxslt -lxml2 -lpam -lssl -lcrypto -lgssapi_krb5 -lz -lreadline -lpthread -lrt -ldl -lm

VERSION           | PostgreSQL 14.2


pg_catalog | pg_cursors  : list of cursos

pg_catalog | pg_file_settings  : all the uncommetned parameters from postgresql.conf file.


```sql
postgres=# select * from pg_file_settings;
```

sourcefile               | sourceline | seqno |            name            |            setting

----------------------------------------+------------+-------+----------------------------+--------------------------------

/var/lib/pgsql/14/data/postgresql.conf |         65 |     1 | max_connections            | 100

/var/lib/pgsql/14/data/postgresql.conf |        127 |     2 | shared_buffers             | 128MB

/var/lib/pgsql/14/data/postgresql.conf |        150 |     3 | dynamic_shared_memory_type | posix

/var/lib/pgsql/14/data/postgresql.conf |        240 |     4 | max_wal_size               | 1GB

/var/lib/pgsql/14/data/postgresql.conf |        241 |     5 | min_wal_size               | 80MB

/var/lib/pgsql/14/data/postgresql.conf |        433 |     6 | log_destination            | stderr

/var/lib/pgsql/14/data/postgresql.conf |        439 |     7 | logging_collector          | on

/var/lib/pgsql/14/data/postgresql.conf |        445 |     8 | log_directory              | pg_log

/var/lib/pgsql/14/data/postgresql.conf |        447 |     9 | log_filename               | postgresql-%Y-%m-%d_%H%M%S.log

/var/lib/pgsql/14/data/postgresql.conf |        451 |    10 | log_rotation_age           | 1d

/var/lib/pgsql/14/data/postgresql.conf |        453 |    11 | log_rotation_size          | 10MB

/var/lib/pgsql/14/data/postgresql.conf |        456 |    12 | log_truncate_on_rotation   | on

/var/lib/pgsql/14/data/postgresql.conf |        505 |    13 | log_min_duration_statement | 0

/var/lib/pgsql/14/data/postgresql.conf |        537 |    14 | log_connections            | on

/var/lib/pgsql/14/data/postgresql.conf |        538 |    15 | log_disconnections         | on

/var/lib/pgsql/14/data/postgresql.conf |        542 |    16 | log_line_prefix            | %a %u %d %h %p

/var/lib/pgsql/14/data/postgresql.conf |        575 |    17 | log_statement              | all

/var/lib/pgsql/14/data/postgresql.conf |        580 |    18 | log_timezone               | America/New_York

/var/lib/pgsql/14/data/postgresql.conf |        694 |    19 | datestyle                  | iso, mdy

/var/lib/pgsql/14/data/postgresql.conf |        696 |    20 | timezone                   | America/New_York

/var/lib/pgsql/14/data/postgresql.conf |        710 |    21 | lc_messages                | en_US.UTF-8

/var/lib/pgsql/14/data/postgresql.conf |        712 |    22 | lc_monetary                | en_US.UTF-8

/var/lib/pgsql/14/data/postgresql.conf |        713 |    23 | lc_numeric                 | en_US.UTF-8

/var/lib/pgsql/14/data/postgresql.conf |        714 |    24 | lc_time                    | en_US.UTF-8

/var/lib/pgsql/14/data/postgresql.conf |        717 |    25 | default_text_search_config | pg_catalog.english


pg_catalog | pg_hba_file_rules   : all entries from pg_gba.conf file will be displayed here.


```sql
postgres=# select * from pg_hba_file_rules;
```

line_number | type  |   database    | user_name  |  address  |                 netmask                 |  auth_method  | options | error

-------------+-------+---------------+------------+-----------+-----------------------------------------+---------------+---------+-------

85 | local | {all}         | {all}      |           |                                         | trust         |         |

87 | local | {rajdb}       | {+rambabu} |           |                                         | reject        |         |

90 | host  | {all}         | {all}      | ::1       | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | scram-sha-256 |         |

93 | local | {replication} | {all}      |           |                                         | peer          |         |

94 | host  | {replication} | {all}      | 127.0.0.1 | 255.255.255.255                         | scram-sha-256 |         |

95 | host  | {replication} | {all}      | ::1       | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | scram-sha-256 |         |


pg_catalog | pg_group : list of roles wheter system created or user created.


```sql
postgres=# create role read_r;
CREATE ROLE
postgres=# select * from pg_group;
```

groname          | grosysid | grolist

---------------------------+----------+---------

pg_database_owner         |     6171 | {}

pg_read_all_data          |     6181 | {}

pg_write_all_data         |     6182 | {24588}

pg_monitor                |     3373 | {}

pg_read_all_settings      |     3374 | {3373}

pg_read_all_stats         |     3375 | {3373}

pg_stat_scan_tables       |     3377 | {3373}

pg_read_server_files      |     4569 | {}

pg_write_server_files     |     4570 | {}

pg_execute_server_program |     4571 | {}

pg_signal_backend         |     4200 | {}

read_r                    |    40973 | {}


pg_catalog | pg_roles  : it will display list of users & system defined roles.


```sql
postgres=# select rolname from pg_roles;
```

rolname

---------------------------

pg_database_owner

pg_read_all_data

pg_write_all_data

pg_monitor

pg_read_all_settings

pg_read_all_stats

pg_stat_scan_tables

pg_read_server_files

pg_write_server_files

pg_execute_server_program

pg_signal_backend

postgres

rambabu

eswar

shiva

tarun

rams

(17 rows)


pg_catalog | pg_shadow    : all the users passwords will be in encrypted format.


```sql
postgres=# alter user eswar with password 'eswar';
ALTER ROLE
postgres=# select usename,passwd from pg_shadow;
```

usename  |                                                                passwd

----------+---------------------------------------------------------------------------------------------------------------------------------------

postgres | SCRAM-SHA-256$4096:QoeWD95BqrVqmvvT9U0DSg==$BesPUU/CPFjLQEamqk7EuUeJhSk2ce+tFD961QJ25Cw=:YNkS1EUo+O67lcztq/mi50/8Bf6yKXGaxn5tLXT7X0A=

rambabu  | SCRAM-SHA-256$4096:TvicGmu2KesqKgYMVY1qaA==$6GqS9MzsJJB8/xQw9yQQCIVJrO6GwizJ0YpM+u/QAMQ=:fNhb5G6p7JaqvXpJW7xIe/s3wrFe3TLSMQg7PHP+mJQ=

shiva    |

tarun    |

rams     | SCRAM-SHA-256$4096:1MX4VTAi11jaeZCJ09Arsw==$3Ut2WuD1HZtp87+x65ZUlEwQmkh2xhi0zgU/EqCcCzM=:Sfw2h7DJGKtLgpaPV5AC0uNS5yPkuGWTgmFc1bzYQtw=

eswar    | SCRAM-SHA-256$4096:791SYMptT+7msZNdlIjTCw==$ZlG7WH+JRhgBx/lj/HCzFIBvGmdwtDA+RWsdHQEqj34=:kphYzsJEQ/bH+ETRDwXtG6gLOS9ybJB7NwfjDyrf7i4=


pg_catalog | pg_user : list of users  same as \du


```sql
postgres=# select * from pg_user;
```

usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  |        valuntil        |    useconfig

----------+----------+-------------+----------+---------+--------------+----------+------------------------+-----------------

postgres |       10 | t           | t        | t       | t            | ******** |                        |

rambabu  |    16395 | t           | f        | f       | f            | ******** | 2022-06-08 00:00:00-04 |

eswar    |    24588 | f           | f        | f       | f            | ******** |                        | {work_mem=10MB}

shiva    |    24589 | f           | f        | f       | f            | ******** |                        |

tarun    |    24590 | f           | f        | f       | f            | ******** |                        |

rams     |    24592 | f           | f        | f       | f            | ******** |                        |

(6 rows)


pg_catalog | pg_stat_all_tables  : all system created tables+ user created tables statistics

pg_catalog | pg_stat_sys_tables : only system created tables stats

pg_catalog | pg_stat_user_tables: only user created tables stats


```sql
postgres=# select relid,schemaname,relname,n_tup_ins,n_tup_del,last_vacuum,last_analyze from pg_stat_user_tables;
```

relid | schemaname | relname | n_tup_ins | n_tup_del | last_vacuum | last_analyze

-------+------------+---------+-----------+-----------+-------------+--------------

32789 | rams       | news    |         0 |         0 |             |

32781 | rams       | emp     |         0 |         0 |             |

32793 | rams       | t1      |         0 |         0 |             |

32786 | rams       | dept    |         0 |         0 |             |

32796 | rams       | t2      |         0 |         0 |             |


pg_catalog | pg_tables  : list of tables with properties.


```sql
postgres=# select * from pg_tables limit 5;
```

schemaname |    tablename     | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity

------------+------------------+------------+------------+------------+----------+-------------+-------------

rams       | emp              | postgres   |            | f          | f        | f           | f

pg_catalog | pg_statistic     | postgres   |            | t          | f        | f           | f

pg_catalog | pg_type          | postgres   |            | t          | f        | f           | f

rams       | dept             | eswar      |            | f          | f        | f           | f

pg_catalog | pg_foreign_table | postgres   |            | t          | f        | f           | f


pg_catalog | pg_indexes: create index defintion.


Expanded display is on.

```sql
postgres=# select schemaname,tablename,indexname,indexdef from pg_indexes limit 1;
```

-[ RECORD 1 ]-----------------------------------------------------------------------------------------------------------------------------

schemaname | pg_catalog

tablename  | pg_statistic

indexname  | pg_statistic_relid_att_inh_index

indexdef   | CREATE UNIQUE INDEX pg_statistic_relid_att_inh_index ON pg_catalog.pg_statistic USING btree (starelid, staattnum, stainherit)


pg_catalog | pg_stat_all_indexes

pg_catalog | pg_stat_sys_indexes

pg_catalog | pg_stat_user_indexes  : index details will be displayed here.


```sql
postgres=# create index emp_idx on emp(no);
CREATE INDEX
postgres=# select * from pg_stat_user_indexes;
```

relid | indexrelid | schemaname | relname | indexrelname | idx_scan | idx_tup_read | idx_tup_fetch

-------+------------+------------+---------+--------------+----------+--------------+---------------

40974 |      40979 | public     | emp     | emp_idx      |        0 |            0 |             0

(1 row)


pg_catalog | pg_locks   : it will display blocking and blocked by sessions details.


pg_catalog | pg_matviews : it will dsiplay materlialized views.


```sql
postgres=# select count(*) from emp;
```

-[ RECORD 1 ]

count | 1000


```sql
postgres=# \x
```

Expanded display is off.

```sql
postgres=# select count(*) from emp;
```

count

-------

1000

(1 row)


```sql
postgres=# create view emp_v as select * from emp;
CREATE VIEW
postgres=# select count(*) from emp_v;
```

count

-------

1000

(1 row)


```sql
postgres=# create materialized view emp_mv as select * from emp;
SELECT 1000
postgres=# select count(*) from emp_mv;
```

count

-------

1000

(1 row)


```sql
postgres=# \dt+
```

List of relations

Schema | Name | Type  |  Owner   | Persistence | Access method | Size  | Description

--------+------+-------+----------+-------------+---------------+-------+-------------

public | emp  | table | postgres | permanent   | heap          | 80 kB |

(1 row)


```sql
postgres=# \dm+
```

List of relations

Schema |  Name  |       Type        |  Owner   | Persistence | Access method | Size  | Description

--------+--------+-------------------+----------+-------------+---------------+-------+-------------

public | emp_mv | materialized view | postgres | permanent   | heap          | 56 kB |

(1 row)


```sql
postgres=# \dv+
```

List of relations

Schema | Name  | Type |  Owner   | Persistence |  Size   | Description

--------+-------+------+----------+-------------+---------+-------------

public | emp_v | view | postgres | permanent   | 0 bytes |

(1 row)


```sql
postgres=# delete from emp;
DELETE 1000
postgres=# select count(*) from emp;
```

count

-------

0

(1 row)


```sql
postgres=# select count(*) from emp_v;
```

count

-------

0

(1 row)


```sql
postgres=# select count(*) from emp_mv;
```

count

-------

1000

(1 row)


```sql
postgres=# refresh materialized view emp_mv;
```

REFRESH MATERIALIZED VIEW

```sql
postgres=# select count(*) from emp_mv;
```

count

-------

0

(1 row)


```sql
postgres=# select * from pg_matviews;
```

schemaname | matviewname | matviewowner | tablespace | hasindexes | ispopulated |   definition

------------+-------------+--------------+------------+------------+-------------+-----------------

public     | emp_mv      | postgres     |            | f          | t           |  SELECT emp.no,+

|             |              |            |            |             |     emp.name   +

|             |              |            |            |             |    FROM emp;

(1 row)


pg_catalog | pg_policies   : list of policies


pg_catalog | pg_publication_tables

pg_catalog | pg_stat_subscription


above 2 related to logical replication.


pg_catalog | pg_replication_slots : list of replication slots

pg_catalog | pg_sequences : list of seuences

pg_catalog | pg_settings : each and every parameter name its value.


```sql
postgres=# select name,setting from pg_settings limit 10;
```

name               |  setting

---------------------------------+------------

allow_system_table_mods         | off

application_name                | psql

archive_cleanup_command         |

archive_command                 | (disabled)

archive_mode                    | off

archive_timeout                 | 0

array_nulls                     | on

authentication_timeout          | 60

autovacuum                      | on

autovacuum_analyze_scale_factor | 0.1

(10 rows)


pg_catalog | pg_stat_activity : to monitor sessions, whether it is active or inactive or idle , whatver it is it.


fyi: historical sessions will not appear.


```sql
postgres=# select datname,pid,usename,client_addr,xact_start,state from pg_stat_activity;
```

datname  | pid  | usename  | client_addr |          xact_start          | state

----------+------+----------+-------------+------------------------------+--------

| 5858 |          |             |                              |

| 5860 | postgres |             |                              |

postgres | 5940 | postgres |             | 2022-03-11 19:42:20.86662-05 | active

| 5856 |          |             |                              |

| 5855 |          |             |                              |

| 5857 |          |             |                              |


pg_catalog | pg_stat_archiver : if we enable archive mode, archiver process will up and running and it will start taking wal files backup.


to archive location. this view will gives statistics of archive processes.


enable archive mode:

```sql
postgres=# show archive_mode;
```

archive_mode

--------------

off

(1 row)


```sql
postgres=#

```

now to enable archive mode we need to make below changes in postgresql.conf file.


1. -bash-4.2$ mkdir /var/lib/pgsql/archive


2. vi postgresql.conf


archive_mode = on

archive_command = 'cp %p /var/lib/pgsql/archive/%f'


```bash
[root@mariadb1 ~]# systemctl stop postgresql-14
[root@mariadb1 ~]# systemctl start postgresql-14

```

3.validatation


```bash
[root@mariadb1 ~]# su - postgres
```

Last login: Fri Mar 11 19:41:07 EST 2022 on pts/0

```bash
-bash-4.2$ ps -ef|grep postgres
```

postgres  6860     1  0 19:48 ?        00:00:00 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/

postgres  6863  6860  0 19:48 ?        00:00:00 postgres: logger

postgres  6865  6860  0 19:48 ?        00:00:00 postgres: checkpointer

postgres  6866  6860  0 19:48 ?        00:00:00 postgres: background writer

postgres  6867  6860  0 19:48 ?        00:00:00 postgres: walwriter

postgres  6868  6860  0 19:48 ?        00:00:00 postgres: autovacuum launcher

postgres  6869  6860  0 19:48 ?        00:00:00 postgres: archiver

postgres  6870  6860  0 19:48 ?        00:00:00 postgres: stats collector

postgres  6871  6860  0 19:48 ?        00:00:00 postgres: logical replication launcher


```bash
-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# show archive_mode;
```

archive_mode

--------------

on

(1 row)


```sql
postgres=# select pg_switch_wal();
```

pg_switch_wal

---------------

0/190BFF8

(1 row)


```sql
postgres=# \x
```

Expanded display is on.

```sql
postgres=# select * from pg_stat_archiver;
```

-[ RECORD 1 ]------+------------------------------

archived_count     | 2

last_archived_wal  | 000000010000000000000002

last_archived_time | 2022-03-11 19:49:41.232147-05

failed_count       | 0

last_failed_wal    |

last_failed_time   |

stats_reset        | 2022-03-11 19:41:03.266665-05


how to cleanup archive files:

we have utility called pg_archivecleanup.


```bash
-bash-4.2$ pwd
```

/var/lib/pgsql/archive

```bash
-bash-4.2$ ls -ltr
```

total 327680

-rw------- 1 postgres postgres 16777216 Mar 11 19:49 000000010000000000000001

-rw------- 1 postgres postgres 16777216 Mar 11 19:49 000000010000000000000002

-rw------- 1 postgres postgres 16777216 Mar 11 19:51 000000010000000000000003

-rw------- 1 postgres postgres 16777216 Mar 11 19:51 000000010000000000000004

-rw------- 1 postgres postgres 16777216 Mar 11 19:51 000000010000000000000005

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000006

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000007

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000008

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000009

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 00000001000000000000000A

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 00000001000000000000000B

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 00000001000000000000000C

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 00000001000000000000000D

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 00000001000000000000000E

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 00000001000000000000000F

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000010

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000011

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000012

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000013

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000014

-bash-4.2$

-bash-4.2$

```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_archivecleanup /var/lib/pgsql/archive/ 000000010000000000000014
-bash-4.2$ ls -ltr
```

total 32768

-rw------- 1 postgres postgres 16777216 Mar 11 19:52 000000010000000000000014

-rw------- 1 postgres postgres 16777216 Mar 11 19:53 000000010000000000000015

-bash-4.2$


->realtime scenorio is we will give commitment to developers/application how manay day point intime recovery will be achieved.


pg_catalog | pg_stat_database : database statistics


```sql
postgres=# select datname,numbackends,temp_files,temp_bytes,deadlocks from pg_stat_database;
```

datname  | numbackends | temp_files | temp_bytes | deadlocks

-----------+-------------+------------+------------+-----------

|           0 |          0 |          0 |         0

template1 |           0 |          0 |          0 |         0

template0 |           0 |          0 |          0 |         0

rajdb     |           0 |          0 |          0 |         0

postgres  |           1 |          0 |          0 |         0

(5 rows)


pg_catalog | pg_stat_database_conflicts    : to find conflicts higher level with respect to database.


```sql
postgres=# select * from pg_stat_database_conflicts;
```

datid |  datname  | confl_tablespace | confl_lock | confl_snapshot | confl_bufferpin | confl_deadlock

-------+-----------+------------------+------------+----------------+-----------------+----------------

1 | template1 |                0 |          0 |              0 |               0 |              0

14485 | template0 |                0 |          0 |              0 |               0 |              0

16384 | rajdb     |                0 |          0 |              0 |               0 |              0

14486 | postgres  |                0 |          0 |              0 |               0 |              0

(4 rows)


pg_catalog | pg_stat_progress_analyze : analyze operation progress here.

pg_catalog | pg_stat_progress_basebackup : fullbackup progress

pg_catalog | pg_stat_progress_cluster : vacuum full progress

pg_catalog | pg_stat_progress_copy : copy command progress

pg_catalog | pg_stat_progress_create_index : create index progress

pg_catalog | pg_stat_progress_vacuum : vacuum progress


pg_catalog | pg_stat_replication  : if replication is in place, we can check the status

pg_catalog | pg_stat_replication_slots   : to make replication more effective, we will create replication slots.


pg_catalog | pg_stat_user_functions    : user created functions.

pg_catalog | pg_user_mappings : during db link , we will map users.

pg_catalog | pg_views: list of views


```sql
postgres=# select * from pg_views limit 1;
```

schemaname | viewname | viewowner |   definition

------------+----------+-----------+-----------------

public     | emp_v    | postgres  |  SELECT emp.no,+

|          |           |     emp.name   +

|          |           |    FROM emp;

(1 row)


pg_catalog | pg_prepared_statements   : it will give prepared statements information.

pg_catalog | pg_prepared_xacts : prepared statement details only will be displayed.


example : running in queries in BTET mode.


in BTET mode, immedietely we have to decide whether commit or rollback before exit from the session.


```sql
postgres=# BEGIN;
```

BEGIN

postgres=*# create table test(no int);

```sql
CREATE TABLE
postgres=*# commit;
COMMIT
postgres=# \q

```

### information_schema metadata views

```sql
postgres=# \dv information_schema.*;
```

List of relations

Schema       |                 Name

--------------------+---------------------------------------

information_schema | _pg_foreign_data_wrappers

information_schema | _pg_foreign_servers

information_schema | _pg_foreign_table_columns

information_schema | _pg_foreign_tables

information_schema | _pg_user_mappings

information_schema | foreign_data_wrapper_options

information_schema | foreign_data_wrappers

information_schema | foreign_server_options

information_schema | foreign_servers

information_schema | foreign_table_options

information_schema | foreign_tables

information_schema | user_mapping_options

information_schema | user_mappings


all the above views related to dblink.


information_schema | administrable_role_authorizations  : role with grant option.


```sql
postgres=# grant rambabu to eswar with admin option;
GRANT ROLE
postgres=# select * from information_schema.administrable_role_authorizations;
```

grantee | role_name | is_grantable

---------+-----------+--------------

eswar   | rambabu   | YES

(1 row)


information_schema | applicable_roles    : user and its assigned role information.


```sql
postgres=# select * from information_schema.applicable_roles;
```

grantee   |      role_name       | is_grantable

------------+----------------------+--------------

postgres   | pg_database_owner    | NO

eswar      | pg_write_all_data    | NO

pg_monitor | pg_read_all_settings | NO

pg_monitor | pg_read_all_stats    | NO

pg_monitor | pg_stat_scan_tables  | NO

eswar      | rambabu              | NO

tarun      | rambabu              | NO

shiva      | eswar                | NO


constraint : it is a condition on the column.


avaible constratins:


pk : only unique values + not null values

fk : to maintain relation ship b/w pk table and other table

not null : mandatory value should insert

unique : no duplicate value

check : based on check condition


```sql
postgres=# create table t1(no int primary key);
CREATE TABLE
postgres=# create table t2(no int references t1(no));
CREATE TABLE
postgres=# create table t3(no int check(no<10));
CREATE TABLE
postgres=# create table t4(no int unique);
CREATE TABLE
postgres=# create table t5(no int not null);
CREATE TABLE
postgres=#


```

information_schema | check_constraints  : list of check constraints


```sql
postgres=# select * from information_schema.check_constraints where constraint_schema='public';
```

constraint_catalog | constraint_schema |    constraint_name    |  check_clause

--------------------+-------------------+-----------------------+----------------

postgres           | public            | t3_no_check           | ((no < 10))

postgres           | public            | 2200_49178_1_not_null | no IS NOT NULL

postgres           | public            | 2200_49200_1_not_null | no IS NOT NULL

(3 rows)


[not null also check constraint]


information_schema | constraint_column_usage  : constraints with table and column names.


```sql
postgres=# select * from information_schema.constraint_column_usage where constraint_schema='public';
```

table_catalog | table_schema | table_name | column_name | constraint_catalog | constraint_schema | constraint_name

---------------+--------------+------------+-------------+--------------------+-------------------+-----------------

postgres      | public       | t3         | no          | postgres           | public            | t3_no_check

postgres      | public       | t1         | no          | postgres           | public            | t1_pkey

postgres      | public       | t1         | no          | postgres           | public            | t2_no_fkey

postgres      | public       | t4         | no          | postgres           | public            | t4_no_key

(4 rows)


information_schema | constraint_table_usage   : only tables with constraints.


```sql
postgres=# select * from information_schema.constraint_table_usage where constraint_schema='public';
```

table_catalog | table_schema | table_name | constraint_catalog | constraint_schema | constraint_name

---------------+--------------+------------+--------------------+-------------------+-----------------

postgres      | public       | t1         | postgres           | public            | t1_pkey

postgres      | public       | t1         | postgres           | public            | t2_no_fkey

postgres      | public       | t4         | postgres           | public            | t4_no_key

(3 rows)


information_schema | referential_constraints  : which fk related to which pk.


```sql
postgres=# select * from information_schema.referential_constraints where constraint_schema='public';
```

-[ RECORD 1 ]-------------+-----------

constraint_catalog        | postgres

constraint_schema         | public

constraint_name           | t2_no_fkey

unique_constraint_catalog | postgres

unique_constraint_schema  | public

unique_constraint_name    | t1_pkey

match_option              | NONE

update_rule               | NO ACTION

delete_rule               | NO ACTION


information_schema | table_constraints   : all constraints


```sql
postgres=# select constraint_schema,constraint_name,table_name,constraint_type from information_schema.table_constraints where constraint_schema='public';
```

constraint_schema |    constraint_name    | table_name | constraint_type

-------------------+-----------------------+------------+-----------------

public            | t1_pkey               | t1         | PRIMARY KEY

public            | t2_no_fkey            | t2         | FOREIGN KEY

public            | t3_no_check           | t3         | CHECK

public            | t4_no_key             | t4         | UNIQUE

public            | 2200_49178_1_not_null | t1         | CHECK

public            | 2200_49200_1_not_null | t5         | CHECK

(6 rows)


only keys details (pk,fk,uk):


```sql
postgres=# select * from information_schema.key_column_usage where constraint_schema='public';
```

constraint_catalog | constraint_schema | constraint_name | table_catalog | table_schema | table_name | column_name | ordinal_position | position_in_unique_c

onstraint

--------------------+-------------------+-----------------+---------------+--------------+------------+-------------+------------------+---------------------

----------

postgres           | public            | t1_pkey         | postgres      | public       | t1         | no          |                1 |


postgres           | public            | t2_no_fkey      | postgres      | public       | t2         | no          |                1 |

1

postgres           | public            | t4_no_key       | postgres      | public       | t4         | no          |                1 |


(3 rows)


information_schema | columns : all columns from all tables, with data types and other details.

information_schema | enabled_roles   : all users and roles.


```sql
postgres=# select * from information_schema.enabled_roles;
```

role_name

---------------------------

pg_database_owner

pg_read_all_data

pg_write_all_data

pg_monitor

pg_read_all_settings

pg_read_all_stats

pg_stat_scan_tables

pg_read_server_files

pg_write_server_files

pg_execute_server_program

pg_signal_backend

postgres

rambabu

shiva

tarun

rams

eswar

read_r

(18 rows)


information_schema | information_schema_catalog_name    : current connected database.


```sql
postgres=# select * from information_schema.information_schema_catalog_name;
```

catalog_name

--------------

postgres

(1 row)


information_schema | role_column_grants

information_schema | role_routine_grants

information_schema | role_table_grants

information_schema | table_privileges


above views will give any user/role having which permission on which table.


```sql
postgres=# grant INSERT on ALL tables in schema public to eswar;
```

GRANT

```sql
postgres=# select * from information_schema.table_privileges where grantee='eswar' and table_schema='public';
```

grantor  | grantee | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy

----------+---------+---------------+--------------+------------+----------------+--------------+----------------

postgres | eswar   | postgres      | public       | t1         | INSERT         | NO           | NO

postgres | eswar   | postgres      | public       | t1         | SELECT         | NO           | YES

postgres | eswar   | postgres      | public       | t2         | INSERT         | NO           | NO

postgres | eswar   | postgres      | public       | t2         | SELECT         | NO           | YES

postgres | eswar   | postgres      | public       | t3         | INSERT         | NO           | NO

postgres | eswar   | postgres      | public       | t3         | SELECT         | NO           | YES

postgres | eswar   | postgres      | public       | t4         | INSERT         | NO           | NO

postgres | eswar   | postgres      | public       | t4         | SELECT         | NO           | YES

postgres | eswar   | postgres      | public       | t5         | INSERT         | NO           | NO

postgres | eswar   | postgres      | public       | t5         | SELECT         | NO           | YES

(10 rows)


information_schema | sequences  : list of sequences

information_schema | tables : list of tables

information_schema | triggered_update_columns  : columns updating as part of trigger.

information_schema | triggers : list of triggers

information_schema | view_column_usage: columns using as part of view creation

information_schema | view_table_usage  :  tables using as part of view creation

information_schema | views: list of views


```sql
postgres=# select * from information_schema.views where table_schema='public';
```

-[ RECORD 1 ]--------------+--------------

table_catalog              | postgres

table_schema               | public

table_name                 | t3_v

view_definition            |  SELECT t3.no+

|    FROM t3;

check_option               | NONE

is_updatable               | YES

is_insertable_into         | YES

is_trigger_updatable       | NO

is_trigger_deletable       | NO

is_trigger_insertable_into | NO


```sql
postgres=# select * from information_schema.view_table_usage where table_schema='public';
```

-[ RECORD 1 ]-+---------

view_catalog  | postgres

view_schema   | public

view_name     | t3_v

table_catalog | postgres

table_schema  | public

table_name    | t3


```sql
postgres=# select * from information_schema.view_column_usage where table_schema='public';
```

-[ RECORD 1 ]-+---------

view_catalog  | postgres

view_schema   | public

view_name     | t3_v

table_catalog | postgres

table_schema  | public

table_name    | t3

column_name   | no


## 21. ENABLE ARCHIVE LOG

->each and every object represented with oid.


->if data we are writing in memory, that area we will call it as Buffer.


->if data we are writing in disk, that area we will call it as page.


->in postgresql each table having its own data file.


->data file size 1gb (segment size), same data file .1,.2,.3..like that extensions will be created for every 1gb.


->default all our data files located under data directory -->base directory -->database folder.


->in the above flow [DATA->BASE->DATABASE FOLDER (db oid) -->Table data file]


->above order schema not coming , schema in not a physical object, schema is logical object, jusst for name shake.


```sql
postgres=# select oid,datname from pg_database;
```

oid  |  datname

-------+-----------

1 | template1

14485 | template0

16384 | rajdb

14486 | postgres

(4 rows)


```sql
postgres=# \q
-bash-4.2$ ls -ltr
```

total 48

drwx------ 2 postgres postgres 8192 Mar  8 19:43 14485

drwx------ 2 postgres postgres 8192 Mar  8 19:43 1

drwx------ 2 postgres postgres 8192 Mar  9 19:38 16384

drwx------ 2 postgres postgres 8192 Mar 11 20:33 14486


under base we have 4 folders, each folder representing database name.


```bash
-bash-4.2$ cd 16384
-bash-4.2$ ls -ltr
```

total 8648

-rw------- 1 postgres postgres 770048 Mar  7 19:43 1255


under database folder by default lot of files will come.


```bash
-bash-4.2$ ls -ltr 2617*
```

-rw------- 1 postgres postgres 114688 Mar  7 19:43 2617: main data file

-rw------- 1 postgres postgres  24576 Mar  7 19:43 2617_fsm : free space map [for upcoming records, it will display avaible storage locations]

-rw------- 1 postgres postgres   8192 Mar  7 19:43 2617_vm : visibility map [all avaible records sotrage locations will be displayed here]

-bash-4.2$


->check table physical location:


```sql
postgres=# select pg_relation_filepath('t1');
```

pg_relation_filepath

----------------------

base/14486/49178

(1 row)


dont touch data files, if corrupted, no way to recovery, only option is if you have old bacup for that table, we can use for restore.


```sql
postgres=# \q
-bash-4.2$ pwd
```

/var/lib/pgsql/14/data/base/16384

```bash
-bash-4.2$ cd ..
-bash-4.2$ cd 14486/
-bash-4.2$ ls -ltr 49178
```

-rw------- 1 postgres postgres 8192 Mar 11 21:01 49178

```bash
-bash-4.2$ rm -rf 49178
-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# select * from t1;
```

ERROR:  could not open file "base/14486/49178": No such file or directory


## 22. TABLESPACE

->tablespace means it is a logical representation for physical location.


->tablespace will decide which database/which object need to store where.


->default tablespaces in postgresql.


```sql
postgres=# \db
```

List of tablespaces

Name    |  Owner   | Location

------------+----------+----------

pg_default | postgres |$DATADIR/base

pg_global  | postgres |$DATADIR/global

(2 rows)


all the objects by default creating with pg_Default tablespace, that is the reason all objects data going to base folder.


```sql
postgres=# \l+
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace

-----------+----------+----------+-------------+-------------+-----------------------+---------+------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +| 8969 kB | pg_default

|          |          |             |             | postgres=CTc/postgres+|         |

|          |          |             |             | eswar=C/postgres      |         |

rajdb     | rambabu  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8641 kB | pg_default

template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8617 kB | pg_default

|          |          |             |             | postgres=CTc/postgres |         |

template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8617 kB | pg_default

|          |          |             |             | postgres=CTc/postgres |         |


advatnages with tablespaces:

1.we can keep our objects in seperate folder.


2.for the specific folder we can implement high level security.


3.if we keep our data in seperate folder, it will increase the pefromance.


4.if my data directory got filled 100%, in that caase if i dont have option to increase the data directory size, only i have option i can

add one more mountpoint with huge space, simply by creating tablespace to that location and i will assign to bigger database,

```bash
at that time bigger database will be moved to new folder.

```

so during space issues also tablespaces will help.


```sql
create tablespace:
```

->different mountpoint or storage location or directory.


```bash
-bash-4.2$ mkdir /var/lib/pgsql/ram_tbs

-bash-4.2$ cd pg_tblspc
-bash-4.2$ pwd
```

/var/lib/pgsql/14/data/pg_tblspc

```bash
-bash-4.2$ ls -ltr
```

total 0


```sql
postgres=# create tablespace ramts location '/var/lib/pgsql/ram_tbs/';
CREATE TABLESPACE
postgres=# \db+
```

List of tablespaces

Name    |  Owner   |        Location        | Access privileges | Options |  Size   | Description

------------+----------+------------------------+-------------------+---------+---------+-------------

pg_default | postgres |                        |                   |         | 34 MB   |

pg_global  | postgres |                        |                   |         | 576 kB  |

ramts      | postgres | /var/lib/pgsql/ram_tbs |                   |         | 0 bytes |

(3 rows)


/var/lib/pgsql/14/data/pg_tblspc

```bash
-bash-4.2$ ls -ltr
```

total 0

lrwxrwxrwx 1 postgres postgres 22 Mar 11 21:14 57370 -> /var/lib/pgsql/ram_tbs

```bash
-bash-4.2$ cd /var/lib/pgsql/ram_tbs
-bash-4.2$ ls -ltr
```

total 0

drwx------ 2 postgres postgres 6 Mar 11 21:14 PG_14_202107181

```bash
-bash-4.2$ cd PG_14_202107181
-bash-4.2$ ls -ltr
```

total 0

-bash-4.2$


we have just created tablespace, not assigned to any object.


in postgresql no concept of adding data files like oracle.


tablespace size details will be depends upon resepct mount point.


```sql
postgres=# create database ram tablespace ramts;
CREATE DATABASE
postgres=# \l+
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description

-----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +| 8969 kB | pg_default | default administrative connection database

|          |          |             |             | postgres=CTc/postgres+|         |            |

|          |          |             |             | eswar=C/postgres      |         |            |

rajdb     | rambabu  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8641 kB | pg_default |

ram       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8617 kB | ramts      |

template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8617 kB | pg_default | unmodifiable empty database

|          |          |             |             | postgres=CTc/postgres |         |            |

template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +| 8617 kB | pg_default | default template for new databases

|          |          |             |             | postgres=CTc/postgres |         |            |

(5 rows)


/var/lib/pgsql/ram_tbs/PG_14_202107181

```bash
-bash-4.2$ ls -ltr
```

total 12

drwx------ 2 postgres postgres 8192 Mar 11 21:17 57371

-bash-4.2$


move database from one tablespace to another tablespace:


```sql
postgres=# alter database rajdb set tablespace ramts;
ALTER DATABASE
postgres=# \l+
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   |  Size   | Tablespace |                Description

-----------+----------+----------+-------------+-------------+-----------------------+---------+------------+--------------------------------------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +| 8969 kB | pg_default | default administrative connection database

|          |          |             |             | postgres=CTc/postgres+|         |            |

|          |          |             |             | eswar=C/postgres      |         |            |

rajdb     | rambabu  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8641 kB | ramts      |

ram       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                       | 8617 kB | ramts      |


during movement exclusive lock will applies, that means no one can access that database/table.


ram=# create table dept(no int) tablespace pg_default;

```sql
CREATE TABLE
ram=# select pg_relation_filepath('dept');
```

pg_relation_filepath

----------------------

base/57371/57375

(1 row)


move existing table tablespace:


ram=# alter table dept set tablespace ramts;

```sql
ALTER TABLE
ram=# select pg_relation_filepath('dept');
```

pg_relation_filepath

---------------------------------------------

pg_tblspc/57370/PG_14_202107181/57371/57378

(1 row)


```sql
drop tablespace:
```

ram=# alter database rajdb set tablespace pg_default;

```sql
ALTER DATABASE
ram=# alter database ram set tablespace pg_default;
```

ERROR:  cannot change the tablespace of the currently open database


rajdb=# alter database ram set tablespace pg_default;

```sql
ALTER DATABASE
```

rajdb=#


rajdb=# drop tablespace ramts;

```sql
DROP TABLESPACE
```

rajdb=#


tablespace limitations:

1.one tablespace can contains only one location.


2.one database/table can contains only one tablespace.


3.one tablespace can be assigned to multiple objects.


4.space utilization depends on respective mount point.


5.if database in one tablesapce, underlying object can store in different tablespace.


6.during tablespace movement exclusive lock will applies on the objects.


fyi: below are the tablespaces details


rajdb=# select * from pg_tablespace;

oid  |  spcname   | spcowner | spcacl | spcoptions

------+------------+----------+--------+------------

1663 | pg_default |       10 |        |

1664 | pg_global  |       10 |        |

(2 rows)


## 23. MAINTAINANCE ACTIVITIES

->to keep database safe

->to increase the database perfromance.

->to reclaim unsed space


in postgresql we have few maintance activties.


1.analyze


2.vacuum


3.vacuum full


4.reindex


### 1.analyze

->to update the statistics of tables. [statistics : information of the tables]


->if stats are not up to date, optimizer will get wrong statistics and it will generate wrong execution plan. if wrong execution plan


system will consume huge resources.


->when we run the analyze it will update the statistics.


->it will not apply any lock.


->it will not require any extra space.


->it will nto make any changes in the data file.


### 2.vacuum

->to remove dead tuples we will perfrom vacuum operations.


->tuple means record


->attribute means column.


->deadtuple: if our table contains 100 records, in the disk page it will occupy 100 locations, if we delete 50 records, now 50 locations should

become free from the disk page, but it will not happen. eventhough we we have deleted records, still sotrage locations are not ready for

reusage, we call those records as dead tuples.


->vacuum just mark deadtuples locations ready for reuse,it will not reclaim any space to disk.


->vacuum will update visibility map file as well.


->it will not apply any lock on the table.


->it no need any extra space.


->it will not make any changes to data file.


### 3.vacuum full

->it will remove bloat


->it will redistribute the data by creating new structure.


->it will reclaim space to disck.


->bloat: unproper data distribution we call it as bloat. [we know we are loading 5gb data, but after loading it occupied 10gb space]


->to perform this activity we need extra space.


->it will applies exclusive lock on the table to avoid the inconcistency of data.


->data file will be changed as it is creating new structure.


recomendations:

->after insert/update/delete analyze mandatory.


->after insert/delete vacuum mandatory.


->if u found bloat value/after delete to reclaim space to disk we need vacuum full.


### 4.reindex

->whenever table get bloated respective index also get bloated, there are huge chances.


->if we found bloat in index, we can perform reindex.


->it will drop and recreate index.


->lock will be applied on the respective table.


->we need extra space as well


->index data file will be changed.


## 24. AUTOVACUUM SETTING

->postgresql by default given option to run vacuum and analyze automatically.


->there is a process called autovacuum launcher process.


->by default autovacuum enabled state.


->autovacuum will be perfromed based on thresholds.


```bash
-bash-4.2$ cat postgresql.conf|grep autovacuum

```

#autovacuum_work_mem = -1               # min 1MB, or -1 to use maintenance_work_mem


log_autovacuum_min_duration = 0       # log autovacuum activity; (it will capture in log file)


autovacuum = on                        # Enable autovacuum subprocess?  'on'


autovacuum_max_workers = 3             # max number of autovacuum subprocesses


autovacuum_naptime = 1min              # time between autovacuum runs


autovacuum_vacuum_threshold = 50       # on any table if 50 rows updated, automatically vacuum operation will trigger.


autovacuum_vacuum_insert_threshold = 1000      # if 1000 records inserted to any table, vacuum will be triggerd.


autovacuum_vacuum_insert_scale_factor = 0.2    # fraction of inserts over table


autovacuum_vacuum_scale_factor = 0.2   # fraction of table size before vacuum


autovacuum_analyze_threshold = 50      # for every 50 rows update analyze will trigger.


autovacuum_analyze_scale_factor = 0.1  # if 10% growth in the table, analyze will trigger automatically.


#autovacuum_freeze_max_age = 200000000  # maximum XID age before forced vacuum


#autovacuum_multixact_freeze_max_age = 400000000        # maximum multixact age


XID : each and every query running will be counted.


->by default limit is 20 crores.


->before 20 crores limit , vacuum mandatory.


->sometimes we missed to perfrom vacuum , in that case once 20 crores count reached, it will trigger mandatory vacuum, ie. transaction id wraparound vacuum.


autovacuum_vacuum_cost_delay = 2ms     # default vacuum cost delay for

# autovacuum, in milliseconds;


## 25. CRON JOB SETUP

```bash
vi postgresql.conf

```

autovacuum=off


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/ reload
```

server signaled


```bash
-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# show autovacuum;
```

autovacuum

------------

off

(1 row)


```sql
postgres=# create table emp(eno int,ename varchar,sal int);
CREATE TABLE
postgres=# insert into emp values(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
INSERT 0 100000

```

table statistics:

```sql
postgres=# select relid,schemaname,relname,n_tup_ins,n_tup_del,n_live_tup,n_dead_tup from pg_stat_user_tables;
```

relid | schemaname | relname | n_tup_ins | n_tup_del | n_live_tup | n_dead_tup

-------+------------+---------+-----------+-----------+------------+------------

65562 | public     | emp     |    100000 |         0 |     100000 |          0

(1 row)


```sql
postgres=# explain select * from emp;
```

QUERY PLAN

-----------------------------------------------------------

Seq Scan on emp  (cost=0.00..1381.60 rows=75360 width=40)

(1 row)


```sql
postgres=# analyze emp;
ANALYZE
postgres=# explain select * from emp;
```

QUERY PLAN

------------------------------------------------------------

Seq Scan on emp  (cost=0.00..1628.00 rows=100000 width=16)

(1 row)


no chnage in the data file:


```sql
postgres=# select pg_relation_filepath('emp');
```

pg_relation_filepath

----------------------

base/14486/65562

(1 row)


```sql
postgres=# analyze emp;
ANALYZE
postgres=# select pg_relation_filepath('emp');
```

pg_relation_filepath

----------------------

base/14486/65562

(1 row)


```bash
find last analyze date:

postgres=# select relid,schemaname,relname,n_tup_ins,n_tup_del,n_live_tup,n_dead_tup,last_vacuum,last_analyze from pg_stat_user_tables;
```

relid | schemaname | relname | n_tup_ins | n_tup_del | n_live_tup | n_dead_tup | last_vacuum |         last_analyze

-------+------------+---------+-----------+-----------+------------+------------+-------------+------------------------------

65562 | public     | emp     |    100000 |         0 |     100000 |          0 |             | 2022-03-14 21:25:18.68399-04

(1 row)


```sql
postgres=# delete from emp where eno<=50000;
DELETE 50000
postgres=#

```

check dead tuples:


```sql
postgres=# analyze emp;
ANALYZE
postgres=# explain select * from emp;
```

QUERY PLAN

-----------------------------------------------------------

Seq Scan on emp  (cost=0.00..1128.00 rows=50000 width=17)

(1 row)


```sql
postgres=# select relid,schemaname,relname,n_tup_ins,n_tup_del,n_live_tup,n_dead_tup,last_vacuum,last_analyze from pg_stat_user_tables;
```

relid | schemaname | relname | n_tup_ins | n_tup_del | n_live_tup | n_dead_tup | last_vacuum |         last_analyze

-------+------------+---------+-----------+-----------+------------+------------+-------------+-------------------------------

65562 | public     | emp     |    100000 |     50000 |      50000 |      50000 |             | 2022-03-14 21:27:45.430111-04

(1 row)


```sql
postgres=# vacuum emp;
VACUUM
postgres=# select relid,schemaname,relname,n_tup_ins,n_tup_del,n_live_tup,n_dead_tup,last_vacuum,last_analyze from pg_stat_user_tables;
```

relid | schemaname | relname | n_tup_ins | n_tup_del | n_live_tup | n_dead_tup |          last_vacuum          |         last_analyze

-------+------------+---------+-----------+-----------+------------+------------+-------------------------------+-------------------------------

65562 | public     | emp     |    100000 |     50000 |      50000 |          0 | 2022-03-14 21:28:44.061601-04 | 2022-03-14 21:27:45.430111-04

(1 row)


fyi: analyze will not clear deadtuples, vacuum will celar dead tuples.


```sql
postgres=# \dt+
```

List of relations

Schema | Name | Type  |  Owner   | Persistence | Access method |  Size  | Description

--------+------+-------+----------+-------------+---------------+--------+-------------

public | emp  | table | postgres | permanent   | heap          | 319 MB |

(1 row)


```sql
postgres=# select count(*) from emp;
```

count

---------

6400000

(1 row)


```sql
postgres=# delete from emp where eno<=25000;
DELETE 0
postgres=# delete from emp where eno<=75000;
DELETE 3200000
postgres=# show shared_buffers;
```

shared_buffers

----------------

128MB

(1 row)


```sql
postgres=# \dt+
```

List of relations

Schema | Name | Type  |  Owner   | Persistence | Access method |  Size  | Description

--------+------+-------+----------+-------------+---------------+--------+-------------

public | emp  | table | postgres | permanent   | heap          | 319 MB |

(1 row)


```sql
postgres=# vacuum emp;
VACUUM
postgres=# \dt+
```

List of relations

Schema | Name | Type  |  Owner   | Persistence | Access method |  Size  | Description

--------+------+-------+----------+-------------+---------------+--------+-------------

public | emp  | table | postgres | permanent   | heap          | 319 MB |

(1 row)


```sql
postgres=# select pg_relation_filepath('emp');
```

pg_relation_filepath

----------------------

base/14486/65562

(1 row)


```sql
postgres=# vacuum FULL emp;
VACUUM
postgres=# select pg_relation_filepath('emp');
```

pg_relation_filepath

----------------------

base/14486/65567

(1 row)


```sql
postgres=# \dt+
```

List of relations

Schema | Name | Type  |  Owner   | Persistence | Access method |  Size  | Description

--------+------+-------+----------+-------------+---------------+--------+-------------

public | emp  | table | postgres | permanent   | heap          | 159 MB |

(1 row)


```sql
postgres=#

```

### reindex

->some times only index will get bloat


->if u dont have scope to perfom vacuum full on the table, if required we can perfrom reindex.


->only tables which are having indexes, those tables only we can perfrom reindex.


```sql
postgres=# \dt+
```

List of relations

Schema | Name | Type  |  Owner   | Persistence | Access method |  Size  | Description

--------+------+-------+----------+-------------+---------------+--------+-------------

public | emp  | table | postgres | permanent   | heap          | 159 MB |

(1 row)


```sql
postgres=# create index emp_idx on emp(eno);
CREATE INDEX
postgres=# \di+
```

List of relations

Schema |  Name   | Type  |  Owner   | Table | Persistence | Access method | Size  | Description

--------+---------+-------+----------+-------+-------------+---------------+-------+-------------

public | emp_idx | index | postgres | emp   | permanent   | btree         | 22 MB |

(1 row)


```sql
postgres=#


if you run reindex on the table, indirectly it will works on indexes only, no change for the table data files.

postgres=# select pg_relation_filepath('emp_idx');
```

pg_relation_filepath

----------------------

base/14486/73754

(1 row)


```sql
postgres=# select pg_relation_filepath('emp');
```

pg_relation_filepath

----------------------

base/14486/65567

(1 row)


```sql
postgres=# reindex table emp;
REINDEX
postgres=# select pg_relation_filepath('emp');
```

pg_relation_filepath

----------------------

base/14486/65567

(1 row)


```sql
postgres=#  select pg_relation_filepath('emp_idx');
```

pg_relation_filepath

----------------------

base/14486/73755

(1 row)


```sql
reindex on whole schema:
```

whole schema objects can perfrom reindex:


```sql
postgres=# reindex schema public;
REINDEX


```

whole system tables of the database also can reindex:

```sql
postgres=# reindex SYSTEM postgres;
REINDEX

whole database vacuum;
```

```sql
postgres=#
postgres=# vacuum;
VACUUM
postgres=# analyze;
ANALYZE
postgres=# vacuum full;
VACUUM


```

multiple tables need to perfrom maintance activties:

```bash
-bash-4.2$ vacuumdb -a(all databases vacuum)
vacuumdb: vacuuming database "postgres"
vacuumdb: vacuuming database "rajdb"
vacuumdb: vacuuming database "ram"
vacuumdb: vacuuming database "template1"

-bash-4.2$ vacuumdb -d rajdb (only rajdb vaccum)
vacuumdb: vacuuming database "rajdb"
```

-bash-4.2$


```bash
-bash-4.2$ vacuumdb -d rajdb -f (vacuum full)
vacuumdb: vacuuming database "rajdb"
```

-bash-4.2$


```bash
-bash-4.2$ vacuumdb -d rajdb -j 2(multiple jobs)
vacuumdb: vacuuming database "rajdb"
```

-bash-4.2$


```bash
-bash-4.2$ vacuumdb -d rajdb -z -j 2 (vacuum+analyze)
vacuumdb: vacuuming database "rajdb"
```

-bash-4.2$


```bash
-bash-4.2$ vacuumdb -d rajdb -Z -j 2 (only analyze)
vacuumdb: vacuuming database "rajdb"

for specific table:
```

```bash
-bash-4.2$ vacuumdb -d postgres -t emp -t t1 -t t2
vacuumdb: vacuuming database "postgres"

```

scheduling maintance activties:

->devide set of tables based on sizes.


->below 5gb tables, daily we can do vacuum + analyze.


->5 to 100 gb tables weekend once.


->bigger tables plan either 15 days once or monthly once.


to schedule jobs we will use crontab:

MIN    HOUR   DOM   MON        DOW     CMD


MIN	Minute field	0 to 59

HOUR	Hour field	0 to 23

DOM	Day of Month	1-31

MON	Month field	1-12

DOW	Day Of Week	0-6

CMD	Command	Any command to be executed.


schedule job for 17th March 08:30 AM


30     08      17      03  *    /home/ramesh/full-backup


example:


```bash
-bash-4.2$ crontab -l
```

no crontab for postgres


```bash
-bash-4.2$ crontab -e
```

no crontab for postgres - using an empty one

crontab: installing new crontab


```bash
-bash-4.2$ crontab -l
```

30 08 17 03 * /tmp/test.sh

-bash-4.2$


test manually and confirm:


```bash
-bash-4.2$ /tmp/test.sh
vacuumdb: vacuuming database "postgres"


```

crontable exmaples link :  https://www.thegeekstuff.com/2009/06/15-practical-crontab-examples/


every day script execute 2 times:


00 11,16 * * * /tmp/test.sh


11am, and 4pm my script should execute.


every one hr continously execute:


00 9-16 * * * /tmp/test.sh  [9 am, 10 am,11 am, 12 am, 1 pm, 2 pm, 3 pm, 4 pm]


```sql
vacuumfull & reindex dont perfrom whenever you want, need confirmation from application/developer team. at the same time need free space as well.


```

->always if any bigger objects vacuum or analyze run in background.


```bash
-bash-4.2$ cat test.sh
vacuumdb -d postgres -t t1 -t t2 -t emp
```

-bash-4.2$

```bash
-bash-4.2$ nohup sh test.sh &
```

[1] 7389

```bash
-bash-4.2$ nohup: ignoring input and appending output to ‘nohup.out’

```

[1]+  Done                    nohup sh test.sh

```bash
-bash-4.2$ cat nohup.out
vacuumdb: vacuuming database "postgres"
```

-bash-4.2$


## 26. DATA IMPORT AND EXPORT

to copy data from table to file(export) or to copy data from file to table(import) we have an utility called COPY.


```sql
postgres=# create table emp(no int,name varchar);
CREATE TABLE
postgres=# insert into emp values(1,'ram'),(2,'sam');
INSERT 0 2
postgres=# select * from emp;
```

no | name

----+------

1 | ram

2 | sam

(2 rows)


1.export data from table to file:


```sql
postgres=# copy emp to '/tmp/emp.txt';
COPY 2
postgres=# \q
-bash-4.2$ cat /tmp/emp.txt
```

1       ram

2       sam


2.import data from file to table:


```sql
postgres=# copy emp from '/tmp/emp.txt';
COPY 2
postgres=# select * from emp;
```

no | name

----+------

1 | ram

2 | sam

1 | ram

2 | sam

(4 rows)


3.export data with delimter [seperator]


```sql
postgres=# copy emp to '/tmp/emp.txt' with delimiter '|';
COPY 4
postgres=# \q
-bash-4.2$ cat /tmp/emp.txt
1|ram
2|sam
1|ram
2|sam
```

-bash-4.2$


4.import data with delimiter:


```sql
postgres=# copy emp from '/tmp/emp.txt' with delimiter '|';
COPY 4


```

5.export data with header


```sql
postgres=# copy emp to '/tmp/emp.txt' with header CSV;
COPY 8
postgres=# \q
```

-bash-4.2$

```bash
-bash-4.2$ cat /tmp/emp.txt
```

no,name

1,ram

2,sam

1,ram

2,sam

1,ram

2,sam

1,ram

2,sam


6.import into another table


```sql
postgres=# create table dept(no int,name varchar);
CREATE TABLE
postgres=# copy dept from '/tmp/emp.txt' with header CSV;
COPY 8


```

7.export specific query output to file.


```sql
postgres=# copy (select * from emp where no<2) to '/tmp/emp.txt';
COPY 4
postgres=# \q
-bash-4.2$ cat /tmp/emp.txt
```

1       ram

1       ram

1       ram

1       ram


## 27. BACKUP AND RESTORE

->we will take backup to keep data safe


->if something crashed, to recover the data.


->to migrate data from one servrer to another server (data refresh)


we have 2 types of backups:


1.logical backup


2.physical backup


## 28. LOGICAL BACKUP

->backup will be in logical format.


->it will not contains any data files of objects.


->not in readbale format.


->to take backup we have 2 utilities


a>pg_dump


b>pg_dumpall


### a>pg_dump

->to take single table backup


->to take single database backup


->to take multiple tables backup


->to take single schema backup


->to take multiple schemas backup


->at a time we can connect only one database, so we can take only one database backup at a time.


->whole server backup not possible.


->backup in custom format (compressed/tar/directory) is possible.


->parallel jobs also possible.


->global object backup not possible (users/user access rights/tablespaces)


### b>pg_dumpall

->only whole server backup possible


->global objects backup possible


->backup only in plain format


->no prallel jobs possible


->usefull to migrate data of whole server to another server.


->this backup not suitable for point intime recovery.


->from this backup whole server restore only possible, single table/object restore not possible.


> **note: while taking backup, no locks will apply**


if you take backup in plain format, to restore we need to use only psql utility.


if you take backup in custom format, to restore we need to use pg_Restore utility.


```bash
      pg_restore is 10 times faster than psql.

```

->due to security conflict, not recomended to take backup in plain format.


### physical backup

->it is exact of data directory.


->it will contains all data files.


->we will call this as file system backup as well


->we will call it as online backup


->we will call it as offline backup as well.


->we will use pg_basebackup utility, so we can call it as basebackup as well.


->this is suitable to perfrom PITR as well.


->from this backup restoring single objects not possible.


### practical scenorios

```bash
-bash-4.2$ cd /var/lib/pgsql/backup/

-bash-4.2$ pwd
```

/var/lib/pgsql/backup


rajdb=# \dt raj_schema.*;

List of relations

Schema   | Name | Type  |  Owner

------------+------+-------+----------

raj_schema | dept | table | postgres

raj_schema | emp  | table | postgres

(2 rows)


scenorio1: take single table backup

```bash
-bash-4.2$ pg_dump -d rajdb -t raj_schema.dept>dep.sql
-bash-4.2$ ls -ltr
```

total 1448

-rw-r--r-- 1 postgres postgres 1478682 Mar 15 21:37 dep.sql

-bash-4.2$


restore table:

```sql
postgres=# \c rajdb
```

You are now connected to database "rajdb" as user "postgres".

rajdb=# drop raj


rajdb=# drop table raj_schema.dept;

```sql
DROP TABLE
```

rajdb=#


```bash
-bash-4.2$ psql -d rajdb -f dep.sql -o dep.log
-bash-4.2$ ls -ltr
```

total 1452

-rw-r--r-- 1 postgres postgres 1478682 Mar 15 21:37 dep.sql

-rw-r--r-- 1 postgres postgres     118 Mar 15 21:39 dep.log

-bash-4.2$

```bash
-bash-4.2$ cat dep.log
```

SET

SET

SET

SET

SET

set_config

------------


(1 row)


SET

SET

SET

SET

SET

SET

```sql
CREATE TABLE
ALTER TABLE
COPY 100000


rajdb=# \dt raj_schema.*;
```

List of relations

Schema   | Name | Type  |  Owner

------------+------+-------+----------

raj_schema | dept | table | postgres

raj_schema | emp  | table | postgres


scenorio2: backup only table data:

```bash
-bash-4.2$ pg_dump -d rajdb -t raj_schema.dept -a>dept.sql

```

scenorio3: backup only table structure:

```bash
-bash-4.2$ pg_dump -d rajdb -t raj_schema.dept -s>dept_strcut.sql

```

scenorio4:restore from structure & data:

```sql
postgres=# \c rajdb
```

You are now connected to database "rajdb" as user "postgres".

rajdb=# drop table raj_schema.dept;

```sql
DROP TABLE

-bash-4.2$ psql -d rajdb -f dept_strcut.sql -o dept_strcut.log
-bash-4.2$ psql -d rajdb -f dept.sql -o dept.log

-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# \c rajdb
```

You are now connected to database "rajdb" as user "postgres".

rajdb=# select count(*) from raj_schema.dept;

count

--------

100000

(1 row)


scenorio5: multiple tables backup at a time

```bash
-bash-4.2$ pg_dump -d rajdb -t raj_schema.dept -t raj_schema.emp>multiple.sql
-bash-4.2$ ls -ltr
```

total 1448

-rw-r--r-- 1 postgres postgres 1479039 Mar 15 21:45 multiple.sql


if you take multiple tables backup in plain format, single table restore not possible.


if we take backup in custom format, then only restore single table is possible.


scenorio6: take schema backup

```bash
-bash-4.2$ pg_dump -d rajdb -n raj_schema>schemabackup.sql
-bash-4.2$ ls -ltr
```

total 1448

-rw-r--r-- 1 postgres postgres 1479387 Mar 15 21:47 schemabackup.sql


for multiple schemas : -n s1  -n s2 (we can use like this)


scenorio7: whole database backup

```bash
-bash-4.2$ pg_dump -d rajdb>wholedb.sql

scenorio8: take the table backup in custom format:(bin)
```

```bash
-bash-4.2$ pg_dump -Fc -d rajdb -t raj_schema.dept -f dept.bin -v 2>dept.log
-bash-4.2$ ls -ltr
```

total 444

-rw-r--r-- 1 postgres postgres   1921 Mar 16 20:40 dept.log

-rw-r--r-- 1 postgres postgres 446936 Mar 16 20:40 dept.bin

-bash-4.2$


make sure no error messages in the log file:


```bash
-bash-4.2$ cat dept.log|grep -i error
-bash-4.2$ cat dept.log|grep -i FATAL

```

scnorio9: restore table:

```bash
-bash-4.2$ pg_restore -Fc -d rajdb -n raj_schema -t dept -c dept.bin -v 2>dept_restore.log
-bash-4.2$ cat dept_restore.log
```

pg_restore: connecting to database for restore

pg_restore: dropping TABLE dept

pg_restore: creating TABLE "raj_schema.dept"

pg_restore: processing data for table "raj_schema.dept"

-bash-4.2$


scnorio10: backup multiple tables:

```bash
-bash-4.2$ pg_dump -Fc -d rajdb -t raj_schema.dept -t raj_schema.emp -f multi.bin -v 2>multi.log
-bash-4.2$ ls -ltr
```

total 444

-rw-r--r-- 1 postgres postgres   2038 Mar 16 20:46 multi.log

-rw-r--r-- 1 postgres postgres 447411 Mar 16 20:46 multi.bin


```bash
-bash-4.2$ cat multi.log|grep 'dumping contents'
```

pg_dump: dumping contents of table "raj_schema.dept"

pg_dump: dumping contents of table "raj_schema.emp"


scenorio11: restore single table using the mult backup

```bash
-bash-4.2$ pg_restore -Fc -d rajdb -n raj_schema -t dept -c multi.bin -v 2>singletab_restore.log
-bash-4.2$ cat singletab_restore.log
```

pg_restore: connecting to database for restore

pg_restore: dropping TABLE dept

pg_restore: creating TABLE "raj_schema.dept"

pg_restore: processing data for table "raj_schema.dept"

-bash-4.2$


scenori12: take the schema backup in tar format:

```bash
-bash-4.2$ pg_dump -Ft -d rajdb -n raj_schema -f raj_schema.tar -v 2>raj_schema_bkp.log
-bash-4.2$ ls -ltr
```

total 1456

-rw-r--r-- 1 postgres postgres    2485 Mar 16 20:51 raj_schema_bkp.log

-rw-r--r-- 1 postgres postgres 1486848 Mar 16 20:51 raj_schema.tar


```bash
-bash-4.2$ cat raj_schema_bkp.log|grep 'dumping contents'
```

pg_dump: dumping contents of table "raj_schema.dept"

pg_dump: dumping contents of table "raj_schema.emp"

-bash-4.2$


scenori12: restore the schema backup :

```bash
-bash-4.2$ pg_restore -Ft -d rajdb -n raj_schema -c raj_schema.tar -v 2>raj_schema_restore.log
-bash-4.2$ cat raj_schema_restore.log
```

pg_restore: connecting to database for restore

pg_restore: dropping VIEW emm_v

pg_restore: dropping TABLE emp

pg_restore: dropping TABLE dept

pg_restore: creating TABLE "raj_schema.dept"

pg_restore: creating TABLE "raj_schema.emp"

pg_restore: creating VIEW "raj_schema.emm_v"

pg_restore: processing data for table "raj_schema.dept"

pg_restore: processing data for table "raj_schema.emp"

-bash-4.2$


scenori13: restore single table from the schema backup :

```bash
-bash-4.2$ pg_restore -Ft -d rajdb -n raj_schema -t dept -c raj_schema.tar -v 2>restore_singletable.log
-bash-4.2$ cat restore_singletable.log
```

pg_restore: connecting to database for restore

pg_restore: dropping TABLE dept

pg_restore: creating TABLE "raj_schema.dept"

pg_restore: processing data for table "raj_schema.dept"

-bash-4.2$


scenori14: take the database backup in tar format :

```bash
-bash-4.2$ pg_dump -Ft -d rajdb -f rajd.tar -j 2 -v 2>dbabckup_parrall.log
-bash-4.2$ ls -ltr
```

total 4

-rw-r--r-- 1 postgres postgres 71 Mar 16 20:55 dbabckup_parrall.log

```bash
-bash-4.2$ cat dbabckup_parrall.log
```

pg_dump: error: parallel backup only supported by the directory format


scenori14: take the database backup in directory  format using parallel :

```bash
-bash-4.2$ pg_dump -Fd -d rajdb -f rajd -j 2 -v 2>dbabckup_parrall.log
-bash-4.2$ ls -ltr
```

total 4

drwx------ 2 postgres postgres   56 Mar 16 20:57 rajd

-rw-r--r-- 1 postgres postgres 2223 Mar 16 20:57 dbabckup_parrall.log

```bash
-bash-4.2$ cd rajd/
-bash-4.2$ ls -ltr
```

total 444

-rw-r--r-- 1 postgres postgres   2064 Mar 16 20:57 toc.dat

-rw-r--r-- 1 postgres postgres     49 Mar 16 20:57 4044.dat.gz

-rw-r--r-- 1 postgres postgres 445181 Mar 16 20:57 4045.dat.gz

-bash-4.2$


scnorio15: restore single table using whole db backup

```bash
-bash-4.2$ pg_restore -Fd -d rajdb -n raj_schema -t emp -c rajd -v 2>restore_single_dir.log
-bash-4.2$ cat restore_single_dir.log
```

pg_restore: connecting to database for restore

pg_restore: dropping TABLE emp

pg_restore: creating TABLE "raj_schema.emp"

pg_restore: processing data for table "raj_schema.emp"


### pg_dumpall: take whole server

```bash
-bash-4.2$ pg_dumpall>fullbkp.sql
-bash-4.2$ ls -ltr
```

total 1452

-rw-r--r-- 1 postgres postgres 1484934 Mar 16 21:03 fullbkp.sql


```sql
postgres=# drop user eswar;
```

ERROR:  role "eswar" cannot be dropped because some objects depend on it

DETAIL:  privileges for database postgres

```sql
postgres=#
postgres=#
postgres=# \dt
```

Did not find any relations.

```sql
postgres=#
postgres=# reassign owned by eswar to postgres;
```

REASSIGN OWNED

```sql
postgres=# drop owned by eswar;
DROP OWNED
postgres=# drop user eswar;
DROP ROLE

```

### restore using dumpall backup


```bash
-bash-4.2$ psql -f fullbkp.sql -o fullbkp.log
```

psql:fullbkp.sql:16: ERROR:  role "postgres" already exists

You are now connected to database "template1" as user "postgres".

You are now connected to database "postgres" as user "postgres".

You are now connected to database "rajdb" as user "postgres".

You are now connected to database "ram" as user "postgres".


### take only global objects backup

```bash
-bash-4.2$ pg_dumpall -g>globalobjects.sql


```

### to take whole sever in custom format

->take global objects backup using pg_dumpall


->take database wise using pg_dump custom format.


## 29. PHYSICAL BACKUP/BASEBACKUP

```bash
-bash-4.2$ pwd
```

/var/lib/pgsql/backup

```bash
-bash-4.2$ ls -ltr
```

total 0

```bash
-bash-4.2$ pg_basebackup -D /var/lib/pgsql/backup/ -X fetch -P
```

66281/66281 kB (100%), 1/1 tablespace

-bash-4.2$


--checkpoint=fast -> in case waiting for checkpoint


```bash
-bash-4.2$ pwd
```

/var/lib/pgsql/backup

```bash
-bash-4.2$ ls -ltr
```

total 292

-rw------- 1 postgres postgres    227 Mar 16 21:15 backup_label

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_commit_ts

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_twophase

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_subtrans

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_snapshots

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_serial

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_notify

drwx------ 4 postgres postgres     34 Mar 16 21:15 pg_multixact

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_dynshmem

drwx------ 7 postgres postgres     62 Mar 16 21:15 base

drwx------ 2 postgres postgres     17 Mar 16 21:15 pg_xact

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_tblspc

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_stat_tmp

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_stat

drwx------ 2 postgres postgres      6 Mar 16 21:15 pg_replslot

-rw------- 1 postgres postgres     88 Mar 16 21:15 postgresql.auto.conf

-rw------- 1 postgres postgres      3 Mar 16 21:15 PG_VERSION

drwx------ 4 postgres postgres     65 Mar 16 21:15 pg_logical

-rw------- 1 postgres postgres   1636 Mar 16 21:15 pg_ident.conf

drwx------ 2 postgres postgres     95 Mar 16 21:15 log

drwx------ 2 postgres postgres   4096 Mar 16 21:15 pg_log

-rw------- 1 postgres postgres   4650 Mar 16 21:15 pg_hba.conf

-rw------- 1 postgres postgres  28821 Mar 16 21:15 postgresql.conf

drwx------ 2 postgres postgres   4096 Mar 16 21:15 global

-rw------- 1 postgres postgres     47 Mar 16 21:15 current_logfiles

drwx------ 3 postgres postgres     58 Mar 16 21:15 pg_wal

-rw------- 1 postgres postgres 227936 Mar 16 21:15 backup_manifest


restore basebackup:

```bash
-bash-4.2$ vi postgresql.conf

```

port=5433


```bash
-bash-4.2$ chmod 0700 /var/lib/pgsql/backup


-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/backup status
```

pg_ctl: no server running

```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/backup start
```

waiting for server to start....    9237LOG:  redirecting log output to logging collector process

9237HINT:  Future log output will appear in directory "pg_log".

.. done

server started

```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/backup status
pg_ctl: server is running (PID: 9237)
```

/usr/pgsql-14/bin/postgres "-D" "/var/lib/pgsql/backup"

-bash-4.2$


-bash-4.2$

```bash
-bash-4.2$ ps -ef|grep postgres
```

postgres  3784     1  0 20:35 ?        00:00:00 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/

postgres  3786  3784  0 20:35 ?        00:00:00 postgres: logger

postgres  3797  3784  0 20:35 ?        00:00:00 postgres: checkpointer

postgres  3798  3784  0 20:35 ?        00:00:00 postgres: background writer

postgres  3799  3784  0 20:35 ?        00:00:00 postgres: walwriter

postgres  3800  3784  0 20:35 ?        00:00:00 postgres: archiver last was 000000010000000000000068.00000028.backup

postgres  3801  3784  0 20:35 ?        00:00:00 postgres: stats collector

postgres  3802  3784  0 20:35 ?        00:00:00 postgres: logical replication launcher

root      4176  3716  0 20:38 pts/0    00:00:00 su - postgres

postgres  4177  4176  0 20:38 pts/0    00:00:00 -bash

postgres  9237     1  0 21:17 ?        00:00:00 /usr/pgsql-14/bin/postgres -D /var/lib/pgsql/backup

postgres  9238  9237  0 21:17 ?        00:00:00 postgres: logger

postgres  9241  9237  0 21:17 ?        00:00:00 postgres: checkpointer

postgres  9242  9237  0 21:17 ?        00:00:00 postgres: background writer

postgres  9243  9237  0 21:17 ?        00:00:00 postgres: walwriter

postgres  9244  9237  0 21:17 ?        00:00:00 postgres: archiver

postgres  9245  9237  0 21:17 ?        00:00:00 postgres: stats collector

postgres  9246  9237  0 21:17 ?        00:00:00 postgres: logical replication launcher

postgres  9299  4177  0 21:18 pts/0    00:00:00 ps -ef

postgres  9300  4177  0 21:18 pts/0    00:00:00 grep --color=auto postgres

-bash-4.2$


```bash
-bash-4.2$ psql -p 5432
psql (14.2)
```

Type "help" for help.


```sql
postgres=# show data_directory;
```

data_directory

------------------------

/var/lib/pgsql/14/data

(1 row)


```sql
postgres=#


-bash-4.2$ psql -p 5433
psql (14.2)
```

Type "help" for help.


```sql
postgres=# show data_directory;
```

data_directory

-----------------------

/var/lib/pgsql/backup

(1 row)


## 30. RESTORE BASEBACKUP

/usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data stop


```bash
mv /var/lib/pgsql/14/data /var/lib/pgsql/14/data_old

mv /var/lib/pgsql/backup /var/lib/pgsql/14/data

```

/usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data start


## 31. PITR

daily backup : 6am, bkp will complete by 6:10am


developer created table by 7am, he done loading upto 8:30 am (1 and 1/2 hr they loaded)


suddenly 8:31 am table got dropped.


eventhough we dont have backup for the specific point of time, still we can able to recover to specific point of time using archive files,


we call this concept as point intime recovery.


->make sure archive mode enabled.


```bash
-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# show archive_mode;
```

archive_mode

--------------

on

(1 row)


```sql
postgres=# show archive_command;
```

archive_command

---------------------------------

```bash
 cp %p /var/lib/pgsql/archive/%f
(1 row)

postgres=#


```

backup taken :

```bash
-bash-4.2$ pg_basebackup -D /var/lib/pgsql/backup/ -X fetch -P
```

66286/66286 kB (100%), 1/1 tablespace

-bash-4.2$


in case user created tablespace is present:- We have to include those tablespace backups as well , we can put any location for those and

******************************************* automatically symbolic link will be updated in that new backup folder pg_tblspac, it will be pointing to that

new location.

We can use below command for the same.

```bash
pg_basebackup -D /var/lib/pgsql_15/backup -X fetch -P -T /var/lib/pgsql_15/tablespace/akash_ts1=/var/lib/pgsql_15/backup/akash_ts1 -T /var/lib/pgsql_15/tablespace/akash_ts2=/var/lib/pgsql_15/backup/akash_ts2

```

Symbolic link update command: ln -vfns target_folder tablespace number

***************************** ln -vfns /var/lib/tablespace/ts1/  163456


in the current running instance pg_wal directory it will contains .backup file


-rw------- 1 postgres postgres      341 Mar 16 21:32 00000001000000000000006A.00000028.backup

drwx------ 2 postgres postgres       94 Mar 16 21:32 archive_status

-rw------- 1 postgres postgres 16777216 Mar 16 21:32 00000001000000000000006B

```bash
-bash-4.2$ pwd
```

/var/lib/pgsql/14/data/pg_wal

-bash-4.2$


below are the backup start time and stop time:

```bash
-bash-4.2$ cat 00000001000000000000006A.00000028.backup
START WAL LOCATION: 0/6A000028 (file 00000001000000000000006A)
STOP WAL LOCATION: 0/6A000100 (file 00000001000000000000006A)
CHECKPOINT LOCATION: 0/6A000060
```

BACKUP METHOD: streamed

BACKUP FROM: primary

START TIME: 2022-03-16 21:32:54 EDT

LABEL: pg_basebackup base backup

START TIMELINE: 1

STOP TIME: 2022-03-16 21:32:55 EDT

STOP TIMELINE: 1

-bash-4.2$


```sql
postgres=# create table pitr_test(eno int,ename varchar,sal int);
CREATE TABLE
postgres=# insert into pitr_test values(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
INSERT 0 100000
postgres=# insert into pitr_test values(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
INSERT 0 100000
postgres=# insert into pitr_test values(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
INSERT 0 100000
postgres=# insert into pitr_test values(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
INSERT 0 100000
postgres=# insert into pitr_test values(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
INSERT 0 100000
postgres=# \dt+
```

List of relations

Schema |   Name    | Type  |  Owner   | Persistence | Access method | Size  | Description

--------+-----------+-------+----------+-------------+---------------+-------+-------------

public | pitr_test | table | postgres | permanent   | heap          | 25 MB |

(1 row)


```sql
postgres=# select count(*) from pitr_test;
```

count

--------

500000

(1 row)


table is avaible in the server upto below time:


```sql
postgres=# select now();
```

now

-------------------------------

2022-03-16 21:36:55.224268-04

(1 row)


```sql
postgres=# \dt
```

List of relations

Schema |   Name    | Type  |  Owner

--------+-----------+-------+----------

public | pitr_test | table | postgres

(1 row)


suddenly table dropped:


```sql
postgres=# drop table pitr_test;
DROP TABLE


```

->only option to recover the table is on top of fullbackup apply archive files until specific point of time.


->copy missed wal files from pg_wal to archive location.


```bash
-bash-4.2$ cp -p /var/lib/pgsql/14/data/pg_wal/00000001000000000000006D .
-bash-4.2$ ls -ltr 00000001000000000000006D
```

-rw------- 1 postgres postgres 16777216 Mar 16 21:38 00000001000000000000006D

```bash
-bash-4.2$ pwd
```

/var/lib/pgsql/archive

-bash-4.2$


->goto backup location and edit postgresql.conf with port 5433


```bash
-bash-4.2$ cd /var/lib/pgsql/backup/
-bash-4.2$ vi postgresql.conf

```

->to perfrom recovery add few entries to postgresql.auto.conf file in backup.


```bash
-bash-4.2$ cat postgresql.auto.conf
```

# Do not edit this file manually!

# It will be overwritten by the ALTER SYSTEM command.

restore_command='cp /var/lib/pgsql/archive/%f %p'

recovery_target_time='2022-03-16 21:36:55.224268-04'


->we need to inform to system, go with recovery, by creating one dummy file recovery.signal.


```bash
-bash-4.2$ touch recovery.signal

```

->-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/backup/ start

waiting for server to start....    12608LOG:  redirecting log output to logging collector process

12608HINT:  Future log output will appear in directory "pg_log".

.. done

server started


->go to log file


```bash
-bash-4.2$ pwd
```

/var/lib/pgsql/backup

```bash
-bash-4.2$ cd pg_log


-bash-4.2$ cat postgresql-2022-03-16_214441.log
```

12608LOG:  starting PostgreSQL 14.2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44), 64-bit

12608LOG:  listening on IPv6 address "::1", port 5433

12608LOG:  listening on IPv4 address "127.0.0.1", port 5433

12608LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5433"

12608LOG:  listening on Unix socket "/tmp/.s.PGSQL.5433"

12610LOG:  database system was interrupted; last known up at 2022-03-16 21:32:54 EDT

cp: cannot stat ‘/var/lib/pgsql/archive/00000002.history’: No such file or directory

12610LOG:  starting point-in-time recovery to 2022-03-16 21:36:55.224268-04

12610LOG:  restored log file "00000001000000000000006A" from archive

12610LOG:  redo starts at 0/6A000028

12610LOG:  consistent recovery state reached at 0/6A000100

12608LOG:  database system is ready to accept read-only connections

12610LOG:  restored log file "00000001000000000000006B" from archive

12610LOG:  restored log file "00000001000000000000006C" from archive

12610LOG:  restored log file "00000001000000000000006D" from archive

12610LOG:  recovery stopping before commit of transaction 1310, time 2022-03-16 21:37:22.501634-04

12610LOG:  pausing at the end of recovery

12610HINT:  Execute pg_wal_replay_resume() to promote.


```bash
-bash-4.2$ psql -p 5433
psql (14.2)
```

Type "help" for help.


```sql
postgres=# select pg_wal_replay_resume();
```

pg_wal_replay_resume

----------------------


(1 row)


validate table created or not:


```sql
postgres=# \dt
```

List of relations

Schema |   Name    | Type  |  Owner

--------+-----------+-------+----------

public | pitr_test | table | postgres

(1 row)


```sql
postgres=# show port;
```

port

------

5433

(1 row)


```sql
postgres=#

```

table we recovered upto specific point of time in backup instance, but we need this table in original instance.


####take table backup and restore it into new instance

```bash
-bash-4.2$ pg_dump -d postgres -t pitr_test -p 5433|psql -d postgres -p 5432
```

SET

SET

SET

SET

SET

set_config

------------


(1 row)


SET

SET

SET

SET

SET

SET

```sql
CREATE TABLE
ALTER TABLE
COPY 500000

-bash-4.2$ psql
psql (14.2)
```

Type "help" for help.


```sql
postgres=# show port;
```

port

------

5432

(1 row)


```sql
postgres=# show data_directory;
```

data_directory

------------------------

/var/lib/pgsql/14/data

(1 row)


```sql
postgres=# \dt
```

List of relations

Schema |   Name    | Type  |  Owner

--------+-----------+-------+----------

public | pitr_test | table | postgres

(1 row)


```sql
postgres=#

```

## 32. DB UPGRADE

->to get the new features


->existing features optimzation


->application dependency


->to overcome security/bug fixes


->organization commitment with clinets


upgrade: move or migrating from older version to higher version we call it as upgrade.


we have 2 types of upgrades:


->minor upgrade : if there is a change in release


->major upgrade : if there is a change in version


ex: 10.13

11.14

12.13


first digit we call it as version


second digit we call it as release.


->we will perform database upgrades in 2 ways.


->1.manual upgrade


->2.auto upgrade


## 33. MANUAL UPGRADE BEFORE 8.4

before 8.4 version, for database upgrade only option is manual upgrade, as there is no auto upgrade utility.


below are the steps for manual upgrade:


ex: upgrade from 8.1 to 8.4 version


below are the 8.1 details:

BIN : /usr/local/pgsql/bin

DD  : /var/lib/pgsql/data/81

port : 5432


install 8.4 binaries in separate folder:

BIN : /usr/local/pgsql_84/bin [./configure --prefix=/usr/local/pgsql_84/]

DD  : /var/lib/pgsql/data/84

port : 5433


start servicces like below.


/usr/local/pgsql_84/bin/pg_ctl -D /var/lib/pgsql/data/84 start


->ask application team to stop jobs for data consistency


->restart 8.1 version instance for consistency.


/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql/data/81 restart


->take full backup from 8.1 using below command.


/usr/local/pgsql/bin/pg_dumpall -p 5432>fullserverbkp.sql


->restore into 8.4 using below command.


/usr/local/pgsql_84/bin/psql -f fullserverbkp.sql -p 5433 -o fullrestore.log


->update the postgresql.conf file and pg_hba.conf file entries same as 8.1 to 8.4.


->run the analyze on all databases in 8.4


->completely stop 8.1 services.


## 34. AUTO UPGRADE

->to perform auto upgrade we have an utility called pg_upgrade.


->this utility came from 8.4 version onwards.


to upgrade below 8.4 to above 8.4 any version we need 2 step upgrade.


8.1 to 9.6 : requires 2 phase upgrade [8.1 to 8.4 manual upgrade and 8.4 to 9.6 auto upgrade] as 8.4 has lot of changes.


/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql/data/81 stop


->go to 8.4 data directory and change port to 5432 and restart services


->now 8.4 running with 5432 port.


->ask application team to start servics.


*************************sample data loading******************************************************

> **note: https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip(sampel data)**


```sql
create database dvdrental;

-bash-4.2$ pg_restore -Ft -d dvdrental dvdrental.tar -v 2>dvdrental.log

```


### database upgrade from 11 to 14

below are the 11 version details.


BIN: /usr/pgsql-11/bin/

DD: /var/lib/pgsql/11/data/

port : 5432


->whenver we have plan for database upgrade we need to check few things. especailly space.


->for new data directory we need extra space, plan before upgrade.


->install new version in different location.


1. https://www.postgresql.org/download/

2. click on package

3. choose linux system(Redhat or Centos, ubuntu)

4. choose required version, platform, architecture(u can get these details by command:hostnamectl or which --version)


# Install the repository RPM:

```bash
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

```

# Install PostgreSQL:

```bash
sudo yum install -y postgresql14-server

```

# Optionally initialize the database and enable automatic start:

```bash
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14

```

below are the 14 details:

BIN : /usr/pgsql-14/bin

DD: /var/lib/pgsql/14/data/

port : 5433  (have to change port number in postgresql.conf file to 5433) then only start 14.


check the database upgrade compatability:

newbin/pg_upgrade -b oldbin -B newbin -d olddatadir -D newdatadir -p oldport -P newport -c


stop new instance services:


```bash
[root@mariadb1 data]# systemctl stop postgresql-14


```

run the preupgrade compatabilty:


```bash
-bash-4.2$ pwd
```

/var/lib/pgsql

-bash-4.2$

```bash
-bash-4.2$ mkdir upgrade
-bash-4.2$ cd upgrade/
-bash-4.2$ pwd
```

/var/lib/pgsql/upgrade

-bash-4.2$


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_upgrade -b /usr/pgsql-11/bin/ -B /usr/pgsql-14/bin/ -d /var/lib/pgsql/11/data/ -D /var/lib/pgsql/14/data/ -p 5432 -P 5433 -c
```

Performing Consistency Checks on Old Live Server

------------------------------------------------

Checking cluster versions                                   ok

Checking database user is the install user                  ok

Checking database connection settings                       ok

Checking for prepared transactions                          ok

Checking for system-defined composite types in user tables  ok

Checking for reg* data types in user tables                 ok

Checking for contrib/isn with bigint-passing mismatch       ok

Checking for user-defined encoding conversions              ok

Checking for user-defined postfix operators                 ok

Checking for tables WITH OIDS                               ok

Checking for invalid "sql_identifier" user columns          ok

Checking for presence of required libraries                 ok

Checking database user is the install user                  ok

Checking for prepared transactions                          ok

Checking for new cluster tablespace directories             ok


*Clusters are compatible*

-bash-4.2$


### during database upgrade

->take the objects counts for validation from old version.


```sql
dvdrental=# select count(*) from pg_stat_user_tables;
```

count

-------

15

(1 row)


```sql
dvdrental=# select count(*) from pg_stat_user_indexes;
```

count

-------

32

(1 row)


```sql
dvdrental=# select count(*) from pg_views where schemaname='public';
```

count

-------

7

(1 row)


```sql
dvdrental=# select count(*) from information_schema.table_privileges where grantee<>'postgres' and table_schema not in('pg_catalog','pg_toast','information_schema');
```

count

-------

0

(1 row)


take the list also.


```sql
dvdrental=# \o grants.sql
dvdrental=# select 'grant '||privilege_type||' on '||table_schema||'.'||table_name||' to '||grantee||';' from information_schema.table_privileges where grantee<>'postgres' and table_schema not in('pg_catalog','pg_toast','information_schema');
dvdrental=# \q

postgres=# select count(*) from pg_user;
```

count

-------

2

(1 row)


->take pg_basebackup from the server.


->ask application team to stop the jobs


->restart database services


->ask vmware team to take snapshot.


->start actual upgrade:

->stop old instance as well.


```bash
[root@mariadb1 data]# systemctl stop postgresql-11

-bash-4.2$ cd upgrade/
-bash-4.2$ ls -ltr
```

total 20

-rw------- 1 postgres postgres  179 Mar 17 21:15 pg_upgrade_utility.log

-rw------- 1 postgres postgres 1249 Mar 17 21:15 pg_upgrade_internal.log

-rw------- 1 postgres postgres 1595 Mar 17 21:15 pg_upgrade_server.log

-rw-r--r-- 1 postgres postgres 4235 Mar 17 21:29 grants.sql

-bash-4.2$

```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_upgrade -b /usr/pgsql-11/bin/ -B /usr/pgsql-14/bin/ -d /var/lib/pgsql/11/data/ -D /var/lib/pgsql/14/data/ -p 5432 -P 5433
```

Performing Consistency Checks

-----------------------------

Checking cluster versions                                   ok

Checking database user is the install user                  ok

Checking database connection settings                       ok

Checking for prepared transactions                          ok

Checking for system-defined composite types in user tables  ok

Checking for reg* data types in user tables                 ok

Checking for contrib/isn with bigint-passing mismatch       ok

Checking for user-defined encoding conversions              ok

Checking for user-defined postfix operators                 ok

Checking for tables WITH OIDS                               ok

Checking for invalid "sql_identifier" user columns          ok

Creating dump of global objects                             ok

Creating dump of database schemas

ok

Checking for presence of required libraries                 ok

Checking database user is the install user                  ok

Checking for prepared transactions                          ok

Checking for new cluster tablespace directories             ok


If pg_upgrade fails after this point, you must re-initdb the

new cluster before continuing.


Performing Upgrade

------------------

Analyzing all rows in the new cluster                       ok

Freezing all rows in the new cluster                        ok

Deleting files from new pg_xact                             ok

Copying old pg_xact to new server                           ok

Setting oldest XID for new cluster                          ok

Setting next transaction ID and epoch for new cluster       ok

Deleting files from new pg_multixact/offsets                ok

Copying old pg_multixact/offsets to new server              ok

Deleting files from new pg_multixact/members                ok

Copying old pg_multixact/members to new server              ok

Setting next multixact ID and offset for new cluster        ok

Resetting WAL archives                                      ok

Setting frozenxid and minmxid counters in new cluster       ok

Restoring global objects in the new cluster                 ok

Restoring database schemas in the new cluster

ok

Copying user relation files

ok

Setting next OID for new cluster                            ok

Sync data directory to disk                                 ok

Creating script to delete old cluster                       ok

Checking for extension updates                              ok


Upgrade Complete

----------------

Optimizer statistics are not transferred by pg_upgrade.

Once you start the new server, consider running:

/usr/pgsql-14/bin/vacuumdb --all --analyze-in-stages


Running this script will delete the old cluster's' data files:

```bash
    ./delete_old_cluster.sh
```

-bash-4.2$


->copy parameters from 11 version postgresql.conf file and pg_hba.conf file entries to 14 server.


```bash
-bash-4.2$ cd /var/lib/pgsql/14/data/
-bash-4.2$ vi postgresql.conf

```

change port to 5432.


```bash
[root@mariadb1 data]# systemctl start postgresql-14

```

->once server up, run analyze


```bash
-bash-4.2$  /usr/pgsql-14/bin/vacuumdb --all --analyze-in-stages
vacuumdb: processing database "dvdrental": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "postgres": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "template1": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "dvdrental": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "postgres": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "template1": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "dvdrental": Generating default (full) optimizer statistics
vacuumdb: processing database "postgres": Generating default (full) optimizer statistics
vacuumdb: processing database "template1": Generating default (full) optimizer statistics
```

-bash-4.2$


->login to database and check object counts.


->ask application team to start jobs and validate.


### rollback: if application team asking to rollback

```bash
[root@mariadb1 data]# systemctl stop postgresql-14
[root@mariadb1 data]# systemctl start postgresql-11

```

ask app team to do their activites.


### rollback after 10 days: if application team asking to rollback

->ask app team to stop jobs

->restart 14 services

```bash
[root@mariadb1 data]# systemctl restart postgresql-14

```

->take pg_dumpall from 14 version


->start 11 version


```bash
[root@mariadb1 data]# systemctl start postgresql-11

```

->restore dump into postgresql-11


->run analyze in 11 and handover to app team.


if app team ok with new version, if they dont need old version, run below things.


```bash
-bash-4.2$ cd upgrade/
-bash-4.2$ ls -ltr
```

total 12

-rw-r--r-- 1 postgres postgres 4235 Mar 17 21:29 grants.sql

-rwx------ 1 postgres postgres   43 Mar 17 21:32 delete_old_cluster.sh

```bash
-bash-4.2$ ./delete_old_cluster.sh
```

-bash-4.2$


s/w uninstall manually:

```bash
[root@mariadb1 data]# yum remove postgresql11-server postgresql11-contrib

[root@mariadb1 data]# rm -rf /usr/pgsql-11

[root@mariadb1 data]# rpm -e postgresql11-libs-11.15-1PGDG.rhel7.x86_64 postgresql11-11.15-1PGDG.rhel7.x86_64


```

## 35. LINK UPGRADE WITHOUT EXTRA SPACE


/usr/pgsql-14/bin/pg_upgrade -b /usr/pgsql-11/bin/ -B /usr/pgsql-14/bin/ -d /var/lib/pgsql/11/data/ -D /var/lib/pgsql/14/data/ -p 5432 -P 5433 -k


upgrade with link mode by using -k flag.


like oracle control files, every data directory, under global folder pg_control file will be there.


/var/lib/pgsql/11/data/global/pg_control.old


if we use link upgrade , it will be renmaed as .old.


To rollback , we need to rename this file and start services.


1. Go to old server data directory global folder.

2. cp pg_control.old pg_control

3. chown postgres:postgres pg_control

4. systemctl start postgresql-11


## 36. MINOR UPGRADE

minor upgrade

14.1 to 14.2--> release change


14.1 binaries can be installed:-/usr/pgsql-14.1/bin/

14.2 binaries can be installed:-/usr/pgsql-14.2/bin/


-> We cannot download minor version directly with yum install from https://www.postgresql.org/download/ directly, instead we have to go to /etc/yum.repos./ and cat pgdg.repo file

inside this file we will get all software url.


- https://download.postgresql.org/pub/repos/yum/

here it will have all software present.


we have to manual download 4 files using (wget address).


1.  ctrl+f  server  	https://download.postgresql.org/pub/repos/yum/14/redhat/rhel-7.9-x86_64/postgresql14-server-14.0-1PGDG.rhel7.x86_64.rpm

2.  ctrl+f  libs	https://download.postgresql.org/pub/repos/yum/14/redhat/rhel-7.9-x86_64/postgresql14-libs-14.0-1PGDG.rhel7.x86_64.rpm

3.  ctrl+f  contrib	https://download.postgresql.org/pub/repos/yum/14/redhat/rhel-7.9-x86_64/postgresql14-contrib-14.0-1PGDG.rhel7.x86_64.rpm

4.  ctrl+f  pgdg        https://download.postgresql.org/pub/repos/yum/14/redhat/rhel-7.9-x86_64/postgresql14-14.0-1PGDG.rhel7.x86_64.rpm


Now install one by one in below order.

1. rpm -ivh postgresql14-libs-14.0-1PGDG.rhel7.x86_64.rpm    (install library)

2. rpm -ivh postgresql14-14.0-1PGDG.rhel7.x86_64.rpm         (install pgdg file)

3. rpm -ivh postgresql14-server-14.0-1PGDG.rhel7.x86_64.rpm  (install server file)

4. rpm -ivh postgresql14-contrib-14.0-1PGDG.rhel7.x86_64.rpm (install contrib file)


/usr/pgsql-14/bin/postgresql-14-setup initdb

```bash
systemctl start postgresql-14
sudo su - postgres
```

psql

14.0 version db running.


--> now we get a request to upgrade from 14.0 to 14.2 (minor upgrade).


1. Download 14.2 same 4 files.

2. stop current instance 14.0 (systemctl stop postgresql-14)

3. Erase 14.0 rpm files:- uninstallation

```bash
rpm -e postgresql14-libs-14.0-1PGDG.rhel7.x86_64 --nodeps
rpm -e postgresql14-14.0-1PGDG.rhel7.x86_64 --nodeps
rpm -e postgresql14-server-14.0-1PGDG.rhel7.x86_64  --nodeps
rpm -e postgresql14-contrib-14.0-1PGDG.rhel7.x86_64  --nodeps

```

4. Now install one by one 14.2 version files in below order

a. rpm -ivh postgresql14-libs-14.2-1PGDG.rhel7.x86_64.rpm    (install library)

b. rpm -ivh postgresql14-14.2-1PGDG.rhel7.x86_64.rpm         (install pgdg file)

c. rpm -ivh postgresql14-server-14.2-1PGDG.rhel7.x86_64.rpm  (install server file)

d. rpm -ivh postgresql14-contrib-14.2-1PGDG.rhel7.x86_64.rpm (install contrib file)


5. start instance(systemctl start postgresql-14)

```bash
sudo su - postgres
```

psql

14.2 version db running.


```sql
Rollback of minor upgrade:-
```

1. Stop service

2. Uninstall/Erase 14.2 rpm files using rpm -e

3. Download and Install 14.0 rpm files

4. start service


## 37. PG BENCH FOR PERFORMANCE TESTING

-> maximum capacity of database with respect to response time.

-> set of queries executed how is database behaviour.

-> to estimate capacity of database, pg_bench is used.


-> requirement from application , hey for this database we want

max response time should be < 1min

how many max queries we can run on this db? we can check using this utility.


-> /usr/pgsql-14/bin/pgbench -i raj        (initiliase pgbench)

-> 4 tables will be created for managing estimation.


/usr/pgsql-14/bin/pgbench -c 5 -t 5  raj   ( 5 clients, 5 transactions total-25)


```bash
[postgres@ip-192-168-92-128 ~]$ pgbench -c 5 -t 5 akash
pgbench (15.5)
```

starting vacuum...end.

transaction type: <builtin: TPC-B (sort of)>

scaling factor: 1

query mode: simple

number of clients: 5

number of threads: 1

maximum number of tries: 1

number of transactions per client: 5

number of transactions actually processed: 25/25

number of failed transactions: 0 (0.000%)

latency average = 6.366 ms				   query response time.

initial connection time = 14.414 ms

tps = 785.397883 (without initial connection time)         785 queries per second can be processed.


->pgbench --help

->pgbench -S -c 100 -t 50 akash         --> select only.

->pgbench -c 10 -t 10 -f test.sql akash --> given queries testing. sample queries also we can run pgbench.


## 38. PG BADGER FOR LOG ANALYSIS REPORT

default log file is in text format and in gb's.


so reading this and preparing report not possible.


but we have tool called pgbadger, to analyze log file and to generate report in html format.


->Download pgbadger


```bash
wget https://sourceforge.net/projects/pgbadger/files/12.4/pgbadger-12.4.tar.gz  --no-check-certificate 

tar -xvf pgbadger-12.4.tar.gz

[postgres@ip-192-168-92-128 ~]$ ll -ltr
```

total 3984

drwxrwxr-x. 6 postgres postgres    4096 Dec 25 11:51 pgbadger-12.4

-rw-rw-r--. 1 postgres postgres 4069487 Dec 25 11:56 pgbadger-12.4.tar.gz

-rw-------. 1 postgres postgres    1348 Mar 21 22:03 logfile

```bash
[postgres@ip-192-168-92-128 ~]$ cd pgbadger-12.4
[postgres@ip-192-168-92-128 pgbadger-12.4]$ ll -ltr
```

total 1840

drwxrwxr-x. 2 postgres postgres    4096 Dec 25 11:51 tools

drwxrwxr-x. 4 postgres postgres    4096 Dec 25 11:51 t

drwxrwxr-x. 3 postgres postgres    4096 Dec 25 11:51 resources

-rw-rw-r--. 1 postgres postgres   37986 Dec 25 11:51 README.md

-rw-rw-r--. 1 postgres postgres   40439 Dec 25 11:51 README

-rwxrwxr-x. 1 postgres postgres 1610878 Dec 25 11:51 pgbadger

-rw-rw-r--. 1 postgres postgres     250 Dec 25 11:51 META.yml

-rw-rw-r--. 1 postgres postgres      81 Dec 25 11:51 MANIFEST

-rw-rw-r--. 1 postgres postgres    2265 Dec 25 11:51 Makefile.PL

-rw-rw-r--. 1 postgres postgres     910 Dec 25 11:51 LICENSE

-rw-rw-r--. 1 postgres postgres    1487 Dec 25 11:51 HACKING.md

drwxrwxr-x. 2 postgres postgres      25 Dec 25 11:51 doc

-rw-rw-r--. 1 postgres postgres    1067 Dec 25 11:51 CONTRIBUTING.md

-rw-rw-r--. 1 postgres postgres  148119 Dec 25 11:51 ChangeLog


below are the steps from README:


```bash
tar xzf pgbadger-11.x.tar.gz
cd pgbadger-11.x/
yum install perl-devel
```

perl Makefile.PL


```bash
[root@ip-192-168-92-128 pgbadger-12.4]# perl Makefile.PL
```

Checking if your kit is complete...

Looks good

Writing Makefile for pgBadger


```bash
[root@ip-192-168-92-128 pgbadger-12.4]# make
which: no pod2markdown in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
```

Makefile:824: You must install pod2markdown to generate README.md from doc/pgBadger.pod

```bash
cp pgbadger blib/script/pgbadger
```

/usr/bin/perl -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/pgbadger

```bash
echo "=head1 SYNOPSIS" > doc/synopsis.pod
./pgbadger --help >> doc/synopsis.pod
echo "=head1 DESCRIPTION" >> doc/synopsis.pod
sed -i.bak 's/ +$//g' doc/synopsis.pod
rm doc/synopsis.pod.bak
sed -i.bak '/^=head1 SYNOPSIS/,/^=head1 DESCRIPTION/d' doc/pgBadger.pod
sed -i.bak '4r doc/synopsis.pod' doc/pgBadger.pod
rm doc/pgBadger.pod.bak
```

Manifying blib/man1/pgbadger.1p

```bash
rm doc/synopsis.pod


[root@ip-192-168-92-128 pgbadger-12.4]# make install
which: no pod2markdown in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
```

Makefile:824: You must install pod2markdown to generate README.md from doc/pgBadger.pod

```bash
echo "=head1 SYNOPSIS" > doc/synopsis.pod
./pgbadger --help >> doc/synopsis.pod
echo "=head1 DESCRIPTION" >> doc/synopsis.pod
sed -i.bak 's/ +$//g' doc/synopsis.pod
rm doc/synopsis.pod.bak
sed -i.bak '/^=head1 SYNOPSIS/,/^=head1 DESCRIPTION/d' doc/pgBadger.pod
sed -i.bak '4r doc/synopsis.pod' doc/pgBadger.pod
rm doc/pgBadger.pod.bak
```

Manifying blib/man1/pgbadger.1p

Installing /usr/local/share/man/man1/pgbadger.1p

Installing /usr/local/bin/pgbadger

Appending installation info to /usr/lib64/perl5/perllocal.pod

```bash
rm doc/synopsis.pod


```

check where pgbadger is installed:-


```bash
[root@ip-192-168-92-128 pgbadger-12.4]# find / -name pgbadger
```

/root/pgbadger-12.4/pgbadger

/root/pgbadger-12.4/blib/script/pgbadger

/usr/local/bin/pgbadger

/home/postgres/pgbadger-12.4/pgbadger


to utilize pgabdger , log related parameters need to update.


log_destination = 'stderr'

logging_collector = on

log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'

log_rotation_age = 1d

log_rotation_size = 10MB

log_min_duration_statement = 0

log_connections = on

log_disconnections = on

log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h'

log_statement = 'all'

log_autovacuum_min_duration = 0

autovacuum=0


Restart instance:-


```bash
[postgres@ip-192-168-92-128 ~]$ cd /var/lib/pgsql_15/DATA
[postgres@ip-192-168-92-128 DATA]$ pg_ctl reload
```

server signaled

```bash
[postgres@ip-192-168-92-128 DATA]$ pg_ctl stop
```

waiting for server to shut down.... done

server stopped

```bash
[postgres@ip-192-168-92-128 DATA]$ pg_ctl start
```

waiting for server to start....2024-04-11 22:42:09 PDT [3944]: [1-1] user=,db=,app=,client=LOG:  redirecting log output to logging collector process

2024-04-11 22:42:09 PDT [3944]: [2-1] user=,db=,app=,client=HINT:  Future log output will appear in directory "pg_log".

done

server started


```bash
[postgres@ip-192-168-92-128 DATA]$ cd pg_log
[postgres@ip-192-168-92-128 pg_log]$ ll -ltr
```

total 6420

-rw-------. 1 postgres postgres   12869 Apr  4 10:41 postgresql-2024-04-04_051842.log

-rw-------. 1 postgres postgres  734867 Apr  7 05:50 postgresql-2024-04-07_005538.log

-rw-------. 1 postgres postgres  367786 Apr  8 09:48 postgresql-2024-04-08_091306.log

-rw-------. 1 postgres postgres   20343 Apr  8 20:56 postgresql-2024-04-08_095155.log

-rw-------. 1 postgres postgres   24446 Apr  8 23:17 postgresql-2024-04-08_225821.log

-rw-------. 1 postgres postgres   11995 Apr  9 05:34 postgresql-2024-04-09_000000.log

-rw-------. 1 postgres postgres    1729 Apr 10 04:33 postgresql-2024-04-10_040046.log

-rw-------. 1 postgres postgres 5327004 Apr 11 03:44 postgresql-2024-04-11_025156.log

-rw-------. 1 postgres postgres    1403 Apr 11 03:57 postgresql-2024-04-11_035703.log

-rw-------. 1 postgres postgres   48323 Apr 11 22:42 postgresql-2024-04-11_214227.log

-rw-------. 1 postgres postgres    1685 Apr 11 22:42 postgresql-2024-04-11_224209.log


```bash
[postgres@ip-192-168-92-128 pg_log]$  pgbadger -f `find /var/lib/pgsql_15/DATA/pg_log/ -name "postgresql-2024-04-11*"` -o pgbadger-report-`date+\%F`.html
```

-bash: date+%F: command not found

LOG: Ok, generating html report... 62431 bytes of 62431 (100.00%), queries: 0, events: 0


```bash
[postgres@ip-192-168-92-128 pg_log]$ ll -ltr
```

total 7192

-rw-------. 1 postgres postgres   12869 Apr  4 10:41 postgresql-2024-04-04_051842.log

-rw-------. 1 postgres postgres  734867 Apr  7 05:50 postgresql-2024-04-07_005538.log

-rw-------. 1 postgres postgres  367786 Apr  8 09:48 postgresql-2024-04-08_091306.log

-rw-------. 1 postgres postgres   20343 Apr  8 20:56 postgresql-2024-04-08_095155.log

-rw-------. 1 postgres postgres   24446 Apr  8 23:17 postgresql-2024-04-08_225821.log

-rw-------. 1 postgres postgres   11995 Apr  9 05:34 postgresql-2024-04-09_000000.log

-rw-------. 1 postgres postgres    1729 Apr 10 04:33 postgresql-2024-04-10_040046.log

-rw-------. 1 postgres postgres 5327004 Apr 11 03:44 postgresql-2024-04-11_025156.log

-rw-------. 1 postgres postgres    1403 Apr 11 03:57 postgresql-2024-04-11_035703.log

-rw-------. 1 postgres postgres   48323 Apr 11 22:42 postgresql-2024-04-11_214227.log

-rw-------. 1 postgres postgres   10925 Apr 11 22:47 postgresql-2024-04-11_224209.log

-rw-------. 1 postgres postgres    1780 Apr 11 22:47 postgresql-2024-04-11_224727.log

-rw-rw-r--. 1 postgres postgres  776521 Apr 11 22:51 pgbadger-report-.html


pgbadger --help

pgbadger -q `find /var/lib/pgsql_15/DATA/pg_log  -name "postgresql*"` -o pgbadger-report-`date +\%F`.html -f stderr


```bash
[postgres@ip-192-168-92-128 DATA]$ cp pgbadger-report-2024-04-11.html /tmp/

```

take out html file through winscp in your local machine and check.


pgbadger -f rds -o rds_out.html /home/CALHEERS.LOCAL/postgres/pgbadger/postgresql.log.2023-05-30


```bash
./pgbadger-12.1/pgbadger -b "2023-07-19 17:00:00" -e "2023-07-19 20:30:00" -u lt02_hbx_app_user -u lt02_hbx_batch_user -f rds -o lt_pgbadger_report_2023-07-19_3.html postgresql.log.2023-07-19;


```

## 39. PASSWORD CHECK EXTENSION


```bash
vi postgresql.conf 

```

shared_preload_libraries = 'passwordcheck'


```bash
pg_ctl stop
pg_ctl start

postgres=# show shared_preload_libraries
postgres-# ;
```

shared_preload_libraries

--------------------------

passwordcheck

(1 row)


->password myst not contain username

->password length muyst 8 character

->password must contain letters

->password must contain numbers


```sql
postgres=# create user test1 with password 'test1';
```

ERROR:  password is too short


```sql
postgres=# create user test1 with password 'test1234';
```

ERROR:  password must not contain user name


log_filename = 'postgresql-%a.log' # log file name pattern,

log_truncate_on_rotation = on


above format hc not required.


log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log' # log file name pattern,

log_rotation_age = 1d                   # Automatic rotation of logfiles will

log_rotation_size = 10MB                        # Automatic rotation of logfiles will


->house keeping manually required.


## 40. REPLICATION SETUP

```sql
copy data from one server to other sever called replication.

```

always both server should be in sync.


it will work as a high availability and can used for disaster recovery.


```bash
source server to destination server.

source server -> primary server, master server
```

destination server-> secondary, slave, standby, dr


data will be copied in the form of transactional logs from one server to another server.

2 ways -


1. synchronus replication : if we run any transaction on the master server, first it will copied to standby server then standby server send acknowledgement

to master server saying that i have received data , then only those transaction commited in master. Means synchronus replication

giving high priority to slave.


-> always master bothers about standby to keep in sync.


-> if issue with standby, not able to send acknowledgement , in that case transaction will not be commited in master server.


-> all write operation will be in hung state, to avoid dependency on standby server.


2. asynchronus replication : if we run any query it will immediately commit in master, it will not bother about standby server whether data is replicating


or not.


There is no gurantee for standby server sync status.


In synchronus replication we can say always master and standby in sync.


by default we have asynchronus replication only.


Types of replication:


1. warm standby replication/log shipping based standby : for every 16 MB, wal file will be backed up and copied to standby. Between master and standby delay 16mb size.


this is until 9.1,


> **Note:-> Log file based shipping replication: Similar to above but we can setup hot standby as well using this, steps to setup this replication are mentioned in Replication folder.**


Streaming replication: Data will be replication continuously from master to standby , so lag will be very less chances. 9.2 onwards this one came.


2. multi slave replication : one master , multiple standby


3. cascading replication : A-->B-->C , less impact on master server.


In postgresql, bydefault  server level replication.


4. logical replication : To replicate set of tables from one server to other server we will use logical replication.


## 41. LOG SHIPPING BASED (WARM/HOT) STANDBY REPLICATION

Warmstandby Replication:


->this is until 9.1 version.


->transaction log/wal files  will be backed up in source and restored to destination.


->always standby serve sitting idle, it will not allow any operation.


->if master completely crashed , then only we will convert slave as master.


->this replication works only for DR purpose, not HA.


Objective: Setup log based shipping in Linux. (hot standby with log file based shipping)

Initial setup on Master server:

1)	Create ssh key on master and copy it to standby

Syntax : ssh-keygen

```sql
           Copy the ssh key to standby server
```

Syntax : ssh-copy-id  postgres@ipadress

2)	Test copying a file from master to standby without entering password

3)	Shutdown Master database

4)	Modify the content of postgres.conf file and edit parameters archive_on, archive_command and archive_timeout.

Archive_command: rsync -a %p postgres@ipaddress:/location of archive folder


Initial Setup Standby Database:

1)	Create a archive directory on standby which is accessible by Master user.

2)	Shutdown postgresql on standby

3)	Delete all the contents of /Data directory


Steps:

1)	Startup postgresql on Master database

2)	Connect to the database using psql shell

3)	Issue connect pg_start_backup

Syntax : select pg_start_backup('dbrep');

4)	Copy Data directory for master to standby.

Syntax : rsync -avz /var/lib/pgsql/12/data/* postgres@192.168.1.9:/var/lib/pgsql/12/data/

5)	End backup on master.

Syntax : select pg_stop_backup();

6)	Comment out archive_on, archive_command and archive_timeout parameters in postgresql.conf in standby server.

7)	Setup recovery command in postgresql.conf

Restore_command = 'cp /var/lib/pgsql/12/archive/%f %p'

8)	Create a standby.signal file in data directory.

9)	Start postgres on standby server

10)	Check the log files whether the postgres has entered standby mode and accepting read only connections.


Test Standby Database:

1)	Try to create table and insert few records on master and check whether they are populated

On the standby .

2 )  try to create a table on standby or delete any records on standby


Failover:

1)	Shutdown postgresql on master and check whether it affects standby

2)	Promote your standby database to primary. Use any of the 3 methods

Pg_ctl promote -D /var/lib/pgsql/12/data

3)	Check signal.standy exist or not.

4)	Bring up the postgresql old master server and try to create a table there and check whether

Its replicated.


## 42. STREAMING REPLICATION

Streaming Replication:   Wal records will be sent  via wal sender process and received on standby by wal receiver process.


->9.2 onwards streaming replication feature came.


-> Replication slot keeps track of how mych standby is lagging from master and need to retained which all wal files in order to get

standby again in sync with master. Earlier in log file based shipping setup we were using wal_keepsegment parameter in order to retained

wal files.


->transaction log/WAL record continuously streamed from master to standby, so lag will very minimal.


->always standby server available.


->we can perform read only operations on standby server.


->it will be suitable for HA and DR.


### Pre-requisite to build replication

1. Both master and standby servers should have same operating system.


uname

```bash
cat /etc/redhat-release  

```

2. Both the sides same database s/w version.


```bash
rpm -qa|grep postgres

```

3. between both the servers port 5432 should be opened for communication.

```bash
systemctl stop firewalld
systemctl disable firewalld 

```

4. make source master server in standby hba conf , and standby ip in master hba conf file.


master ip : 192.168.56.201


standby ip : 192.168.56.202


5. Build master server:


->go to data directory.


-> vi postgresql.conf


listen_addresses = '*'


wal_level = replica    --> default value.


max_wal_senders = 10   --> default value


wal_keep_size = 1024   --> 1GB wal file keep in master server


wal_log_hints = on


hot_standby = on


-> vi pg_hba.conf

TYPE DATABASE     USER 		ADDRESS		  METHOD

host replication replication   192.168.56.202/32  trust             --> standby ip address


```bash
systemctl start postgresql-14

```

-> login to db and create user


```sql
create user replication with replication;

show listen_addresses;

show wal_log_hints;

show wal_keep_size;

```

6. Build standby server:

->in standby server make sure data directory is empty.


->for the first time sync run the full backup of master server by connecting from standby server to master server in standby directory.


```bash
pg_basebackup -D /var/lib/pgsql/14/data/ -X fetch -P -R -h 192.168.56.201 -U replication	

where -R: replication, standby server should have primary server detail, -R will create connection string entry in postgresql.auto.conf file of standby server.

```

: for replication standby.signal file is mandatory, which will create automatically.


->in standby server data directory where backup has been taken , one extra file will be present called standby.signal automatically it will create.


->vi postgresql.auto.conf


primary_conninfo = 'user=replication host=192.168.56.201 port=5432'


only these details should be present.


until postgres 11 version , this primary conninfo entry in recovery.conf file, there is no standby.signal file.


-> always in pg_hba.conf files should have ip addresses and vice-versa.


-> vi pg_hba.conf

TYPE DATABASE     USER 		ADDRESS		  METHOD

host replication replication   192.168.56.201/32  trust             --> standby ip address


```bash
systemctl start postgresql-14

```

both the sides services up and running, so need to perform replication validation.


Validation from master side:


->ps -ef|grep sender

->login to db.


```sql
select * from pg_stat_replication;

```

make sure sent_lsn, write_lsn, replay_lsn, flush_lsn values are same, that means no lag between master and standby.


sent_lsn : transaction record sent to standby


write_lsn: transaction reached memory in standby


flush_lsn: transaction flushed to disk in standby


replay_lsn: transaction committed in disk and sending reply back to master.


below these column value should be blank , means there is no lag.

write_lag

flush_lag

replay_lag


current wal sent from primary:

```sql
select * from pg_current_wal_lsn();  

```

recevie on standby, both should be matching:

```sql
select * from pg_last_wal_receive_lsn(); 

```

This will give the size differene between master and standby in bytes.

```sql
select * from pg_wal_lsn_diff('0/C0197F8','0/8000110'); 

```

check lsn is actually in which physical WAL file:

```sql
select pg_walfile_name('0/C0197F8');

select * from pg_replication_slots;

```

Validation from standby server:


->ps -ef|grep receiver

->login to db.


```sql
select pg_is_in_recovery();          

```

o/p: t


standby server should be always in recovery mode true.


data validation:

do object counts and run some create,drop statement and valdiate from standby.


asynchronus replication behaviour testing:


-> stop standby.

-> create some objects in master. -> it will create in master and not bother about standby.


-> in postgresql replication nice feature is whenever standby up, replication gap will be applied automatically if you have enough wal files in master server

pg_wal directory.


-> if wal files removed from master before receiving to standby server.


-> if above situation comes replication will be break, we need to rebuild replication by running pg_basebackup.


## 43. CONVERT ASYNC TO SYNC


->go to master server data directory.


->vi postgresql.conf


synchronous_standby_names = '*'


-> to reflect parameter reload services is enough. (pg_ctl reload)


validate by : select sync_state from pg_stat_replication;  --> sync output.


test sync behaviour:


-> stop standby

-> create database sync_test;


master server will be in hung state till standby server comes up.


-> Now if we cancel this statement, it will commit in master server and giving info to user saying it has committed locally but not committed to standby server.


-> all people will go for async replication.


## 44. CONVERT SYNC TO ASYNC

->go to master server data directory.


->vi postgresql.conf

#synchronous_standby_names = '*'    --> comment this out


-> to reflect parameter reload services is enough. (pg_ctl reload)


validate by : select sync_state from pg_stat_replication;  --> sync output.


## 45. MULTI SLAVE REPLICATION SETUP

192.168.56.201 : master

192.168.56.202 : st1

192.168.56.203 : st2


->add ip address in master .201 data directory hba conf file.

-> vi pg_hba.conf

TYPE DATABASE     USER 		ADDRESS		  METHOD

host replication replication   192.168.56.202/32  trust             --> standby1 ip address

host replication replication   192.168.56.203/32  trust             --> standby2 ip address


-> Now take pgbasebackup in st1(.202) server of .201 server.

```bash
pg_basebackup -D /var/lib/pgsql/14/data/ -X fetch -P -R -h 192.168.56.201 -U replication	

pg_ctl reload 

cd /var/lib/pgsql/14/data/

```

->vi postgresql.auto.conf

primary_conninfo = 'user=replication host=192.168.56.201 port=5432'


only these details should be present.


->systemctl start postgresql-14


-> Now take pgbasebackup in st2(.203) server of .201 server.

```bash
pg_basebackup -D /var/lib/pgsql/14/data/ -X fetch -P -R -h 192.168.56.201 -U replication	


pg_ctl reload 

cd /var/lib/pgsql/14/data/

```

->vi postgresql.auto.conf

primary_conninfo = 'user=replication host=192.168.56.201 port=5432'


only these details should be present.


->systemctl start postgresql-14


Validation:

```sql
select * from pg_stat_replication;--it will show 2 records. 1st node will show as sync state and 2nd node will show potential(ready for converting to sync)

```

## 46. CASCADE REPLICATION SETUP

1->2,3 (impact on master:multi slave).

1->2->3 (impact will be less on master)


-> In standby1 server 202 hba_conf file add entry for 203.

-> vi pg_hba.conf

TYPE DATABASE     USER 		ADDRESS		  METHOD

host replication replication   192.168.56.203/32  trust


```bash
pg_ctl reload 

```

-> take pg_basebackup in 203 server for 202 standby1 server.

```bash
pg_basebackup -D /var/lib/pgsql/14/data/ -X fetch -P -R -h 192.168.56.202 -U replication

```

->systemctl start postgresql-14


->select * from pg_stat_replication; -> run in 201 master server only 1 row for 202.

-> run in 202 first standby server only 1 row for 203.


202 server acting as standby for 201.

202 server acting as master for 203.


eventhough 202 acting as a master , still able to perform read operations.


```sql
select pg_is_in_recovery();          

```

o/p: t


for both 202 and 203, for 201 it should be false.


## 47. MANUAL FAILOVER

1-> master       2-> standby


-> if 1 goes down , can make 2 as master/standalone.  (master can be make if standby is available else it is called as standalone server)


standby-> only read operations


standalone-> can allow write operations, but no standby.


master down:


```bash
systemctl stop postgresql-14

```

->cannot perform any write operation on standby as recovery mode is true.


```sql
postgres=# select pg_is_in_recovery();
```

pg_is_in_recovery

-------------------

t

(1 row)


->promote standby server as stand alone server using below command.


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/ promote
```

waiting for server to promote.... done

server promoted

-bash-4.2$


```sql
postgres=# select pg_is_in_recovery();
```

pg_is_in_recovery

-------------------

f

(1 row)


->now it is standalone server, u can perform write operations.


->now load some data , run few insert queries.


->now old master is ready, can i make my old master as master directly? no, as stand alone server has more data.


->first we will take extra changes/transactions from stand alone to old master.


->to take only modified blocks from stand alone  server to old master, we have one utility i.e. pg_rewind. (we can take pg_basebackup as well but that will take time)


->in old master perform below step:

```bash
systemctl start postgresql-14
systemctl stop postgresql-14

```

->in wal files under pg_wal numbers 1,2,3.. we will call it as time lines.


->both old master and stand alone are in different timelines.


-> in 202 server hba conf file, 201 entry should be present.


-> vi pg_hba.conf(202 server)

TYPE DATABASE     USER 		ADDRESS		  METHOD

host replication replication   192.168.56.201/32  trust

local all        postgres                         trust

host  all         all           0.0.0.0/0         trust


```bash
psql -h 202 server --> this should connect.

```

->pg_ctl reload


->listen_addresses = "*"


201->old master having less data. this is the target.

202->stand alone more data. this is the source.


->In old master run below:


pg_rewind  --target-pgdata=/var/lib/pgsql/14/data/ --source-server='host=192.168.56.202 user=replication dbname=replication'

pg_rewind: servers diverged at WAL location 0/1F0000D8 on timeline 1

pg_rewind: rewinding from last common checkpoint at 0/1F000060 on timeline 1

pg_rewind: Done!

-bash-4.2$


->start old master server

```bash
systemctl start postgresql-14

```

->in 202 which was a standby server under its data directory create standby.signal file. Now again it will become standby.

```bash
touch standby.signal
systemctl stop postgresql-14
systemctl start postgresql-14

```

-> Now old master has become master again and 202 become as standby server. This is called failover (master down, promote standby as master) and


failback(making old master as master and new master as standby to the original state).


-> But this is manual failover, failback which in real time not good, so there is autofailover.


## 48. AUTOFAILOVER EXTENSION (PG_AUTOCTL)

requirements 4 nodes.


1.monitor node : 192.168.56.201 : 5000 port


2.master db: 192.168.56.202 : 5001 port


3.stdb1    : 192.168.56.203 : 5002 port


4.standby 2  : 192.168.56.204 : 5003 port


install autofailover s/w across all the 4 nodes.


```bash
wget https://install.citusdata.com/community/rpm.sh . (it will create repo like our pgdg)

[root@mariadb2 ~]# chmod 777 rpm.sh
[root@mariadb2 ~]# ./rpm.sh
```

Detected operating system as centos/7.

Checking for curl...

Detected curl...

Checking for postgresql14-server...

Installing pgdg14 repo... done.

Checking for EPEL repositories...

Detected EPEL repoitories

Downloading repository file: https://repos.citusdata.com/community/config_file.repo?os=centos&dist=7&source=script... done.

Installing pygpgme to verify GPG signatures... done.

Installing yum-utils... done.

Generating yum cache for citusdata_community... done.


The repository is set up! You can now install packages.

You have new mail in /var/spool/mail/root

```bash
[root@mariadb2 ~]#


```

it will create repo as below.


```bash
[root@mariadb1 yum.repos.d]# pwd
```

/etc/yum.repos.d

```bash
[root@mariadb1 yum.repos.d]#

```

-rw-r--r-- 1 root root  555 Mar 20 21:18 citusdata_community.repo


This will install autofailover software 16 version and postgresql database software 14 version as well.


```bash
[root@mariadb2 ~]# yum install pg-auto-failover16_14 full -y

create monitor server:
```

======================

su - postgres


```bash
mkdir /var/lib/pgsql/14/data  (all 4 nodes)

-bash-4.2$ cat .bash_profile
export PATH=$PATH:/usr/pgsql-14/bin
export PGDATA=/var/lib/pgsql/14/data/
export PGPORT=5000

pg_autoctl --version

-bash-4.2$ pg_autoctl create monitor --ssl-self-signed --auth trust
```

21:25:27 24187 INFO  Using default --ssl-mode "require"

21:25:27 24187 INFO  Using --ssl-self-signed: pg_autoctl will create self-signed certificates, allowing for encrypted network traffic

21:25:27 24187 WARN  Self-signed certificates provide protection against eavesdropping; this setup does NOT protect against Man-In-The-Middle attacks nor Impersonation attacks.

21:25:27 24187 WARN  See https://www.postgresql.org/docs/current/libpq-ssl.html for details

21:25:27 24187 INFO  Initialising a PostgreSQL cluster at "/var/lib/pgsql/14/data/monitor"

21:25:27 24187 INFO  /usr/pgsql-14/bin/pg_ctl initdb -s -D /var/lib/pgsql/14/data/monitor --option '--auth=trust'

21:25:39 24187 INFO   /bin/openssl req -new -x509 -days 365 -nodes -text -out /var/lib/pgsql/14/data/monitor/server.crt -keyout /var/lib/pgsql/14/data/monitor/server.key -subj "/CN=mariadb2"

21:25:39 24187 INFO  Started pg_autoctl postgres service with pid 24247

21:25:39 24187 INFO  Started pg_autoctl monitor-init service with pid 24248

21:25:39 24247 INFO   /usr/pgsql-14/bin/pg_autoctl do service postgres --pgdata /var/lib/pgsql/14/data/monitor -v

21:25:39 24253 INFO   /usr/pgsql-14/bin/postgres -D /var/lib/pgsql/14/data/monitor -p 5000 -h *

21:25:40 24247 INFO  Postgres is now serving PGDATA "/var/lib/pgsql/14/data/monitor" on port 5000 with pid 24253

21:25:41 24248 WARN  NOTICE:  installing required extension "btree_gist"

21:25:42 24248 INFO  Granting connection privileges on 192.168.56.0/24

21:25:43 24248 INFO  Reloading Postgres configuration and HBA rules

21:25:43 24248 INFO  Your pg_auto_failover monitor instance is now ready on port 5000.

21:25:43 24248 INFO  Monitor has been successfully initialized.

21:25:43 24187 WARN  pg_autoctl service monitor-init exited with exit status 0

21:25:43 24247 INFO  Postgres controller service received signal SIGTERM, terminating

21:25:43 24247 INFO  Stopping pg_autoctl postgres service

21:25:43 24247 INFO  /usr/pgsql-14/bin/pg_ctl --pgdata /var/lib/pgsql/14/data/monitor --wait stop --mode fast

21:25:43 24187 INFO  Waiting for subprocesses to terminate.

21:25:46 24187 INFO  Stop pg_autoctl

-bash-4.2$


```bash
-bash-4.2$ nohup pg_autoctl run &
```

[1] 24337


```bash
-bash-4.2$ nohup: ignoring input and appending output to ‘nohup.out’

```

-bash-4.2$

```bash
-bash-4.2$ jobs
```

[1]+  Running                 nohup pg_autoctl run &

-bash-4.2$


```bash
pg_autoctl show uri

```

add below enyries in monitor node hba file.


host    all             all             0.0.0.0/0            trust

host    replication     all             0.0.0.0/0            trust

hostssl "pg_auto_failover" "autoctl_node" 0.0.0.0/0 trust # Auto-generated by pg_auto_failover


-> enter all 3 nodes ip in hba conf file instead of 0.0.0.0/0


```bash
-bash-4.2$ /usr/pgsql-14/bin/pg_ctl -D /var/lib/pgsql/14/data/monitor/ reload
```

server signaled

```bash
-bash-4.2$ psql -p5000
psql (14.2)
```

Type "help" for help.


```sql
postgres=# select * from pg_hba_file_rules;
```

line_number |  type   |      database      |   user_name    | address |                 netmask                 | auth_method | options | error

-------------+---------+--------------------+----------------+---------+-----------------------------------------+-------------+---------+-------

89 | local   | {all}              | {all}          |         |                                         | trust       |         |

91 | host    | {all}              | {all}          | 0.0.0.0 | 0.0.0.0                                 | trust       |         |

93 | host    | {all}              | {all}          | ::1     | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | trust       |         |

96 | local   | {replication}      | {all}          |         |                                         | trust       |         |

97 | host    | {replication}      | {all}          | 0.0.0.0 | 0.0.0.0                                 | trust       |         |

98 | host    | {replication}      | {all}          | ::1     | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | trust       |         |

99 | hostssl | {pg_auto_failover} | {autoctl_node} | 0.0.0.0 | 0.0.0.0                                 | trust       |         |

(7 rows)


```bash
-bash-4.2$ pg_autoctl show uri
```

Type |    Name | Connection String

-------------+---------+-------------------------------

monitor | monitor | postgres://autoctl_node@mariadb3:5000/pg_auto_failover?sslmode=require

formation | default |


build master server:

```bash
-bash-4.2$ cat .bash_profile
export PATH=$PATH:/usr/pgsql-14/bin
export PGDATA=/var/lib/pgsql/14/data/
export PGPORT=5001

-bash-4.2$ pg_autoctl create postgres --dbname autodb --auth trust --no-ssl --monitor postgres://autoctl_node@mariadb3:5000/pg_auto_failover

```

nohup pg_autoctl run &


->make 0.0.0.0/0 entries in pg_hba.conf file.[instead of all zeros keep each and every servers]


->reload postgresql services


check formation URI:


```bash
-bash-4.2$ pg_autoctl show uri
```

Type |    Name | Connection String

-------------+---------+-------------------------------

monitor | monitor | postgres://autoctl_node@mariadb3:5000/pg_auto_failover?sslmode=require

formation | default | postgres://mariadb4:5001/autodb?target_session_attrs=read-write&sslmode=require


-bash-4.2$


build standby1:

```bash
vi .bash_profile

export PATH=$PATH:/usr/pgsql-14/bin
export PGDATA=/var/lib/pgsql/14/data/
export PGPORT=5002

-bash-4.2$  pg_autoctl create postgres --dbname autodb --auth trust --no-ssl --monitor postgres://autoctl_node@mariadb3:5000/pg_auto_failover

-bash-4.2$ nohup pg_autoctl run &


-bash-4.2$ pg_autoctl show uri
```

Type |    Name | Connection String

-------------+---------+-------------------------------

monitor | monitor | postgres://autoctl_node@mariadb3:5000/pg_auto_failover?sslmode=require

formation | default | postgres://mariadb4:5001,crash4:5002/autodb?target_session_attrs=read-write&sslmode=require


```bash
-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   1: 0/3000148 |   read-write |             primary |             primary

node_4 |     4 |   crash4:5002 |   1: 0/3000148 |    read-only |           secondary |           secondary


->always to make it consitent across the cluster, add entries in pg_hba.conf file.


build standby server2:

```bash
vi .bash_profile

export PATH=$PATH:/usr/pgsql-14/bin
export PGDATA=/var/lib/pgsql/14/data/
export PGPORT=5003

-bash-4.2$ pg_autoctl create postgres --dbname autodb --auth trust --no-ssl --monitor postgres://autoctl_node@mariadb3:5000/pg_auto_failover

-bash-4.2$ nohup pg_autoctl run &

-bash-4.2$ pg_autoctl show uri
```

Type |    Name | Connection String

-------------+---------+-------------------------------

monitor | monitor | postgres://autoctl_node@mariadb3:5000/pg_auto_failover

formation | default | postgres://mariadb1:5003,crash4:5002,mariadb4:5001/autodb?target_session_attrs=read-write&sslmode=prefer


-bash-4.2$


```sql
cluster state:

-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   1: 0/5000060 |   read-write |             primary |             primary

node_4 |     4 |   crash4:5002 |   1: 0/5000060 |    read-only |           secondary |           secondary

node_5 |     5 | mariadb1:5003 |   1: 0/5000060 |    read-only |           secondary |           secondary


-bash-4.2$


go to application server , test the connectivity using connection URI.


```bash
-bash-4.2$ psql postgres://mariadb4:5001,crash4:5002/autodb?target_session_attrs=read-write
psql (14.2)
```

Type "help" for help.


autodb=#


autodb=# select * from pg_stat_replication;

-[ RECORD 1 ]----+------------------------------

pid              | 2283

usesysid         | 16411

usename          | pgautofailover_replicator

application_name | pgautofailover_standby_4

client_addr      | 192.168.56.154

client_hostname  |

client_port      | 44578

backend_start    | 2022-02-21 20:03:47.514884-05

backend_xmin     |

state            | streaming

sent_lsn         | 0/5006208

write_lsn        | 0/5006208

flush_lsn        | 0/5006208

replay_lsn       | 0/5006208

write_lag        | 00:00:00.001784

flush_lag        | 00:00:00.005483

replay_lag       | 00:00:00.005985

sync_priority    | 1

sync_state       | quorum

reply_time       | 2022-02-21 20:24:46.348304-05

-[ RECORD 2 ]----+------------------------------

pid              | 3400

usesysid         | 16411

usename          | pgautofailover_replicator

application_name | pgautofailover_standby_5

client_addr      | 192.168.56.201

client_hostname  |

client_port      | 52548

backend_start    | 2022-02-21 20:12:40.067375-05

backend_xmin     |

state            | streaming

sent_lsn         | 0/5006208

write_lsn        | 0/5006208

flush_lsn        | 0/5006208

replay_lsn       | 0/5006208

write_lag        | 00:00:00.001684

flush_lag        | 00:00:00.005444

replay_lag       | 00:00:00.007192

sync_priority    | 1

sync_state       | quorum

reply_time       | 2022-02-21 20:24:44.426264-05


get the ip details:


autodb=# select inet_server_addr();

inet_server_addr

------------------

192.168.56.204

(1 row)


->switchover:

```bash
-bash-4.2$ pg_autoctl perform switchover
```

20:28:33 18931 INFO  Waiting 60 secs for a notification with state "primary" in formation "default" and group 0

20:28:33 18931 INFO  Listening monitor notifications about state changes in formation "default" and group 0

20:28:33 18931 INFO  Following table displays times when notifications are received

Time |   Name |  Node |     Host:Port |       Current State |      Assigned State

---------+--------+-------+---------------+---------------------+--------------------

20:28:34 | node_3 |     3 | mariadb4:5001 |             primary |            draining

20:28:34 | node_3 |     3 | mariadb4:5001 |            draining |            draining

20:28:34 | node_3 |     3 | mariadb4:5001 |            draining |          report_lsn

20:28:34 | node_4 |     4 |   crash4:5002 |           secondary |          report_lsn

20:28:34 | node_5 |     5 | mariadb1:5003 |           secondary |          report_lsn

20:28:35 | node_5 |     5 | mariadb1:5003 |          report_lsn |          report_lsn

20:28:35 | node_4 |     4 |   crash4:5002 |          report_lsn |          report_lsn

20:28:36 | node_3 |     3 | mariadb4:5001 |          report_lsn |          report_lsn

20:28:36 | node_4 |     4 |   crash4:5002 |          report_lsn |   prepare_promotion

20:28:36 | node_4 |     4 |   crash4:5002 |   prepare_promotion |   prepare_promotion

20:28:36 | node_4 |     4 |   crash4:5002 |   prepare_promotion |        wait_primary

20:28:37 | node_3 |     3 | mariadb4:5001 |          report_lsn |      join_secondary

20:28:37 | node_5 |     5 | mariadb1:5003 |          report_lsn |      join_secondary

20:28:37 | node_4 |     4 |   crash4:5002 |        wait_primary |        wait_primary

20:28:38 | node_3 |     3 | mariadb4:5001 |      join_secondary |      join_secondary

20:28:38 | node_3 |     3 | mariadb4:5001 |      join_secondary |           secondary

20:28:38 | node_5 |     5 | mariadb1:5003 |      join_secondary |      join_secondary

20:28:38 | node_5 |     5 | mariadb1:5003 |      join_secondary |           secondary

20:28:40 | node_3 |     3 | mariadb4:5001 |           secondary |           secondary

20:28:40 | node_4 |     4 |   crash4:5002 |        wait_primary |             primary

20:28:40 | node_4 |     4 |   crash4:5002 |             primary |             primary

-bash-4.2$

```bash
-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   2: 0/5006618 |    read-only |           secondary |           secondary

node_4 |     4 |   crash4:5002 |   2: 0/5006618 |   read-write |             primary |             primary

node_5 |     5 | mariadb1:5003 |   2: 0/5006618 |    read-only |           secondary |           secondary


but application should have connection retry facitiy, then only connection will be established to new master.


autodb=# select inet_server_addr();

inet_server_addr

------------------

192.168.56.204

(1 row)


autodb=# select inet_server_addr();

FATAL:  terminating connection due to administrator command

server closed the connection unexpectedly

This probably means the server terminated abnormally

before or while processing the request.

The connection to the server was lost. Attempting reset: Succeeded.

```bash
psql (14.2, server 14.1)
autodb=# select inet_server_addr();
```

inet_server_addr

------------------

192.168.56.154

(1 row)


```sql
set the priority for failover and switchover:
```

```bash
-bash-4.2$ pg_autoctl set node candidate-priority --name node_5 90
```

20:39:45 26279 INFO  Waiting for the settings to have been applied to the monitor and primary node

20:39:46 26279 INFO  New state is reported by node 3 "node_3" (mariadb4:5001): "apply_settings"

20:39:46 26279 INFO  Setting goal state of node 3 "node_3" (mariadb4:5001) to primary after it applied replication properties change.

20:39:46 26279 INFO  New state is reported by node 3 "node_3" (mariadb4:5001): "primary"

90

```bash
-bash-4.2$ pg_autoctl show settings
```

Context |    Name |                   Setting | Value

----------+---------+---------------------------+-------------------------------------------------------------

formation | default |      number_sync_standbys | 1

primary |  node_3 | synchronous_standby_names | 'ANY 1 (pgautofailover_standby_5, pgautofailover_standby_4)'

node |  node_3 |        candidate priority | 50

node |  node_4 |        candidate priority | 50

node |  node_5 |        candidate priority | 90

node |  node_3 |        replication quorum | true

node |  node_4 |        replication quorum | true

node |  node_5 |        replication quorum | true


```bash
-bash-4.2$ pg_autoctl perform switchover

-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   4: 0/5006CA8 |    read-only |           secondary |           secondary

node_4 |     4 |   crash4:5002 |   4: 0/5006CA8 |    read-only |           secondary |           secondary

node_5 |     5 | mariadb1:5003 |   4: 0/5006CA8 |   read-write |             primary |             primary


-bash-4.2$


failover:

shut down master node and check the clustert status


```bash
-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   5: 0/5007008 |   read-write |             primary |             primary

node_4 |     4 |   crash4:5002 |   5: 0/5007008 |    read-only |           secondary |           secondary

node_5 |     5 | mariadb1:5003 |   4: 0/5006CE0 | read-write ! |             primary |             demoted


rejoin failed node to cluster:

start services in failed node:


```bash
-bash-4.2$ nohup pg_autoctl run &
```

[1] 14272

```bash
-bash-4.2$ nohup: ignoring input and appending output to ‘nohup.out’


```

automatically failed node will be added as secondary.


```bash
-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   5: 0/5007008 |   read-write |             primary |             primary

node_4 |     4 |   crash4:5002 |   5: 0/5007008 |    read-only |           secondary |           secondary

node_5 |     5 | mariadb1:5003 |   5: 0/5007008 |    read-only |           secondary |           secondary


if both standby's down at a time, all the writes will be hung state in master, in that case we need to change configuration.


```bash
-bash-4.2$ pg_autoctl show settings
```

Context |    Name |                   Setting | Value

----------+---------+---------------------------+-------------------------------------------------------------

formation | default |      number_sync_standbys | 1

primary |  node_3 | synchronous_standby_names | 'ANY 1 (pgautofailover_standby_5, pgautofailover_standby_4)'


->make number_sync_standbys to 0.


```bash
-bash-4.2$ pg_autoctl set formation number-sync-standbys 0
```

20:47:46 31602 INFO  Waiting for the settings to have been applied to the monitor and primary node

20:47:46 31602 INFO  New state is reported by node 3 "node_3" (mariadb4:5001): "apply_settings"

20:47:46 31602 INFO  Setting goal state of node 3 "node_3" (mariadb4:5001) to primary after it applied replication properties change.

20:47:46 31602 INFO  New state is reported by node 3 "node_3" (mariadb4:5001): "primary"

20:47:46 31602 INFO  primary node has now set synchronous_standby_names = 'ANY 1 (pgautofailover_standby_5, pgautofailover_standby_4)'

0

```bash
-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   5: 0/50070B8 |   read-write |             primary |             primary

node_4 |     4 |   crash4:5002 |   5: 0/50070B8 |    read-only |           secondary |           secondary

node_5 |     5 | mariadb1:5003 |   5: 0/50070B8 |    read-only |           secondary |           secondary


```bash
-bash-4.2$ pg_autoctl show settings
```

Context |    Name |                   Setting | Value

----------+---------+---------------------------+-------------------------------------------------------------

formation | default |      number_sync_standbys | 0


```sql
set maintance mode for the servers:
```

```bash
-bash-4.2$  pg_autoctl enable maintenance
```

20:50:43 15967 INFO  Listening monitor notifications about state changes in formation "default" and group 0

20:50:43 15967 INFO  Following table displays times when notifications are received

Time |   Name |  Node |     Host:Port |       Current State |      Assigned State

---------+--------+-------+---------------+---------------------+--------------------

20:50:44 | node_5 |     5 | mariadb1:5003 |           secondary |         maintenance

20:50:44 | node_5 |     5 | mariadb1:5003 |         maintenance |         maintenance


```bash
-bash-4.2$ pg_autoctl show state
```

Name |  Node |     Host:Port |       TLI: LSN |   Connection |      Reported State |      Assigned State

-------+-------+---------------+----------------+--------------+---------------------+--------------------

node_3 |     3 | mariadb4:5001 |   5: 0/50071D8 |   read-write |             primary |             primary

node_4 |     4 |   crash4:5002 |   5: 0/50071D8 |    read-only |           secondary |           secondary

node_5 |     5 | mariadb1:5003 |   5: 0/50071D8 |         none |         maintenance |         maintenance


-bash-4.2$


database upgrade :

stop 2 standbys and cleanup, install new s/w.


->take downtime and upgrade master.


->rebuild standby's using create database command.


[in postgresql replication will not work 2 different version]


## 49. REP MANAGER

Rep Manager to setup auto failover:-


Replication-repmanager folder having docs for all rep manager setup.

1. Repmgr primary-standby setup

2. Repmgr Failover node rejoin   -- failover and rejoin primary node as standby again.

3. Revert_new_standby to primary -- again need to perform failover by shut down standby server.

4. Standby_follow -- Add one more standby in the configuration which will follow primary always in case of failover(it should follow 1st standby).

5. Cloning and cascading replication

6. Switchover

7. Witness server: In case standby is unable to reach out to primary server because of network issue, it thinks primary is not available and it can get promote itself for that witness server needs to configured so that it can monitor primary and keep inform standby about primary.

8. Uninstall: Clearing and unregister instance from repmanager cluster.


## 50. LOGICAL REPLICATION

Use Cases - When is Logical Replication Useful?


If you want to consolidate multiple databases tables into a single database for analytical purposes.

If your requirement is to replicate data between different major versions of PostgreSQL.

If you want to send incremental changes in a single database or a subset of a database to other databases.

If giving access to replicated data to different groups of users.

If sharing a subset of the database between multiple databases.


there is no concept of master and slave.


server a: table a

server b: table a (as part of DR)


Limitations Of Logical Replication

---------------------------------

this will be usefull if you want to replicate only set of tables data to different server/different database.

in that case only logical replication will work, what ever your requirement is if you want to perform replication b/w two different version

of servers this is the only option.


Logical Replication has some limitations:


Tables must have the same fully qualified name between publication(master) and subscription(standby).

Tables must have primary key or unique key

Mutual (bi-directional) Replication is not supported

Does not replicate schema/DDL changes.(DRCA--drop rename create alter)

only DML operations will be reflect (UPDATE,INSERT,DELETE,TRUNCATE)

Does not replicate sequences

Does not replicate Large Objects(lob.blob,clob)


only table data only (either insert/update/delete operations will be replicate, remaining nothing will work)


Subscriptions can have more columns or different order of columns, but the types and column names must match between Publication and Subscription.


Superuser privileges to add all tables.


You cannot stream over to the same host (subscription will get locked).


server should be different or instance should be different.


Implementation:

---------------

you need two postgresql servers either same version or not,same database or not. it will works.


------------------------------------------------------------------

publication server(master):192.168.56.201

-------------------------------------------------------------------

in postgresql.conf file you need to keep below parameter:


listen_address='*'

wal_level= logical (for streaming replication : replica)

wal_keep_size=1024


->make changes in pg_hba.conf file


after making changes to reflect we need to restart services.


```bash
[postgres@localhost ram]$ psql -p5435
psql (11.0)
```

Type "help" for help.


```sql
postgres=# \l
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges

-----------+----------+----------+-------------+-------------+-----------------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

ram       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

(4 rows)


```sql
postgres=# create database source_rep;
CREATE DATABASE
postgres=# \c source_rep
```

You are now connected to database "source_rep" as user "postgres".

source_rep=# create table test_rep(id int primary key, name varchar);

```sql
CREATE TABLE
source_rep=# create table test_rep_other(id int primary key, name varchar);
CREATE TABLE
```

source_rep=# \dt

List of relations

Schema |      Name      | Type  |  Owner

--------+----------------+-------+----------

public | test_rep       | table | postgres

public | test_rep_other | table | postgres

(2 rows)


source_rep=# insert into test_rep values(generate_series(1,100),'data'||generate_series(1,100));

```sql
INSERT 0 100
source_rep=# insert into test_rep_other  values(generate_series(1,100),'data'||generate_series(1,100));
INSERT 0 100
source_rep=# select count(1) from test_rep;
```

count

-------

100

(1 row)


source_rep=# select count(1) from test_rep_other;

count

-------

100

(1 row)


source_rep=# CREATE PUBLICATION mypub FOR TABLE test_rep,test_rep_other;

```sql
CREATE PUBLICATION
```

source_rep=# \q


source_rep=# select * from pg_publication;

oid  | pubname | pubowner | puballtables | pubinsert | pubupdate | pubdelete | pubtruncate

-------+---------+----------+--------------+-----------+-----------+-----------+-------------

24597 | mypub   |       10 | f            | t         | t         | t         | t


source_rep=# select * from pg_publication_rel;

oid  | prpubid | prrelid

-------+---------+---------

24598 |   24597 |   24581

24599 |   24597 |   24589

(2 rows)


tables part of publication:


source_rep=# select relname from pg_class where oid in(select prrelid from pg_publication_rel);

relname

----------------

test_rep

test_rep_other

(2 rows)


---------------------------------------------

subscription(slave server): 192.168.56.203

----------------------------------------------

change below parameter in slave postgresql.conf file and restart services.


wal_level= logical


```bash
[root@mariadb2 data]# systemctl restart postgresql-14

[postgres@localhost rams]$ psql -p5433
psql (11.0)
```

Type "help" for help.


```sql
postgres=# \l
```

List of databases

Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges

-----------+----------+----------+-------------+-------------+-----------------------

postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |

template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +

|          |          |             |             | postgres=CTc/postgres

(3 rows)


```sql
postgres=# create database target_rep;
CREATE DATABASE
postgres=# \c target_rep
```

You are now connected to database "target_rep" as user "postgres".

target_rep=# create table test_rep_other(id int primary key, name varchar);

```sql
CREATE TABLE
target_rep=# create table test_rep(id int primary key, name varchar);
CREATE TABLE

target_rep=# CREATE SUBSCRIPTION mysub CONNECTION 'dbname=source_rep host=192.168.56.201 user=postgres port=5432' PUBLICATION mypub;
```

NOTICE:  created replication slot "mysub" on publisher

```sql
CREATE SUBSCRIPTION
```

target_rep=#


your log file should contain below steps.


4041 2020-03-27 19:19:16 PDT LOG:  logical replication apply worker for subscription "mysub" has started

4043 2020-03-27 19:19:17 PDT LOG:  logical replication table synchronization worker for subscription "mysub", table "test_rep_other" has started

4042 2020-03-27 19:19:17 PDT LOG:  logical replication table synchronization worker for subscription "mysub", table "test_rep" has started

4043 2020-03-27 19:19:17 PDT LOG:  logical replication table synchronization worker for subscription "mysub", table "test_rep_other" has finished

4042 2020-03-27 19:19:17 PDT LOG:  logical replication table synchronization worker for subscription "mysub", table "test_rep" has finished


replication done, for validation check the record count in tables.

```sql
select * from pg_stat_replication; if replication not working there will be no entries, if no entry means again you have to create scubscrtion.

```

to monitor replication:


pg_publication_tables:


source_rep=# select * from pg_publication_tables;

pubname | schemaname |   tablename

---------+------------+----------------

mypub   | public     | test_rep

mypub   | public     | test_rep_other


pg_stat_subscription:


target_rep=# select * from pg_stat_subscription;

-[ RECORD 1 ]---------+------------------------------

subid                 | 16399

subname               | mysub

pid                   | 2499

relid                 |

received_lsn          | 0/6AE8FF0

last_msg_send_time    | 2021-12-06 20:31:24.853348-05

last_msg_receipt_time | 2021-12-06 20:31:24.84882-05

latest_end_lsn        | 0/6AE8FF0

latest_end_time       | 2021-12-06 20:31:24.853348-05


target_rep=#


you can give all the tables under one schema, that means yor are not doing schema replication.


pg_publication:

--------------

```sql
postgres=# \c source_rep;
```

You are now connected to database "source_rep" as user "postgres".

source_rep=# select * from pg_publication;

pubname | pubowner | puballtables | pubinsert | pubupdate | pubdelete | pubtruncate

---------+----------+--------------+-----------+-----------+-----------+-------------

mypub   |       10 | f            | t         | t         | t         | t

(1 row)


source_rep=#


if you want to replciation all tables of database , below is the approach.


source_rep=# CREATE PUBLICATION mytestpub FOR ALL TABLES;

```sql
CREATE PUBLICATION

```

take dump from publication like below.


```bash
pg_dump -d source_Rep -p5434 -s>publicationtables_sourcedb.sql


```

goto subsctiption server:

------------------------

```bash
psql -p5432 -d target_Rep -f publicationtables_sourcedb.sql


create subscription as mentioned above.

if any errors means you have to create scubscription again.


```

master server:

-------------

source_rep=# select * from pg_stat_replication;

-[ RECORD 1 ]----+------------------------------

pid              | 3162

usesysid         | 10

usename          | postgres

application_name | mysub

client_addr      | ::1

client_hostname  |

client_port      | 52930

backend_start    | 2018-12-28 23:38:49.357683-08

backend_xmin     |

state            | streaming

sent_lsn         | 0/92867A8

write_lsn        | 0/92867A8

flush_lsn        | 0/92867A8

replay_lsn       | 0/92867A8

write_lag        |

flush_lag        |

replay_lag       |

sync_priority    | 0

sync_state       | async


source_rep=#


source_rep=# select * from pg_publication_tables;

pubname | schemaname |   tablename

---------+------------+----------------

mypub   | public     | test_rep

mypub   | public     | test_rep_other

(2 rows)


source_rep=#


target_rep=# \x

Expanded display is on.

target_rep=# select * from pg_stat_subscription;

-[ RECORD 1 ]---------+------------------------------

subid                 | 16401

subname               | mysub

pid                   | 3161

relid                 |

received_lsn          | 0/92867A8

last_msg_send_time    | 2018-12-28 23:46:01.286717-08

last_msg_receipt_time | 2018-12-28 23:46:01.289782-08

latest_end_lsn        | 0/92867A8

latest_end_time       | 2018-12-28 23:46:01.286717-08


target_rep=#


if some one perfromed any DDL changes on top of publication node, replication will break, then there will be no output for

```sql
select * from pg_stat_replication;

source_rep=# alter table test_rep add column sal int;
ALTER TABLE
source_rep=# select * from pg_stat_replication;
```

pid  | usesysid | usename  | application_name |  client_addr   | client_hostname | client_port |         backend_start         | backend_xmin

|   state   | sent_lsn  | write_lsn | flush_lsn | replay_lsn | write_lag | flush_lag | replay_lag | sync_priority | sync_state |          repl

y_time

-------+----------+----------+------------------+----------------+-----------------+-------------+-------------------------------+-------------

-+-----------+-----------+-----------+-----------+------------+-----------+-----------+------------+---------------+------------+--------------

-----------------

15808 |       10 | postgres | mysub            | 192.168.56.202 |                 |       48730 | 2022-02-16 20:59:59.019631-05 |

| streaming | 0/5788E80 | 0/5788E80 | 0/5788E80 | 0/5788E80  |           |           |            |             0 | async      | 2022-02-16 21

:05:31.201127-05

(1 row)


source_rep=# insert into test_rep values(101,'ram',10000);

```sql
INSERT 0 1
source_rep=# select * from pg_stat_replication;
```

pid | usesysid | usename | application_name | client_addr | client_hostname | client_port | backend_start | backend_xmin | state | sent_lsn |

write_lsn | flush_lsn | replay_lsn | write_lag | flush_lag | replay_lag | sync_priority | sync_state | reply_time

-----+----------+---------+------------------+-------------+-----------------+-------------+---------------+--------------+-------+----------+-

----------+-----------+------------+-----------+-----------+------------+---------------+------------+------------

(0 rows)


2022-02-16 21:05:53.067 EST [21711] ERROR:  logical replication target relation "public.test_rep" is missing replicated column: "sal"

2022-02-16 21:05:53.070 EST [20981] LOG:  background worker "logical replication worker" (PID 21711) exited with exit code 1


```sql
postgres=# select * from pg_stat_subscription; (subscription)


```

again we have to fix the issue and refresh subscription.


target_rep=# ALTER SUBSCRIPTION mysub REFRESH PUBLICATION;

```sql
ALTER SUBSCRIPTION

if above command not resolves the issue after fix you have drop and recreate the subscription.

```

the only tracking method is pg log for logical replication.


```sql
ALTER SUBSCRIPTION mysub  REFRESH PUBLICATION WITH ( COPY_DATA );

```

to drop/stop logical replication:


target_rep=# drop subscription mysub;

NOTICE:  dropped replication slot "mysub" on publisher

```sql
DROP SUBSCRIPTION


source_rep=# drop publication mypub;
DROP PUBLICATION
source_rep=# drop publication mytestpub;
DROP PUBLICATION

```

\dRs -> to show subscription.

\dRp -> to show publication.


Below are the test cases for which document is present in part 2 replication course:-

1. Add a table to existing publication and limit only to insert,delete means only required operation.

2. Add a table without primary key and add it to publication.

3. Alter a column of existing table added to publication.

4. Setup cascading logical replication.

5. Remove subscription and publication.


'''' Logical replication new features added in 15 version.'''''

1. Replicate required column only.

2. Replicate required rows using row filtering.

3. Alter subscription skip (disable on error) and then skip that transaction on standby using lsn found in log file.

4. All tables inside a schema can be specify to publish.

5. pg_stat_subscription_stats new view is added to track error detected in replication.


'''' Logical replication new features added in 16 version. '''''

1. Logical replication can be setup from a standby instance.

2. Subscribers can use parallel in order to apply transaction.

3. pg_create_subscription role is introduced in version 16 , which can grant to a user and then user will be able to create subscription.


4. Bi-directional replication before 16 version - (without origin): Bidirectional -infinite loop doc is present to setup this.

-> Before 16 version, if you want to setup bi-directional replciation then

1 node -create publication for a table.

2 node -create subscription of node 1.

2 node -create new publication on this node.

1 node -create subscription for 2 node.


-> this above setup will result into a infinite loop , as once u insert data on node 1, it replicate to node 2 and then node 1 again as

postgres does not aware about the record origin from where it came, it thinks it is a new record inserted and has to be replicated.

-> to avoid this , just dropped the subscription.


Bi-directional Replication with origin: Bidirectional with origin doc can be used to setup this.

Origin: None -> Origin of the record should be local, those should not be replicated record from some other machine.

Any  -> Any record whether locallly inserted or replicated, needs to get replicate.


5. Binary copy during initial load:

-> significant improvement 30% during initial load of logical replication.


6. Parallel applying on standby:

-> The number of parallel workers is controlled by the new server variable max_parallel_apply_workers_per_subscription.

-> CREATE SUBSCRIPTION streaming option now supports parallel to enable application of large transactions by parallel workers.

-> PostgreSQL version 16 adds a new feature that improves performance by parallelizing the process of applying changes to the subscriber node by using multiple background workers.


7. Logical replication from standby:

-> Document available for setting up another logical standby from a standby source.


## 51. TABLE INHERITANCE


```sql
Create Table:
create table orders(orderno serial, flightname varchar(100),boarding varchar(100),status varchar(100),source varchar(100));
create table online_booking (price int) inherits(orders);
create table agent_booking (commision int) inherits(orders);

Insert Records:

insert into orders(flightname,boarding,status,source) values('aircanada','xyz','ontime','employees');
insert into online_booking(flightname,boarding,status,source,price) values('nippon','chn','ontime','website',5000);
insert into online_booking(flightname,boarding,status,source,price) values('luftansa','chn','ontime','app',3000);
 insert into agent_booking(flightname,boarding,status,source,commision) values('etihad','aud','ontime','agent001',1000);
insert into agent_booking(flightname,boarding,status,source,commision) values('emirates','dxb','ontime','agent007',1300);

Select Parent Table:

```

nano=# \set AUTOCOMMIT off

nano=# select * from orders;

orderno | flightname | boarding | status |  source

---------+------------+----------+--------+-----------

1 | aircanada  | xyz      | ontime | employees

2 | nippon     | chn      | ontime | website

3 | luftansa   | chn      | ontime | app

5 | etihad     | aud      | ontime | agent001

6 | emirates   | dxb      | ontime | agent007

(5 rows)


```sql
Update Parent:

nano=# update orders set status='Cancelled';
UPDATE 5
nano=# select * from orders;
```

orderno | flightname | boarding |  status   |  source

---------+------------+----------+-----------+-----------

1 | aircanada  | xyz      | Cancelled | employees

2 | nippon     | chn      | Cancelled | website

3 | luftansa   | chn      | Cancelled | app

5 | etihad     | aud      | Cancelled | agent001

6 | emirates   | dxb      | Cancelled | agent007

(5 rows)


nano=# rollback;

```sql
ROLLBACK
nano=# select * from orders;
```

orderno | flightname | boarding | status |  source

---------+------------+----------+--------+-----------

1 | aircanada  | xyz      | ontime | employees

2 | nippon     | chn      | ontime | website

3 | luftansa   | chn      | ontime | app

5 | etihad     | aud      | ontime | agent001

6 | emirates   | dxb      | ontime | agent007

(5 rows)


nano=# update only orders set status='Cancelled';

```sql
UPDATE 1
nano=# select * from orders;
```

orderno | flightname | boarding |  status   |  source

---------+------------+----------+-----------+-----------

1 | aircanada  | xyz      | Cancelled | employees

2 | nippon     | chn      | ontime    | website

3 | luftansa   | chn      | ontime    | app

5 | etihad     | aud      | ontime    | agent001

6 | emirates   | dxb      | ontime    | agent007

(5 rows)


Delete:

nano=# delete from orders;

```sql
DELETE 5
nano=# select * from orders;
```

orderno | flightname | boarding | status | source

---------+------------+----------+--------+--------

(0 rows)


nano=# rollback;

WARNING:  there is no transaction in progress

```sql
ROLLBACK
nano=# select * from orders;
```

orderno | flightname | boarding | status | source

---------+------------+----------+--------+--------

(0 rows)


```sql
Drop Table:
nano=# drop table orders;
```

ERROR:  cannot drop table orders because other objects depend on it

DETAIL:  table online_booking depends on table orders

table agent_booking depends on table orders

HINT:  Use DROP ... CASCADE to drop the dependent objects too.

nano=# drop table orders cascade;

NOTICE:  drop cascades to 2 other objects

DETAIL:  drop cascades to table online_booking

```sql
drop cascades to table agent_booking
DROP TABLE

nano=# delete from orders;
DELETE 5
nano=# select * from orders;
```

orderno | flightname | boarding | status | source

---------+------------+----------+--------+--------

(0 rows)


nano=# rollback;

WARNING:  there is no transaction in progress

```sql
ROLLBACK


nano=# select * from orders;
```

orderno | flightname | boarding | status | source

---------+------------+----------+--------+--------

(0 rows)


nano=# drop table orders;

ERROR:  cannot drop table orders because other objects depend on it

DETAIL:  table online_booking depends on table orders

table agent_booking depends on table orders

HINT:  Use DROP ... CASCADE to drop the dependent objects too.

nano=# drop table orders cascade;

NOTICE:  drop cascades to 2 other objects

DETAIL:  drop cascades to table online_booking

```sql
drop cascades to table agent_booking
DROP TABLE

```

## 52. COPY TABLE


```sql
create table train_dest as table train_bookings;
create table train_dest as table train_bookings with no data;

```

## 53. PG_BACKREST


types:

1. Full backup

2. Differential backup: changed files from last full backup

3. Incremental backup: changed files from last any backup (differential or full)


```bash
yum install -y epel-release;
yum install -y libzstd-devel;
yum install -y pgbackrest.x86_64;

sudo yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm 

sudo yum -y install postgresql12-server postgresql12-contrib pgbackrest

mkdir /etc/pgbackrest
chown -R postgres:postgres /etc/pgbackrest
chown -R postgres:postgres /usr/bin/pgbackrest
chown -R postgres:postgres /var/lib/pgbackrest
chown -R postgres:postgres /var/log/pgbackrest

```

Configuration:


```bash
vi /etc/pgbackrest/pgbackrest.conf
```

[global]

repo1-path=/var/lib/pgbackrest

repo1-retention-full=2


[pg0app]

pg1-path=/app01/data

pg1-port=5432

pg1-user=postgres

:wq

=======================

PostgreSQL Configuration :-


```bash
cd /app01/data

vi postgresql.conf 

```

wal_level = 'replica'

archive_mode = 'on'

archive_command = 'pgbackrest --stanza=pg0app archive-push %p'

max_wal_senders = '10'


:wq


```bash
pg_ctl -D /app01/data restart 

```

==================================


```bash
$ pgbackrest stanza-create --stanza=pg0app --log-level-console=info

Full Backup: (The first backup will always be a full backup even if we dont mention type)

$ pgbackrest backup --stanza=pg0app --log-level-console=info

```

Check the backup:


$pgbackrest info


=======================

```bash
psql -p 5432 

create table tbl1 as select * from pg_class,pg_description limit 1000000;

create table tbl2 as select * from pg_class,pg_description limit 1000000;

```

===============================

Incremental backup:


```bash
$ pgbackrest backup --type=incr --stanza=pg0app --log-level-console=info


```

=======================

```bash
psql -p 5432 

create table tbl3 as select * from pg_class,pg_description limit 1000000;

create table tbl4 as select * from pg_class,pg_description limit 1000000;

```

===============================


```bash
$ pgbackrest backup --type=incr --stanza=pg0app --log-level-console=info


```

Differential backup and PITR:


```bash
$ pgbackrest backup --type=diff --stanza=pg0app --log-level-console=info

```

psql

```sql
postgres=# create table t1 (id int);
CREATE TABLE
postgres=# insert into t1 values(generate_series(1,100));
INSERT 0 100
postgres=# select current_timestamp;
```

current_timestamp

----------------------------------

2019-12-24 13:22:22.329155+05:30

(1 row)


```sql
postgres=# drop table t1;
DROP TABLE
postgres=#\q

-bash-4.2$ pg_ctl -D data/ stop
```

waiting for server to shut down.... done

server stopped


```bash
-bash-4.2$ pgbackrest --stanza=pg0app --delta --type=time "--target=2019-12-24 13:22:22.329155+05:30" --target-action=promote restore

```

** The delta option allows pgBackRest to automatically determine which files in the database cluster directory can be preserved and which ones need to be restored from the backup


```bash
-bash-4.2$ cat data/recovery.conf 
```

# Recovery settings generated by pgBackRest restore on 2019-12-24 13:27:32

restore_command = 'pgbackrest --stanza=pg0app archive-get %f "%p"'

recovery_target_time = '2019-12-24 13:22:22.329155+05:30'

recovery_target_action = 'promote'


```bash
-bash-4.2$ pg_ctl -D data/ start

postgres=# select count (*) from t1;
```

count

-------

100

(1 row)


## 54. POSTGRES 13 FEATURES


1. there is no architectural changes in 13 than 12.


2. B tree de duplication:

->Merging duplicate values together and forming a single list for each value.

->It result in small index size

->Improved queries that used index scanning.

->After pg_upgrade usage , need to run reindex to make use of same index.


3. Incremental sorting

-> which accelerates sorting data when data that is sorted from earlier parts of a query are already sorted.

-> index on c1 and you need to sort dataset by c1, c2. Then incremental sort can help you because it wouldn’t sort the whole dataset,

but sort individual groups whose have the same value of c1 instead.

-> The incremental sort is extremely helpful when you have a LIMIT clause.


4. Parallel vacuum

->Max_parallel_maintenance_workers , Min_parallel_index_scan_size parameter governs parallel vacuum

->The degree of parallelization is either specified by the user or determined based on the number of indexes that the table has.

->Syntax : VACUUM (PARALLEL 2, VERBOSE) <TableName>


5. Trusted Extensions

->Can install extensions without super user privileges if we have create privilege on database.

->DROP DATABASE DBNAME WITH (FORCE)

->Explain tracks wal_usage:

EX:  EXPLAIN (ANALYZE, WAL, COSTS OFF) UPDATE t1 SET id = 1 WHERE id = 1;


6. New system views introduced

-> Pg_stat_progress_basebackup to report the progress of streaming base backups.

-> Pg_stat_progress_analyze to report ANALYZE progress.

-> Pg_shmem_allocations to display shared memory usage.


## 55. POSTGRES 15 FEATURES


1. Architectural changes introduced in 15.


2. Merge command introduced using which multile dml can execute in single statement.(update, insert).


3. Selective publication of tables'' contents within logical replication publications, through the ability to specify column

lists and row filter conditions.(explained more in Logical replication section above)


4. Support for structured server log output using the JSON format.


5. Compression options including support for Zstandard (zstd) compression.

-> Pg_Basebackup has compress option, we  can now specify the compression method and compression level.

-> Syntax :

```bash
       pg_basebackup --compress=none --format=tar -D back.1
       pg_basebackup --compress=zstd:level=9,workers=2 --format=tar -D back.2

```

6. Stats Collector process has been effectively removed from the architecture. Statistics are now stored in dynamic shared memory.

-> In Postgresql-15, stats updates are first accumulated locally in each process as marked as pending. This stats are later flushed

to shared memory on commit.

-> Information on the shared memory area for statistics can be obtained by searching the pg_backend_memory_contexts view.

Syntax:  SELECT name, total_bytes, free_bytes, used_bytes FROM  pg_backend_memory_contexts WHERE name LIKE 'PgStat%' ;


7. New Roles like pg_read_all_data and pg_write_all_data has been provided.


8.       Operation                       PostgreSQL 14               PostgreSQL 15

Start backup                    pg_start_backup             pg_backup_start

Stop backup                     pg_stop_backup              pg_backup_stop


9. Roles can now be granted permission to change specific parameters through the ALTER SYSTEM statement.

Syntax:

```sql
     GRANT ALTER SYSTEM ON PARAMETER log_statement TO demo


```

## 56. POSTGRES 16 FEATURES


1. Logical Replication Improvements

->PostgreSQL 16 allows users to setup logical replication from a standby instance.

->This helps in distributing workloads, such as opting to use a standby server instead of the primary server to logically replicate

changes to downstream systems. More Logical replication features discussed in logical replication section.


2. pg_stat_io system view introduced which gives information about input/output of cluster, how much data read, write in disk.

To reset io: select pg_stat_reset_shared('io');


3. Copy utility

-> Significant improvment by 300%

-> pg_stat_progress_copy , to monitor copy operation details.


4. Parallel support for full and right joins

```sql
set max_parallel_worker_per_gather- to increase parallel worker for a query.

```

5. Auto_explain

-> Makes more readable by logging values passed into parametrized statement.

-> it will track queries in log files.

-> bind values can be seen logged in log files in 16 version.

-> How to set this on session level:

LOAD 'auto_explain';

```sql
    SET auto_explain.log_min_duration=0;   // it will log all queries.
    SET auto_explain.log_analyze=true;

```

6. Incremental sort- Select Distinct

-> Incremental sort means optimizer will take chunk of rows and sort it , instead of sorting everything at once. 16 version onwards

incremental sort is supporting select distinct statement as well.


## 57. POSTGRES 17 FEATURES


## 58. INSTALL POSTGRES CLIENT (PSQL, PGDUMP ON WINDOWS SUBSYSTEM FOR LINUX)


Check version for WSL :

window+r -> wsl -> open


lsb_release -a


# Step 1: Update package list

```bash
sudo apt update

```

# Step 2: Install dependencies

```bash
sudo apt install wget gnupg2 lsb-release -y

```

# Step 3: Add PostgreSQL GPG key

```bash
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

```

# Step 4: Add PostgreSQL official APT repository

```bash
echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | \
  sudo tee /etc/apt/sources.list.d/pgdg.list

```

# Step 5: Update again to fetch new repo data

```bash
sudo apt update

# Step 6: Install client tools only (no database server)
sudo apt install postgresql-client-15 -y

```

Verify installation

```bash
psql --version
pg_dump --version

```

akashgupta@A-GM0ND3BT:~$ which psql

/usr/bin/psql

akashgupta@A-GM0ND3BT:~$ which pg_dump

/usr/bin/pg_dump


