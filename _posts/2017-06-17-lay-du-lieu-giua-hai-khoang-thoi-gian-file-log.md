---
layout: post
title: Lấy dữ liệu giữa hai khoảng thời gian từ file log
tags: [Oracle, Linux]
comments: true
---

Giả sử có file log rất rất rất dài như bên dưới:

```sh
...
Mon Dec 19 15:14:25 2016
Completed checkpoint up to RBA [0xed8.2.10], SCN: 49559582207
Mon Dec 19 15:20:50 2016
Incremental checkpoint up to RBA [0xed8.475dd.0], current log tail at RBA [0xed8.e56c1.0]
Mon Dec 19 15:36:32 2016
Beginning log switch checkpoint up to RBA [0xed9.2.10], SCN: 49561019703
Thread 2 advanced to log sequence 3801 (LGWR switch)
Current log# 5 seq# 3801 mem# 0: +DATA/pe2/onlinelog/group_5.1009.925254353
Current log# 5 seq# 3801 mem# 1: +DATA/pe2/onlinelog/group_5.1001.925254505
Mon Dec 19 15:36:40 2016
Archived Log entry 362384 added for thread 2 sequence 3800 ID 0xf2b2824a dest 2:
Archived Log entry 362385 added for thread 2 sequence 3800 ID 0xf2b2824a dest 10:
Mon Dec 19 15:41:50 2016
Completed checkpoint up to RBA [0xed9.2.10], SCN: 49561019703
Mon Dec 19 15:51:00 2016
Incremental checkpoint up to RBA [0xed9.6dd91.0], current log tail at RBA [0xed9.96f20.0]
Mon Dec 19 16:04:21 2016
Beginning log switch checkpoint up to RBA [0xeda.2.10], SCN: 49562407178
Thread 2 advanced to log sequence 3802 (LGWR switch)
Current log# 6 seq# 3802 mem# 0: +DATA/pe2/onlinelog/group_6.1005.925254357
Current log# 6 seq# 3802 mem# 1: +DATA/pe2/onlinelog/group_6.2199.925254509
Mon Dec 19 16:04:30 2016
Archived Log entry 362388 added for thread 2 sequence 3801 ID 0xf2b2824a dest 2:
Archived Log entry 362389 added for thread 2 sequence 3801 ID 0xf2b2824a dest 10:
Mon Dec 19 16:09:54 2016
Completed checkpoint up to RBA [0xeda.2.10], SCN: 49562407178
...
```

Bây giờ ta cần lấy log trong 1 khoảng thời gian để kiểm tra. Giả sử cần lấy log từ `Mon Dec 19 15:36:40 2016` đến `Mon Dec 19 16:04:30 2016`:
```sh
sed -n "/Mon Dec 19 15:36:40 2016/,/Mon Dec 19 16:04:30 2016/p" file_name > new_alert.log
```

Đơn giản phải không nào :D
