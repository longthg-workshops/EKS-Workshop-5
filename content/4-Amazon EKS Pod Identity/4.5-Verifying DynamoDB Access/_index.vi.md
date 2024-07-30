---
title: "Xác nhận khả năng truy cập DynamoDB"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 4.5 </b>"
---

Bây giờ, với `carts` Service Account được liên kết với vai trò IAM được ủy quyền, pod `carts` có quyền truy cập vào bảng DynamoDB. Truy cập cửa hàng web lại và điều hướng đến giỏ hàng.

```bash
$ kubectl -n ui get service ui-nlb -o jsonpath='{.status.loadBalancer.ingress[*].hostname}{"\n"}'
k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com
```

Pod `carts` có thể tiếp cận dịch vụ DynamoDB và giỏ hàng bây giờ có thể truy cập!

![Cart](../../../static/img/sample-app-screens/shopping-cart.png)

Sau khi vai trò IAM của AWS được liên kết với Service Account, bất kỳ Pods mới nào được tạo bằng Service Account đó sẽ bị chặn bởi [webhook EKS Pod Identity](https://github.com/aws/amazon-eks-pod-identity-webhook). Webhook này chạy trên bảng điều khiển của cụm Amazon EKS và được quản lý hoàn toàn bởi AWS. Hãy xem xét kỹ hơn pod `carts` mới để xem các biến môi trường mới.

```bash
$ kubectl -n carts exec deployment/carts -- env | grep AWS
AWS_STS_REGIONAL_ENDPOINTS=regional
AWS_DEFAULT_REGION=us-west-2
AWS_REGION=us-west-2
AWS_CONTAINER_CREDENTIALS_FULL_URI=http://169.254.170.23/v1/credentials
AWS_CONTAINER_AUTHORIZATION_TOKEN_FILE=/var/run/secrets/pods.eks.amazonaws.com/serviceaccount/eks-pod-identity-token
```

Những điều đáng chú ý là:

* `AWS_DEFAULT_REGION` Vùng được tự động đặt cùng với cụm EKS của chúng ta
* `AWS_STS_REGIONAL_ENDPOINTS` các điểm cuối STS vùng được cấu hình để tránh đặt áp lực quá lớn lên điểm cuối toàn cầu tại `us-east-1`
* Biến `AWS_CONTAINER_CREDENTIALS_FULL_URI` cho biết cho các SDK của AWS cách lấy thông tin xác thực sử dụng [HTTP credential provider](https://docs.aws.amazon.com/sdkref/latest/guide/feature-container-credentials.html). Điều này có nghĩa là EKS Pod Identity không cần phải tiêm thông tin xác thực thông qua một cặp `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`, và thay vào đó các SDK có thể có thông tin xác thực tạm thời được cung cấp cho họ thông qua cơ chế EKS Pod Identity. Bạn có thể đọc thêm về cách hoạt động này trong [tài liệu AWS](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html).

Bạn đã thành công cấu hình Pod Identity trong Ứng dụng của bạn!!!