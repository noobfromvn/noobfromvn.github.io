---
layout: post
title: Redo Transport User
subtitle: 
tags: [Oracle, Data Guard]
comments: true
---

> As you know when we create Data Guard Configuration, we must copy password file. Because SYS user is default redo transport user on Data Guard configurations. 
When we change SYS password on primary database, redo transport service is stopping and we are getting error: `ORA-01031: insufficient privileges`. When redo transport failed for this reason we can recreate standby password file with copy from primary side. Same time we can change redo transport user, by setting REDO_TRANSPORT_USER with SYSOPER privileged user.

## SYSTEM INFORMATIONS
```
+---------+-----------------+-----------------+
|         | DC              | DR              |
+---------+-----------------+-----------------+
| Host    | dc-testdb-01/02 | dr-testdb-01/02 |
+---------+-----------------+-----------------+
| DB Name | testdb          | testdr          |
+---------+-----------------+-----------------+
```

## CONFIGURATION STEPS

### 1) Check that user who authorized with SYSDBA or SYSOPER.

```sql
SYS@testdb1 > SELECT * FROM V$PWFILE_USERS;

USERNAME                       SYSDB SYSOP SYSAS
------------------------------ ----- ----- -----
SYS                            TRUE  TRUE  FALSE
```

### 2) Create a user for REDO_TRANSPORT_USER parameter and give the SYSOPER privilege.
***-- Primary database***

Create user:
```sql
SYS@testdb1 > CREATE USER REDO_TRANS IDENTIFIED BY password;

User created.
```

Grant SYSOPER privilege:
```sql
SYS@testdb1 > GRANT SYSOPER TO REDO_TRANS;

Grant succeeded.
```

Check user privilege:
```sql
SYS@testdb1 > SELECT * FROM V$PWFILE_USERS;

USERNAME                       SYSDB SYSOP SYSAS
------------------------------ ----- ----- -----
SYS                            TRUE  TRUE  FALSE
REDO_TRANS                     FALSE TRUE  FALSE

```

### 3) Make log switch to applying redo log that which include create user statement and copy password file to standby database.
**-- Primary database**

Switch logfile:
```sql
ALTER SYSTEM SWITCH ALL LOGFILE;
```

Copy password file to another nodes:
```sh
[oracle@dc-testdb-01 ~]$ scp /u01/app/oracle/product/11.2.0/dbhome_1/dbs/orapwtestdb1 dc-testdb-02:/u01/app/oracle/product/11.2.0/dbhome_1/dbs/orapwtestdb2
[oracle@dc-testdb-01 ~]$ scp /u01/app/oracle/product/11.2.0/dbhome_1/dbs/orapwtestdb1 dr-testdb-01:/u01/app/oracle/product/11.2.0/dbhome_1/dbs/orapwtestdr1
[oracle@dr-testdb-01 ~]$ scp /u01/app/oracle/product/11.2.0/dbhome_1/dbs/orapwtestdr1 dr-testdb-02:/u01/app/oracle/product/11.2.0/dbhome_1/dbs/orapwtestdr2
```

### 4) Set REDO_TRANSPORT_USER parameter both primary and standby database.
***-- Primary database***

```sql
ALTER SYSTEM SET REDO_TRANSPORT_USER=REDO_TRANS scope=both sid='*';
```

***-- Standby database***

```sql
ALTER SYSTEM SET REDO_TRANSPORT_USER=REDO_TRANS scope=both sid='*';
```

### 5) Restart the log apply process

```sql
$ dgmgrl /

EDIT DATABASE TESTDR SET STATE=APPLY-OFF;
EDIT DATABASE TESTDR SET STATE=APPLY-ON;
```

***Done***
