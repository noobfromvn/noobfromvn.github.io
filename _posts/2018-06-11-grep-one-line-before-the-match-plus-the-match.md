---
layout: post
title: Grep one line before the match plus the match
subtitle: 
tags: [TIL, Linux]
comments: true
---

Mình có một log file của database Oracle có dạng sau:

```sh
Mon Jan 22 08:15:19 2018
Thread 1 advanced to log sequence 137 (LGWR switch)
  Current log# 11 seq# 137 mem# 0: +DATA04/orcl/onlinelog/group_11.302.965663691
  Current log# 11 seq# 137 mem# 1: +FRA/orcl/onlinelog/group_11.3276.965663691
Mon Jan 22 08:15:19 2018
Archived Log entry 450 added for thread 1 sequence 136 ID 0xf0a99e3 dest 1:
Mon Jan 22 08:21:43 2018
Thread 1 advanced to log sequence 138 (LGWR switch)
  Current log# 12 seq# 138 mem# 0: +DATA04/orcl/onlinelog/group_12.300.965663693
  Current log# 12 seq# 138 mem# 1: +FRA/orcl/onlinelog/group_12.3237.965663693
Mon Jan 22 08:21:43 2018
Archived Log entry 454 added for thread 1 sequence 137 ID 0xf0a99e3 dest 1:
Mon Jan 22 08:22:10 2018
Global Enqueue Services Deadlock detected. More info in file
 /u01/app/oracle/diag/rdbms/orcl/orcl1/trace/orcl1_lmd0_49897.trc.
Mon Jan 22 08:28:13 2018
Thread 1 advanced to log sequence 139 (LGWR switch)
  Current log# 13 seq# 139 mem# 0: +DATA04/orcl/onlinelog/group_13.305.965663693
  Current log# 13 seq# 139 mem# 1: +FRA/orcl/onlinelog/group_13.3170.965663693
Mon Jan 22 08:28:13 2018
Expanded controlfile section 11 from 456 to 912 records
Requested to grow by 456 records; added 16 blocks of records
Mon Jan 22 08:28:13 2018
Archived Log entry 458 added for thread 1 sequence 138 ID 0xf0a99e3 dest 1:
```

Mục đích của mình muốn tìm tất cả các thời điểm xảy ra Deadlock trên database, mà đối với một file log hàng trăm ngàn dòng thì không thể kéo xuống đọc theo cách của người trần mắt hột được.

Trong trường hợp này mình cần lấy ra các đoạn log như sau:

```sh
Mon Jan 22 08:22:10 2018
Global Enqueue Services Deadlock detected. More info in file
```

Nhìn qua log file trên, có thể thấy cấu trúc của log file sẽ là:

```
dòng trên: thời điểm xảy ra sự kiện
dòng dưới: thông tin sự kiện log
```

Vậy mình sẽ tìm tất cả các dòng có chữ Deadlock, và lấy ra một dòng trước đó để lấy thời gian xảy ra.

Sử dụng công cụ có sẵn của Linux sẽ giúp ta thực hiện việc này:

```sh
grep -B1 "Deadlock" alert_orcl.log

# Option -B1: Cho phép lấy 1 dòng trước của chuỗi được khớp

# Result
Mon Jan 22 08:22:10 2018
Global Enqueue Services Deadlock detected. More info in file
```

Easy ha :D 
