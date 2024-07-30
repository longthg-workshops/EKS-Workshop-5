---
title: "Network-Policies (Chính sách Mạng)"
date: "`r Sys.Date()`"
weight: 11
chapter: false
pre: "<b> 1.11 </b>"
---

Lưu lượng đi qua một máy chủ web frontend cho người dùng một máy chủ ứng dụng phục vụ API phía sau và một máy chủ cơ sở dữ liệu.

- Có hai loại lưu lượng
  - Ingress
  - Egress


#### Bảo mật Mạng


#### Chính sách Mạng

#### Bộ chọn Chính sách Mạng

#### Quy tắc Chính sách Mạng

#### Tạo chính sách mạng

- Để tạo một chính sách mạng
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
   name: db-policy
  spec:
    podSelector:
      matchLabels:
        role: db
    policyTypes:
    - Ingress
    ingress:
    - from:
      - podSelector:
          matchLabels:
            role: api-pod
      ports:
      - protocol: TCP
        port: 3306
  ```
  
  ```
  $ kubectl create -f policy-definition.yaml
  ```
  
#### Ghi chú

#### Bài giảng bổ sung về [Phát triển Chính sách Mạng](https://kodekloud.com/topic/developing-network-policies/)

#### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/