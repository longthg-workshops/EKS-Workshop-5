---
title: "Kubernetes Security Primitives"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

#### Kubernetes Security Primitives

#### Secure Hosts

![EKS](/images/0001/0001.png?featherlight=false&width=90pc)


#### Secure Kubernetes
Chúng ta cần đưa ra hai loại quyết định:

- Ai có thể truy cập?
- Họ có thể làm gì?

![EKS](/images/0001/0002.png?featherlight=false&width=90pc)

#### Xác thực (Authentication)

- Ai có thể truy cập vào máy chủ API được xác định bởi các cơ chế xác thực.

#### Ủy quyền (Authorization)

- Sau khi họ có quyền truy cập vào cụm, những gì họ có thể làm được được xác định bởi các cơ chế ủy quyền.

#### Chứng chỉ TLS

- Tất cả các giao tiếp với cụm, giữa các thành phần khác nhau như Cụm ETCD, kube-controller-manager, lập lịch, máy chủ API, cũng như những thành phần đang chạy trên các nút làm việc như kubelet và kubeproxy đều được bảo vệ bằng mã hóa TLS.

![EKS](/images/0001/0003.png?featherlight=false&width=90pc)

#### Chính sách Mạng
Cách giao tiếp giữa các ứng dụng trong cụm.

