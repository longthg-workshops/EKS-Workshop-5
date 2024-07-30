---
title: "Niêm phong khoá bí mật"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 5.3.3 </b>"
---

### Niêm phong khoá bí mật

#### Khám phá Pod trong danh mục

Việc triển khai `catalog` trong Namespace `catalog` truy cập các giá trị cơ sở dữ liệu sau từ secret `catalog-db` thông qua biến môi trường:

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
- name: DB_NAME
  valueFrom:
    configMapKeyRef:
      key: name
      name: catalog
- name: DB_READ_ENDPOINT
  valueFrom:
    secretKeyRef:
      key: endpoint
      name: catalog-db
- name: DB_ENDPOINT
  valueFrom:
    secretKeyRef:
      key: endpoint
      name: catalog-db
```

Khi khám phá Secret `catalog-db`, chúng ta thấy rằng nó chỉ được mã hóa bằng base64, có thể dễ dàng giải mã như sau, gây khó khăn cho các biểu mẫu secrets khi tham gia vào quy trình làm việc GitOps.

```file
manifests/base-application/catalog/secrets.yaml
```

```bash
$ kubectl -n catalog get secrets catalog-db --template {{.data.username}} | base64 -d
catalog_user%                                                                                                                                                                                                   
$ kubectl -n catalog get secrets catalog-db --template {{.data.password}} | base64 -d
default_password% 
```

Hãy tạo một secret mới `catalog-sealed-db`. Chúng ta sẽ tạo một tập tin mới `new-catalog-db.yaml` với các khóa và giá trị giống như Secret `catalog-db`.

```file
manifests/modules/security/sealed-secrets/new-catalog-db.yaml
```

Bây giờ, hãy tạo các biểu mẫu SealedSecret YAML với kubeseal.

```bash
$ kubeseal --format=yaml < ~/environment/eks-workshop/modules/security/sealed-secrets/new-catalog-db.yaml \
  > /tmp/sealed-catalog-db.yaml
```

Hoặc, bạn có thể lấy khóa công khai từ bộ điều khiển và sử dụng nó ngoại tuyến để niêm phong Secret của bạn:

```bash test=false
$ kubeseal --fetch-cert > /tmp/public-key-cert.pem
$ kubeseal --cert=/tmp/public-key-cert.pem --format=yaml < ~/environment/eks-workshop/modules/security/sealed-secrets/new-catalog-db.yaml \
  > /tmp/sealed-catalog-db.yaml
```

Nó sẽ tạo một sealed-secret với nội dung sau:

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: catalog-sealed-db
  namespace: catalog
spec:
  encryptedData:
    password: AgBe(...)R91c
    username: AgBu(...)Ykc=
  template:
    data: null
    metadata:
      creationTimestamp: null
      name: catalog-sealed-db
      namespace: catalog
    type: Opaque
```

Hãy triển khai SealedSecret này lên cụm EKS của bạn:

```bash
$ kubectl apply -f /tmp/sealed-catalog-db.yaml
```

Nhật ký của bộ điều khiển cho thấy rằng nó nhận SealedSecret tùy chỉnh vừa được triển khai, mở niêm phong nó để tạo ra một Secret thường.

```bash
$ kubectl logs deployments/sealed-secrets-controller -n kube-system

2022/11/07 04:28:27 Updating catalog/catalog-sealed-db
2022/11/07 04:28:27 Event(v1.ObjectReference{Kind:"SealedSecret", Namespace:"catalog", Name:"catalog-sealed-db", UID:"a2ae3aef-f475-40e9-918c-697cd8cfc67d", APIVersion:"bitnami.com/v1alpha1", ResourceVersion:"23351", FieldPath:""}): type: 'Normal' reason: 'Unsealed' SealedSecret unsealed successfully
```

Kiểm tra xem Secret `catalog-sealed-db` đã được mở niêm phong từ SealedSecret và được triển khai bởi bộ điều khiển vào namespace `secure-secrets`.

```bash
$ kubectl get secret -n catalog catalog-sealed-db 

NAME                       TYPE     DATA   AGE
catalog-sealed-db          Opaque   4      7m51s
```

Hãy triển khai lại triển khai **catalog** đọc từ Secret trên. Chúng tôi đã cập nhật triển khai `catalog` để đọc Secret `catalog-sealed-db` như sau:

```kustomization
modules/security/sealed-secrets/deployment.yaml
Deployment/catalog
```

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/security/sealed-secrets
$ kubectl rollout status -n catalog deployment/catalog --timeout 30s
```

**catalog-sealed-db** là một tài nguyên SealedSecret an toàn để lưu trữ trong một kho Git cùng với các biểu mẫu YAML liên quan đến các tài nguyên Kubernetes khác như DaemonSets, Deployments, ConfigMaps vv. được triển khai trong cụm của bạn. Bạn sau đó có thể sử dụng quy trình làm việc GitOps để quản lý việc triển khai các tài nguyên này lên cụm của bạn.