# MariaDB
## Install
    mariadb_secure_install

## Root password 
### Hot change
    mysqladmin -u root -p'OLDPASSWORD' password 'NEWPASSWORD';
### Cold change
    systemctl mariadb stop
    mysqld_safe --skip-grant-tables &
    mysql mysql -u root
    UPDATE user SET password=PASSWORD('NEWPASSWORD') WHERE user="root";
    FLUSH PRIVILEGES;
    exit
    systemctl mariadb start

## User split
    mysqladmin -u root -p create MYDB
    mysql -u root -p mysql
    GRANT ALL ON MYDB.* TO MYUSER@'%' IDENTIFIED BY 'NEWPASSWORD';
    GRANT ALL ON MYDB.* TO MYUSER@localhost IDENTIFIED BY 'NEWPASSWORD';
    FLUSH PRIVILEGES;
    exit
* % : from any ip
* localhost: only from localhost

## Extract permissions
    use mysql
    select distinct concat('SHOW GRANTS FOR ''' , user, '''@''',host,''';') as query from user;

## Tables
### Copy
    CREATE TABLE new_table_name AS
    SELECT * FROM old_table_name;

## Log requests
### Activate
    SET global general_log = 1;
### Destination
#### Table
```
SET global log_output = 'table';
```
```
select * from general_log;
select * from mysql.general_log where command_type = 'Query' and user_host = 'user[user] @ localhost []'
```
#### File
```
SET global log_output = 'file';
SET global general_log_file = '/tmp/mysql_trace.log';
```
### Desactivate
    SET global general_log = 0;

## Backup
### All
    
## Restore

## Upgrade
    # service mariadb stop
    # yum remove MariaDB-*
    # cat /etc/yum.repos.d/MariaDB.repo
    # sed -i 's/11\.1/11\.2/' /etc/yum.repos.d/MariaDB.repo
    # yum install MariaDB-server
    # service mariadb start
    # /usr/bin/mariadb-upgrade -u root -p
