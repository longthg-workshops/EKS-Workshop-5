---
title: "Cấp quyền (Authorization)"
weight: 6
chapter: false
pre: "<b> 1.6 </b>"
---


####  Cấp quyền (Authorization)

- Kubernetes hỗ trợ một số cơ chế cấp quyền khác nhau, bao gồm:

  - Node Authorization (Xác thực node)

  - Xác thực dựa trên thuộc tính (Attribute-based Authorization - ABAC)

  - Xác thực dựa trên vai trò (Role-Based Authorization - RBAC)

  - Webhook

#### Xác thực node
![node-auth](/images/1/6/0001.png?width=80pc)

#### ABAC
![abac](/images/1/6/0002.png?width=80pc)

#### RBAC
![rbac](/images/1/6/0003.png?width=80pc)

#### Webhook
![webhook](/images/1/6/0004.png?width=80pc)

### Thiết lập chế độ xác thực trên Kubernetes API Server

Các tùy chọn chế độ có thể được xác định trên kube-apiserver

![mode](/images/1/6/0005.png?width=80pc)

Khi bạn chỉ định nhiều chế độ xác thực, hệ thống sẽ cho phép theo thứ tự mà nó được chỉ định

![mode1](/images/1/6/0006.png?width=80pc)

#### K8s Reference Docs
  - https://kubernetes.io/docs/reference/access-authn-authz/authorization/