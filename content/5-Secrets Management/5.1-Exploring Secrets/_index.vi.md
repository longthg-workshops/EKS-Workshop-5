---
title: "Tìm hiểu khoá bí mật"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 5.1 </b>"
---


### Tìm hiểu khoá bí mật

#### Tiết lộ Secrets dưới dạng Environment Variables

Có thể tiết lộ các khóa, chẳng hạn như, tên người dùng và mật khẩu, trong Secret database-credentials tới một Pod dưới dạng Environment Variables sử dụng một tài liệu Pod như sau (dưới đây):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: someName
  namespace: someNamespace
spec:
  containers:
  - name: someContainer
    image: someImage
    env:
    - name: DATABASE_USER
      valueFrom:
        secretKeyRef:
          name: database-credentials
          key: username
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: database-credentials
          key: password
```

#### Tiết lộ Secrets dưới dạng Volumes

Secrets cũng có thể được gắn kết như các khối dữ liệu trên một Pod và bạn có thể kiểm soát các đường dẫn bên trong Volumes nơi các khóa Secret được chiếu ra sử dụng một tài liệu Pod như sau (dưới đây):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: someName
  namespace: someNamespace
spec:
  containers:
  - name: someContainer
    image: someImage
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/data"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: database-credentials
      items:
      - key: username
        path: DATABASE_USER 
      - key: password
        path: DATABASE_PASSWORD

```

Với đặc tả Pod trên, điều sau sẽ xảy ra:

- Giá trị cho khóa username trong Secret database-credentials được lưu trữ trong tệp `/etc/data/DATABASE_USER` bên trong Pod.
- Giá trị cho khóa mật khẩu được lưu trữ trong tệp `/etc/data/DATABASE_PASSWORD`.