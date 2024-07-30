
---
title: "Cluster Roles"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b> 1.8 </b>"
---

#### Cluster Roles
  
Trong phần này, chúng ta sẽ tìm hiểu về các vai trò cụm

#### Role (Vai trò)
- Vai trò và Rolebindings được gắn với không gian tên, có nghĩa là chúng được tạo trong các không gian tên.
  

  
#### Namespace (Không gian tên)
- Bạn có thể nhóm hoặc cô lập các nút trong một không gian tên không?
  - Không, những thứ đó là tài nguyên phạm vi cụm hoặc phạm vi cụm. Chúng không thể được liên kết với bất kỳ không gian tên cụ thể nào.
  

  
- Vì vậy, các tài nguyên được phân loại là tài nguyên phạm vi tên hoặc phạm vi cụm.
  
- Để xem tài nguyên phạm vi tên
  ```bash
  $ kubectl api-resources --namespaced=true
  ```
- Để xem tài nguyên không phạm vi tên
  ```bash
  $ kubectl api-resources --namespaced=false
  ```
  
  
#### ClusterRole (Vai trò cụm) và ClusterRoleBinding (Ràng buộc Vai trò cụm)
- Vai trò cụm là các vai trò ngoại trừ chúng được sử dụng cho các tài nguyên phạm vi cụm. Loại như **`CLusterRole`** 
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: cluster-administrator
  rules:
  - apiGroups: [""] # "" chỉ ra nhóm API lõi
    resources: ["nodes"]
    verbs: ["get", "list", "delete", "create"]
  ```
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: cluster-admin-role-binding
  subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: cluster-administrator
    apiGroup: rbac.authorization.k8s.io
  ```
  ```bash
  $ kubectl create -f cluster-admin-role.yaml
  $ kubectl create -f cluster-admin-role-binding.yaml
  ```
  
  
- Bạn cũng có thể tạo một vai trò cụm cho các tài nguyên không gian tên. Khi bạn làm điều đó, người dùng sẽ có quyền truy cập vào các tài nguyên này trên tất cả các không gian tên.

#### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities