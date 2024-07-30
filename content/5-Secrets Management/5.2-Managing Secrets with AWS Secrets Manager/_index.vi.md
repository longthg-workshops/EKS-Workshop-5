---
title: "Quản lý khoá bí mật bằng AWS Secrets Manager"
date: "`r Sys.Date()`"
weight: 2 
chapter: false
pre: "<b> 5.2 </b>"
---

#### Quản lý khoá bí mật bằng AWS Secrets Manager

#### Chuẩn bị môi trường cho phần này:

```bash timeout=600 wait=30 hook=install
$ prepare-environment security/secrets-manager
```

Điều này sẽ thực hiện những thay đổi sau đây trong môi trường thí nghiệm của bạn:

Cài đặt các phần mở rộng Kubernetes sau đây trong Cụm EKS của bạn:
* Kubernetes Secrets Store CSI Driver
* AWS Secrets and Configuration Provider
* External Secrets Operator

Bạn có thể xem Terraform áp dụng những thay đổi này [tại đây](https://github.com/aws-samples/eks-workshop-v2/tree/main/manifests/modules/security/secrets/secrets-manager/.workshop/terraform).

[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) cho phép bạn dễ dàng xoay vòng, quản lý và lấy dữ liệu nhạy cảm như thông tin đăng nhập, khóa API, chứng chỉ, và những thông tin khác. Bạn có thể sử dụng [AWS Secrets and Configuration Provider (ASCP)](https://github.com/aws/secrets-store-csi-driver-provider-aws) cho [Kubernetes Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/) để gắn kết các bí mật được lưu trữ trong Secrets Manager dưới dạng khối lượng trong các Pod Kubernetes.

Với ASCP, bạn có thể lưu trữ và quản lý các thông tin bí mật của mình trong Secrets Manager và sau đó lấy chúng thông qua các khối công việc đang chạy trên Amazon EKS. Bạn có thể sử dụng vai trò và chính sách IAM để giới hạn quyền truy cập vào các bí mật của bạn cho các Pod Kubernetes cụ thể trong một cụm. ASCP lấy danh tính Pod và trao đổi danh tính đó để đổi lấy một vai trò IAM. ASCP giả định vai trò IAM của Pod, sau đó nó có thể lấy các bí mật từ Secrets Manager mà được ủy quyền cho vai trò đó.

Một cách khác để tích hợp AWS Secrets Manager với Kubernetes Secrets là thông qua [External Secrets](https://external-secrets.io/). External Secrets là một toán tử có thể tích hợp và đồng bộ hóa các bí mật từ AWS Secrets Manager đọc thông tin từ nó và tự động tiêm các giá trị vào một Kubernetes Secret với một dạng trừu tượng hóa lưu trữ và quản lý vòng đời của các bí mật cho bạn.

Nếu bạn sử dụng xoay vòng tự động của Secrets Manager cho các bí mật của bạn, bạn có thể phụ thuộc vào khoảng thời gian làm mới của External Secrets hoặc sử dụng tính năng hòa giải xoay vòng của Secrets Store CSI Driver để đảm bảo bạn đang lấy khoá bí mật mới nhất từ Secrets Manager, tùy thuộc vào công cụ bạn chọn để quản lý các bí mật bên trong Cụm Amazon EKS của bạn.

Trong phần tiếp theo của thí nghiệm này, chúng ta sẽ tạo một vài tình huống mẫu về việc sử dụng các bí mật từ AWS Secrets Manager và External Secrets.