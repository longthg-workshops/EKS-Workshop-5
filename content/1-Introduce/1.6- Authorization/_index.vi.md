
---
title: "Cấp quyền (Authorization)"
date: "`r Sys.Date()`"
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

#### K8s Reference Docs
  - https://kubernetes.io/docs/reference/access-authn-authz/authorization/