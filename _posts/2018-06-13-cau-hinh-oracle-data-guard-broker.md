---
layout: post
title: Cấu hình Oracle Data Guard Broker
subtitle: 
tags: [Oracle, Data Guard]
comments: true
---

# MỤC TIÊU 
Note này thực hiện cấu hình DG Broker cho hệ thống RAC to RAC Data Guard đã được cấu hình sẵn.

```
PRIMARY db_unique_name : testdb
STANDBY db_unique_name : testdr
```

# CÁCH THỰC HIỆN

### Thiết lập tham số vị trí lưu file cấu hình của Broker

Đăng nhập SQL\*Plus với quyền SYSDBA trên cả Primary và Physical Standby database:

```
export ORACLE_SID=testdb1
sqlplus / as sysdba
```

#### Trên Primary database:

```sql
ALTER SYSTEM SET DG_BROKER_CONFIG_FILE1='+DATA03/TESTDB/DG_TESTDB_CONFIG1.DAT' SID='*';
ALTER SYSTEM SET DG_BROKER_CONFIG_FILE2='+DATA03/TESTDB/DG_TESTDB_CONFIG2.DAT' SID='*';
```

#### Trên Physical Standby database

```sql
ALTER SYSTEM SET DG_BROKER_CONFIG_FILE1='+DATA03/TESTDR/DG_TESTDR_CONFIG1.DAT' SID='*';
ALTER SYSTEM SET DG_BROKER_CONFIG_FILE2='+DATA03/TESTDR/DG_TESTDR_CONFIG2.DAT' SID='*';
```

### Dừng tiến trình đồng bộ trên Physical Standby database

```sql
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
```

### Thực hiện set tham số DG_BROKER_START sang TRUE

```sql
ALTER SYSTEM SET DG_BROKER_START=TRUE SID='*';
```

### Tạo cấu hình Broker

Đứng trên một node của Primary, đăng nhập DGMGRL:

```
export ORACLE_SID=testdb1
dgmgrl /
```

Tạo cấu hình Broker:

```sql
CREATE CONFIGURATION TESTDB_TESTDR_CONFIG AS
PRIMARY DATABASE IS TESTDB
CONNECT IDENTIFIER IS 'TESTDB'; -- TESTDB là chuỗi kết nối được khai báo trong file $ORACLE_HOME/network/admin/tnsnames.ora
```

### Add Physical Standby database

```sql
ADD DATABASE TESTDR AS
CONNECT IDENTIFIER IS 'TESTDR'; -- TESTDR là chuỗi kết nối được khai báo trong file $ORACLE_HOME/network/admin/tnsnames.ora
```

### Enable cấu hình

```sql
ENABLE CONFIGURATION;
```

# SWITCHOVER và FAILOVER

### Switchover
```sql
SWITCHOVER TO TESTDR;
```

### Failover
```sql
FAILOVER TO TESTDR;
```

# MỘT SỐ LỆNH VẬN HÀNH KHÁC


### Disable cấu hình

```
DISABLE CONFIGURATION;
```

### Kiểm tra cấu hình Broker

```
DGMGRL> SHOW CONFIGURATION

Configuration - testdb_testdr_config

  Protection Mode: MaxPerformance
  Databases:
    testdb - Primary database
    testdr - Physical standby database

Fast-Start Failover: DISABLED

Configuration Status:
SUCCESS


-- or

SHOW CONFIGURATION VERBOSE

```

### Kiểm tra database

```
DGMGRL> SHOW DATABASE TESTDB

Database - testdb

  Role:            PRIMARY
  Intended State:  TRANSPORT-ON
  Instance(s):
    testdb1
    testdb2

Database Status:
SUCCESS

DGMGRL> SHOW DATABASE TESTDR

Database - testdr

  Role:            PHYSICAL STANDBY
  Intended State:  APPLY-ON
  Transport Lag:   0 seconds (computed 1 second ago)
  Apply Lag:       0 seconds (computed 1 second ago)
  Apply Rate:      1.12 MByte/s
  Real Time Query: OFF
  Instance(s):
    testdr1 (apply instance)
    testdr2

Database Status:
SUCCESS

-- or

SHOW DATABASE VERBOSE TESTDB
SHOW DATABASE VERBOSE TESTDR

```

### Bật tắt TRANSPORT trên Primary

```
EDIT DATABASE TESTDB SET STATE=TRANSPORT-OFF;

EDIT DATABASE TESTDB SET STATE=TRANSPORT-ON;
```

### Bật tắt APPLY trên Physical Standby

```
EDIT DATABASE TESTDR SET STATE=APPLY-OFF;

EDIT DATABASE TESTDR SET STATE=APPLY-ON;
```

### Thiết lập PROPERTY

```
EDIT DATABASE TESTDR SET PROPERTY DelayMins=240;
```

### Thiết lập Protection Mode

```
EDIT CONFIGURATION SET PROTECTION MODE AS MAXAVAILABILITY;
```

### Silent mode


```
dgmgrl -silent / "show database 'testdr'"
dgmgrl -silent / "show database 'testdb'"
```

# Mọi người cùng chia sẻ kinh nghiệm DG Broker bên dưới phần bình luận nhé!
