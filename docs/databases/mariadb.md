# MariaDB
This section covers various commands and procedures for managing MariaDB.

## Install
Installs MariaDB and runs the secure installation script.
```sh
mariadb_secure_install
```

## Root password
This subsection provides methods for changing the root password in MariaDB.

### Hot change
Changes the root password without restarting the MariaDB service.
```sh
mysqladmin -u root -p'OLDPASSWORD' password 'NEWPASSWORD';
```

### Cold change
Changes the root password after stopping and starting the MariaDB service.
```sh
systemctl mariadb stop
mysqld_safe --skip-grant-tables &
mysql mysql -u root
UPDATE user SET password=PASSWORD('NEWPASSWORD') WHERE user="root";
FLUSH PRIVILEGES;
exit
systemctl mariadb start
```

## User split
Creates a new database and grants privileges to a user.
```sh
mysqladmin -u root -p create MYDB
mysql -u root -p mysql
GRANT ALL ON MYDB.* TO MYUSER@'%' IDENTIFIED BY 'NEWPASSWORD';
GRANT ALL ON MYDB.* TO MYUSER@localhost IDENTIFIED BY 'NEWPASSWORD';
FLUSH PRIVILEGES;
exit
```
* % : from any ip
* localhost: only from localhost

## Extract permissions
Displays the SQL commands needed to recreate the grants for each user.
```sh
use mysql
select distinct concat('SHOW GRANTS FOR ''' , user, '''@''',host,''';') as query from user;
```

## Tables
This subsection provides commands related to database tables.

### Copy
Creates a new table by copying an existing one.
```sh
CREATE TABLE new_table_name AS
SELECT * FROM old_table_name;
```

## Log requests
This subsection explains how to log MariaDB requests.

### Activate
Enables the general query log.
```sh
SET global general_log = 1;
```

### Destination
Specifies where the general query log should be stored.

#### Table
```sh
SET global log_output = 'table';
```
```sh
select * from general_log;
select * from mysql.general_log where command_type = 'Query' and user_host = 'user[user] @ localhost []'
```

#### File
```sh
SET global log_output = 'file';
SET global general_log_file = '/tmp/mysql_trace.log';
```

### Deactivate
Disables the general query log.
```sh
SET global general_log = 0;
```

## Upgrade
This subsection explains how to upgrade MariaDB.

```sh
# service mariadb stop
# yum remove MariaDB-*
# cat /etc/yum.repos.d/MariaDB.repo
# sed -i 's/11\.1/11\.2/' /etc/yum.repos.d/MariaDB.repo
# yum install MariaDB-server
# service mariadb start
# /usr/bin/mariadb-upgrade -u root -p
```
