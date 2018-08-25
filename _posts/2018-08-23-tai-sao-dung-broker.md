---
layout: post
title: Tại sao lại là Broker?
subtitle: ... Xàm xí về Broker
tags: [Oracle, Data Guard]
comments: true
---

Bạn có đang cấu hình sử dụng Oracle Data Guard Broker? Một số người thường không muốn triển khai Broker vì một vài lý do. Hoặc họ không thấy được lợi ích của Broker mang lại, hoặc họ băn khoăn về sự phức tạp khi phải bổ sung cấu hình, hoặc đơn giản là họ-không-thích. Bài viết này sẽ thảo luận về những lợi ích của Broker và hy vọng mọi người thấy triển khai nó thật đơn giản. 

### Tại sao sử dụng Broker?

Điều hiển nhiên là việc triển khai Data Guard có thể hoàn thành mà chẳng cần đến Broker. Broker không phải là yêu cầu và bạn vẫn có thể sống hạnh phúc mà chẳng cần quan tâm đến nó. Vậy tại sao lại sử dụng nó? Chỉ có một lý do tại sao bạn bạn muốn triển khai Broker, đó là: ***Its makes your life easier.***. Đúng, bạn phải thực hiện cấu hình Broker và mất công quản lý nó, nhưng cuối cùng công việc của bạn trở nên dễ dàng hơn rất nhiều. Dưới đây là một số lợi ích mà DG Broker cung cấp cho bạn:

- Tự động bật tiến trình managed recovery: Bạn không cần phải viết script tự động bật tiến trình managed recovery mỗi khi khởi động lại server, database. Broker giúp bạn làm điều đó. Broker ghi nhớ cấu hình managed recovery lần cuối trước khi DB tắt và tiếp tục bật đồng bộ ở trạng thái đó trong lần khởi động tiếp theo.

- Switchover/Failover chỉ với 1 câu lệnh: Nếu bạn không cấu hình Broker, mỗi lần thực hiện Switchover bạn cần thực hiện rất nhiều câu lệnh. Bạn cần có 1 session SQL\*Plus kết nối vào Primary, một session kết nối vào Standby và thực hiện hàng tá câu lệnh để có thể Switchover. Với Broker, nó trở nên đơn giản chỉ với một câu lệnh SWITCHOVER DATABASE. Một câu lệnh ... là nó đó.

- Tích hợp với Enterprise Manager: Nếu bạn muốn dùng chuột click click để thực hiện việc Switchover thông qua màn hình quản trị GUI kiểu như EM chả hạn, bạn cần Broker. EM không thể thực hiện mấy câu lệnh Switchover bằng SQL, chỉ có thể dùng Broker.

- Đứng một chỗ để cấu hình: Nếu bạn muốn thay đổi cấu hình primary/standby, bật tắt apply, bật tắt transport, bạn có thể đăng nhập vào Broker command line từ primary hoặc standby server và gõ lệnh. Broker có thể xử lý nhiều hệ thống khác nhau, định tuyến câu lệnh đến hệ thống mà câu lệnh của bạn chỉ ra. Broker quản lý nhiều standby database cùng lúc một cách dễ dàng.

- Đứng một chỗ để Monitor: Broker cung cấp công cụ để bạn chỉ cần ngồi im trên một server cũng có thể kiểm tra trạng thái của các database. Tình trạng cấu hình, hiệu năng transport, apply lag,... Dễ ha.

### Cấu hình Broker như thế nào?

[Note cấu hình nhanh Broker cho hệ thống Oracle RAC được cấu hình sẵn Dataguard.](https://datoracle.github.io/2018-06-13-cau-hinh-oracle-data-guard-broker/)

Trên đây là một số lợi ích khi sử dụng Broker, chia sẻ ở bên dưới phần bình luận nếu bạn thấy nó có gì hay, hoặc không hay. Thực ra chính bản thân mình và những người bạn DBA của mình cũng từng gặp những rắc rối to khi sử dụng Broker, nhưng hơn hết Oracle vẫn ngày càng hoàn thiện và khắc phục các bug của nó.

Oracle là một gã khổng lồ, tạm thời thì tôi tin hắn.
