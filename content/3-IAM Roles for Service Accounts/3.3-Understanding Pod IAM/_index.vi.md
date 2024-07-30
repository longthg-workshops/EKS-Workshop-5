---
title: "Hiểu về Pod IAM"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.3 </b>"
---

Điểm đầu tiên để tìm vấn đề là trong log của dịch vụ `carts`:

```bash
$ kubectl logs -n carts deployment/carts
```

Điều này sẽ trả về nhiều log, vì vậy hãy lọc để có cái nhìn ngắn gọn về vấn đề:

```bash
$ kubectl -n carts logs deployment/carts \
  | grep DynamoDbException
2024-01-09T18:54:10.818Z ERROR 1 --- ${sys:LOGGED_APPLICATION_NAME}[nio-8080-exec-1] o.a.c.c.C.[.[.[.[dispatcherServlet]      : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: software.amazon.awssdk.services.dynamodb.model.DynamoDbException: User: arn:aws:sts::123456789012:assumed-role/eksctl-eks-workshop-nodegroup-defa-NodeInstanceRole-vniVa7QtGHXO/i-075976199b049a358 is not authorized to perform: dynamodb:Query on resource: arn:aws:dynamodb:us-west-2:123456789012:table/eks-workshop-carts/index/idx_global_customerId because no identity-based policy allows the dynamodb:Query action (Service: DynamoDb, Status Code: 400, Request ID: QEQBV8R44MI1DSRQFGIAAAOS8FVV4KQNSO5AEMVJF66Q9ASUAAJG)] with root cause
software.amazon.awssdk.services.dynamodb.model.DynamoDbException: User: arn:aws:sts::123456789012:assumed-role/eksctl-eks-workshop-nodegroup-defa-NodeInstanceRole-vniVa7QtGHXO/i-075976199b049a358 is not authorized to perform: dynamodb:Query on resource: arn:aws:dynamodb:us-west-2:123456789012:table/eks-workshop-carts/index/idx_global_customerId because no identity-based policy allows the dynamodb:Query action (Service: DynamoDb, Status Code: 400, Request ID: QEQBV8R44MI1DSRQFGIAAAOS8FVV4KQNSO5AEMVJF66Q9ASUAAJG)
```

Ứng dụng của chúng ta đang tạo ra một `AccessDeniedException`, cho thấy vai trò IAM mà Pod của chúng ta đang sử dụng để truy cập DynamoDB không có các quyền cần thiết. Điều này xảy ra vì Pod của chúng ta mặc định đang sử dụng vai trò IAM được gán cho nút làm việc EC2 mà nó đang chạy, không có IAM Policy cho phép truy cập vào DynamoDB.

Một cách chúng ta có thể giải quyết vấn đề này là mở rộng quyền IAM của các nút làm việc EC2 của chúng ta, nhưng điều này sẽ cho phép bất kỳ Pod nào chạy trên chúng truy cập vào bảng DynamoDB của chúng ta, điều này không phải là một thực hành tốt và cũng không an toàn. Thay vào đó, chúng ta sẽ sử dụng IAM Roles for Service Accounts (IRSA) để cụ thể cho phép các Pod trong dịch vụ `carts` truy cập.

