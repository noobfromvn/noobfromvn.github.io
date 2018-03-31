---
layout: post
title: Phân vùng bảng, index trong Oracle Database
tags: [Oracle]
comments: true
---

Phân vùng giải quyết vấn đề quan trọng trong việc hỗ trợ các bảng rất lớn bằng cách phân chia chúng thành các phần nhỏ để dễ quản lý. Các câu truy vấn SQL và câu lệnh DML không cần phải chỉnh sửa để có thể truy xuất các phân vùn bảng. Tuy nhiên sau khi phân vùng được định nghĩa, các câu lệnh DDL có thể thao tác trực tiếp tới từng phân vùng chứ không cần tới toàn bộ các tables hoặc indexes. Phân vùng là cách đơn giản hoá việc quản lý các cơ sở dữ liệu lớn. Phân vùng hoàn toàn trong suốt với lớp ứng dụng.

# Lợi thế của việc phân vùng:
- Phân vùng cho phép các hoạt động quản lý cơ sở dữ liệu như tải, tạo index và xây dựng lại, sao lưu/phục hồi ở mức phân vùng chứ không cần thao tác trên toàn bộ bảng. Điều này giúp giảm đáng kể thời gian cho những hoạt động đó.
- Phân vùng cải thiện hiệu suất truy vấn. Trong nhiều trường hợp, kết quả của truy vấn có thể có được bằng cách truy cập vào một tập hợp con của các phân vùng chứ không phải toàn bộ bảng. Đối với một số truy vấn kỹ thuật này được gọi là phân vùng tỉa (partition pruning), có thể tăng hiệu suất hoạt động.
- Phân vùng làm giảm đáng kể các ảnh hưởng bởi thời gian bảo trì.
- Phân vùng độc lập cho các hoạt động bảo trì phân vùng, cho phép đồng thời bảo trì trên nhiều phân vùng bảng hoặc chỉ mục khác nhau. Có thể chạy đồng thời các hành động SELECT, DML đối với phân vùng mà không ảnh hưởng đến quá trình bảo trì.
- Phân vùng có thể thực hiện mà không cần sự thay đổi nào trong ứng dụng, không cần viết lại ứng dụng.
 ![](/img/table_partition_h1.jpg)
_Hình 1. Phân vùng trong bảng._

# Khoá phân vùng (Partition Key)
Mỗi hàng thuộc bảng đã được phân vùng được gán rõ ràng cho một phân vùng duy nhất. Khoá phân vùng là tập hợp một hoặc nhiều cột xác định phân vùng cho mỗi hàng. Oracle trực tiếp thực thi các câu lệnh INSERT, UPDATE, DELETE vào các phân vùng thích hợp thông qua khoá phân vùng.

**Một khoá phân vùng chứa:**
- Danh sách có thứ tự các cột từ 1 - 16.
- Không thể chứa một LEVEL, ROWID, MLSLABEL, pseudocolumn hoặc một cột kiểu ROWID.
- Không thể chứa cột NULL.

# Phân vùng bảng
- Bảng có thể chia thành 64.000 phân vùng riêng biệt.
- Tất cả các bảng đều có thể được phân vùng trừ các bảng có chứa các cột có kiểu LONG hoặc RAW LONG. Tuy nhiên có thể sử dụng các bảng có kiểu CLOB, BLOB.

# Phương pháp phân vùng
- Phân vùng theo phạm vi (Range Partition)
- Phân vùng theo danh sách (List Partition)
- Phân vùng Hash
- Phân vùng tổng hợp
 ![](/img/table_partition_h2.jpg)
_Hình 2. Phân vùng Range, List, Hash_
 ![](/img/table_partition_h3.jpg)
_Hình 3. Phân vùng tổng hợp_

## Phân vùng phạm vi (Range Partition)
Phân vùng phạm vi dữ liệu dựa trên phạm vi của khoá phân vùng mà ta thiết lập. Đây là loại phân vùng phổ biết sử dụng hàng ngày. Ví dụ, có thể phân vùng dữ liệu bán hàng theo từng tháng.
Khi sử dụng phân vùng cần xem xét các nguyên tắc sau:
- Mỗi phân vùng đều có một mệnh đề GIÁ TRỊ NHỎ HƠN, không bao gồm cận trên. Bất kỳ giá trị nhị phân nào của khoá phân vùng lớn hơn hoặc bằng nó đều thuộc phân vùng tiếp theo cao hơn.
- Một giá trị MAXVALUES (lớn nhất theo nghĩa đen) được định nghĩa cho phân vùng cao nhất. MAXVALUES định nghĩa cho một giá trị ảo vô hạn, lớn hơn bất kỳ giá trị khoá phân vùng nào khác, bao gồm cả NULL.
Ví dụ tạo một bảng `sale_range` được phân vùng phạm vi theo trường `sale_date`:

```sql
CREATE TABLE sales_range 
(salesman_id NUMBER(5), 
salesman_name VARCHAR2(30), 
sales_amount NUMBER(10), 
sales_date DATE)
PARTITION BY RANGE(sales_date) 
(
PARTITION sales_jan2000 VALUES LESS THAN(TO_DATE('02/01/2000','DD/MM/YYYY')),
PARTITION sales_feb2000 VALUES LESS THAN(TO_DATE('03/01/2000','DD/MM/YYYY')),
PARTITION sales_mar2000 VALUES LESS THAN(TO_DATE('04/01/2000','DD/MM/YYYY')),
PARTITION sales_apr2000 VALUES LESS THAN(TO_DATE('05/01/2000','DD/MM/YYYY'))
);
```

## Phân vùng danh sách (List Partition)
Phân vùng theo danh sách cho phép chỉ rõ hàng nào thuộc phân vùng nào. Khác với phân vùng theo phạm vi, phân vùng theo danh sách chỉ rõ giá trị cho khoá phân vùng.
Ưu điểm của phân vùng danh sách là có thể nhóm và tổ chức các dữ liệu chưa được sắp xếp hoặc dữ liệu không liên quan một cách tự nhiên.

Ví dụ sau, phân vùng bảng bán hàng theo khu vực:
```sql
CREATE TABLE sales_list
(salesman_id NUMBER(5), 
salesman_name VARCHAR2(30),
sales_state VARCHAR2(20),
sales_amount NUMBER(10), 
sales_date DATE)
PARTITION BY LIST(sales_state)
(
PARTITION sales_west VALUES('California', 'Hawaii'),
PARTITION sales_east VALUES ('New York', 'Virginia', 'Florida'),
PARTITION sales_central VALUES('Texas', 'Illinois')
PARTITION sales_other VALUES(DEFAULT)
);
```
## Phân vùng băm (Hash Parition)
Phân vùng băm dễ dàng cho phép phân vùng dữ liệu mà không phù hợp với phân vùng phạm vi và phân vùng danh sách. Phân vùng băm dễ dàng thực hiện với cú pháp đơn giản. Phân vùng băm tốt hơn sơ với phân vùng phạm vi khi:
- Không biết trước có bao nhiêu dữ liệu trong một phạm vi
- Các phạm vi của phân vùng phạm vi khác nhau, khó tính toán bằng tay
- Phân vùng phạm vi có thể nhóm những dữ liệu không mong muốn
- Các tính năng hiệu năng như DML song song, phân vùng tỉa (partition pruning), gộp phân vùng thận trọng (partition-wise joins) là quan trọng.

Các khái nhiệm tách, gỡ, gộp không áp dụng cho phân vùng băm. Thay vào đó phân vùng băm có thể được bổ sung và kết hợp lại.

_Ví dụ về phân vùng băm:_
```sql
CREATE TABLE sales_hash
(salesman_id NUMBER(5),
salesman_name VARCHAR2(30),
sales_amount NUMBER(10),
week_no NUMBER(2))
PARTITION BY HASH(salesman_id)
PARTITIONS 4
STORE IN (data1, data2, data3, data4);
```

## Phân vùng tổng hợp (Composite Partition)
Phân vùng tổng hợp phân chia dữ liệu bằng cách sử dụng phân vùng phạm vi, trong mỗi phân vùng, phân vùng phụ (subpartition) sử dụng phương pháp phân vùng danh sách hoặc phân vùng băm. Phân vùng tổng hợp phạm vi-băm cải tiến khả năng của phân vùng phạm vi và khả năng phân chia, định vị dữ liệu của phân vùng băm. Phân vùng tổng hợp phạm vi-danh sách cải tiếng khả năng của phân vùng phạm vi và cơ chế điều khiển rõ ràng của phân vùng danh sách.
Phân vùng tổng hợp hỗ trợ các hành động như thêm phân vùng phạm vi mới, nhưng cũng cung cấp mức độ cao hơn như DML song song, chi tiết hơn về vị trí dữ liệu trong phân vùng phụ.

_Ví dụ tạo phân vùng tổng hợp phạm vi-băm:_
```sql
CREATE TABLE sales_composite 
(salesman_id NUMBER(5), 
salesman_name VARCHAR2(30), 
sales_amount NUMBER(10), 
sales_date DATE)
PARTITION BY RANGE(sales_date) 
SUBPARTITION BY HASH(salesman_id)
SUBPARTITION TEMPLATE(
SUBPARTITION sp1 TABLESPACE data1,
SUBPARTITION sp2 TABLESPACE data2,
SUBPARTITION sp3 TABLESPACE data3,
SUBPARTITION sp4 TABLESPACE data4)
(PARTITION sales_jan2000 VALUES LESS THAN(TO_DATE('02/01/2000','DD/MM/YYYY'))
PARTITION sales_feb2000 VALUES LESS THAN(TO_DATE('03/01/2000','DD/MM/YYYY'))
PARTITION sales_mar2000 VALUES LESS THAN(TO_DATE('04/01/2000','DD/MM/YYYY'))
PARTITION sales_apr2000 VALUES LESS THAN(TO_DATE('05/01/2000','DD/MM/YYYY'))
PARTITION sales_may2000 VALUES LESS THAN(TO_DATE('06/01/2000','DD/MM/YYYY')));
```
còn tiếp ...
