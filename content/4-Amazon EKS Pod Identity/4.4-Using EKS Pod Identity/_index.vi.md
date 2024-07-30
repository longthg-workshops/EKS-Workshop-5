---
title: "Using EKS Pod Identity"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 4.4 </b>"
---

#### Using EKS Pod Identity

Để sử dụng EKS Pod Identity trong cụm của bạn, tiện ích `EKS Pod Identity Agent` phải được cài đặt trên cụm EKS của bạn. Hãy cài đặt nó bằng lệnh dưới đây.

```bash timeout=300 wait=60
$ aws eks create-addon --cluster-name $EKS_CLUSTER_NAME --addon-name eks-pod-identity-agent 
{
    "addon": {
        "addonName": "eks-pod-identity-agent",
        "clusterName": "eks-workshop",
        "status": "CREATING",
        "addonVersion": "v1.1.0-eksbuild.1",
        "health": {
            "issues": []
        },
        "addonArn": "arn:aws:eks:us-west-2:123456789000:addon/eks-workshop/eks-pod-identity-agent/9ec6cfbd-8c9f-7ff4-fd26-640dda75bcea",
        "createdAt": "2024-01-12T22:41:01.414000+00:00",
        "modifiedAt": "2024-01-12T22:41:01.434000+00:00",
        "tags": {}
    }
}

$ aws eks wait addon-active --cluster-name $EKS_CLUSTER_NAME --addon-name eks-pod-identity-agent
```

Bây giờ hãy xem tiện ích mới đã tạo ra những gì trong cụm EKS của bạn. Bạn có thể thấy một DaemonSet được triển khai trên Namespace `kube-system`, sẽ chạy một Pod trên mỗi Node trong Cụm của chúng tôi.

```bash
$ kubectl -n kube-system get daemonset eks-pod-identity-agent 
NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
eks-pod-identity-agent    3         3         3       3            3           <none>          3d21h
$ kubectl -n kube-system get pods -l app.kubernetes.io/name=eks-pod-identity-agent
NAME                           READY   STATUS    RESTARTS   AGE
eks-pod-identity-agent-4tn28   1/1     Running   0          3d21h
eks-pod-identity-agent-hslc5   1/1     Running   0          3d21h
eks-pod-identity-agent-thvf5   1/1     Running   0          3d21h
```

Một Vai trò IAM cung cấp các quyền cần thiết cho dịch vụ `carts` để đọc và ghi vào bảng DynamoDB đã được tạo khi bạn chạy kịch bản `prepare-environment` trong bước đầu tiên của mô-đun này. Bạn có thể xem chính sách như được hiển thị dưới đây.

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

Vai trò cũng đã được cấu hình với mối quan hệ tin cậy phù hợp cho phép Chủ thể Dịch vụ EKS giả định vai trò này cho Pod Identity. Bạn có thể xem như dưới đây với lệnh sau.

```bash
$ aws iam get-role \
  --query 'Role.AssumeRolePolicyDocument' \
  --role-name ${EKS_CLUSTER_NAME}-carts-dynamo | jq .
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "pods.eks.amazonaws.com"
            },
            "Action": [
                "sts:AssumeRole",
                "sts:TagSession"
            ]
        }
    ]
}
```

Tiếp theo, chúng ta sẽ sử dụng tính năng Amazon EKS Pod Identity để liên kết một vai trò IAM AWS với tài khoản dịch vụ Kubernetes sẽ được sử dụng bởi triển khai của chúng tôi. Để tạo sự kết hợp, hãy chạy lệnh sau.

```bash wait=30
$ aws eks create-pod-identity-association --cluster-name ${EKS_CLUSTER_NAME} \
  --role-arn arn:aws:iam::${AWS_ACCOUNT_ID}:role/${EKS_CLUSTER_NAME}-carts-dynamo \
  --namespace carts --service-account carts
{
    "association": {
        "clusterName": "eks-workshop",
        "namespace": "carts",
        "serviceAccount": "carts",
        "roleArn": "arn:aws:iam::123456789000:role/eks-workshop-carts-dynamo",
        "associationArn": "arn:aws::123456789000:podidentityassociation/eks-workshop/a-abcdefghijklmnop1",
        "associationId": "a-abcdefghijklmnop1",
        "tags": {},
        "createdAt": "2024-01-09T16:16:38.163000+00:00",
        "modifiedAt": "2024-01-09T16:16:38.163000+00:00"
    }
}
```

Tất cả những gì còn lại là xác minh rằng Triển khai `carts` đang sử dụng Tài khoản Dịch vụ `carts`.

```bash
$ kubectl -n carts describe deployment carts | grep 'Service Account'
  Service Account:  carts
```

Với Tài khoản Dịch vụ được xác minh, khởi động lại các Pod `carts`.

```bash hook=enable-pod-identity hookTimeout=430
$ kubectl -n carts rollout restart deployment/carts
deployment.apps/carts restarted
$ kubectl -n carts rollout status deployment/carts
Waiting

 for deployment "carts" rollout to finish: 1 old replicas are pending termination...
deployment "carts" successfully rolled out
```
