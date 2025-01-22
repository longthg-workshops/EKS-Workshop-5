---
title: "TLS in Kubernetes"
weight: 4
chapter: false
pre: "<b> 1.4 </b>"
---

### Usage and Components

Make sure all different services in the cluster use server certificates and all clients use client certificates to verify that they are who they claim to be.

- Server Certificates for Servers
- Client Certificates for Clients

![EKS](/images/0003/0001.png?featherlight=false&width=90pc)

Look at the different components in the k8s cluster and identify the different servers and clients along with who is talking to whom.

![EKS](/images/0003/0002.png?featherlight=false&width=90pc)

### Commonly used commands

#### Generate Certificates

There are different tools (e.g. easyrsa, openssl or cfssl etc.) to generate certificates.

#### Certificate Authority (CA)

- Generate a new key

```
$ openssl genrsa -out ca.key 2048
```

- Generate CSR

```
$ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

- Sign a certificate

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