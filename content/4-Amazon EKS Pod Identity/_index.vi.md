---
title: "Amazon EKS Pod Identity"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 4. </b>"
---

#### Chuẩn bị môi trường của bạn cho phần này:

```bash timeout=300 wait=30
$ prepare-environment security/eks-pod-identity
```

Điều này sẽ thực hiện các thay đổi sau đây vào môi trường lab của bạn:

- Tạo một bảng Amazon DynamoDB
- Tạo một vai trò IAM cho các tải trọng công việc AmazonEKS để truy cập vào bảng DynamoDB
- Cài đặt EKS Managed Addon cho EKS Pod Identity Agent
- Cài đặt AWS Load Balancer Controller trong cụm Amazon EKS

Ứng dụng trong các container của một Pod có thể sử dụng một SDK AWS được hỗ trợ hoặc AWS CLI để thực hiện các yêu cầu API đến các dịch vụ AWS bằng cách sử dụng quyền quản lý danh tính và truy cập (IAM) AWS. Ví dụ, các ứng dụng có thể cần tải tệp lên một bucket S3 hoặc truy vấn một bảng DynamoDB, và để làm điều đó, họ phải ký các yêu cầu API AWS của mình bằng các thông tin đăng nhập AWS. [Amazon EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html) cung cấp khả năng quản lý thông tin đăng nhập cho các ứng dụng của bạn, tương tự như cách Mẫu Amazon EC2 Instance cung cấp thông tin đăng nhập cho các thể hiện. Thay vì tạo và phân phối thông tin đăng nhập AWS của bạn cho các container hoặc sử dụng Amazon EC2 instance's Role, bạn có thể liên kết một Vai trò IAM với một Tài khoản Dịch vụ Kubernetes và cấu hình Pods của bạn với nó. Kiểm tra tài liệu EKS [tại đây](https://docs.aws.amazon.com/eks/latest/userguide/pod-id-minimum-sdk.html) để biết danh sách chính xác các phiên bản được hỗ trợ.

Trong chương này, chúng ta sẽ cấu hình lại một trong các thành phần ứng dụng mẫu để tận dụng API AWS và cung cấp cho nó các đặc quyền phù hợp.
