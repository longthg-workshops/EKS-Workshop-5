---
title: "Cài đặt Sealed Secrets"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 5.3.2 </b>"
---

#### Cài đặt Sealed Secrets


Lệnh `kubeseal` được sử dụng để tương tác với bộ điều khiển sealed secrets, và đã được cài đặt sẵn trong Cloud9.

Điều đầu tiên chúng ta sẽ làm là cài đặt bộ điều khiển sealed secrets trong cụm EKS:

```bash
$ kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml
$ kubectl wait --for=condition=Ready --timeout=30s pods -l name=sealed-secrets-controller -n kube-system
```

Bây giờ chúng ta sẽ kiểm tra trạng thái của pod

```bash
$ kubectl get pods -n kube-system -l name=sealed-secrets-controller
sealed-secrets-controller-77747c4b8c-snsxp      1/1     Running   0          5s
```

Các bản log của bộ điều khiển sealed secrets cho thấy rằng bộ điều khiển cố gắng tìm bất kỳ khóa riêng nào hiện có trong quá trình khởi động. Nếu không có khóa riêng nào được tìm thấy, sau đó nó sẽ tạo một khoá bí mật mới với các chi tiết chứng chỉ.

```bash
$ kubectl logs deployments/sealed-secrets-controller -n kube-system
controller version: 0.18.0
2022/10/18 09:17:01 Starting sealed-secrets controller version: 0.18.0
2022/10/18 09:17:01 Searching for existing private keys
2022/10/18 09:17:02 New key written to kube-system/sealed-secrets-keyvkl9w
2022/10/18 09:17:02 Certificate is 
-----BEGIN CERTIFICATE-----
MIIEzTCCArWgAwIBAgIRAPsk+UrW9GlPu4gXN1qKqGswDQYJKoZIhvcNAQELBQAw
ADAeFw0yMjEwMTgwOTE3MDJaFw0zMjEwMTUwOTE3MDJaMAAwggIiMA0GCSqGSIb3
(...)
q5P11EvxPBfIt9xDx5Jz4JWp5M7wWawGaeBqTmTDbSkc
-----END CERTIFICATE-----

2022/10/18 09:17:02 HTTP server serving on :8080
```

Chúng ta có thể xem nội dung của khoá bí mật chứa khóa niêm phong dưới dạng cặp khóa công khai / khóa riêng trong định dạng YAML như sau:

```bash
$ kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    tls.crt: LS0tL(...)LQo=
    tls.key: LS0tL(...)LS0K
  kind: Secret
  metadata:
    creationTimestamp: "2022-10-18T09:17:02Z"
    generateName: sealed-secrets-key
    labels:
      sealedsecrets.bitnami.com/sealed-secrets-key: active
    name: sealed-secrets-keyvkl9w
    namespace: kube-system
    resourceVersion: "129381"
    uid: 23f5e70c-2537-4c38-a85c-b410f1dcf9a6
  type: kubernetes.io/tls
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```