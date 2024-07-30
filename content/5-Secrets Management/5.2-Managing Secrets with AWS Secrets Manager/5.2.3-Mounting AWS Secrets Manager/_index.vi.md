---
title: "Gắn secret trong AWS Secrets Manager lên Pod Kubernetes"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 5.2.3 </b>"
---

#### Gắn secret trong AWS Secrets Manager lên Pod Kubernetes

Bây giờ chúng ta đã lưu một Secret trong AWS Secrets Manager và đồng bộ hóa với một Secret trong Kubernetes, hãy gắn nó vào trong Pod. Đầu tiên, chúng ta nên xem qua `catalog` Deployment và các Secrets hiện có trong namespace `catalog`.

Deployment `catalog` truy cập các thông tin xác thực cơ sở dữ liệu sau từ secret `catalog-db` thông qua biến môi trường:

- `DB_USER`
- `DB_PASSWORD`

```bash
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.containers[] | .env'

- name: DB_USER
  valueFrom:
    secretKeyRef:
      key: username
      name: catalog-db
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      key: password
      name: catalog-db
```

Chú ý rằng Deployment `catalog` không có bất kỳ `volumes` hoặc `volumeMounts` nào ngoại trừ một `emptyDir` được gắn vào `/tmp`.

```bash
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.volumes'
- emptyDir:
    medium: Memory
  name: tmp-volume
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.containers[] | .volumeMounts'
- mountPath: /tmp
  name: tmp-volume
```

Hãy áp dụng các thay đổi trong Deployment `catalog` để sử dụng secret được lưu trong AWS Secrets Manager làm nguồn của thông tin xác thực.

```kustomization
modules/security/secrets-manager/mounting-secrets/kustomization.yaml
Deployment/catalog
```

Ở đây, chúng ta sẽ gắn secret trong AWS Secrets Manager bằng cách sử dụng CSI driver với SecretProviderClass chúng ta đã xác thực trước đó trên mountPath `/etc/catalog-secret` bên trong Pod. Khi điều này xảy ra, AWS Secrets Manager sẽ đồng bộ nội dung của secret đã lưu với Amazon EKS và tạo một Secret trong Kubernetes với cùng nội dung có thể được tiêu thụ như biến môi trường trong Pod.

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/security/secrets-manager/mounting-secrets/ \
  | envsubst | kubectl apply -f-
$ kubectl rollout status -n catalog deployment/catalog --timeout=120s
```

Hãy xác thực các thay đổi đã được thực hiện trong Namespace `catalog`.

Trong Deployment, chúng ta sẽ có thể kiểm tra rằng nó có một `volume` mới và `volumeMount` tương ứng trỏ đến CSI Secret Store Driver, và được gắn vào đường dẫn `/etc/catalog-secrets`.

```bash
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.volumes'
- csi:
    driver: secrets-store.csi.k8s.io
    readOnly: true
    volumeAttributes:
      secretProviderClass: catalog-spc
  name: catalog-secret
- emptyDir:
    medium: Memory
  name: tmp-volume
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.containers[] | .volumeMounts'                                                                                                                                                                             
- mountPath: /etc/catalog-secret
  name: catalog-secret
  readOnly: true
- mountPath: /tmp
  name: tmp-volume
```

Các Secrets được gắn kết là một cách tốt để có thông tin nhạy cảm có sẵn dưới dạng một tệp trong hệ thống tệp của một hoặc nhiều container trong Pod. Một số lợi ích là không tiết lộ giá trị của secret như biến môi trường và khi một volume chứa dữ liệu từ một Secret và khi Secret đó được cập nhật, Kubernetes sẽ theo dõi và cập nhật dữ liệu trong volume.

Bạn có thể xem nội dung của Secret đã được gắn kết bên trong Pod của bạn.

```bash
$ kubectl -n catalog exec deployment/catalog -- ls /etc/catalog-secret/ 
eks-workshop-catalog-secret  password  username
$ kubectl -n catalog exec deployment/catalog -- cat /etc/catalog-secret/${SECRET_NAME}
{"username":"catalog_user", "password":"default_password"}
$ kubectl -n catalog exec deployment/catalog -- cat /etc/catalog-secret/username
catalog_user
$ kubectl -n catalog exec deployment/catalog -- cat /etc/catalog-secret/password
default_password
```

Có 3 tệp trong mountPath `/etc/catalog-secret`.
1. `eks-workshop-catalog-secret`: Giá trị của secret dưới dạng JSON.
2. `password`: Giá trị đã được lọc và định dạng theo jmesPath của mật khẩu.
3. `username`: Giá trị đã được lọc và định dạng theo jmesPath của tên người dùng.

Ngoài ra, các biến môi trường hiện đang được tiêu thụ từ Secret mới, `catalog-secret`, không tồn tại trước đó, và nó đã được tạo bởi SecretProviderClass thông qua CSI Secret Store driver.

```bash
$ kubectl -n catalog get deployment catalog -o yaml | yq '.spec.template.spec.containers[] | .env'
- name: DB_USER
  valueFrom:
    secretKeyRef:
      key: username
      name: catalog-secret
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      key: password
      name: catalog-secret
$ kubectl -n catalog get secrets
NAME             TYPE     DATA   AGE
catalog-db       Opaque   2      15h
catalog-secret   Opaque   2      43s
```

Chúng ta có thể xác nhận điều này bằng cách kiểm tra các biến môi trường trong Pod đang chạy:

```bash
$ kubectl -n catalog exec -ti deployment/catalog -- env | grep DB_
```

Bây giờ chúng ta đã có một Kubernetes Secret tích hợp hoàn toàn với AWS Secrets Manager có thể tận dụng quá trình secret rotation. Đây là một thực hành tốt cho Quản lý Secrets. Mỗi khi một secret được rotate hoặc cập nhật trên AWS Secrets Manager, chúng ta có thể triển khai một phiên bản mới của Deployment để CSI Secret Store driver có thể đồng bộ nội dung của Kubernetes Secrets với giá trị đã được xoay.