---
title: "TLS in Kubernetes"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 1.4 </b>"
---

#### TLS in Kubernetes

Đảm bảo tất cả các dịch vụ khác nhau trong cụm sử dụng chứng chỉ máy chủ và tất cả các máy khách sử dụng chứng chỉ máy khách để xác minh họ là người họ nói họ là.

- Server Certificates for Servers
- Client Certificates for Clients

![EKS](/images/0003/0001.png?featherlight=false&width=90pc)

Hãy xem các thành phần khác nhau trong cụm k8s và xác định các máy chủ và máy khách khác nhau cùng với việc ai nói chuyện với ai.

![EKS](/images/0003/0002.png?featherlight=false&width=90pc)

#### Generate Certificates

Có các công cụ khác nhau như easyrsa, openssl hoặc cfssl vv. hoặc nhiều công cụ khác để tạo chứng chỉ.

#### Certificate Authority (CA)

- Generate Keys

```
$ openssl genrsa -out ca.key 2048
```

- Generate CSR

```
$ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

- Sign certificates

```
$ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

![EKS](/images/0003/0003.png?featherlight=false&width=90pc)

#### Generating Client Certificates

- Generate Keys

```
$ openssl genrsa -out admin.key 2048
```

- Generate CSR

```
$ openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
```

- Sign certificates

```
$ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

![EKS](/images/0003/0004.png?featherlight=false&width=90pc)

- Certificate with admin privilages

```
$ openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
```