---
title: "Sử dụng Amazon DynamoDB"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 4.2 </b>"
---

#### Sử dụng Amazon DynamoDB

Quy trình đầu tiên trong quá trình này là cấu hình lại dịch vụ carts để sử dụng một bảng DynamoDB đã được tạo trước cho chúng ta. Ứng dụng tải hầu hết cấu hình từ một ConfigMap, hãy xem nó:

```bash
$ kubectl -n carts get -o yaml cm carts
apiVersion: v1
data:
  AWS_ACCESS_KEY_ID: key
  AWS_SECRET_ACCESS_KEY: secret
  CARTS_DYNAMODB_CREATETABLE: true
  CARTS_DYNAMODB_ENDPOINT: http://carts-dynamodb:8000
  CARTS_DYNAMODB_TABLENAME: Items
kind: ConfigMap
metadata:
  name: carts
  namespace: carts
```

Ngoài ra, kiểm tra tình trạng hiện tại của ứng dụng bằng cách sử dụng trình duyệt. Một dịch vụ kiểu `LoadBalancer` có tên `ui-nlb` được cung cấp trong không gian tên `ui`, từ đó có thể truy cập giao diện người dùng của ứng dụng.

```bash
$ kubectl -n ui get service ui-nlb -o jsonpath='{.status.loadBalancer.ingress[*].hostname}{"\n"}'
k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com
```

Sử dụng URL được tạo ra từ lệnh trên để mở giao diện người dùng trong trình duyệt của bạn. Nó nên mở trang chủ cửa hàng như được hiển thị dưới đây.

![Home](../../../static/img/sample-app-screens/home.png)

Bây giờ, kustomization sau đây ghi đè lên ConfigMap, loại bỏ cấu hình điểm cuối DynamoDB, làm cho SDK mặc định sử dụng dịch vụ DynamoDB thực sự thay vì Pod kiểm thử của chúng tôi. Chúng tôi cũng đã cung cấp tên của bảng DynamoDB đã được tạo cho chúng tôi, được lấy từ biến môi trường `CARTS_DYNAMODB_TABLENAME`.

```kustomization
modules/security/eks-pod-identity/dynamo/kustomization.yaml
ConfigMap/carts
```

Hãy kiểm tra giá trị của `CARTS_DYNAMODB_TABLENAME` sau đó chạy Kustomize để sử dụng dịch vụ DynamoDB thực sự:

```bash
$ echo $CARTS_DYNAMODB_TABLENAME
eks-workshop-carts
$ kubectl kustomize ~/environment/eks-workshop/modules/security/eks-pod-identity/dynamo \
  | envsubst | kubectl apply -f-
```

Điều này sẽ ghi đè lên ConfigMap của chúng tôi với các giá trị mới:

```bash
$ kubectl -n carts get cm carts -o yaml
apiVersion: v1
data:
  CARTS_DYNAMODB_TABLENAME: eks-workshop-carts
kind: ConfigMap
metadata:
  labels:
    app: carts
  name: carts
  namespace: carts
```

Chúng ta sẽ cần khởi động lại tất cả các Pod của ứng dụng `carts` để lấy nội dung ConfigMap mới của chúng ta.

```bash hook=enable-dynamo hookTimeout=430
$ kubectl -n carts rollout restart deployment/carts
deployment.apps/carts restarted
$ kubectl -n carts rollout status deployment/carts
```

Vậy bây giờ ứng dụng của chúng ta nên đang sử dụng DynamoDB phải không? Hãy thử tải nó lên trong trình duyệt bằng URL được xuất ra trong lệnh trước đó và điều hướng đến giỏ hàng.

```bash
$ kubectl -n ui get service ui-nlb -o jsonpath='{.status.loadBalancer.ingress[*].hostname}{"\n"}'
k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com
```

![Error500](../../../static/img/sample-app-screens/error-500.png)

Trang giỏ hàng không thể truy cập được! Điều gì đã xảy ra không ổn?