---
title: "Áp dụng IRSA"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 3.4 </b>"
---

#### Áp dụng IRSA
 
Để sử dụng IAM Role cho các Service Account trong EKS cluster của bạn, một `IAM OIDC Identity Provider` phải được tạo và liên kết với một cụm. Một OIDC đã được cung cấp và liên kết với EKS clusterEKS của bạn:

Đi đến Identity Providers trong Bảng điều khiển IAM:

[https://console.aws.amazon.com/iamv2/home#/identity_providers](https://console.aws.amazon.com/iamv2/home#/identity_providers)

Bạn sẽ thấy một nhà cung cấp OIDC đã được tạo cho cluster EKS của bạn:

![Nhà cung cấp IAM OIDC](./assets/oidc.png)

Một lựa chọn khác là sử dụng AWS CLI để xác minh `IAM OIDC Identity Provider`.

```bash
$ aws iam list-open-id-connect-providers

{
    "OpenIDConnectProviderList": [
        {
            "Arn": "arn:aws:iam::012345678901:oidc-provider/oidc.eks.us-east-2.amazonaws.com/id/7185F12D2B62B8DA97B0ECA713F66C86"
        }
    ]
}
```

Và xác minh sự liên kết của nó với EKS cluster của chúng tôi.

```bash
$ aws eks describe-cluster --name ${EKS_CLUSTER_NAME} --query 'cluster.identity'

{
    "oidc": {
        "issuer": "https://oidc.eks.us-west-2.amazonaws.com/id/7185F12D2B62B8DA97B0ECA713F66C86"
    }
}
```


Một IAM Role cung cấp các quyền cần thiết cho dịch vụ `carts` để đọc và ghi vào bảng DynamoDB đã được tạo cho bạn. Bạn có thể xem chính sách như sau:

```bash
$ aws iam get-policy-version \
  --version-id v1 --policy-arn \
  --query 'PolicyVersion.Document' \
  arn:aws:iam::${AWS_ACCOUNT_ID}:policy/${EKS_CLUSTER_NAME}-carts-dynamo | jq .
{
  "Statement": [
    {
      "Action": "dynamodb:*",
      "Effect": "Allow",
      "Resource": [
        "arn:aws:dynamodb:us-west-2:1234567890:table/eks-workshop-carts",
        "arn:aws:dynamodb:us-west-2:1234567890:table/eks-workshop-carts/index/*"
      ],
      "Sid": "AllAPIActionsOnCart"
    }
  ],
  "Version": "2012-10-17"
}
```

Role cũng đã được cấu hình với mối quan hệ tin cậy phù hợp cho phép nhà cung cấp OIDC liên kết với EKS clusterEKS của chúng tôi giả định Role này miễn là chủ đề là Service Account cho thành phần carts. Bạn có thể xem như sau:

```bash
$ aws iam get-role \
  --query 'Role.AssumeRolePolicyDocument' \
  --role-name ${EKS_CLUSTER_NAME}-carts-dynamo | jq .
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::1234567890:oidc-provider/oidc.eks.us-west-2.amazonaws.com/id/22E1209C76AE64F8F612F8E703E5BBD7"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-west-2.amazonaws.com/id/22E1209C76AE64F8F612F8E703E5BBD7:sub": "system:serviceaccount:carts:carts"
        }
      }
    }
  ]
}
```

Tất cả những gì còn lại là để cấu hình lại đối tượng Service Account liên kết với ứng dụng `carts`, thêm chú thích cần thiết vào đó, để IRSA có thể cung cấp quyền hợp lệ cho Pods sử dụng IAM Role ở trên. 
Hãy xác minh SA liên kết với Triển khai `carts`.

```bash
$ kubectl -n carts describe deployment carts | grep 'Service Account'
  Service Account:  cart
```

Bây giờ hãy kiểm tra giá trị của `CARTS_IAM_ROLE` sẽ cung cấp ARN của IAM Role cho chú thích Service Account.

```bash
$ echo $CARTS_IAM_ROLE
arn:aws:iam::1234567890:role/eks-workshop-carts-dynamo
```

Sau khi chúng ta đã xác minh IAM Role sẽ được sử dụng, chúng ta có thể chạy Kustomize để áp dụng thay đổi lên Service Account.

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/security/irsa/service-account \
  | envsubst | kubectl apply -f-
```

Điều này sẽ sửa đổi Service Account như sau:

```kustomization
modules/security/irsa/service-account/carts-serviceAccount.yaml
ServiceAccount/carts
```

Xác minh xem Service Account đã được chú thích hay không.

```bash
$ kubectl describe sa carts -n carts | grep Annotations
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::1234567890:role/eks-workshop-carts-dynamo
```

Với Service Account được cập nhật, bây giờ chúng ta chỉ cần khởi động lại Pod carts để nó nhận thay đổi:

```

bash
$ kubectl rollout restart -n carts deployment/carts
deployment.apps/carts restarted
$ kubectl rollout status -n carts deployment/carts
```

