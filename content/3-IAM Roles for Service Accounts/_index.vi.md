---
title: "IAM Roles for Service Accounts"
weight: 3
chapter: false
pre: "<b> 3. </b>"
---

### Trước khi bắt đầu

Chuẩn bị môi trường của bạn cho phần này:

```bash
$ prepare-environment security/irsa
```

Việc này sẽ tạo những thay đổi sau cho môi trường thực hành của bạn:

- Tạo bảng Amazon DynamoDB

- Tạo vai trò IAM cho khối lượng công việc Amazon EKS để truy cập bảng DynamoDB

- Cài đặt AWS Load Balancer Controller trong cụm Amazon EKS

{{% notice info %}}
You can view the Terraform that applies these changes [here](https://github.com/aws-samples/eks-workshop-v2/tree/stable/manifests/modules/security/irsa/.workshop/terraform).
{{% /notice %}}

### Tổng quan về phần thực hành này

Các ứng dụng trong container của Pod có thể sử dụng AWS SDK hoặc AWS CLI để tạo yêu cầu API tới các dịch vụ AWS bằng quyền AWS Identity and Access Management (IAM). Ví dụ: các ứng dụng có thể cần tải tệp lên thùng S3 hoặc truy vấn bảng DynamoDB. Để thực hiện như vậy, các ứng dụng phải ký các yêu cầu API AWS của mình bằng thông tin xác thực AWS. [IAM Roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) (IRSA) cung cấp khả năng quản lý thông tin xác thực cho các ứng dụng của bạn, tương tự như cách Hồ sơ phiên bản IAM cung cấp thông tin xác thực cho các phiên bản Amazon EC2. Thay vì tạo và phân phối thông tin xác thực AWS của bạn tới các container hoặc dựa vào Hồ sơ phiên bản Amazon EC2 để ủy quyền, bạn liên kết vai trò IAM với Tài khoản dịch vụ Kubernetes và định cấu hình Pod của mình để sử dụng Tài khoản dịch vụ đó.

Trong chương này, chúng ta sẽ định cấu hình lại một trong các thành phần ứng dụng mẫu để tận dụng API AWS và cung cấp cho nó xác thực phù hợp.

### Phần thực hành này có gì ?

{{% children / %}}