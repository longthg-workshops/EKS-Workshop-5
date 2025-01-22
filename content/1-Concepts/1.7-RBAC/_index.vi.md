---
title: "Kiểm soát truy cập dựa trên vai trò (Role-Based Access Control)"
weight: 7
chapter: false
pre: "<b> 1.7 </b>"
---
Kiểm soát truy cập dựa trên vai trò (Role-Based Access Control - **RBAC**)

### Làm thế nào để tạo một vai trò?

- Mỗi quy tắc (rule) trong một vai trò (role) có 3 phần:

  - apiGroups

  - resources

  - verbs

- Tạo vai trò bằng lệnh kubectl:

```bash
$ kubectl create -f developer-role.yaml
```

### Liên kết người dùng với vai trò đã tạo.

- Đối với điều này, chúng ta tạo một đối tượng khác được gọi là **`RoleBinding`**. Đối tượng liên kết vai trò này liên kết một đối tượng người dùng với một vai trò.

- Tạo liên kết vai trò bằng lệnh kubectl

```bash
$ kubectl create -f devuser-developer-binding.yaml
```

- Cũng lưu ý rằng các vai trò và liên kết vai trò nằm trong phạm vi của namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""] # "" biểu thị nhóm API cốt lõi
  resources: ["pods"]
  verbs: ["get", "list", "update", "delete", "create"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user # "name" phân biệt chữ hoa chữ thường
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```
![rbac1](/images/1/7/0001.PNG)

#### Xem RBAC
  
- Để liệt kê các vai trò
  ```bash
  $ kubectl get roles
  ```
- Để liệt kê các liên kết vai trò
  ```bash
  $ kubectl get rolebindings
  ```
- Để mô tả vai trò 
  ```bash
  $ kubectl describe role developer
  ```

![rbac2](/images/1/7/0002.PNG)
    
- Để mô tả liên kết vai trò
  ```bash
  $ kubectl describe rolebinding devuser-developer-binding
  ```
![rbac3](/images/1/7/0003.PNG)

**Nếu bạn là một người dùng muốn xem, liệu bạn có quyền truy cập vào một tài nguyên cụ thể trong cluster không?**

### Kiểm tra quyền truy cập

Bạn có thể sử dụng lệnh kubectl auth

```bash
$ kubectl auth can-i create deployments
$ kubectl auth can-i delete nodes
```
```bash
$ kubectl auth can-i create deployments --as dev-user
$ kubectl auth can-i create pods --as dev-user
```
```bash
$ kubectl auth can-i create pods --as dev-user --namespace test
```
![rbac5](/images/1/7/0004.PNG)
  
### Tên Tài Nguyên
- Lưu ý về tên tài nguyên, chúng ta vừa thấy cách bạn có thể cung cấp quyền truy cập cho người dùng đối với các tài nguyên như pods trong namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""] # "" biểu thị nhóm API cốt lõi
  resources: ["pods"]
  verbs: ["get", "update", "create"]
  resourceNames: ["blue", "orange"]
```  
![rbac4](/images/1/7/0005.PNG)
### Tài liệu Tham Khảo K8s
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities