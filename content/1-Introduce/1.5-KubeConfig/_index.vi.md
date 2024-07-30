---
title: "KubeConfig"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 1.5 </b>"
---

Trong phần này, chúng ta sẽ tìm hiểu về kubeconfig trong Kubernetes.

- Client sử dụng tệp chứng chỉ và khóa để truy vấn API Rest của Kubernetes để lấy danh sách các pod bằng cách sử dụng curl.
- Bạn có thể chỉ định điều này bằng cách sử dụng kubectl

- Chúng ta có thể di chuyển các thông tin này vào một tệp cấu hình được gọi là kubeconfig. Và chỉ định tệp này như là tùy chọn kubeconfig trong câu lệnh.

```
$ kubectl get pods --kubeconfig config

```

#### Tệp Kubeconfig

Tệp kubeconfig có 3 phần
- Cụm (Clusters)
- Ngữ cảnh (Contexts)
- Người dùng (Users)

Để xem tệp hiện tại đang được sử dụng

```
$ kubectl config view

```

Bạn có thể chỉ định tệp kubeconfig với kubectl config view bằng cờ "--kubeconfig"

```
$ kubectl config veiw --kubeconfig=my-custom-config

```

Làm thế nào để cập nhật ngữ cảnh hiện tại của bạn? Hoặc thay đổi ngữ cảnh hiện tại

```
$ kubectl config view --kubeconfig=my-custom-config

```

Xem các trợ giúp của kubectl config

```
$ kubectl config -h

```

#### Tài liệu tham khảo K8s
https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config