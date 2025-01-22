---
title: "Cơ sở cho bảo mật Kubernetes"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

### Đặt vấn đề

Chúng ta cần đưa ra hai loại quyết định:

- Ai có thể truy cập?
- Họ có thể làm gì?

![KubeSec](../../../images/0001/0002.png?featherlight=false&width=90pc)

Từ đó, ta có hai quá trình sau:

- **Xác thực (Authentication):** Quyết định ai có thể truy cập vào máy chủ API - được xác định bởi các cơ chế xác thực.

- **Ủy quyền (Authorization):** Sau khi họ có quyền truy cập vào cụm, cho phép họ làm những gì ? - được được xác định bởi các cơ chế ủy quyền.

### Chứng chỉ TLS

Tất cả các giao tiếp với cụm, giữa các thành phần khác nhau như Cụm ETCD, kube-controller-manager, lập lịch, máy chủ API, cũng như những thành phần đang chạy trên các nút làm việc như kubelet và kubeproxy đều được bảo vệ bằng mã hóa TLS.

![EKS](../../../images/0001/0003.png?featherlight=false&width=90pc)

### Chính sách Mạng

Các chính sách mạng quyết định cách giao tiếp giữa các ứng dụng trong cụm. Chúng cho phép bạn quy định các pod được và không được phép giao tiếp với nhau, cũng như với thế giới bên ngoài.

### Một số chính sách bảo mật cho máy chủ
- Chặn truy cập bằng mật khẩu
- Chỉ cho phép truy cập SSH bằng khóa

![EKS](../../../images/0001/0001.png?featherlight=false&width=90pc)