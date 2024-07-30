---
title: "Quản lý khoá bí mật"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 5. </b>"
---

#### Quản lý khoá bí mật

[**Kubernetes Secret (Khoá bí mật Kubernetes)**](https://kubernetes.io/docs/concepts/configuration/secret/) là một tài nguyên giúp các nhà quản trị cụm quản lý việc triển khai thông tin nhạy cảm như mật khẩu, mã thông báo OAuth và khóa SSH v.v. Các secret này có thể được gắn kết như các thư mục dữ liệu hoặc được tiết lộ như các biến môi trường cho các container trong một Pod, từ đó tách biệt việc triển khai Pod khỏi việc quản lý dữ liệu nhạy cảm cần thiết cho các ứng dụng được container hóa trong một Pod.

Đã trở thành một thực hành phổ biến cho một Nhóm DevOps quản lý các biểu mẫu YAML cho các nguồn lực Kubernetes khác nhau và kiểm soát phiên bản chúng bằng cách sử dụng một kho Git. Điều này cho phép họ tích hợp một kho Git với luồng làm việc GitOps để thực hiện Cung cấp Liên tục của các nguồn lực này đến một cụm EKS.
Kubernetes ẩn dữ liệu nhạy cảm trong một Secret bằng cách sử dụng mã hóa base64 đơn giản, cũng như việc lưu trữ các tệp như vậy trong một kho Git là cực kỳ không an toàn vì nó rất dễ giải mã dữ liệu đã được mã hóa base64. Điều này làm cho việc quản lý các biểu mẫu YAML cho Kubernetes Secrets bên ngoài một cụm trở nên khó khăn.

Có một số phương pháp khác nhau bạn có thể sử dụng để quản lý secrets, trong chương này về Quản lý Secrets, chúng tôi sẽ bao gồm một vài phương pháp, [Sealed Secrets for Kubernetes](https://github.com/bitnami-labs/sealed-secrets) và [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html).