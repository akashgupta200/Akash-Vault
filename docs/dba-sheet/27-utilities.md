---
layout: default
title: "Utilities"
parent: "DBA Reference Sheet"
nav_order: 27
---

# Utilities

Performance tools, third-party utilities, text editors, and diagnostic collection for Oracle DBAs.

---

## Performance Tools

### Best Oracle performance tools

- [Oracle Real World - Best Oracle Performance Tools](http://www.oraclerealworld.com/best-oracle-performance-tools/)
- [ba6.us - Oracle performance](http://ba6.us/node/177)
- [Windows Podnova - Oracle performance trace](http://windows.podnova.com/trends/oracle_performance_trace.html)

---

## OSWatcher

### OSWatcher GUI

```bash
server123:$ java -jar -Xmx1024m oswg.jar -is /work/oracle/server123_OSW/osw/archive
```

---

## Notepad++

### Add text at end of each line (table list formatting)

**Find with:** `[\r]+`  
**Replace with:** `,`

*Search Mode: Regular expression*

**Alternative:** For all tables in one line use `[\r\n]+`

### Add text at start of line

**Find with:** `^`  
**Replace with:** `'`

### Add text at end of line

**Find with:** `$`  
**Replace with:** `,`

### Column insert (Alt+C)

Place cursor at start of first line, press **Alt+C**, enter text under "Text to Insert", apply OK.

Reference: [Stack Overflow - Add text after every line](https://stackoverflow.com/questions/13806355/add-text-after-every-line-multiple-entries) | [Replace new lines with comma](https://stackoverflow.com/questions/15740853/replace-new-lines-with-a-comma-delimiter-with-notepad)

### Find NON-ASCII characters

**Find with:** `[^\x00-\x7F]+`

*Search Mode: Regular expression*

---

## SQL Monitoring

### asqlmon for visual explain plan in PuTTY

Reference: [Tanel Poder - asqlmon SQL monitoring](http://blog.tanelpoder.com/2013/03/17/asqlmon-sql-sql-monitoring-like-execution-plan-line-level-drilldown-into-sql-response-time/)

---

## PuTTY

### PuTTY autologin script (VBScript)

```vbscript
Dim UserName
Dim Passwrd
ServerName = InputBox("Please Enter Your Servername:")
Passwrd  = InputBox("Please Enter Your RSA TOKEN:")
If ServerName ="" Then
  Wscript.Quit
Else
 Set shell = WScript.CreateObject("WScript.Shell")
 pcmd = "C:\Program Files (x86)\Putty\putty.exe "&Servername & " -pw firstpwd"&Passwrd
Set exec = shell.Exec(pcmd)
Set pout = exec.StdOut
End If
```

---

## SQL*Plus

### Set SQL prompt

```sql
set sqlprompt "_user '@' _connect_identifier >"
```

### Get user input

```sql
accept sid default '' prompt 'Please provide the sid: '
```

### Reset input value

```sql
undefine sid
```

---

## Testing & Diagnostics

### Simulate ORA-600 error

```sql
execute dbms_system.ksdwrt(2,'ORA-600: test');
```

### Burn CPU 100% from database level

```sql
DECLARE
L_n NUMBER;
BEGIN
WHILE (TRUE)
LOOP
L_n:= dbms_random.random();
END LOOP;
END;
/
```

Reference: [DBI Services - Oracle 11g Instance Caging](https://blog.dbi-services.com/oracle-11g-instance-caging-limit-database-cpu-consumption/)

---

## Diagnostic Collection

### Oracle diag collection

```bash
$GI_HOME/bin/diagcollection.pl --collect --chmos --incidenttime 11/04/201702:00:00 --incidentduration 02:00
```

### Root TFA diag collection

```bash
tfactl diagcollect
```

For specific date:

```bash
tfactl diagcollect -for "yyyy-mm-dd"
```

For specific time range:

```bash
tfactl diagcollect -from "yyyy-mm-dd hh:mm:ss" -to "yyyy-mm-dd hh:mm:ss"
```

**Alternative:**

```bash
$TFA_HOME/bin/tfactl diagcollect -srdc dbperf
```

Reference: [TFA Collector - TFA with Database Support Tools Bundle (Doc ID 1513912.1)](https://support.oracle.com)

### TFA collection for database

Run as root:

```bash
./tfactl diagcollect -database mydb -from "Nov/16/2017 21:00:00" -to "Nov/17/2017 02:00:00"
```

---

Back to [Main Index](README.md)
