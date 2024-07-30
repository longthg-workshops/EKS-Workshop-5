---
title: "Xác nhận khả năng truy cập DynamoDB"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 3.5 </b>"
---

#### Xác nhận khả năng truy cập DynamoDB

Bây giờ, với `carts` Service Account đã được chú thích với vai trò IAM được ủy quyền, pod `carts` có quyền truy cập vào bảng DynamoDB. Truy cập cửa hàng web lại và điều hướng đến giỏ hàng.

```bash
$ kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}"
k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com
```

Pod `carts` có thể truy cập vào dịch vụ DynamodDB và giỏ hàng bây giờ có thể truy cập được!

<browser url="http://k8s-ui-uinlb-647e781087-6717c5049aa96bd9.elb.us-west-2.amazonaws.com/cart">
<img src={require('@site/static/img/sample-app-screens/shopping-cart.png').default}/>
</browser>

Hãy xem xét kỹ hơn Pod mới `carts` để xem đang xảy ra điều gì.

```bash
$ kubectl -n carts exec deployment/carts -- env | grep AWS
AWS_STS_REGIONAL_ENDPOINTS=regional
AWS_DEFAULT_REGION=us-west-2
AWS_REGION=us-west-2
AWS_ROLE_ARN=arn:aws:iam::1234567890:role/eks-workshop-carts-dynamo
AWS_WEB_IDENTITY_TOKEN_FILE=/var/run/secrets/eks.amazonaws.com/serviceaccount/token
```

Các biến môi trường này không được truyền vào bằng cách sử dụng một thứ tương tự ConfigMap hoặc cấu hình trực tiếp trên Deployment. Thay vào đó, các biến này đã được thiết lập bởi IRSA tự động để cho phép SDK AWS nhận các thông tin xác thực tạm thời từ dịch vụ AWS STS.

Những điều đáng chú ý là:

* Khu vực được tự động đặt cùng với cụm EKS của chúng ta
* Các điểm cuối khu vực STS được cấu hình để tránh đặt áp lực quá nhiều lên điểm cuối toàn cầu ở `us-east-1`
* ARN của vai trò khớp với vai trò mà chúng ta đã sử dụng để chú thích ServiceAccount của Kubernetes của chúng ta trước đó

Cuối cùng, biến `AWS_WEB_IDENTITY_TOKEN_FILE` cho SDK AWS biết cách nhận thông tin xác thực bằng cách sử dụng liên minh danh tính web. Điều này có nghĩa là IRSA không cần phải tiêm các thông tin xác thực thông qua một cặp `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`, và thay vào đó các SDK có thể nhận được thông tin xác thực tạm thời thông qua cơ chế OIDC. Bạn có thể đọc thêm về cách hoạt động này trong [tài liệu AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc.html).