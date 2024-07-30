---
title: "Sử dụng DynamoDB"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---


#### Sử dụng DynamoDB


Bước đầu tiên trong quy trình này là cấu hình lại dịch vụ giỏ hàng để sử dụng một bảng DynamoDB đã được tạo trước đó cho chúng ta. Ứng dụng tải hầu hết các cấu hình từ một ConfigMap, hãy xem nó:

```bash
kubectl -n carts get -o yaml cm carts
```

```yaml
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

Kustomization sau đây ghi đè lên ConfigMap, loại bỏ cấu hình đầu cuối DynamoDB endpoint để SDK mặc định sử dụng dịch vụ DynamoDB thực tế thay vì Pod kiểm tra của chúng tôi. Chúng tôi cũng đã cung cấp tên bảng DynamoDB đã được tạo sẵn cho chúng ta, được rút ra từ biến môi trường CARTS_DYNAMODB_TABLENAME.

Kustomize Patch
ConfigMap/carts
Diff
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../../../base-application/carts
configMapGenerator:
- name: carts
  namespace: carts
  env: config.properties
  behavior: replace
  options:
    disableNameSuffixHash: true
```

Hãy kiểm tra giá trị của CARTS_DYNAMODB_TABLENAME sau đó chạy Kustomize để sử dụng dịch vụ DynamoDB thực tế:

```bash
echo $CARTS_DYNAMODB_TABLENAME
eks-workshop-carts

kubectl kustomize ~/environment/eks-workshop/modules/security/irsa/dynamo \
  | envsubst | kubectl apply -f-
```

Điều này sẽ ghi đè lên ConfigMap của chúng tôi với các giá trị mới:

```bash
kubectl get -n carts cm carts -o yaml
```

```yaml
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

Bây giờ chúng ta cần khởi động lại tất cả các Pod `carts` để lấy nội dung ConfigMap mới của chúng tôi:

```bash
kubectl rollout restart -n carts deployment/carts
```

```bash
kubectl rollout status -n carts deployment/carts
```

Hãy thử truy cập ứng dụng của chúng ta bằng trình duyệt. Một dịch vụ kiểu LoadBalancer có tên ui-nlb được cung cấp trong không gian tên ui từ đó có thể truy cập giao diện người dùng của ứng dụng.

```bash
kubectl get service -n ui ui-nlb -o jsonpath='{.status.loadBalancer.ingress[*].hostname}{"\n"}'
k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com
```

Vậy bây giờ ứng dụng của chúng ta phải đang sử dụng DynamoDB đúng không? Hãy mở nó trong trình duyệt bằng cách sử dụng đầu ra của lệnh trên và điều hướng đến giỏ hàng:

Trang giỏ hàng không thể truy cập! Đã xảy ra vấn đề gì?