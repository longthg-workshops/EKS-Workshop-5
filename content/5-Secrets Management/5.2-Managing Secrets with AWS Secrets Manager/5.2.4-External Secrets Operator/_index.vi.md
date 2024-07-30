---
title: "External Secrets Operator"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 5.2.4 </b>"
---

#### External Secrets Operator (Trình vận hành secret từ bên thứ ba)

Bây giờ chúng ta có thể khám phá tích hợp với Secrets Managed sử dụng External Secrets operator. Điều này đã được cài đặt trong cluster EKS của chúng ta:

```bash
$ kubectl -n external-secrets get pods
NAME                                                READY   STATUS    RESTARTS   AGE
external-secrets-6d95d66dc8-5trlv                   1/1     Running   0          7m
external-secrets-cert-controller-774dff987b-krnp7   1/1     Running   0          7m
external-secrets-webhook-6565844f8f-jxst8           1/1     Running   0          7m
$ kubectl -n external-secrets get sa 
NAME                  SECRETS   AGE
default               0         7m
external-secrets-sa   0         7m
```

Như chúng ta có thể thấy, có một ServiceAccount có tên `external-secrets-sa` được liên kết với một vai trò IAM thông qua [IRSA](../../iam-roles-for-service-accounts/), với quyền truy cập vào AWS Secrets Manager để lấy thông tin về secrets.

```bash
$ kubectl -n external-secrets describe sa external-secrets-sa | grep Annotations
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::068535243777:role/eks-workshop-external-secrets-sa-irsa
```

Ngoài ra, chúng ta cần tạo một tài nguyên cluster mới có tên là `ClusterSecretStore`, đây là một SecretStore trên toàn cụm có thể được tham chiếu bởi tất cả ExternalSecrets từ mọi namespace.

```file
manifests/modules/security/secrets-manager/cluster-secret-store.yaml
```

```bash
$ cat ~/environment/eks-workshop/modules/security/secrets-manager/cluster-secret-store.yaml \
  | envsubst | kubectl apply -f -
```

Hãy xem xét kỹ hơn về các thông số của tài nguyên mới được tạo.

```bash
$ kubectl get clustersecretstores.external-secrets.io 
NAME                   AGE   STATUS   CAPABILITIES   READY
cluster-secret-store   81s   Valid    ReadWrite      True
$ kubectl get clustersecretstores.external-secrets.io cluster-secret-store  -o yaml | yq '.spec'
provider:
  aws:
    auth:
      jwt:
        serviceAccountRef:
          name: external-secrets-sa
          namespace: external-secrets
    region: us-west-2
    service: SecretsManager

```

Bạn có thể thấy ở đây, nó đang sử dụng một [JSON Web Token (jwt)](https://jwt.io/), được tham chiếu đến ServiceAccount mà chúng ta vừa kiểm tra, để đồng bộ với AWS Secrets Manager.

Hãy tiếp tục và tạo một `ExternalSecret` mô tả những dữ liệu nào sẽ được lấy từ AWS Secrets Manager, cách dữ liệu sẽ được biến đổi và lưu trữ như một Kubernetes Secret. Sau đó, chúng ta có thể patch Deployment `catalog` của mình để sử dụng External Secret làm nguồn cho thông tin xác thực.

```kustomization
modules/security/secrets-manager/external-secrets/kustomization.yaml
Deployment/catalog
ExternalSecret/catalog-external-secret
```

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/security/secrets-manager/external-secrets/ \
  | envsubst | kubectl apply -f-
$ kubectl rollout status -n catalog deployment/catalog --timeout=120s
```

Kiểm tra tài nguyên `ExternalSecret` mới tạo.

```bash
$ kubectl -n catalog get externalsecrets.external-secrets.io
NAME                      STORE                  REFRESH INTERVAL   STATUS         READY
catalog-external-secret   cluster-secret-store   1h                 SecretSynced   True
```

Xác minh rằng tài nguyên có trạng thái `SecretSynced`, có nghĩa là nó đã được đồng bộ thành công từ AWS Secrets Manager. Hãy xem xét kỹ hơn các thông số của tài nguyên này.

```bash
$ kubectl -n catalog get externalsecrets.external-secrets.io catalog-external-secret -o yaml | yq '.spec'
dataFrom:
  - extract:
      conversionStrategy: Default
      decodingStrategy: None
      key: eks-workshop/catalog-secret
refreshInterval: 1h
secretStoreRef:
  kind: ClusterSecretStore
  name: cluster-secret-store
target:
  creationPolicy: Owner
  deletionPolicy: Retain
```

Chú ý đến các tham số `key` và `secretStoreRef` trỏ đến secret mà chúng ta đã lưu trữ trên AWS Secrets Manager, và `ClusterSecretStore` đã tạo trước đó. Cũng như `refreshInterval` được đặt là 1 giờ, có nghĩa là giá trị từ secret này sẽ được kiểm tra và cập nhật mỗi giờ.

Nhưng làm thế nào chúng ta sử dụng ExternalSecret này trong các Pod của mình? Sau khi chúng ta tạo tài nguyên này, nó tự động tạo ra một secret Kubernetes có cùng tên trong namespace.

```bash
$ kubectl -n catalog get secrets
NAME                      TYPE     DATA   AGE
catalog-db                Opaque   2      21h
catalog-external-secret   Opaque   2      1m
catalog-secret            Opaque   2      5h40m
```

Hãy xem xét kỹ hơn về secret này.

```bash
$ kubectl -n catalog get secret catalog-external-secret -o yaml | yq '.metadata.ownerReferences'
- apiVersion: external-secrets.io/v1beta1
  blockOwnerDeletion: true
  controller: true
  kind: ExternalSecret
  name: catalog-external-secret
  uid: b8710001-366c-44c2-8e8d-462d85b1b8d7
```

Thấy rằng nó có một `ownerReference` trỏ đến External Secrets Operator.

Bây giờ kiểm tra rằng pod `catalog` đã được cập nhật với các giá trị từ secret mới này, và...

Nó đã hoạt động!

```bash
$ kubectl -n catalog get pods
NAME                       READY   STATUS    RESTARTS   AGE
catalog-777c4d5dc8-lmf6v   1/1     Running   0          1m
catalog-mysql-0            1/1     Running   0          24h
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.containers[] | .env'
- name: DB_USER
  valueFrom:
    secretKeyRef:
      key: username
      name: catalog-external-secret
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      key: password
      name: catalog-external-secret
```

### Kết luận

Tóm lại không có lựa chọn tốt nhất giữa **AWS Secrets and Configuration Provider (ASCP)** và **External Secrets Operator (ESO)** để quản lý secrets được lưu trữ trên **AWS Secrets Manager**.

Cả hai công cụ đều có những ưu điểm cụ thể của chúng, ví dụ, ASCP có thể giúp bạn tránh tiết lộ secrets như biến môi trường, gắn chúng như volumes trực tiếp từ AWS Secrets Manager vào một Pod, nhược điểm là cần phải quản lý những volumes đó. Trong khi đó, ESO làm cho việc quản lý vòng đời của Kubernetes Secrets dễ dàng hơn, có cũng một SecretStore trên toàn cụm, tuy nhiên nó không cho phép bạn sử dụng Secrets như volumes. Tất cả phụ thuộc vào trường hợp sử dụng của bạn, và có cả hai có thể mang lại cho bạn nhiều linh hoạt và bảo mật hơn với quản lý Secrets.