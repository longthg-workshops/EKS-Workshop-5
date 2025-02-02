---
title: "Storing secrets in AWS Secrets Manager"
weight: 1
chapter: false
pre: "<b> 5.2.1 </b>"
---

####  Storing secrets in AWS Secrets Manager


Trước tiên, chúng ta cần lưu một khoá bí mật trong AWS Secrets Manager, hãy làm điều đó bằng cách sử dụng AWS CLI:

```bash
$ export SECRET_SUFFIX=$(openssl rand -hex 4)
$ export SECRET_NAME="$EKS_CLUSTER_NAME-catalog-secret-${SECRET_SUFFIX}"
$ aws secretsmanager create-secret --name "$SECRET_NAME" \
  --secret-string '{"username":"catalog_user", "password":"default_password"}' --region $AWS_REGION
{
    "ARN": "arn:aws:secretsmanager:$AWS_REGION:$AWS_ACCOUNT_ID:secret:$EKS_CLUSTER_NAME/catalog-secret-ABCdef",
    "Name": "eks-workshop/static-secret",
    "VersionId": "7e0b352d-6666-4444-aaaa-cec1f1d2df1b"
}
```

Lệnh trên đang lưu một khoá bí mật với nội dung key/value được mã hóa JSON, cho các thông tin `username` và `password`.

Xác thực bí mật mới đã lưu trong [AWS Secrets Manager Console](https://console.aws.amazon.com/secretsmanager/listsecrets) hoặc sử dụng lệnh này:

```bash
$ aws secretsmanager describe-secret --secret-id "$SECRET_NAME"
{
    "ARN": "arn:aws:secretsmanager:us-west-2:068535243777:secret:eks-workshop/catalog-secret-WDD8yS",
    "Name": "eks-workshop/catalog-secret",
    "LastChangedDate": "2023-10-10T20:44:51.882000+00:00",
    "VersionIdsToStages": {
        "94d1fe43-87f5-42fb-bf28-f6b090f0ca44": [
            "AWSCURRENT"
        ]
    },
    "CreatedDate": "2023-10-10T20:44:51.439000+00:00"
}
```