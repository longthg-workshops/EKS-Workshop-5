---
title: "Liệt kê tài nguyên"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1 </b>"
---

#### Liệt kê tài nguyên

#### Giới thiệu
Thành phần `carts` của kiến ​​trúc của chúng tôi sử dụng Amazon DynamoDB làm cơ sở dữ liệu lưu trữ của nó, đây là một trường hợp sử dụng phổ biến bạn sẽ thấy cho việc tích hợp cơ sở dữ liệu không quan hệ với Amazon EKS. Cách mà API `carts` hiện đang triển khai sử dụng một phiên bản nhẹ của Amazon DynamoDB chạy như một container trong cụm EKS.

Bạn có thể thấy điều này bằng cách chạy lệnh sau:

```bash
kubectl -n carts get pod
```

```plaintext
NAME                              READY   STATUS    RESTARTS   AGE
carts-5d7fc9d8f-xm4hs             1/1     Running   0          14m
carts-dynamodb-698674dcc6-hw2bg   1/1     Running   0          14m
```

Trong trường hợp trên, Pod `carts-dynamodb-698674dcc6-hw2bg` là dịch vụ DynamoDB nhẹ của chúng tôi. Chúng ta có thể xác minh ứng dụng giỏ hàng của chúng ta đang sử dụng nó bằng cách kiểm tra môi trường của nó:

```bash
kubectl -n carts exec deployment/carts -- env | grep CARTS_DYNAMODB_ENDPOINT
```

```plaintext
CARTS_DYNAMODB_ENDPOINT=http://carts-dynamodb:8000
```

Cách tiếp cận này có thể hữu ích cho việc kiểm thử, nhưng chúng tôi muốn di chuyển ứng dụng của mình để sử dụng dịch vụ Amazon DynamoDB quản lý hoàn toàn để tận dụng đầy đủ quy mô và độ tin cậy mà nó cung cấp.