---
title: "ASCP"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 5.2.2 </b>"
---

#### AWS Secrets and Configuration Provider (ASCP)

Khi chúng ta thực thi kịch bản `prepare-environment` được mô tả trong một [bước trước đó](./index.md), nó đã cài đặt thành công AWS Secrets and Configuration Provider (ASCP) cho Kubernetes Secrets Store CSI Driver cần thiết cho bài thực hành này.

Tiếp theo, hãy kiểm tra xem các addons đã được triển khai hay chưa.

Kiểm tra DaemonSet và các Pods tương ứng của Kubernetes Secrets Store CSI Driver.

```bash
$ kubectl -n secrets-store-csi-driver get pods,daemonsets -l app=secrets-store-csi-driver
NAME                                                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/csi-secrets-store-secrets-store-csi-driver   3         3         3       3            3           kubernetes.io/os=linux   3m57s

NAME                                                   READY   STATUS    RESTARTS   AGE
pod/csi-secrets-store-secrets-store-csi-driver-bzddm   3/3     Running   0          3m57s
pod/csi-secrets-store-secrets-store-csi-driver-k7m6c   3/3     Running   0          3m57s
pod/csi-secrets-store-secrets-store-csi-driver-x2rs4   3/3     Running   0          3m57s
```

Kiểm tra DaemonSet và các Pods tương ứng của Driver CSI Secrets Store Provider cho AWS.

```bash
$ kubectl -n kube-system get pods,daemonset -l "app=secrets-store-csi-driver-provider-aws"  
NAME                                                   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/secrets-store-csi-driver-provider-aws   3         3         3       3            3           kubernetes.io/os=linux   2m3s

NAME                                              READY   STATUS    RESTARTS   AGE
pod/secrets-store-csi-driver-provider-aws-4jf8f   1/1     Running   0          2m2s
pod/secrets-store-csi-driver-provider-aws-djtf5   1/1     Running   0          2m2s
pod/secrets-store-csi-driver-provider-aws-dzg9r   1/1     Running   0          2m2s
```

Để cung cấp quyền truy cập vào các khoá bí mật được lưu trữ trong AWS Secrets Manager qua Kubernetes Secrets Store CSI Driver, bạn cần một *SecretProviderClass*, đây là một tài nguyên tùy chỉnh có phạm vi được sử dụng để cung cấp cấu hình của trình điều khiển và các tham số cụ thể phù hợp với thông tin trong AWS Secrets Manager.

```file
manifests/modules/security/secrets-manager/secret-provider-class.yaml
```

Trong tài nguyên trên, chúng ta có hai cấu hình chính mà chúng ta nên tập trung. Vì vậy, hãy tiến hành tạo tài nguyên để khám phá các thông số đó.

```bash
$ cat ~/environment/eks-workshop/modules/security/secrets-manager/secret-provider-class.yaml \
  | envsubst | kubectl apply -f -
```

Tham số *objects*, trỏ đến một khoá bí mật có tên là `eks-workshop/catalog-secret` mà chúng ta sẽ lưu trữ trong AWS Secrets Manager trong bước tiếp theo. Lưu ý rằng chúng tôi đang sử dụng [jmesPath](https://jmespath.org/) để trích xuất một cặp key-value cụ thể từ bí mật được định dạng JSON.

```bash
$ kubectl get secretproviderclass -n catalog catalog-spc -o yaml | yq '.spec.parameters.objects'

- objectName: "eks-workshop/catalog-secret"
  objectType: "secretsmanager"
  jmesPath:
    - path: username
      objectAlias: username
    - path: password
      objectAlias: password
```

Và *secretObjects*, sẽ tạo và/hoặc đồng bộ hóa một Kubernetes Secrets với dữ liệu từ bí mật được lưu trữ trong AWS Secrets Manager. Điều này có nghĩa là khi được gắn vào một Pod, SecretProviderClass sẽ tạo ra một Kubernetes Secrets, nếu nó chưa tồn tại, và đồng bộ hóa các giá trị được lưu trữ trong AWS Secrets Manager với Kubernetes Secrets này, trong trường hợp của chúng ta, nó có tên là `catalog-secret`.

```bash
$ kubectl get secretproviderclass -n catalog catalog-spc -o yaml | yq '.spec.secretObjects'

- data:
    - key: username
      objectName: username
    - key: password
      objectName: password
  secretName: catalog-secret
  type: Opaque
```

