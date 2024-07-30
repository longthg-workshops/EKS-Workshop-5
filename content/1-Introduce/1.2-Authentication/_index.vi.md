---
title: "Xác thực (Authentication)"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---


#### Tài khoản (Accounts)

![EKS](/images/0001/0004.png?featherlight=false&width=90pc)

1. Các người dùng khác nhau có thể truy cập vào bảo mật của cụm được quản lý bởi các ứng dụng tự quản lý bên trong cụm mà họ truy cập.

![EKS](/images/0001/0005.png?featherlight=false&width=90pc)

2. Vậy, chúng ta còn lại 2 loại người dùng:

    - Người: như Quản trị viên và Nhà phát triển
    - Robot: như các quy trình/dịch vụ hoặc ứng dụng đòi hỏi truy cập vào cụm.

![EKS](/images/0001/0006.png?featherlight=false&width=90pc)


3. Tất cả quản lý truy cập người dùng đều được thực hiện bởi apiserver và tất cả các yêu cầu đi qua apiserver.

![EKS](/images/0001/0007.png?featherlight=false&width=90pc)

#### Cơ chế Xác thực (Authentication Mechanisms)

Có các cơ chế xác thực khác nhau có thể được cấu hình.

![EKS](/images/0001/0008.png?featherlight=false&width=90pc)

#### Cơ chế Xác thực cơ bản

![EKS](/images/0001/0009.png?featherlight=false&width=90pc)

#### Cấu hình kube-apiserver

- Nếu bạn thiết lập thông qua kubeadm thì cập nhật tệp mẫu kube-apiserver.yaml với tùy chọn.

![EKS](/images/0001/00010.png?featherlight=false&width=90pc)

#### Xác thực Người dùng (Authenticate User)

- Để xác thực bằng các thông tin đăng nhập cơ bản khi truy cập vào máy chủ API, chỉ định tên người dùng và mật khẩu trong một lệnh curl.

```
$ curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
```

![EKS](/images/0001/00011.png?featherlight=false&width=90pc)


Chúng ta có thể có thêm cột trong tập tin user-details.csv để gán người dùng vào các nhóm cụ thể.


#### K8s Reference Docs

- https://kubernetes.io/docs/reference/access-authn-authz/authentication/

