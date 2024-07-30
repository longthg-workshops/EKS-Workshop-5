---
title: "Giới thiệu"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1 </b>"
---

#### Giới thiệu

Thành phần `carts` trong kiến trúc của chúng tôi sử dụng Amazon DynamoDB làm nền tảng lưu trữ, đây là một trường hợp sử dụng phổ biến bạn sẽ tìm thấy cho việc tích hợp cơ sở dữ liệu không quan hệ với Amazon EKS. Cách mà API của `carts` đang triển khai hiện nay sử dụng một [phiên bản nhẹ của Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html) chạy như một container trong cụm EKS.

Bạn có thể thấy điều này bằng cách chạy lệnh sau:

```bash
$ kubectl -n carts get pod 
NAME                              READY   STATUS    RESTARTS        AGE
carts-5d7fc9d8f-xm4hs             1/1     Running   0               14m
carts-dynamodb-698674dcc6-hw2bg   1/1     Running   0               14m
```

Trong trường hợp trên, Pod `carts-dynamodb-698674dcc6-hw2bg` là dịch vụ DynamoDB nhẹ của chúng tôi. Chúng ta có thể xác minh ứng dụng `carts` của chúng ta đang sử dụng nó bằng cách kiểm tra môi trường của nó:

```bash
$ kubectl -n carts exec deployment/carts -- env | grep CARTS_DYNAMODB_ENDPOINT
CARTS_DYNAMODB_ENDPOINT=http://carts-dynamodb:8000
```

Cách tiếp cận này có thể hữu ích cho việc kiểm thử, nhưng chúng tôi muốn di chuyển ứng dụng của mình để sử dụng dịch vụ Amazon DynamoDB quản lý hoàn toàn để tận dụng đầy đủ quy mô và độ tin cậy mà nó cung cấp.