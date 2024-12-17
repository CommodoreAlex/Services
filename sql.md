Update and patch the server:

```shell
# Updating repositories and upgrading the OS:
sudo apt-get update && sudo apt-get dist-upgrade
```

# The following changes below will occur in `/etc/mysql/my.cnf:

To install MySQL-Server (if required):
```bash
# Installing
sudo apt-get install mysql-server -y
```

1) Secure MySQL Installation:

After installing MySQL, run the `mysql_secure_installation` script to automate some of the common security configurations:
```bash
# Installing
sudo mysql_secure_installation
```

This will prompt you to:
* Set a strong root password (if not configured)
* Remove insecure default settings, like the **anonymous user** and **test database**.
* Disable remote root login.
* Reload privilege tables.

2) Use Strong Passwords

Ensure that all MySQL accounts especially the root account, use strong passwords. 

In the `/etc/my.cnf` or `/etc/mysql/my.cnf` ensure the following is present to enforce a password policy:
```sql
[mysqld]
validate-password-policy=STRONG
validate-passowrd-length=14
```
Source: [MySQL :: MySQL 8.0 Reference Manual :: 8.4.3 The Password Validation Component](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html)

You can adjust this policy to meet your requirements. 

3) Configure MySQL to Listen Only on Localhost:

By default, MySQL may listen on all network interfaces. This can be dangerous if exposed to the internet.

In the `/etc/my.cnf` or `/etc/mysql/my.cnf` files configure MySQL to only listen on 127.0.0.1:
```sql
[mysqld]
bind-address = 127.0.0.1
```

4) Disable Symbolic-links

Disable Symbolic-links to prevent symbolic link attacks (which may be used to access unauthorized files). You can disable this by deferring the configuration file and adding:
```sql
UPDATE
```

5) Limit Max Connections and Connections per Host:

Limit the number of simultaneous connections and set restrictions on the number of connections per host to prevent DoS (Denial of Service) attacks:
```sql
[mysqld]
max_connections = 100
max_conenctions_per_hour = 1000
```

6) Disable LOAD DATA LOCAL INFILE:

Disable the LOAD DATA LOCAL functionality, which can be exploited for remote code execution (RCE):
```sql
[mysqld]
local-infile=0
```

7) Enable and Configure SSL/TLS:

Make sure to generate SSL certificates and set up MySQL to use them. Instructions: [MySQL :: MySQL 8.4 Reference Manual :: 8.3.3.2 Creating SSL Certificates and Keys Using openssl](https://dev.mysql.com/doc/refman/8.4/en/creating-ssl-files-using-openssl.html)

Ensure that all connections to MySQL are encrypted (`/path/to/` means point to the location the certs are in the filesystem):
```sql
[mysqld]
ssl-ca=/path/to/ca-cert.pem
ssl-cert=/path/to/server-cert.pem
ssl-key=/path/to/server-key.pem
require_secure_transport = ON
```

8) Enable General and Error Logging:

Enable error logs and general logs to track suspicious activity:
```sql
[mysqld]
log_error = /var/log/mysql/error.log
general_log = 1
general_log_file = /var/log/mysql/mysql.log
```

Enable binary logging to maintain an audit trail of all changes made to your database:
```sql
[mysqld]
log-bin = /var/log/mysql/mysql-bin
server-id = 1
```

9) Enable MySQL Audit Plugins (Auditing and Logging):

Consider using the MySQL Enterprise Audit plugin or third-party auditing tools (like `audit_log`) to log all SQL statements, user logins, and other sensitive actions:
```sql
ISNTALL PLUGIN audit_log SONAME 'audit_log.so';
```

---
User Privileges Management:
---

You can connect locally to your database using:
```bash
# Notice -p does not present a password first, it will prompt you
mysql -h 127.0.0.1 -u root -p 
```
For granular SQL command guidance: [SQL Cheat Sheet ( Basic to Advanced) - GeeksforGeeks](https://www.geeksforgeeks.org/sql-cheat-sheet/)

1) Use Least Privilege Principle:

Grant users only the least amount of privilege necessary for their tasks. Avoid granting superuser (root) access unless absolutely required.

Only give SELECT privileges for read-only users. Avoid 'GRANT ALL' unless absolutely necessary.
```SQL
# Grant users the least privileges necesasary
GRANT SELECT, INSERT ON database_name.* TO 'user'@'localhost';
```

2) Create Individual User Accounts:

Do not use shared accounts. Each user should have a unique MySQL account for accountability and auditing.

3) Revoke Unused User Accounts:

If there are users who no longer require access, revoke their permissions:
```SQL
REVOKE ALL PRIVILEGES ON *.* FROM 'user'@'localhost';
DROP USER 'user'@'localhost';
```

4) Use MySQL Roles:

MySQL 8.0+ supports roles, which allow you to define specific sets of privielges and assign them to users. This makes privilege management easier and more scalable.

Example of creating a role and assigning it:
```SQL
CREATE ROLE 'read_only';
GRANT SELECT ON database_name.* TO 'read_only';
GRANT 'read_only' TO 'user'@'localhost';
```

Restart the Service to Apply Changes:
--

Configure the service to start on boot:
```bash
sudo systemctl enable mysql
```

Restarting the MySQL Service:
```bash
sudo systemctl restart mysql
```
