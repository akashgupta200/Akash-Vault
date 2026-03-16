---
layout: default
title: "Security"
parent: "DBA Reference Sheet"
nav_order: 26
---

# Security

Password management, profiles, auditing, wallet, backdoor access, and Oracle Database Vault for Oracle DBAs.

---

## Password & Verification

### Pwd verify function extract password

Reference: [OraFAQ - Pwd verify function](http://orafaq.com/node/58)

### Oracle password hack

Reference: [Sooner or Later - Oracle password](http://www.soonerorlater.hu/index.khtml?article_id=513)

Reference: [Pete Finnigan's weblog](http://www.petefinnigan.com/weblog/archives/00001103.htm)

---

## Backdoor Access

### Backdoor entry to Oracle database

```bash
sqlplus -prelim / as sysdba
```

**Alternative:**

```sql
sqlplus /nolog
SQL> set _prelim on
SQL> conn / as sysdba
```

Reference: [The backdoor entry to Oracle database](https://taliphakanozturken.wordpress.com/2012/06/12/the-backdoor-entry-to-oracle-database/)

---

## DB Link Password

### Extract DB link password

Works under 11.2.0.2.

```sql
select passwordx from sys.link$ where name='mydblink';

set serveroutput on
declare
 db_link_password varchar2(500);
begin
 db_link_password := '06D05F5E36F13A08FD3C5FE489EB89B094701C114FF156A92D84A5724EF5FC2BA4F25BFBE99146C22075BEF3012D0F9DC6231FBD1A5EFBFA97DCD8FD13737243992EA16AD5A23B7DC823346DEB4CD69FE6F20B3F15821FEFF9F44430EE40C78CAEE37DF25F25C2BEDED1DD2A61C72351E462BF1B844B2599E5125AE0135EAF7';

 dbms_output.put_line ('Plain password: ' || utl_raw.cast_to_varchar2 ( dbms_crypto.decrypt ( substr (db_link_password, 19) , dbms_crypto.DES_CBC_PKCS5 , substr (db_link_password, 3, 16) ) ) );
end;
/
```

Reference: [Decrypting Oracle database DB link password](http://murty4all.blogspot.ch/2013/11/decrypting-oracle-database-db-link_29.html)

---

## Session Spoofing

### Spoof OSUSER in v$session

Reference: [Spoofing v$session osuser](https://oraganism.wordpress.com/2009/10/06/spoofing-vsession-osuser/)

---

## Wallet

### ORA-28365: wallet is not open

Connect to container and check whether WALLET is OPEN. If closed at Container DB, open it:

```sql
select status from v$encryption_wallet;
```

```sql
ADMINISTER KEY MANAGEMENT SET KEYSTORE open IDENTIFIED BY abc123 container=all;
```

If wallet is OPEN in container but CLOSED at PDB level:

```sql
ADMINISTER KEY MANAGEMENT SET KEYSTORE close IDENTIFIED BY abc123 container=all;
ADMINISTER KEY MANAGEMENT SET KEYSTORE open IDENTIFIED BY abc123 container=all;
```

**Alternative:**

```sql
administer key management set keystore close;
```

Reference: [Encryption in Oracle Public Cloud](https://blog.dbi-services.com/encryption-in-oracle-public-cloud/)

### Wallet status

```sql
set linesize 750 pages 9999
col WRL_PARAMETER for a50
select INST_ID,WRL_TYPE,WRL_PARAMETER,STATUS,WALLET_TYPE from gv$encryption_wallet;
```

---

## Auditing

### SP parameter modification in last 7 days

```sql
set linesize 155
col time for a15
col parameter_name format a50
col old_value format a30
col new_value format a30
break on instance skip 3
select instance_number instance, snap_id, time, parameter_name, old_value, new_value from (
select a.snap_id,to_char(end_interval_time,'DD-MON-YY HH24:MI') TIME,  a.instance_number, parameter_name, value new_value,
lag(parameter_name,1) over (partition by parameter_name, a.instance_number order by a.snap_id) old_pname,
lag(value,1) over (partition by parameter_name, a.instance_number  order by a.snap_id) old_value ,
decode(substr(parameter_name,1,2),'__',2,1) calc_flag
from dba_hist_parameter a, dba_Hist_snapshot b , v$instance v
where a.snap_id=b.snap_id
and a.instance_number=b.instance_number
and parameter_name like nvl('&parameter_name',parameter_name)
and a.instance_number like nvl('&instance_number',v.instance_number)
)
where
new_value != old_value
and calc_flag not in (decode('&show_calculated','Y',3,2))
order by 1,2
/
```

### Who locked my account

```sql
set lines 750 pages 9999
column USERNAME format a20
column OS_USERNAME format a20
column USERHOST format a40
column EXTENDED_TIMESTAMP format a60

SELECT USERNAME, OS_USERNAME, USERHOST, EXTENDED_TIMESTAMP,returncode
FROM SYS.DBA_AUDIT_SESSION WHERE returncode != 0 and username = '&Account_Locked'
and EXTENDED_TIMESTAMP > (systimestamp-1) order by 4;
```

Common return codes: ORA-01017 (invalid username/password), ORA-28000 (account is locked)

### User audit info

```sql
select 'standard audit', sessionid,
    proxy_sessionid, statementid, entryid, extended_timestamp, global_uid,
    username, client_id, null, os_username, userhost, os_process, terminal,
    instance_number, owner, obj_name, null, new_owner,
    new_name, action, action_name, audit_option, transactionid, returncode,
    scn, comment_text, sql_bind, sql_text,
    obj_privilege, sys_privilege, admin_option, grantee, priv_used,
    ses_actions, logoff_time, logoff_lread, logoff_pread, logoff_lwrite,
    logoff_dlock, session_cpu
  from dba_audit_trail;
```

### Audited privileges

```sql
SELECT * FROM dba_stmt_audit_opts union SELECT * FROM dba_priv_audit_opts;
```

### User creation time

```sql
select a.* from sys.aud$ a, dba_users b
where a.action# = 51
 and a.OBJ$NAME = b.username;
```

### CREATE/ALTER USER statements

```sql
select * From dba_audit_trail
 where action_name like '%ALTER USER%'
  or action_name like '%CREATE USER%';
```

### Profile creation time

```sql
select * From dba_audit_trail
 where action_name like '%ALTER PROFILE%'
  or action_name like '%CREATE PROFILE%';
```

---

## Table Modification Tracking

### When was the table last accessed

```sql
select INSERTS,UPDATES,DELETES,TABLE_NAME,to_char(TIMESTAMP,'DD-MM-YY HH24:MI') from dba_tab_modifications where table_owner not like '%SYS%' order by TIMESTAMP desc;
```

*Note: Table-level monitoring must be enabled for this query.*

Reference: [When was a table last changed](http://blog.tanelpoder.com/2009/02/07/when-was-a-table-last-changed/)

### When was the password changed for a user

```sql
set lines 750 pages 9999
select USER#,NAME, TO_CHAR(ptime, 'DD-MON-YYYY HH24:MI')  from user$ order by 1;
```

---

## Shell Script Security

### Mask password in shell script

Reference: [Tek-Tips - Mask password](http://www.tek-tips.com/viewthread.cfm?qid=1605767)

Reference: [Linux crypt command](http://www.idevelopment.info/data/Unix/Linux/LINUX_CryptCommand.shtml)

---

## Oracle Database Vault

Reference: [Oracle Database Vault documentation](http://oradb-srv.wlv.ac.uk/E16655_01/server.121/e17608/dvdisabl.htm)

### Check if Database Vault is enabled

```sql
SELECT PARAMETER, VALUE FROM V$OPTION WHERE PARAMETER = 'Oracle Database Vault';
```

### Disable Database Vault

```sql
EXEC DVSYS.DBMS_MACADM.DISABLE_DV;
```

### Enable Database Vault

```sql
EXEC DVSYS.DBMS_MACADM.ENABLE_DV;
```

*Oracle Label Security must be enabled before using Database Vault. Restart required.*

### Enable Oracle Label Security

```sql
SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Oracle Label Security';
```

```sql
EXEC LBACSYS.CONFIGURE_OLS;
EXEC LBACSYS.OLS_ENFORCEMENT.ENABLE_OLS;
```

---

Back to [Main Index](README.md)
