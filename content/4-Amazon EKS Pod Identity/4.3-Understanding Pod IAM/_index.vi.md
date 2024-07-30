---
title: "Hiểu về Pod IAM"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 4.3 </b>"
---

#### Hiểu về Pod IAM

Điểm đầu tiên để tìm vấn đề là logs của dịch vụ `carts`:

```bash
$ kubectl -n carts logs deployment/carts
... đoạn output đã bị lược bỏ
```

Điều này sẽ trả về nhiều logs, vì vậy hãy lọc để có cái nhìn tổng quan về vấn đề:

```bash
$ kubectl -n carts logs deployment/carts | grep -i Exception
2024-02-12T20:20:47.547Z ERROR 1 --- [nio-8080-exec-7] o.a.c.c.C.[.[.[.[dispatcherServlet]      : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: com.amazonaws.services.dynamodbv2.model.AmazonDynamoDBException: User: arn:aws:sts::123456789000:assumed-role/eksctl-eks-workshop-nodegroup-defa-NodeInstanceRole-Q1p0w2o9e3i8/i-0p1qaz2wsx3edc4rfv is not authorized to perform: dynamodb:Query on resource: arn:aws:dynamodb:us-west-2:123456789000:table/Items/index/idx_global_customerId because no identity-based policy allows the dynamodb:Query action (Service: AmazonDynamoDBv2; Status Code: 400; Error Code: AccessDeniedException; Request ID: MA54K0UDUOCLJ96UP6PT76VTBBVV4KQNSO5AEMVJF66Q9ASUAAJG; Proxy: null)] with root cause
com.amazonaws.services.dynamodbv2.model.AmazonDynamoDBException: User: arn:aws:sts::123456789000:assumed-role/eksctl-eks-workshop-nodegroup-defa-NodeInstanceRole-Q1p0w2o9e3i8/i-0p1qaz2wsx3edc4rfv is not authorized to perform: dynamodb:Query on resource: arn:aws:dynamodb:us-west-2:123456789000:table/Items/index/idx_global_customerId because no identity-based policy allows the dynamodb:Query action (Service: AmazonDynamoDBv2; Status Code: 400; Error Code: AccessDeniedException; Request ID: MA54K0UDUOCLJ96UP6PT76VTBBVV4KQNSO5AEMVJF66Q9ASUAAJG; Proxy: null)
```

Ứng dụng đang tạo ra một `AccessDeniedException` cho thấy rằng IAM Role mà Pod của chúng ta đang sử dụng để truy cập DynamoDB không có các quyền cần thiết. Điều này xảy ra vì mặc định, nếu không có IAM Roles hoặc Policies nào được liên kết với Pod của chúng ta, nó sẽ sử dụng IAM Role liên kết với Instance Profile được gán cho instance EC2 đang chạy, trong trường hợp này Role này không có một IAM Policy nào cho phép truy cập vào DynamoDB.

Một cách chúng ta có thể giải quyết vấn đề này là mở rộng IAM permissions của Instance Profile của EC2, nhưng điều này sẽ cho phép bất kỳ Pod nào chạy trên chúng có thể truy cập vào bảng DynamoDB của chúng ta, điều này không an toàn và không phải là một thực hành tốt cho việc cấp quyền ít nhất cần thiết. Thay vào đó, chúng ta sẽ sử dụng EKS Pod Identity để cho phép truy cập cụ thể cần thiết bởi ứng dụng `carts` tại mức Pod.