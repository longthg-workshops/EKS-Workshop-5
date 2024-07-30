---
title: "Amazon GuardDuty for EKS"
date: "`r Sys.Date()`"
weight: 6
chapter: false
pre: "<b> 6. </b>"
---

#### Amazon GuardDuty for EKS

Chuẩn bị môi trường của bạn cho phần này:

```bash timeout=300 wait=30
$ prepare-environment
```

Amazon GuardDuty cung cấp các tính năng phát hiện mối đe dọa cho phép bạn liên tục giám sát và bảo vệ các tài khoản AWS, khối lượng công việc và dữ liệu được lưu trữ trong Amazon Simple Storage Service (Amazon S3). GuardDuty phân tích các luồng siêu dữ liệu liên tục được tạo ra từ hoạt động của tài khoản và mạng trong AWS CloudTrail Events, Amazon Virtual Private Cloud (VPC) Flow Logs, và các bản ghi DNS (Domain Name System). GuardDuty cũng sử dụng thông tin tình báo mối đe dọa tích hợp như các địa chỉ IP độc hại đã biết, phát hiện bất thường và học máy (ML) để xác định mối đe dọa một cách chính xác hơn.

Amazon GuardDuty giúp bạn dễ dàng giám sát liên tục các tài khoản AWS, khối lượng công việc và dữ liệu được lưu trữ trong Amazon S3. GuardDuty hoạt động hoàn toàn độc lập với các tài nguyên của bạn, vì vậy không có rủi ro về hiệu suất hoặc ảnh hưởng đến sẵn có đối với khối lượng công việc của bạn. Dịch vụ được quản lý hoàn toàn và được tích hợp thông tin tình báo mối đe dọa, phát hiện bất thường và ML. Amazon GuardDuty cung cấp các cảnh báo chi tiết và hữu ích dễ tích hợp với các hệ thống quản lý sự kiện và luồng công việc hiện có. Không có chi phí ban đầu và bạn chỉ trả tiền cho các sự kiện được phân tích, không cần triển khai phần mềm bổ sung hoặc đăng ký dịch vụ thông tin tình báo mối đe dọa.

GuardDuty có hai loại bảo vệ cho EKS:
1. EKS Audit Log Monitoring giúp bạn phát hiện các hoạt động đáng ngờ trong các cụm EKS của bạn bằng cách sử dụng hoạt động nhật ký kiểm toán Kubernetes
2. EKS Runtime Monitoring cung cấp phạm vi phát hiện mối đe dọa thời gian chạy cho các nút Amazon Elastic Kubernetes Service (Amazon EKS) và các container trong môi trường AWS của bạn

Trong phần này, chúng ta sẽ xem xét cả hai loại bảo vệ với các ví dụ thực tế.
