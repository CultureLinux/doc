# MongoDB
## Install 
### Debian 11
```
# apt install gpg wget
# wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | apt-key add -
# echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/5.0 main" | tee /etc/apt/sources.list.d/mongodb-org-5.0.list
# apt update
# apt install mongodb-org
# systemctl enable --now mongod
```

## Authentication
### Create admin first
```
$ mongosh admin
switched to db admin
```
```
db.createUser(
  {
    user: "admin",
    pwd: "admin_pwd",
    roles: [
      { role: "userAdminAnyDatabase", db: "admin" },
      { role: "readWriteAnyDatabase", db: "admin" }
    ]
  }
)
```
### Force auth
```
# systemctl stop mongod
# vi /etc/mongod.conf
```
```
...
security:
  authorization: enabled
...
```
```
# systemctl start mongod
```
### Login 
* failed
```
$ mongosh
test> db.getCollectionNames()
MongoServerError: Command listCollections requires authentication
```
* success
```
$ mongosh mongodb://admin:admin_pwd@localhost:27017/test --authenticationDatabase admin
test> db.getCollectionNames()
[]
```

## Users
### get
```
$ mongosh mongodb://admin:admin_pwd@localhost:27017/test --authenticationDatabase admin
admin> db.getUsers()
{
  users: [
    {
      _id: 'admin.admin',
      userId: new UUID("58625790-8e6b-4433-9e54-777ede29afea"),
      user: 'admin',
      db: 'admin',
      roles: [
        { role: 'readWriteAnyDatabase', db: 'admin' },
        { role: 'userAdminAnyDatabase', db: 'admin' }
      ],
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
    }
  ],
  ok: 1
}
```
### Add role
```
admin> db.grantRolesToUser('admin',['dbAdminAnyDatabase'])
{ ok: 1 }
admin> db.getUsers()
{
  users: [
    {
      _id: 'admin.admin',
      userId: new UUID("58625790-8e6b-4433-9e54-777ede29afea"),
      user: 'admin',
      db: 'admin',
      roles: [
        { role: 'dbAdminAnyDatabase', db: 'admin' },
        { role: 'userAdminAnyDatabase', db: 'admin' },
        { role: 'readWriteAnyDatabase', db: 'admin' }
      ],
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
    }
  ],
  ok: 1
}
```

### Change password
```
admin> db.changeUserPassword("svc_backup","mysqlpwd")
{ ok: 1 }
```

## Connect
### auto connect
vi ~/.mongoshrc.js
```
db = connect("localhost:27017/admin");
db.auth('admin','mysqlpwd')
```

```
mongosh --authenticationDatabase admin
```

### no autoconnect
```
mongosh --authenticationDatabase admin --norc -u otheruser -p
```



## Database
### Create 
```
$ mongosh 
test> use ex_db
switched to db ex_db
ex_db> db.createCollection("posts")
{ ok: 1 }
ex_db> show dbs
admin   116.00 KiB
config  108.00 KiB
ex_db     8.00 KiB
local    72.00 KiB
```
### Drop 
* User must have role : dbAdmin or dbAdminAnyDatabase
```
$ mongosh 
test> use ex_db
switched to db ex_db
ex_db> db.dropDatabase()
{ ok: 1, dropped: 'ex_db' }
ex_db> show dbs
admin   116.00 KiB
config  108.00 KiB
local    72.00 KiB
```

