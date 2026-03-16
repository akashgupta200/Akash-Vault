---
layout: default
title: "Memory Management Reference"
parent: "DBA Reference Sheet"
nav_order: 22
---

# Memory Management Reference

SGA, PGA, shared pool, buffer cache, and memory tuning queries for Oracle DBAs.

---

## Shared Pool Resources

- [Shared pool management (coskan.wordpress.com)](https://coskan.wordpress.com/2007/09/14/what-i-learned-about-shared-pool-management/)
- [Shared pool purging (yong321.freeshell.org)](http://yong321.freeshell.org/computer/SharedPoolPurging.html)

---

## SGA Queries

### SGA Dynamic Allocation

```sql
select component,current_size/1024/1024 || 'M' from v$sga_dynamic_components;
```

### Free/Used SGA

```sql
column TOTAL_SGA for a20
column USEED for a20
column FREE for a20
select  round(sum(bytes)/1024/1024,2)||' MB' total_sga,
      round(round(sum(bytes)/1024/1024,2) - round(sum(decode(name,'free memory',bytes,0))/1024/1024,2))||' MB' used,
      round(sum(decode(name,'free memory',bytes,0))/1024/1024,2)||' MB' free
    from v$sgastat;
```

### Memory Utilization in SGA by Shared Pool (Top 25)

```sql
select * from ( select POOL, NAME, BYTES, BYTES/1048576 as MBytes from v$sgastat where pool='shared pool' order by BYTES desc ) where rownum <= 25;
```

### Free Memory in Shared Pool

```sql
SELECT * FROM v$sgastat
WHERE name = 'free memory';
```

### Amount of Session Memory Used in Shared Pool

```sql
select SUM(value)/1024/1024||' Mb' "Total session memory"
     from v$sesstat, v$statname
     where name = 'session uga memory'
     and v$sesstat.statistic# = v$statname.statistic#;
```

### Statements Using Lots of Shared Pool Memory

```sql
set lines 1000 pages 9999
col stmt for a150
SELECT substr(sql_text,1,150) "Stmt", count(*),
                sum(sharable_mem)    "Mem",
                sum(users_opening)   "Open",
                sum(executions)      "Exec"
          FROM v$sql
         GROUP BY substr(sql_text,1,150)
        HAVING sum(sharable_mem) > 20000
        order by sum(sharable_mem);
```

---

## PGA Queries

### Total PGA Used (MB)

```sql
select sum(pga_used_mem)/1024/1024 pga from v$process;
```

### Determine PGA and Process Memory by Process

```sql
set lines 110
col unm format a30 hea "USERNAME (SID,SERIAL#)"
col pus format 999,990.9 hea "PROC KB|USED"
col pal format 999,990.9 hea "PROC KB|MAX ALLOC"
col pgu format 99,999,990.9 hea "PGA KB|USED"
col pga format 99,999,990.9 hea "PGA KB|ALLOC"
col pgm format 99,999,990.9 hea "PGA KB|MAX MEM"

select s.username||' ('||s.sid||','||s.serial#||')' unm, round((sum(m.used)/1024),1) pus,
round((sum(m.max_allocated)/1024),1) pal, round((sum(p.pga_used_mem)/1024),1) pgu,
round((sum(p.pga_alloc_mem)/1024),1) pga, round((sum(p.pga_max_mem)/1024),1) pgm
from v$process_memory m, v$session s, v$process p
where m.serial# = p.serial# and p.pid = m.pid and p.addr=s.paddr and
s.username is not null group by s.username, s.sid, s.serial# order by unm;
```

### History of PGA Memory Used

```sql
select min(a.begin_time),max(a.end_time),b.USED_TOTAL,b.ALLOCATED_TOTAL from dba_hist_sysmetric_summary a,DBA_HIST_PROCESS_MEM_SUMMARY b where begin_time between SYSDATE-3 and SYSDATE and a.snap_id=b.snap_id group by b.USED_TOTAL,b.ALLOCATED_TOTAL order by 1 asc;
```

### PGA Memory Used by Each Process

```sql
SELECT to_char(ssn.sid, '9999') || ' - ' || nvl(ssn.username, nvl(bgp.name, 'background')) ||
nvl(lower(ssn.machine), ins.host_name) "SESSION",
to_char(prc.spid, '999999999') "PID/THREAD",
to_char((se1.value/1024)/1024, '999G999G990D00') || ' MB' " CURRENT SIZE",
to_char((se2.value/1024)/1024, '999G999G990D00') || ' MB' " MAXIMUM SIZE"
FROM v$sesstat se1, v$sesstat se2, v$session ssn, v$bgprocess bgp, v$process prc,
v$instance ins, v$statname stat1, v$statname stat2
WHERE se1.statistic# = stat1.statistic# and stat1.name = 'session pga memory'
AND se2.statistic# = stat2.statistic# and stat2.name = 'session pga memory max'
AND se1.sid = ssn.sid
AND se2.sid = ssn.sid
AND se1.sid=&sid
AND ssn.paddr = bgp.paddr (+)
AND ssn.paddr = prc.addr (+);
```

**Alternative:** For a single SID, replace `AND se1.sid=&sid` with `AND se1.sid=114`.

### PGA Memory Usage by SQL ID

Reference: [Link huge PGA temp (bdrouvot.wordpress.com)](https://bdrouvot.wordpress.com/2013/03/19/link-huge-pga-temp/)

```sql
col percent head '%' for 99990.99
col star for A10 head ''

accept seconds prompt "Last Seconds [60] : " default 60;
accept top prompt "Top  Rows    [10] : " default 10;

select SQL_ID,round(PGA_MB,1) PGA_MB,percent,rpad('*',percent*10/100,'*') star
from
(
select SQL_ID,sum(DELTA_PGA_MB) PGA_MB ,(ratio_to_report(sum(DELTA_PGA_MB)) over ())*100 percent,rank() over(order by sum(DELTA_PGA_MB) desc) rank
from
(
select SESSION_ID,SESSION_SERIAL#,sample_id,SQL_ID,SAMPLE_TIME,IS_SQLID_CURRENT,SQL_CHILD_NUMBER,PGA_ALLOCATED,
greatest(PGA_ALLOCATED - first_value(PGA_ALLOCATED) over (partition by SESSION_ID,SESSION_SERIAL# order by sample_time rows 1 preceding),0)/power(1024,2) "DELTA_PGA_MB"
from
v$active_session_history
where
IS_SQLID_CURRENT='Y'
and sample_time > sysdate-&seconds/86400
order by 1,2,3,4
)
group by sql_id
having sum(DELTA_PGA_MB) > 0
)
where rank < (&top+1)
order by rank
/
```

### PGA Memory Used by Particular SPID

```sql
SELECT spid, program,
            pga_max_mem      max,
            pga_alloc_mem    alloc,
            pga_used_mem     used,
            pga_freeable_mem free
       FROM V$PROCESS
      WHERE spid = 7735;
```

### PGA Memory Used by Background Processes

```sql
select PID,SPID,serial#,USERNAME,PROGRAM,PGA_USED_MEM,PGA_ALLOC_MEM,PGA_FREEABLE_MEM,PGA_MAX_MEM from v$process order by PGA_USED_MEM desc;
```

### Max Memory Used by a Person or Process

```sql
set linesize 750
set pagesize 250
column box format a20
column username format a7
column program format a20
column os_user format a20

select s.sid, p.pid, p.spid,substr(s.machine,1,20) box,s.logon_time logon_date,to_char (s.logon_time, 'hh24:mi:ss') logon_time,
substr(s.username,1,7) username,
substr(s.osuser,1,20) os_user,
substr(s.program,1,35) program from v$session s, v$process p where s.paddr = p.addr and
s.sid=(select s.sid from v$session s where paddr=(select addr from v$process where
pga_used_mem=(select max(pga_used_mem) from v$process)));
```

### Memory Used by Each User (PGA, UGA)

```sql
select s.osuser osuser,s.username,s.program,s.serial# serial,se.sid,n.name,
       max(se.value) maxmem
from v$sesstat se,
     v$statname n
,v$session s
where n.statistic# = se.statistic#
and   n.name in ('session pga memory','session pga memory max',
                 'session uga memory','session uga memory max')
and s.sid=se.sid
group by n.name,se.sid,s.osuser,s.username,s.program,s.serial#
order by 2;
```

### Session Memory Used in PGA (sort_area_size + hash_area_size)

```sql
set pages 999
column pga_size format 999,999,999
select
    1048576+a.value+b.value   pga_size
from
   v$parameter a,
   v$parameter b
where
   a.name = 'sort_area_size'
and
   b.name = 'hash_area_size';
```

---

## Total Memory

### Total Memory Used (SGA + PGA)

```sql
select (sga+pga)/1024/1024 as "sga_pga"
      from
      (select sum(value) sga from v$sga),
      (select sum(pga_used_mem) pga from v$process);
```

---

## I/O Process

### Data File and Temp File I/O Statistics

```sql
COL ts FORMAT a10 HEADING "Tablespace";
COL reads FORMAT 9999900;
COL writes FORMAT 9999900;
COL br FORMAT 9999900 HEADING "BlksRead";
COL bw FORMAT 9999900 HEADING "BlksWrite";
COL rtime FORMAT 9999900;
COL wtime FORMAT 9999900;
SELECT ts.name AS ts, fs.phyrds "Reads", fs.phywrts "Writes"
, Fs.phyblkrd AS br, fs.phyblkwrt AS bw
, Fs.readtim "RTime", fs.writetim "WTime"
FROM v$tablespace ts, v$datafile df, v$filestat fs
WHERE ts.ts# = df.ts# AND df.file# = fs.file#
UNION
SELECT ts.name AS ts, ts.phyrds "Reads", ts.phywrts "Writes"
, Ts.phyblkrd AS br, ts.phyblkwrt AS bw
, Ts.readtim "RTime", ts.writetim "WTime"
FROM v$tablespace ts, v$tempfile tf, v$tempstat ts
WHERE ts.ts# = tf.ts# AND tf.file# = ts.file# ORDER BY 1;
```

---

## ORA-04031 and Memory Issues

### Resources

- [5 Easy Steps to Solve ORA-04031 (dbas-oracle.com)](http://www.dbas-oracle.com/2013/05/5-Easy-Step-to-Solve-ORA-04031-with-Oracle-Support-Provided-Tool.html)
- [X$KSMLRU, X$KSMSP shared pool monitoring (dba-oracle.com)](http://www.dba-oracle.com/t_x$ksmlru_x$ksmsp_shared_pool_monitoring.htm)
- MOSC Note: 146599.1 — Diagnosing and Resolving Error ORA-04031

### ORA-04031 History (10g)

```sql
set pages 999
set lines 130
col component for a25 head "Component"
col status format a10 head "Status"
col initial_size for 999,999,999,999 head "Initial"
col parameter for a25 heading "Parameter"
col final_size for 999,999,999,999 head "Final"
col changed head "Changed At"

select component, parameter, initial_size, final_size, status,
to_char(end_time ,'mm/dd/yyyy hh24:mi:ss') changed
from gv$sga_resize_ops
order by 6
/
```

### ORA-04031 History (11g)

```sql
set pages 999
set lines 130
col component for a25 head "Component"
col status format a10 head "Status"
col initial_size for 999,999,999,999 head "Initial"
col parameter for a25 heading "Parameter"
col final_size for 999,999,999,999 head "Final"
col changed head "Changed At"

select component, parameter, initial_size, final_size, status,
to_char(end_time ,'mm/dd/yyyy hh24:mi:ss') changed
from gv$memory_resize_ops
order by 6
/
```

### Troubleshoot Tool

Metalink ID 559339.1

---

[Back to Main Index](README.md)
