---
layout: post
title: Lấy thông tin các host truy cập vào database thông qua listener log
tags: [Oracle, Regex, Linux]
comments: true
---


---
layout: post
title: Lấy thông tin các host truy cập vào database thông qua listener log
tags: [Oracle, Regex, Linux]
comments: true
---

Hôm nay mình lại chia sẻ về chủ đề lấy thông tin trong một file log rất rất lớn, lần trước mình có chia sẻ về việc cắt nội dung log theo khoảng thời gian, các bạn có thể xem [tại đây](https://datoracle.github.io/2017-06-17-lay-du-lieu-giua-hai-khoang-thoi-gian-file-log/).

OK giờ thì vào chủ đề hôm nay thôi!
# Bài toán
Mình có file listener log của một hệ thống khách hàng, bây giờ mình cần lấy thông tin tất cả các host đã truy cập vào hệ thống. File log của mình khoảng 17 triệu dòng (~3GB).

- Log có dạng như sau _(tượng trưng vài cái thôi chứ nó nhiều kinh khủng o_o )_
```sh
$ cat listener.log #(tên hostname đã được thay đổi)
...
... dài vcđ luôn ...
...
17-AUG-2017 14:37:11 * (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=PCPACK)(CID=(PROGRAM=d:\oracle\product\11.2.0\dbhome_1\bin\ORACLE.EXE)(HOST=HOSTNAME1)(USER=SYSTEM))) * (ADDRESS=(PROTOCOL=tcp)(HOST=10.250.160.89)(PORT=31097)) * establish * PCPACK * 0
17-AUG-2017 14:37:15 * (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=PCPACK)(CID=(PROGRAM=d:\oracle\product\11.2.0\dbhome_1\bin\ORACLE.EXE)(HOST=HOSTNAME2)(USER=SYSTEM))) * (ADDRESS=(PROTOCOL=tcp)(HOST=10.250.160.40)(PORT=51444)) * establish * PCPACK * 0
17-AUG-2017 14:37:16 * (CONNECT_DATA=(CID=(PROGRAM=)(HOST=HOSTNAME3)(USER=SYSTEM$))(SERVICE_NAME=PCPACK)) * (ADDRESS=(PROTOCOL=tcp)(HOST=10.250.160.31)(PORT=59307)) * establish * PCPACK * 0
17-AUG-2017 14:38:05 * (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=PCPACK)(CID=(PROGRAM=d:\oracle\product\11.2.0\dbhome_1\bin\ORACLE.EXE)(HOST=HOSTNAME4)(USER=SYSTEM))) * (ADDRESS=(PROTOCOL=tcp)(HOST=10.250.160.89)(PORT=31322)) * establish * PCPACK * 0
...
```

Nếu lấy một cách thủ công, mình sẽ cần nhặt ra các hostname ở giữa chuỗi `"HOST"` và `"USER"`, với log sinh ra mỗi giây vài dòng và hostname trùng nhau như này quả là rất khó khăn để lấy thông tin.

# Giải quyết
Mình sử dụng grep và regex. Mình cần làm 2 việc:

1. Lấy tất cả hostname trong file log sau đó đẩy ra một file mới có tên `listener_host.log`
```sh
$ cat listener.log | grep -oP 'HOST=\K.*?(?=\)\(USER)' >> listener_host.log
```
**Grep option:**

`-o : Print only the matched (non-empty) parts of a matching line, with each such part on a separate output line.`

`-P : Interpret PATTERN as a Perl regular expression.`


- Giải thích Regex tý: Regular Expression (biểu thức chính quy - hay Regex) nôm na là một chuỗi ký tự đặc biệt dùng để làm mẫu, giúp tìm kiếm các chuỗi trùng khớp với mẫu đó trong nội dung văn bản nào đó. Cái này mọi người có thể lên Gu gồ tìm hiểu rất hữu ích.
- Đoạn regex của mình lấy thông tin giữa hai chuỗi `"HOST="` và `")(USER"`

2. Khi đã có được một mớ hỗn độn các hostname xếp lần lượt theo từng dòng, trong đó có rất nhiều tên trùng nhau thì giờ mình chỉ việc lấy ra những giá trị duy nhất và đẩy ra một file mới `listener_host_uniq.log`.
```sh
cat listener_host.log | sort | uniq >> listener_host_uniq.log
```
`sort - sort lines of text files`

`uniq - report or filter out repeated lines in a file`


Các bạn có thể kết hợp 2 câu lệnh này thành một câu lệnh, còn mình thì thích làm 2 lần cơ :D

Xin chào!
