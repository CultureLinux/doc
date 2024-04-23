# Sql Server
## Install (debian12)
### Repository
    # apt install gnupg2 apt-transport-https wget curl
    # wget -q -O- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /usr/share/keyrings/microsoft.gpg > /dev/null 2>&1
    # echo "deb [signed-by=/usr/share/keyrings/microsoft.gpg arch=amd64,armhf,arm64] https://packages.microsoft.com/ubuntu/22.04/mssql-server-2022 jammy main" | sudo tee /etc/apt/sources.list.d/mssql-server-2022.list
### Install
    # apt install mssql-server
### setup
    /opt/mssql/bin/mssql-conf setup
### install cli
    # echo "deb [signed-by=/usr/share/keyrings/microsoft.gpg arch=amd64,armhf,arm64] https://packages.microsoft.com/ubuntu/22.04/prod jammy main" | sudo tee /etc/apt/sources.list.d/prod.list
    # apt update
    # apt install mssql-tools unixodbc-dev
    # ln -s /opt/mssql-tools/bin/sqlcmd /usr/local/bin/sqlcmd


## Usage
### Connect
#### interactive
    $ /opt/mssql-tools/bin/sqlcmd -S 192.168.1.110 -U SA
#### not interactive
* not secure
```
    $ /opt/mssql-tools/bin/sqlcmd -S 192.168.1.110 -U SA -P 'xXxXxxX'
```
* a little better
```
    $ export SQLCMDPASSWORD='xXxXxxX'
    $ /opt/mssql-tools/bin/sqlcmd -S 192.168.1.110 -U SA
```    
#### query
    $ /opt/mssql-tools/bin/sqlcmd -S 192.168.1.110 -U SA -Q "SELECT @@version GO"
#### script
    $ /opt/mssql-tools/bin/sqlcmd -S 192.168.1.110 -U SA -i script-file.sql

## User
### Create
    use master 
    CREATE LOGIN clinux WITH PASSWORD = 'Clinux';

    IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'db_clinux')
    BEGIN
    CREATE DATABASE db_clinux;
    END;
    GO

    use db_clinux
    CREATE USER u_clinux FOR LOGIN clinux;
    ALTER ROLE db_owner ADD MEMBER u_clinux ;
    GO


### Query 
#### Database
* get current
```
    select DB_NAME()
    GO
```    
* get all 
```
    select name from sys.databases
    GO
```    
* create
```
    IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'datalake')
    BEGIN
    CREATE DATABASE datalake;
    END;
    GO
```    
#### Table
* get all
```
    use datalake
    SELECT name,crdate FROM SYSOBJECTS WHERE xtype = 'U';
    GO
```    
* Describe
```
    exec sp_columns MyTable
```    
#### Columns
* add column 
```
    ALTER TABLE MyTable ADD zone varchar(255)
```
* delete column 
```
    ALTER TABLE MyTable DROP COLUMN zone
```

#### insert
##### datetime
```
    INSERT [dbo].[MyTable] ([ID], [DATEMyDatetime]) VALUES (500, convert(datetime, '2022-11-17 00:00:00', 121))
``` 