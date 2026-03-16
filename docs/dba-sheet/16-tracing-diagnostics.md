---
layout: default
title: "Tracing & Diagnostics"
parent: "DBA Reference Sheet"
nav_order: 16
---

# Tracing & Diagnostics

SQL trace, 10046 trace, tkprof, oradebug, ADRCI, and hang analysis for Oracle DBAs.

---

## Trace Enable / Disable

### Enable trace (multiple methods)

```sql
ALTER SESSION SET EVENTS '10046 TRACE NAME CONTEXT FOREVER, LEVEL 12';
```

**Alternative:**

```sql
EXEC SYS.DBMS_SYSTEM.SET_EV(SID,SERIAL#,10046,12,'');
EXEC SYS.DBMS_SUPPORT.START_TRACE_IN_SESSION(SID,SERIAL#,WAITS=>TRUE,BINDS=>FALSE);
EXEC DBMS_MONITOR.SESSION_TRACE_ENABLE(SESSION_ID=>SID,SERIAL_NUM=>SERIAL#,WAITS=>TRUE,BINDS=>TRUE);
EXEC SYS.DBMS_SYSTEM.SET_SQL_TRACE_IN_SESSION(SID,SERIAL#,TRUE);
```

### Disable trace

```sql
ALTER SESSION SET EVENTS '10046 TRACE NAME CONTEXT OFF';
```

**Alternative:**

```sql
EXEC SYS.DBMS_SYSTEM.SET_EV(SID,SERIAL#,10046,0,'');
EXEC SYS.DBMS_SUPPORT.STOP_TRACE_IN_SESSION(SID,SERIAL#);
EXEC SYS.DBMS_SYSTEM.SET_SQL_TRACE_IN_SESSION(SID,SERIAL#,FALSE);
```

---

## Session Identification

### Find own SID and serial number

```sql
SELECT username, inst_id, sid, serial# FROM gV$SESSION WHERE audsid = userenv('sessionid');
```

**Alternative:**

```sql
SELECT sys_context('USERENV', 'SID') OwnSID FROM dual;
SELECT DISTINCT sid OwnSID FROM v$mystat;
```

---

## SQL Trace (Basic)

### Start/stop session trace

```sql
ALTER SESSION SET sql_trace = true;
ALTER SESSION SET sql_trace = false;
```

### Add tracefile identifier for identification

```sql
ALTER SESSION SET sql_trace = true;
ALTER SESSION SET tracefile_identifier = mysqltrace;
```

---

## Tracing Other Sessions

### Tracing other user's sessions (10g)

```sql
EXECUTE dbms_system.set_sql_trace_in_session(501, 44396, true);
EXECUTE dbms_system.set_sql_trace_in_session(501, 44396, false);
```

Look for trace file in `USER_DUMP_DEST`. Get SID and serial number from the session identification query above.

### Tracing in 11g (recommended)

```sql
EXEC dbms_monitor.session_trace_enable(session_id=>3, serial_num=>5027, binds=>true, waits=>true);
EXEC dbms_monitor.session_trace_disable(session_id=>3, serial_num=>5027);
```

---

## Trace by Module or SQL ID

### Trace a module

```sql
EXECUTE DBMS_MONITOR.SERV_MOD_ACT_TRACE_ENABLE(
    service_name=>'vasont.world', module_name=>'VasontU.exe',
    action_name=>DBMS_MONITOR.ALL_ACTIONS, waits=>TRUE,
    binds=>TRUE, instance_name=>NULL);
EXECUTE DBMS_MONITOR.SERV_MOD_ACT_TRACE_DISABLE(
    service_name=>'vasont.world', module_name=>'VasontU.exe');
```

### Trace a specific sql_id

```sql
ALTER SYSTEM SET EVENTS 'sql_trace[SQL:8kybysnu4nn34] plan_stat=all_executions,wait=true,bind=true';
ALTER SYSTEM SET EVENTS 'sql_trace[SQL:8kybysnu4nn34] off';
```

**Alternative:**

```sql
ALTER SYSTEM SET EVENTS 'sql_trace[sql:cjrha4bzuupzf] level=12';
ALTER SYSTEM SET EVENTS 'sql_trace[SQL:8kybysnu4nn34] off';
```

---

## Check Active Events

### List active events in database

```sql
DECLARE
  event_level NUMBER;
BEGIN
  FOR i IN 10000..10999 LOOP
    sys.dbms_system.read_ev(i, event_level);
    IF (event_level > 0) THEN
      dbms_output.put_line('Event '||TO_CHAR(i)||' set at level '|| TO_CHAR(event_level));
    END IF;
  END LOOP;
END;
/
```

---

## Trace Using Trigger

### Enable trace on logon for specific user

```sql
CREATE OR REPLACE TRIGGER sqltrace
AFTER LOGON ON DATABASE
BEGIN
  IF user='XXXXX' THEN
    EXECUTE IMMEDIATE 'alter session set events ''10046 trace name context forever, level 4''';
  END IF;
END;
/
```

**Alternative (with tracefile identifier):**

```sql
CREATE OR REPLACE TRIGGER trace_trigger_scott
    AFTER LOGON ON DATABASE
    WHEN (USER='SCOTT')
DECLARE
   stmt VARCHAR2(100);
   hname VARCHAR2(20);
   uname VARCHAR2(20);
BEGIN
   SELECT sys_context('USERENV','HOST'), sys_context('USERENV','SESSION_USER')
   INTO hname, uname FROM dual;
   stmt := 'alter session set tracefile_identifier='||hname||'_'||uname;
   EXECUTE IMMEDIATE stmt;
   EXECUTE IMMEDIATE 'alter session set sql_trace=true';
END;
/
```

### Disable tracing on logoff

```sql
CREATE OR REPLACE TRIGGER trace_trigger_off
  BEFORE LOGOFF ON DATABASE
  WHEN(user='SCOTT')
BEGIN
  EXECUTE IMMEDIATE 'alter session set sql_trace=false';
END;
/
```

---

## Tracing Entire Database

```sql
ALTER SYSTEM SET sql_trace = true SCOPE=MEMORY;
ALTER SYSTEM SET sql_trace = false SCOPE=MEMORY;
```

---

## 10046 Trace

### 10046 trace by SID/serial (from v$session/v$process)

```sql
SELECT p.spid, s.sid, s.serial#
FROM v$session s, v$process p
WHERE s.paddr = p.addr AND p.spid = 24078;

BEGIN dbms_system.set_ev(18, 5, 10046, 12, ''); END;  -- trace on
-- collect trace for ~15 minutes during the problem
BEGIN dbms_system.set_ev(18, 5, 10046, 0, ''); END;   -- trace off
```

**Alternative (oradebug by OS PID):**

```sql
-- connect / as sysdba
oradebug setospid 9834
oradebug unlimit
oradebug event 10046 trace name context forever, level 12
oradebug tracefile_name
-- wait for 5 mins then trace off
oradebug event 10046 trace name context off
```

**Alternative (with errorstack for DBLINK):**

```sql
ALTER SESSION SET tracefile_identifier='mytrace_1089';
ALTER SESSION SET EVENTS '10046 trace name context forever, level 12:1089 trace name errorstack level 3';
SELECT * FROM dual@DBLINK;
ALTER SESSION SET EVENTS '10046 trace name context off';
```

### 10046 tracing existing process (by SID)

```sql
SELECT p.PID, p.SPID, s.SID FROM v$process p, v$session s
WHERE s.paddr = p.addr AND s.sid = &SESSION_ID;
```

Then use `oradebug setospid <spid>` (Linux/Unix) or `oradebug setorapid <pid>` (Windows):

```sql
oradebug setospid <spid>
oradebug unlimit
oradebug event 10046 trace name context forever, level 12;
```

### Wait events and bind variables for different session

```sql
EXEC sys.dbms_system.set_ev([SID], [SERIAL], 10046, 12, '');
EXEC dbms_system.set_ev(si=>123, se=>1234, ev=>10046, le=>8, nm=>' ');
```

---

## Dump Trace for SQL in Shared Pool (11gR2+)

```sql
EXECUTE DBMS_SQLDIAG.DUMP_TRACE(p_sql_id=>'&sql_id',
  p_child_number=>0,
  p_component=>'Optimizer',
  p_file_id=>'TRACE_10053');
```

### Compiler trace for specific sql_id

```sql
EXEC dbms_sqldiag.dump_trace(p_sql_id=>'<SQL_ID>', p_child_number=>0, p_component=>'Compiler', p_file_id=>'CBO_TRACE_DEV');
EXEC dbms_sqldiag.dump_trace(p_sql_id=>'<SQL_ID>', p_child_number=>0, p_component=>'Compiler', p_file_id=>'CBO_TRACE_PRD');
```

---

## Parallel Session Trace

```sql
ALTER SESSION SET "_px_trace"=high,all;
```

---

## 10132 Trace

```sql
ALTER SESSION SET tracefile_identifier='My_trace';
ALTER SESSION SET EVENTS '10132 trace name context forever, level 12';
ALTER SESSION SET EVENTS '10132 trace name context off';
```

---

## Tkprof

### Basic tkprof usage

```bash
tkprof orcl102_ora_3064.trc output.prf EXPLAIN=scott/tiger SYS=NO
```

### Sort by CPU (recommended)

```bash
tkprof ora10g_ora_5868.trc tracea.prf sort=fchcpu,prscpu,execpu
```

---

## Database Hung – OS Level

### Solaris

```bash
ps -ef | egrep '4711|4713'
truss -aefo /tmp/truss.out -p <PID>
```

### Linux

```bash
strace -f -o output.txt sqlplus / as sysdba
```

---

## Database Hung – DB Level

### RAC (prelim connection)

```bash
sqlplus /nolog
```

```sql
SET _prelim on
CONNECT / as sysdba
oradebug setorapname reco
oradebug -g all hanganalyze 3
oradebug -g all dump systemstate 266
oradebug setorapname diag
oradebug tracefile_name
```

### Single instance

```bash
sqlplus /nolog
```

```sql
SET _prelim on
CONNECT / as sysdba
oradebug setorapname diag
oradebug dump hanganalyze 3
oradebug dump systemstate 266
oradebug tracefile_name
```

---

## Login to DB When Hung

```bash
sqlplus -prelim / as sysdba
```

```sql
oradebug setmypid
oradebug unlimit
oradebug dump systemstate 266
```

**Alternative (hanganalyze):**

```sql
oradebug setmypid
oradebug unlimit
oradebug hanganalyze 3
-- wait one minute before getting the second hanganalyze
oradebug hanganalyze 3
oradebug tracefile_name
```

---

## Dump ASH When DB Is Hung

```sql
oradebug setmypid
oradebug dump ashdump 5
oradebug tracefile_name
```

Dumps last 5 minutes of ASH content.

---

## Tracing RMAN

```bash
rman rcvcat rman/rman@catalog target user/passwd@target debug trace trace_file_name
```

---

## Tracing Oracle Process at OS Level

```bash
strace -fF -v -p 16311 -o output.txt
```

---

## Tracing SQL*Plus at OS Level

```bash
strace /oracle/product/10.2.0.1/bin/sqlplus -V 2>&1 | less
```

---

## Oradebug Utility

Find OS PID or Oracle PID:

```sql
SELECT s.sid, s.serial#, p.spid, p.pid, s.username, s.osuser
FROM v$session s, v$process p
WHERE s.paddr = p.addr;
```

Set trace via oradebug:

```sql
oradebug setospid 864
-- or: oradebug setorapid 13
oradebug unlimit
oradebug event 10046 trace name context forever, level 12
-- run queries in target session, then:
oradebug event 10046 trace name context off
```

---

## ADRCI

### Show homes and purge diagnostic data

```bash
adrci> show homes
adrci> set home diag/tnslsnr/dbname/listener
adrci> purge -age 2880 -type ALERT
adrci> purge -age 2880 -type TRACE
adrci> purge -age 2880
```

### Create incident package

```bash
adrci> show homes
adrci> set homepath diag/rdbms/dbname/instancename
adrci> show incident
adrci> ips pack incident 90737 in /tmp
```

### Diagcollection (11.2+)

```bash
$GRID_HOME/bin/diagcollection.sh
```

For 10.2 and 11.1:

```bash
$CRS_HOME/bin/diagcollection.pl -crshome=$CRS_HOME --collect
```

---

## Reference Links

- [Correct way to trace a session](http://tinky2jed.wordpress.com/technical-stuff/oracle-stuff/what-is-the-correct-way-to-trace-a-session-in-oracle/)
- [How to trace Oracle sessions](http://alexzeng.wordpress.com/2008/08/01/how-to-trace-oracle-sessions/)
- [Oracle Developer Net – Tracing](http://www.oracle-developer.net/display.php?id=516)
- [trcextprof – raw trace 10046 profiler](https://mahmoudhatem.wordpress.com/2015/07/23/trcextprof-sql-the-raw-trace-file-10046-profiler-based-on-external-tables-regexp/)
- [SQL Trace – ORAFAQ](http://www.orafaq.com/wiki/SQL_Trace)
- [TKProf – ORAFAQ](http://www.orafaq.com/wiki/TKProf)
- [Event 10132](https://jonathanlewis.wordpress.com/2006/11/27/event-10132/)
- [Oracle hanging – hanganalyze and systemstate](https://blog.dbi-services.com/oracle-is-hanging-dont-forget-hanganalyze-and-systemstate/)
- Metalink Note: 452358.1

---

Back to [Main Index](README.md)
