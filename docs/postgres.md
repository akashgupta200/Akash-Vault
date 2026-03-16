---
layout: default
title: PostgreSQL Queries
nav_order: 3
has_toc: true
---

# PostgreSQL — Complete DBA Reference

This document is a comprehensive DBA reference for PostgreSQL, covering architecture, installation, configuration, user management, backup/restore, replication, maintenance, and version-specific features. It serves as a complete handbook for database administrators working with PostgreSQL.

---

## 1. POSTGRES DATABASE INTRODUCTION

### Database Concepts

**Database**: It is a storage space or device which is used to manage or maintain only summarized data.

**Summarized data:**
- Data which will gives proper meaning
- Data will be used for proper purpose
- Data will be having proper format

**RAW data:**
- Data will not give proper meaning
- Not useful for proper purpose
- Not contains proper format

Default RAW data will be there; everyone will get the summarized data from RAW data.

### Centralized Database Alternatives

While deciding centralized database we have a lot of alternatives:

1. Oracle
2. MS SQL
3. MySQL
4. PostgreSQL
5. MongoDB
6. Teradata
7. Greenplum
8. Redis
9. MemSQL
10. Couchbase
11. Aerospike
12. DynamoDB
13. DB2

Almost 30+ technologies in the market.

### Why Choose PostgreSQL

- Oracle or MS SQL or all other database support & licensing cost will be very high
- Not that much user-friendly
- These database licensing costs will depend on number of CPUs in the server
- If data size grows, definitely system performance will be reduced; to increase performance we increase the number of CPUs and RAM
- If number of CPUs increases, performance will be good, but cost also increases

So most people started searching for alternatives. People expect a database with these features:

1. Database software should be either free of cost or cheap price
2. It should support data in relational (table) format
3. It should provide high level security
4. It should support High Availability: if server goes down, immediately another server plays its role
5. It should support DR (Disaster Recovery): if server completely crashes, immediately another server plays its role permanently
6. It should support point-in-time recovery: even without enough backup, we should be capable to restore up to specific point of time
7. If DBA is not able to do things, someone should be there to provide support
8. If client asks for certification for the product, we need certification as well

With these alternatives, one product came into market: **PostgreSQL**.

### History Behind PostgreSQL

In the 1980s, 2 PhD students from Berkeley University built a database named INGRES. After their project, they left.

After some time, a corporate company started funding for further development. After development, they renamed it as **PostgreSQL** in 1989.

As this product was developed by funding, the product should not be sold as commercial. The PostgreSQL team started a community website with software and documentation. Anyone can download and use.

Starting version is 1.0, 1.6, later directly jumped to version 6. Current PostgreSQL version 14.2. From version 6 to 14, almost 160+ major and minor versions released.

As it is open source, no one gives support if required. Some companies started services: OpenSCG, 2nd Quadrant, Fujitsu, Cybertec, DBI, DBOps, MigOps, Percona, PGExperts, Serverlines, EDB.

EDB (EnterpriseDB) gives funding to open source team for new features/product development. EDB should not make any changes to product developed by open source team. EDB gives user-friendly tools for DBA team: EFM, PEM, BART.

- 2017, 2019, 2020: Best growing RDBMS of the year award goes to PostgreSQL
- 2018: Award goes to MySQL due to Oracle

---

## 2. POSTGRES ARCHITECTURE

### General Architecture

From the starting point of query execution until getting the result set—the complete backend flow—we call it as architecture. Based on process and work, architecture is divided into 3 layers:

1. General architecture
2. Memory based architecture
3. Process based architecture

### General Architecture Flow

- Running query we call it as raising request
- End user raises request from client interface; request reaches PostgreSQL server operating system
- In operating system we call worker as process
- On top of OS there is a process called **postmaster process**
- This process stops and starts PostgreSQL instance services
- Postmaster performs host-based authentication for new requests

### Host-Based Authentication

- In PostgreSQL there is a configuration file called `pg_hba.conf`
- It contains all databases, usernames, and client IP addresses
- This file is located under DATA directory
- Postmaster reads `pg_hba.conf` and checks whether username, database name, and client IP are in the file
- If no entries: error. If entries exist: connection authorized
- If authorized, postmaster creates new postgres process for the request
- That process performs: syntaxer, security check, resolver, optimizer, dispatcher

**Brief view:** User → Client interface → PostgreSQL server OS → Postmaster → Host-based auth → Postgres process → Generic steps (syntaxer, security check, resolver, optimizer, dispatcher)

---

## 3. GENERAL ARCHITECTURE

### Memory Based Architecture

PostgreSQL is completely operating system based. If you run any query, that query appears on the operating system. Memory plays a key role in performance.

Based on work type, memory is categorized into:

1. shared_buffers
2. work_mem
3. temp_buffers
4. maintenance_work_mem
5. autovacuum_work_mem
6. effective_cache_size

### shared_buffers

- Piece of RAM for PostgreSQL internal operations
- Recommended: 25% of total RAM (e.g., 16GB system → 4GB)
- Configured in `postgresql.conf`
- Requires restart to change
- Instance level
- Default: 128MB, Minimum: 128KB

### work_mem

- Used for external operations (hashing or sorting)
- Operation specific—each query gets work_mem allocated
- Plan: max_connections × work_mem ≤ 50% of remaining RAM
- Default: 4MB
- Released after query execution

### temp_buffers

- Used for temp result sets, temporary tables
- Default: 8MB

### maintenance_work_mem

- For VACUUM, ANALYZE, VACUUM FULL, REINDEX
- Default: 64MB

### autovacuum_work_mem

- For automated vacuum & analyze
- Default: -1 (uses maintenance_work_mem)

### effective_cache_size

- Dummy value for optimizer
- Assign 85% of total RAM
- Improves index effectiveness
- Default: 128MB

---

## 4. HOSTBASED AUTHENTICATION

See section 2 and section 18 for detailed pg_hba.conf configuration.

---

## 5. MEMORY BASED ARCHITECTURE

See section 3 for memory parameter details.

---

## 6. TUNE MEMORY RELATED PARAMETER

### Tuning Guidelines

1. **Shared buffers**: If increased, transaction per second may decrease—tune carefully
2. **Work mem**: For ORDER BY/sorting—increase work_mem for that session to speed up
3. **Maintenance_work_mem**: For slow index creation—increase temporarily on session level
4. **effective_cache_size**: If less, optimizer may not use index—try increasing
5. **checkpoint_timeout**: Default 5 min. Increasing to 20 min improves performance (less disk flushing) but increases crash recovery risk

---

## 7. PROCESS BASED ARCHITECTURE

### Mandatory Processes

- **postmaster**: Start/stop/restart/reload PostgreSQL services
- **WAL writer**: Writes committed transactions to WAL file
- **bg writer**: Every 200ms flushes dirty buffers to disk
- **checkpointer**: Every 5 minutes flushes gaps
- **stats collector**: Updates statistics
- **autovacuum launcher**: Automatic vacuum and analyze
- **logical replication launcher**: For logical replication

### Optional Processes

- **archiver**: Copies WAL files to archive (when archive_mode enabled)
- **logger**: Audit log collection
- **wal sender**: Sends data to standby
- **wal receiver**: Receives data from master

### WAL Files

- Each transaction recorded in WAL file
- Only committed transactions recorded
- Uncommitted in wal_buffer
- New WAL file every 16MB
- Located in pg_wal (pg_xlog until 9.6)
- Default pg_wal: 1GB (64 files)
- Switch WAL: `SELECT pg_switch_wal();`
- Enable archive_mode for WAL backup

---

## 8. HOSTBASED AUTHENTICATION

(Refer to section 18 for pg_hba.conf details)

---

## 9. LINUX SETUP

### VMware Setup

Download VMware Player and Linux ISO. Create new virtual machine. Use PuTTY for SSH access.

### Linux Basic Commands

Check current user:

```bash
whoami
```

Check hostname and IP:

```bash
hostname
hostname -i
```

List mount points:

```bash
df -h
```

Switch user:

```bash
su - username
```

Current directory:

```bash
pwd
```

Change directory:

```bash
cd /path
```

List files:

```bash
ls -ltr
du -sh *
```

Create directory and file:

```bash
mkdir dirname
vi filename
```

Change ownership:

```bash
chown -R postgres:postgres /path
chmod 700 filename
```

Display users:

```bash
cat /etc/passwd | grep /bin/bash
```

Create user and add to sudoers:

```bash
useradd username
vi /etc/sudoers
```

---

## 10. VMWARE INSTALLATION

See section 9 for VMware and Linux setup steps.

---

## 11. POSTGRESQL INSTALLATION

Two types: Source code installation and Binaries installation.

**Source code**: Manual, can run multiple instances of same version
**Binaries**: Automated, service auto-startup, same version can't install twice

---

## 12. SOURCE CODE INSTALLATION

### Steps

Download from https://www.postgresql.org/ftp/source/

Extract and configure:

```bash
tar -xvf postgresql-14.2.tar.gz
cd postgresql-14.2
./configure
make
make install
cd contrib
make
make install
```

Create data directory and initialize:

```bash
mkdir /var/lib/pgsql_feb22/DATA
chown -R postgres:postgres /var/lib/pgsql_feb22/DATA/
su - postgres
/usr/local/pgsql/bin/initdb -D /var/lib/pgsql_feb22/DATA/
```

Start/stop services:

```bash
/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ start
/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ stop
/usr/local/pgsql/bin/pg_ctl -D /var/lib/pgsql_feb22/DATA/ reload
```

### Uninstallation

```bash
make clean
make uninstall
rm -rf /usr/local/pgsql/
rm -rf /var/lib/pgsql_feb22/DATA/
```

---

## 13. BINARIES INSTALLATION

### Install Repository and Packages

```bash
wget https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
rpm -ivh pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql14-server postgresql14-contrib
```

### Initialize and Start

```bash
/usr/pgsql-14/bin/postgresql-14-setup initdb
systemctl start postgresql-14
systemctl status postgresql-14
```

### Uninstallation

```bash
systemctl stop postgresql-14
yum remove -y postgresql14-server postgresql14-contrib
rpm -e postgresql14 postgresql14-libs
rm -rf /var/lib/pgsql/14/data/
```

---

## 14. UNINSTALLATION

See sections 12 and 13 for source and binary uninstallation steps.

---

## 15. DATA DIRECTORY EXPLANATION

### Key Directories and Files

- **pg_twophase**: Two-phase commit data
- **pg_snapshots**: Snapshots (like AWR in Oracle)
- **pg_serial**: Serial transactions, deadlock handling
- **pg_notify**: Notifications, errors, warnings
- **pg_dynshmem**: Runtime shared_buffers utilization
- **pg_commit_ts**: Committed transactions
- **PG_VERSION**: Current database version
- **pg_tblspc**: User tablespace symbolic links
- **pg_stat**: Object statistics
- **pg_replslot**: Replication slot data
- **postgresql.conf**: Main configuration file
- **postgresql.auto.conf**: ALTER SYSTEM parameters
- **pg_hba.conf**: Host-based authentication
- **pg_ident.conf**: OS/DB user mapping (LDAP)
- **pg_wal**: WAL files
- **global**: Global objects (users, tablespaces)
- **base**: Actual data files
- **pg_logical**: Logical replication data

---

## 16. USER MANAGEMENT

### Object Flow

Instance → Databases → Schemas → Objects (tables, functions, views...)

### Connect to Database

```bash
psql
psql -U postgres -d rajdb
```

### List Databases and Users

List databases:

```sql
\l
```

List users/roles:

```sql
\du
```

User attributes: SUPERUSER, CREATEDB, CREATEROLE, INHERIT, LOGIN, REPLICATION, BYPASSRLS, CONNECTION LIMIT, PASSWORD, VALID UNTIL.

### System Roles

- **pg_database_owner**: Act as database owner
- **pg_read_all_data**: SELECT from all tables
- **pg_write_all_data**: INSERT, UPDATE, DELETE on all tables (with schema USAGE)
- **pg_monitor**: Monitor whole instance
- **pg_read_all_settings**: View all parameters
- **pg_read_all_stats**: View pg_stat* views
- **pg_stat_scan_tables**: Execute monitoring functions
- **pg_read_server_files**, **pg_write_server_files**: COPY from/to file
- **pg_execute_server_program**: Run COPY command
- **pg_signal_backend**: Cancel/terminate sessions

```sql
GRANT pg_write_all_data TO eswar;
```

### Grant Role to Multiple Users

```sql
GRANT rambabu TO shiva;
GRANT rambabu TO eswar;
GRANT rambabu TO tarun;
```

Then in pg_hba.conf use `+rambabu` to apply to all members of rambabu role.

### Create User and Set Attributes

```sql
CREATE USER rambabu;
ALTER USER rambabu WITH PASSWORD 'rambabu';
ALTER USER rambabu SUPERUSER;
ALTER USER rambabu NOSUPERUSER;
ALTER USER rambabu CREATEDB;
ALTER USER rambabu VALID UNTIL '06-08-2022';
ALTER USER rambabu CONNECTION LIMIT 10;
```

### Schema and Object Management

```sql
CREATE SCHEMA ram_schema;
CREATE TABLE ram_schema.emp(eno int, ename varchar);
ALTER SCHEMA ram_schema OWNER TO rambabu;
ALTER SCHEMA ram_schema RENAME TO raj_schema;
ALTER DATABASE rajdb OWNER TO rambabu;
```

---

## 17. AUDITLOG MANAGEMENT

### Enable Audit Log in postgresql.conf

```ini
log_destination = 'stderr'
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 10MB
log_min_duration_statement = 0
log_connections = on
log_disconnections = on
log_line_prefix = '%a %u %d %h %p'
log_statement = 'all'
```

### Housekeeping

- Remove files older than 30 days
- Compress files older than 3 days
- Use `zcat` to read compressed logs

---

## 18. HOSTBASED AUTHENTICATION

### pg_hba.conf Format

```
# TYPE  DATABASE  USER  ADDRESS  METHOD
```

**TYPE**: local (Unix socket) or host (remote)
**METHOD**: trust, scram-sha-256, reject, peer

### Examples

Allow rambabu on rajdb without password (local):

```
local   rajdb   rambabu   trust
```

Require password:

```
local   rajdb   rambabu   scram-sha-256
```

Reject connections:

```
local   rajdb   rambabu   reject
```

CIDR for IP ranges:

```
host    all   all   0.0.0.0/0   scram-sha-256
```

Reload after changes:

```bash
pg_ctl -D /var/lib/pgsql/14/data/ reload
```

---

## 19. USER ACCESS MANAGEMENT

### Three Levels

1. **DB level**: pg_hba.conf
2. **Schema level**: USAGE (select/insert/update/delete), ALL (create/drop)
3. **Object level**: SELECT, INSERT, UPDATE, DELETE on objects

### Grant Permissions

```sql
GRANT USAGE ON SCHEMA rams TO eswar;
GRANT SELECT, INSERT, DELETE, UPDATE ON rams.emp TO eswar;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA rams TO eswar;
GRANT ALL ON SCHEMA rams TO eswar;
ALTER DEFAULT PRIVILEGES IN SCHEMA rams GRANT SELECT, INSERT, DELETE, UPDATE ON TABLES TO eswar;
GRANT CREATE ON DATABASE postgres TO eswar;
GRANT eswar TO shiva;
```

---

## 20. METADATA TABLES/VIEWS

Metadata means data about data. In PostgreSQL each object is represented by object ID (OID). Two hidden schemas: pg_catalog and information_schema.

### pg_catalog Tables

**pg_auth_members** — Users and assigned roles:

```sql
SELECT * FROM pg_auth_members;
```

**pg_authid** — Users and system roles with OID:

```sql
SELECT oid, rolname, rolsuper, rolcreatedb FROM pg_authid;
```

**pg_class** — Objects, types, pages:

```sql
SELECT oid, relname, relnamespace, reltype, relowner, relpages, relkind FROM pg_class LIMIT 10;
```

**pg_database** — Databases:

```sql
SELECT oid, datname, datallowconn, datconnlimit, dattablespace FROM pg_database;
```

**pg_namespace** — Schemas:

```sql
SELECT * FROM pg_namespace;
```

**pg_tablespace** — Tablespaces:

```sql
SELECT * FROM pg_tablespace;
```

**pg_index** — Indexes:

```sql
SELECT indexrelid, indrelid, indisvalid, indisready, indislive FROM pg_index LIMIT 10;
```

**pg_extension** — Extensions:

```sql
SELECT * FROM pg_extension;
```

### pg_catalog Views

**pg_available_extensions**:

```sql
SELECT * FROM pg_available_extensions LIMIT 10;
```

**pg_file_settings** — Uncommented postgresql.conf parameters:

```sql
SELECT * FROM pg_file_settings;
```

**pg_hba_file_rules** — pg_hba.conf entries:

```sql
SELECT * FROM pg_hba_file_rules;
```

**pg_stat_activity** — Sessions (active/idle):

```sql
SELECT datname, pid, usename, client_addr, xact_start, state FROM pg_stat_activity;
```

**pg_stat_replication** — Replication status:

```sql
SELECT * FROM pg_stat_replication;
```

**pg_stat_user_tables** — User table statistics:

```sql
SELECT relid, schemaname, relname, n_tup_ins, n_tup_del, last_vacuum, last_analyze FROM pg_stat_user_tables;
```

**pg_settings** — All parameters:

```sql
SELECT name, setting FROM pg_settings LIMIT 10;
```

**pg_stat_archiver** — Archive process stats (when archive_mode on):

```sql
SELECT * FROM pg_stat_archiver;
```

**pg_stat_database** — Database statistics:

```sql
SELECT datname, numbackends, temp_files, temp_bytes, deadlocks FROM pg_stat_database;
```

### information_schema Views

**table_privileges** — Table permissions:

```sql
SELECT * FROM information_schema.table_privileges WHERE grantee='eswar' AND table_schema='public';
```

**check_constraints**:

```sql
SELECT * FROM information_schema.check_constraints WHERE constraint_schema='public';
```

**referential_constraints** — FK to PK mapping:

```sql
SELECT * FROM information_schema.referential_constraints WHERE constraint_schema='public';
```

**table_constraints** — All constraints:

```sql
SELECT constraint_schema, constraint_name, table_name, constraint_type FROM information_schema.table_constraints WHERE constraint_schema='public';
```

---

## 21. ENABLE ARCHIVE LOG

### Configuration

```bash
mkdir /var/lib/pgsql/archive
```

Edit postgresql.conf:

```ini
archive_mode = on
archive_command = 'cp %p /var/lib/pgsql/archive/%f'
```

Restart required. Validate:

```sql
SHOW archive_mode;
SELECT pg_switch_wal();
SELECT * FROM pg_stat_archiver;
```

### Archive Cleanup

```bash
pg_archivecleanup /var/lib/pgsql/archive/ 000000010000000000000014
```

---

## 22. TABLESPACE

### Create Tablespace

```bash
mkdir /var/lib/pgsql/ram_tbs
```

```sql
CREATE TABLESPACE ramts LOCATION '/var/lib/pgsql/ram_tbs/';
CREATE DATABASE ram TABLESPACE ramts;
ALTER DATABASE rajdb SET TABLESPACE ramts;
ALTER TABLE dept SET TABLESPACE ramts;
```

### Limitations

- One tablespace = one location
- One database/table = one tablespace
- Exclusive lock during movement

### Physical Storage

Each object has OID. Buffer = memory, Page = disk. Each table has its own data file. Segment size 1GB; extensions .1, .2 for every 1GB. Path: DATA → base → database folder (db OID) → table data file.

Check table physical location:

```sql
SELECT pg_relation_filepath('t1');
```

Data files: main file, _fsm (free space map), _vm (visibility map). Do not touch data files—corruption means no recovery without backup.

---

## 23. MAINTAINANCE ACTIVITIES

### 1. ANALYZE

- Updates table statistics
- No lock, no extra space
- Run after INSERT/UPDATE/DELETE

```sql
ANALYZE emp;
```

### 2. VACUUM

- Removes dead tuples
- Marks space for reuse
- No lock, no extra space

```sql
VACUUM emp;
```

### 3. VACUUM FULL

- Removes bloat, reclaims space
- Exclusive lock, needs extra space

```sql
VACUUM FULL emp;
```

### 4. REINDEX

- Drops and recreates index
- For index bloat

```sql
REINDEX TABLE emp;
REINDEX SCHEMA public;
REINDEX SYSTEM postgres;
```

### vacuumdb Utility

```bash
vacuumdb -a
vacuumdb -d rajdb
vacuumdb -d rajdb -f
vacuumdb -d rajdb -z -j 2
vacuumdb -d postgres -t emp -t t1
```

### Maintenance Practical Example

Disable autovacuum, create table, insert, analyze, delete, vacuum:

```sql
CREATE TABLE emp(eno int, ename varchar, sal int);
INSERT INTO emp VALUES(generate_series(1,100000),'emp'||generate_series(1,100000),generate_series(1,100000));
ANALYZE emp;
DELETE FROM emp WHERE eno<=50000;
VACUUM emp;
```

Check dead tuples in pg_stat_user_tables. VACUUM clears dead tuples; VACUUM FULL reclaims space and changes data file. REINDEX changes index file only.

### View vs Materialized View

View: No storage, always current. Materialized view: Physical storage, refresh needed:

```sql
CREATE VIEW emp_v AS SELECT * FROM emp;
CREATE MATERIALIZED VIEW emp_mv AS SELECT * FROM emp;
REFRESH MATERIALIZED VIEW emp_mv;
```

### Constraints

```sql
CREATE TABLE t1(no int PRIMARY KEY);
CREATE TABLE t2(no int REFERENCES t1(no));
CREATE TABLE t3(no int CHECK(no<10));
CREATE TABLE t4(no int UNIQUE);
CREATE TABLE t5(no int NOT NULL);
```

---

## 24. AUTOVACUUM SETTING

### Key Parameters in postgresql.conf

```ini
autovacuum = on
autovacuum_max_workers = 3
autovacuum_naptime = 1min
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_scale_factor = 0.2
autovacuum_analyze_threshold = 50
autovacuum_analyze_scale_factor = 0.1
```

---

## 25. CRON JOB SETUP

### Crontab Format

```
MIN  HOUR  DOM  MON  DOW  CMD
```

Example—daily at 8:30 AM:

```bash
30 08 * * * /tmp/test.sh
```

Every day at 11am and 4pm:

```bash
00 11,16 * * * /tmp/test.sh
```

Run in background:

```bash
nohup sh test.sh &
```

---

## 26. DATA IMPORT AND EXPORT

### COPY Command

Export to file:

```sql
COPY emp TO '/tmp/emp.txt';
COPY emp TO '/tmp/emp.txt' WITH DELIMITER '|';
COPY emp TO '/tmp/emp.txt' WITH HEADER CSV;
```

Import from file:

```sql
COPY emp FROM '/tmp/emp.txt';
COPY dept FROM '/tmp/emp.txt' WITH HEADER CSV;
```

Export query result:

```sql
COPY (SELECT * FROM emp WHERE no<2) TO '/tmp/emp.txt';
```

---

## 27. BACKUP AND RESTORE

### Logical Backup

- **pg_dump**: Single database, table, schema
- **pg_dumpall**: Whole server, global objects
- Plain format: restore with psql
- Custom format: restore with pg_restore (10x faster)

### Physical Backup

- **pg_basebackup**: Exact data directory copy
- Suitable for PITR

---

## 28. LOGICAL BACKUP

### pg_dump Examples

Single table:

```bash
pg_dump -d rajdb -t raj_schema.dept > dep.sql
```

Data only / structure only:

```bash
pg_dump -d rajdb -t raj_schema.dept -a > dept.sql
pg_dump -d rajdb -t raj_schema.dept -s > dept_struct.sql
```

Schema backup:

```bash
pg_dump -d rajdb -n raj_schema > schemabackup.sql
```

Custom format (single table restore possible):

```bash
pg_dump -Fc -d rajdb -t raj_schema.dept -f dept.bin
pg_restore -Fc -d rajdb -n raj_schema -t dept -c dept.bin
```

Directory format with parallel:

```bash
pg_dump -Fd -d rajdb -f rajd -j 2
```

### pg_dumpall

```bash
pg_dumpall > fullbkp.sql
pg_dumpall -g > globalobjects.sql
```

Restore dumpall (expect "role already exists" for postgres):

```bash
psql -f fullbkp.sql -o fullbkp.log
```

### Drop User with Dependencies

```sql
REASSIGN OWNED BY eswar TO postgres;
DROP OWNED BY eswar;
DROP USER eswar;
```

---

## 29. PHYSICAL BACKUP/BASEBACKUP

### Take Base Backup

```bash
pg_basebackup -D /var/lib/pgsql/backup/ -X fetch -P
```

With tablespaces:

```bash
pg_basebackup -D /var/lib/pgsql_15/backup -X fetch -P -T /var/lib/pgsql_15/tablespace/akash_ts1=/var/lib/pgsql_15/backup/akash_ts1
```

---

## 30. RESTORE BASEBACKUP

### Restore to New Instance

Edit postgresql.conf (change port):

```ini
port=5433
```

```bash
chmod 0700 /var/lib/pgsql/backup
pg_ctl -D /var/lib/pgsql/backup start
```

### Restore to Existing Instance

```bash
pg_ctl -D /var/lib/pgsql/14/data stop
mv /var/lib/pgsql/14/data /var/lib/pgsql/14/data_old
mv /var/lib/pgsql/backup /var/lib/pgsql/14/data
pg_ctl -D /var/lib/pgsql/14/data start
```

---

## 31. PITR

### Prerequisites

- Archive mode enabled
- Full backup taken
- WAL files in archive

### Recovery Steps

1. Copy missed WAL from pg_wal to archive
2. Edit postgresql.auto.conf in backup:

```ini
restore_command='cp /var/lib/pgsql/archive/%f %p'
recovery_target_time='2022-03-16 21:36:55.224268-04'
```

3. Create recovery.signal:

```bash
touch recovery.signal
```

4. Start instance:

```bash
pg_ctl -D /var/lib/pgsql/backup/ start
```

5. Promote when ready:

```sql
SELECT pg_wal_replay_resume();
```

6. Restore table to original instance:

```bash
pg_dump -d postgres -t pitr_test -p 5433 | psql -d postgres -p 5432
```

### Replication Slot for Tablespaces

When user tablespaces exist, include -T for pg_basebackup:

```bash
pg_basebackup -D /var/lib/pgsql_15/backup -X fetch -P -T /var/lib/pgsql_15/tablespace/akash_ts1=/var/lib/pgsql_15/backup/akash_ts1 -T /var/lib/pgsql_15/tablespace/akash_ts2=/var/lib/pgsql_15/backup/akash_ts2
```

Symbolic link update: `ln -vfns target_folder tablespace_number`

---

## 32. DB UPGRADE

### Reasons

- New features, optimization, security/bug fixes, application dependency

### Types

- **Minor**: Release change (14.1 → 14.2)
- **Major**: Version change (11 → 14)

---

## 33. MANUAL UPGRADE BEFORE 8.4

For versions before 8.4: pg_dumpall from old, psql restore to new. No pg_upgrade.

---

## 34. AUTO UPGRADE

### pg_upgrade (8.4+)

Pre-upgrade check:

```bash
/usr/pgsql-14/bin/pg_upgrade -b /usr/pgsql-11/bin/ -B /usr/pgsql-14/bin/ -d /var/lib/pgsql/11/data/ -D /var/lib/pgsql/14/data/ -p 5432 -P 5433 -c
```

Run upgrade:

```bash
/usr/pgsql-14/bin/pg_upgrade -b /usr/pgsql-11/bin/ -B /usr/pgsql-14/bin/ -d /var/lib/pgsql/11/data/ -D /var/lib/pgsql/14/data/ -p 5432 -P 5433
```

Post-upgrade:

```bash
vacuumdb --all --analyze-in-stages
```

### Pre-Upgrade Validation Queries

Take object counts from old version for validation:

```sql
SELECT count(*) FROM pg_stat_user_tables;
SELECT count(*) FROM pg_stat_user_indexes;
SELECT count(*) FROM pg_views WHERE schemaname='public';
SELECT count(*) FROM pg_user;
```

### Rollback

If application requests rollback: stop new version, start old version. After 10 days: pg_dumpall from new, restore to old, run analyze.

### Delete Old Cluster After Success

```bash
./delete_old_cluster.sh
```

### Two-Phase Upgrade (Below 8.4)

8.1 to 9.6: First manual upgrade 8.1 → 8.4 (pg_dumpall/psql), then pg_upgrade 8.4 → 9.6.

---

## 35. LINK UPGRADE WITHOUT EXTRA SPACE

```bash
pg_upgrade -b /usr/pgsql-11/bin/ -B /usr/pgsql-14/bin/ -d /var/lib/pgsql/11/data/ -D /var/lib/pgsql/14/data/ -p 5432 -P 5433 -k
```

Rollback: Rename pg_control.old to pg_control in global folder.

---

## 36. MINOR UPGRADE

For 14.0 to 14.2: Stop, erase old RPMs, install new RPMs, start.

---

## 37. PG BENCH FOR PERFORMANCE TESTING

pgbench estimates maximum capacity with respect to response time. Use when application requires max response time (e.g., &lt; 1 min) and you need to determine max queries.

### Initialize and Run

```bash
pgbench -i raj
pgbench -c 5 -t 5 raj
pgbench -S -c 100 -t 50 akash
pgbench -c 10 -t 10 -f test.sql akash
```

Output includes: latency average (query response time), tps (transactions per second). -S for SELECT only. -f for custom SQL file.

---

## 38. PG BADGER FOR LOG ANALYSIS REPORT

Default logs are text format and large. pgbadger analyzes logs and generates HTML report.

### Install and Use

```bash
yum install perl-devel
wget https://sourceforge.net/projects/pgbadger/files/12.4/pgbadger-12.4.tar.gz
tar -xvf pgbadger-12.4.tar.gz
cd pgbadger-12.4
perl Makefile.PL
make
make install
```

### Required log_format in postgresql.conf

```ini
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
```

### Generate Report

```bash
pgbadger -f `find /var/lib/pgsql_15/DATA/pg_log/ -name "postgresql-2024-04-11*"` -o pgbadger-report-$(date +%F).html
pgbadger -q `find /var/lib/pgsql_15/DATA/pg_log -name "postgresql*"` -o pgbadger-report-$(date +%F).html -f stderr
```

With time range: `pgbadger -b "2023-07-19 17:00:00" -e "2023-07-19 20:30:00" -o report.html postgresql.log`

---

## 39. PASSWORD CHECK EXTENSION

### Enable in postgresql.conf

```ini
shared_preload_libraries = 'passwordcheck'
```

Restart required. Enforces: no username in password, min 8 chars, letters and numbers.

---

## 40. REPLICATION SETUP

### Types

1. **Synchronous**: Master waits for standby ack before commit
2. **Asynchronous**: Master commits immediately (default)

### Streaming Replication (9.2+)

Master postgresql.conf:

```ini
listen_addresses = '*'
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1024
wal_log_hints = on
hot_standby = on
```

Master pg_hba.conf:

```
host replication replication 192.168.56.202/32 trust
```

Create replication user:

```sql
CREATE USER replication WITH REPLICATION;
```

Standby: pg_basebackup from master:

```bash
pg_basebackup -D /var/lib/pgsql/14/data/ -X fetch -P -R -h 192.168.56.201 -U replication
```

-R creates standby.signal and primary_conninfo in postgresql.auto.conf.

### Replication Validation

From master:

```sql
SELECT * FROM pg_stat_replication;
SELECT pg_current_wal_lsn();
```

From standby:

```sql
SELECT pg_is_in_recovery();
SELECT pg_last_wal_receive_lsn();
```

sent_lsn, write_lsn, flush_lsn, replay_lsn should match for no lag. Check write_lag, flush_lag, replay_lag are blank.

### LSN Utilities

```sql
SELECT pg_wal_lsn_diff('0/C0197F8','0/8000110');
SELECT pg_walfile_name('0/C0197F8');
```

---

## 41. LOG SHIPPING BASED (WARM/HOT) STANDBY REPLICATION

### Initial Setup on Master

1. Create SSH key, copy to standby: `ssh-copy-id postgres@standby_ip`
2. Shutdown master, edit postgresql.conf: archive_mode=on, archive_command, archive_timeout
3. archive_command: `rsync -a %p postgres@standby_ip:/archive_location`

### Standby Setup

1. Create archive directory on standby
2. Shutdown standby, delete DATA directory contents
3. On master: `SELECT pg_start_backup('dbrep');`
4. Copy: `rsync -avz /var/lib/pgsql/12/data/* postgres@standby_ip:/var/lib/pgsql/12/data/`
5. On master: `SELECT pg_stop_backup();`
6. On standby: Comment archive params, set restore_command, create standby.signal
7. Start standby

### Promote Standby

```bash
pg_ctl promote -D /var/lib/pgsql/12/data
```

---

## 42. STREAMING REPLICATION

See section 40 for configuration. WAL records streamed continuously via wal sender/receiver.

---

## 43. CONVERT ASYNC TO SYNC

```ini
synchronous_standby_names = '*'
```

Reload: `pg_ctl reload`

---

## 44. CONVERT SYNC TO ASYNC

Comment out synchronous_standby_names, reload.

---

## 45. MULTI SLAVE REPLICATION SETUP

Add standby IPs to master pg_hba.conf. Run pg_basebackup on each standby from master.

---

## 46. CASCADE REPLICATION SETUP

Master → Standby1 → Standby2. Add Standby2 IP to Standby1 pg_hba.conf. pg_basebackup from Standby1 on Standby2.

---

## 47. MANUAL FAILOVER

Promote standby:

```bash
pg_ctl -D /var/lib/pgsql/14/data/ promote
```

Use pg_rewind to resync old master:

```bash
pg_rewind --target-pgdata=/var/lib/pgsql/14/data/ --source-server='host=192.168.56.202 user=replication dbname=replication'
```

---

## 48. AUTOFAILOVER EXTENSION (PG_AUTOCTL)

Requires 4 nodes: monitor (port 5000), master (5001), standby1 (5002), standby2 (5003).

### Install

```bash
wget https://install.citusdata.com/community/rpm.sh
chmod 777 rpm.sh
./rpm.sh
yum install pg-auto-failover16_14 -y
```

### Create Monitor

```bash
export PGPORT=5000
pg_autoctl create monitor --ssl-self-signed --auth trust
nohup pg_autoctl run &
pg_autoctl show uri
```

### Create Primary and Standbys

```bash
export PGPORT=5001
pg_autoctl create postgres --dbname autodb --auth trust --no-ssl --monitor postgres://autoctl_node@monitor_host:5000/pg_auto_failover
nohup pg_autoctl run &
```

Repeat for standby nodes with PGPORT=5002, 5003. Add all node IPs to pg_hba.conf on each node.

### Cluster State

```bash
pg_autoctl show state
pg_autoctl show uri
```

### Switchover

```bash
pg_autoctl perform switchover
```

### Set Failover Priority

```bash
pg_autoctl set node candidate-priority --name node_5 90
```

### Maintenance Mode

```bash
pg_autoctl enable maintenance
```

### When Both Standbys Down

```bash
pg_autoctl set formation number-sync-standbys 0
```

---

## 49. REP MANAGER

See Replication-repmanager folder for Repmgr setup: primary-standby, failover, rejoin, witness server, etc.

---

## 50. LOGICAL REPLICATION

### Use Cases

- Replicate subset of tables
- Different PostgreSQL versions
- Consolidate data for analytics

### Limitations

- Tables need primary key or unique key
- No DDL replication (only DML)
- No sequences, LOBs
- No mutual replication (without origin handling)

### Setup

Publisher (postgresql.conf):

```ini
wal_level = logical
```

```sql
CREATE PUBLICATION mypub FOR TABLE test_rep, test_rep_other;
```

Subscriber:

```sql
CREATE SUBSCRIPTION mysub CONNECTION 'dbname=source_rep host=192.168.56.201 user=postgres port=5432' PUBLICATION mypub;
```

Refresh after DDL fix:

```sql
ALTER SUBSCRIPTION mysub REFRESH PUBLICATION;
```

If DDL (e.g. ALTER TABLE ADD COLUMN) breaks replication, fix on subscriber then refresh. If not resolved, drop and recreate subscription.

### Publication for All Tables

```sql
CREATE PUBLICATION mytestpub FOR ALL TABLES;
```

Take schema dump: `pg_dump -d source_rep -p 5434 -s > publicationtables_sourcedb.sql`. Restore on subscriber, then create subscription.

### Logical Replication Monitoring

Publisher: `SELECT * FROM pg_publication_tables;` and `SELECT * FROM pg_stat_replication;`
Subscriber: `SELECT * FROM pg_stat_subscription;`

### Drop Subscription/Publication

```sql
DROP SUBSCRIPTION mysub;
DROP PUBLICATION mypub;
```

\dRs shows subscriptions. \dRp shows publications.

### Logical Replication v15+ Features

- Column list (replicate specific columns)
- Row filtering
- ALTER SUBSCRIPTION ... SKIP (disable on error)
- Schema-level publication
- pg_stat_subscription_stats for errors

### Logical Replication v16+ Features

- Replicate from standby
- Parallel apply (max_parallel_apply_workers_per_subscription)
- pg_create_subscription role
- Bi-directional with origin
- Binary copy for initial load (30% improvement)

---

## 51. TABLE INHERITANCE

```sql
CREATE TABLE orders(orderno serial, flightname varchar(100), boarding varchar(100), status varchar(100), source varchar(100));
CREATE TABLE online_booking (price int) INHERITS(orders);
CREATE TABLE agent_booking (commission int) INHERITS(orders);
```

Insert and select:

```sql
INSERT INTO orders(flightname,boarding,status,source) VALUES('aircanada','xyz','ontime','employees');
INSERT INTO online_booking(flightname,boarding,status,source,price) VALUES('nippon','chn','ontime','website',5000);
INSERT INTO agent_booking(flightname,boarding,status,source,commission) VALUES('etihad','aud','ontime','agent001',1000);
SELECT * FROM orders;
```

Update only parent (not children): `UPDATE ONLY orders SET status='Cancelled';`

Delete from parent cascades to children. Drop parent requires CASCADE due to dependencies.

---

## 52. COPY TABLE

```sql
CREATE TABLE train_dest AS TABLE train_bookings;
CREATE TABLE train_dest AS TABLE train_bookings WITH NO DATA;
```

---

## 53. PG_BACKREST

### Types

- Full, Differential, Incremental

### Configuration

```ini
[global]
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2

[pg0app]
pg1-path=/app01/data
pg1-port=5432
pg1-user=postgres
```

### Commands

```bash
pgbackrest stanza-create --stanza=pg0app
pgbackrest backup --stanza=pg0app
pgbackrest backup --type=incr --stanza=pg0app
pgbackrest backup --type=diff --stanza=pg0app
pgbackrest --stanza=pg0app --delta --type=time "--target=2019-12-24 13:22:22.329155+05:30" --target-action=promote restore
```

### PITR with pgBackRest

After restore, recovery.conf (or postgresql.auto.conf) contains:

```ini
restore_command = 'pgbackrest --stanza=pg0app archive-get %f "%p"'
recovery_target_time = '2019-12-24 13:22:22.329155+05:30'
recovery_target_action = 'promote'
```

Start instance: `pg_ctl -D data/ start`

### postgresql.conf for pgBackRest

```ini
wal_level = 'replica'
archive_mode = 'on'
archive_command = 'pgbackrest --stanza=pg0app archive-push %p'
max_wal_senders = '10'
```

---

## 54. POSTGRES 13 FEATURES

- B-tree deduplication
- Incremental sorting
- Parallel VACUUM
- Trusted extensions
- DROP DATABASE WITH (FORCE)
- EXPLAIN (ANALYZE, WAL)
- pg_stat_progress_basebackup, pg_stat_progress_analyze, pg_shmem_allocations

---

## 55. POSTGRES 15 FEATURES

- MERGE command
- Logical replication: column lists, row filters
- JSON log format
- pg_basebackup compression (zstd)
- Stats in shared memory (no stats collector process)
- pg_read_all_data, pg_write_all_data roles
- pg_backup_start, pg_backup_stop
- GRANT ALTER SYSTEM ON PARAMETER

---

## 56. POSTGRES 16 FEATURES

- Logical replication from standby
- pg_stat_io
- COPY 300% improvement, pg_stat_progress_copy
- Parallel full/right joins
- auto_explain with bind values
- Incremental sort for SELECT DISTINCT
- pg_create_subscription role
- Bi-directional replication with origin

---

## 57. POSTGRES 17 FEATURES

(Content in original was placeholder—refer to PostgreSQL 17 release notes)

---

## 58. INSTALL POSTGRES CLIENT (PSQL, PGDUMP ON WINDOWS SUBSYSTEM FOR LINUX)

### WSL Installation

```bash
sudo apt update
sudo apt install wget gnupg2 lsb-release -y
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt install postgresql-client-15 -y
psql --version
pg_dump --version
```

---

## 59. ADDITIONAL REFERENCE

### AWS RDS PostgreSQL

Connect to RDS instance:

```bash
psql -h pgsqlfeb22.crfk9g7requz.us-east-1.rds.amazonaws.com -U postgres -d postgres
```

### Environment Variables

Add to .bash_profile for postgres user:

```bash
export PATH=$PATH:/usr/local/pgsql/bin/
export PGDATA=/var/lib/pgsql_feb22/DATA/
```

For binaries: `export PGDATA=/var/lib/pgsql/14/data/`

### Enable Auto Startup (rc.local)

```bash
vi /etc/rc.d/rc.local
su - postgres -c 'pg_ctl -D $PGDATA start'
chmod +x /etc/rc.d/rc.local
```

### PostgreSQL 15 Backup Commands

- `pg_start_backup` → `pg_backup_start`
- `pg_stop_backup` → `pg_backup_stop`

### Sample Data (dvdrental)

```bash
# Download from https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip
createdb dvdrental
pg_restore -Ft -d dvdrental dvdrental.tar -v 2>dvdrental.log
```

### Crontab Examples

Reference: https://www.thegeekstuff.com/2009/06/15-practical-crontab-examples/

---

This document consolidates PostgreSQL DBA knowledge from installation through advanced replication and version-specific features. For detailed step-by-step procedures, refer to the specific sections above.
