---
layout: post
title: Đổi tên nhiều file cùng một lúc dùng Regex
subtitle: 
tags: [TIL]
comments: true
bigimg: /img/xkcd-regex.jpg
---

Giả sử có một folder chứa 3 file có tên na ná nhau nhưng khác phần đuôi và khác luôn kiểu file như sau:
```sh
├── squared-checkbox-directive.js
├── squared-checkbox-style.scss
└── squared-checkbox-template.html
```
Giờ chúng ta muốn đổi tên cả 3 file này thành dạng circle-checkbox-*.*, thì không thể nào dùng lệnh mv được (nếu ai làm được thì bày mình với :D)
Có thể dùng lệnh rename để đổi:
```sh
rename 's/squared/circle/' *.*
```
Kết quả thu được:
```sh
├── circle-checkbox-directive.js
├── circle-checkbox-style.scss
└── circle-checkbox-template.html
```
Lệnh này có sẵn trên môi trường Linux, còn trên Mac thì phải cài thêm thông qua brew:
```sh
brew install rename
```
