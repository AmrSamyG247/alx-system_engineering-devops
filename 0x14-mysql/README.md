##0x14-mysql ALX SE program tasks By Amr Samy

## How I go with tasks ?

## Installation Guide for mysql 5.7.*

- First go to site [dev.mysql.com](https://dev.mysql.com/doc/refman/5.7/en/checking-gpg-signature.html) and copy the PGP PUBLIC KEY just immediately under the _Notice_ section to your clipboard.

- Create a new file in your terminal with a .key extension and paste the PGP PUB KEY copied to clipboard.
- Then do the following

```bash
$ sudo apt-key add name_of_file.key
OK

# adding it to the apt repo
$ sudo sh -c 'echo "deb http://repo.mysql.com/apt/ubuntu bionic mysql-5.7" >> /etc/apt/sources.list.d/mysql.list'

# updating the apt repo to add the url i added earlier
$ sudo apt-get update

# now check your available versions
$ sudo apt-cache policy mysql-server
mysql-server:
  Installed: (none)
  Candidate: 8.0.31-0ubuntu0.20.04.2
  Version table:
     8.0.31-0ubuntu0.20.04.2 500
        500 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages
     8.0.31-0ubuntu0.20.04.1 500
        500 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages
     8.0.19-0ubuntu5 500
        500 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/main amd64 Packages
     5.7.40-1ubuntu18.04 500
        500 http://repo.mysql.com/apt/ubuntu bionic/mysql-5.7 amd64 Packages

# Now am installing mysql 5.7.*
$ sudo apt-get install -f mysql-client=5.7* mysql-community-server=5.7* mysql-server=5.7* -y
```
## Project Task

### Creating a user and Granting Priviledges in mysql
```mysql
$ mysql -uroot -p
Password:	/* Type root password

mysql> CREATE USER 'holberton_user'@'localhost' IDENTIFIED BY 'projectcorrection280hbtn';

mysql> GRANT REPLICATION CLIENT ON *.* TO 'holberton_user'@'localhost';

mysql> FLUSH PRIVILEGES;
```

### Creating Database, Tables and adding Data to the Tables

```mysql

mysql> CREATE DATABASE db_name_;

-- To verify if db is created
mysql> SHOW DATABASES;

mysql> USE db_name;

mysql> CREATE TABLE nexus6 (
    -> col_1 VARCHAR(50) NOT NULL,
    -> col_2 VARCHAR(50) NOT NULL);
-- continue adding more coloums to your taste for me i just added two coloumns

mysql> INSERT INTO nexus6(col_1, col_2) VALUES ('val_1', 'val_2');
 Query OK, 1 row affected (0.01 sec)
mysql> INSERT INTO nexus6(col_1, col_2) VALUES ('Amr', 'Samy');
 Query OK, 1 row affected (0.00 sec)
 
-- Verify if data was added succesfully do
mysql> select * from nexus6;
 +-------+-------+
| col_1 | col_2 |
+-------+-------+
| val_1 | val_2 |
| Amr   | Samy  |
+-------+-------+
2 rows in set (0.00 sec)

-- Grant user select access on DB
mysql> GRANT ALL PRIVILEGES ON *.*  TO 'holberton_user'@'localhost';
Query OK, 0 rows affected (0.04 sec)

### Setting Up MySQL Replication

- First create replication user and grant replication priviledge ( best practice).

```mysql

mysql> CREATE USER 'replica_user'@'%' IDENTIFIED BY 'replica_user_pwd';

mysql> GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';

mysql> FLUSH PRIVILEGES;

-- to verify
mysql> SELECT user, Repl_slave_priv FROM mysql.user;

mysql> exit
```
- Next up you go to the /etc/mysql/mysql.conf.d/mysqld.cnf and comment the bind address and then add this lines to it

```bash
# By default we only accept connections from localhost
# bind-address = 127.0.0.1
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = db_name
```
- Then you enable incoming connection to port 3306 and restart mysql-server
```bash

$ sudo ufw allow from 'replica_server_ip/web-02' to any port 3306

$ sudo service mysql restart
```
- Now log back in to mysql-server to lock db and prepare binary file for replication.

```bash
$ mysql -uroot -p
password:
```
```mysql
mysql> 

mysql> FLUSH TABLES WITH READ LOCK;

mysql> SHOW MASTER STATUS;

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      149 | db           |                  |                   |
+------------------+----------+--------------+------------------+-------------------+

```
_Take note of the binary log and the position, jot it down or you leave this window open and you open another window to continue_

- you then export the db from myql-server to local machine and then copy this db to replica machine

```bash
$ mysqldump -uroot -p db_name > export_db_name.sql

$ scp -i _idenetity_file_ export_db_name.sql user@machine_ip:location
```
- Then ssh to replica machine ip_adress to import this tables to replica mysql-server

```bash

$ mysql -uroot -p 
password:


mysql> CREATE DATABASE db_name;

mysql>exit
bye

$ mysql -uroot p db_name < export_db_name.sql
password:

# Now edit the config file in /etc/mysql/mysql.conf.d/mysqld.cnf and then reload mysql-server

```bash

server-id = 2
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = db_name_from_master_mysql-server
relay_log = /var/log/mysql/mysql-relay-bin.log

$ sudo service mysql restart
```

- Login to mysql server in replica to configure replication

```bash

$ mysql -uroot -p
password:


mysql>
```
```mysql

mysql> CHANGE MASTER TO
    -> MASTER_HOST='source_host_name',
    -> MASTER_USER='replication_user_name',
    -> MASTER_PASSWORD='replication_password',
    -> MASTER_LOG_FILE='recorded_log_file_name',
    -> MASTER_LOG_POS=recorded_log_position;

-- Then you start slave
mysql> START SLAVE;
```
__Done__
