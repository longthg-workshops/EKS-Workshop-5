---
title: "Bảo vệ khoá bí mật bằng Seal Secrets"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 5.3 </b>"
---

#### Bảo vệ khoá bí mật bằng Seal Secrets


Dự án [Sealed Secrets](https://docs.bitnami.com/tutorials/sealed-secrets) không liên quan đến Dịch vụ AWS mà là một công cụ mã nguồn mở từ bên thứ ba của [Btinami Labs](https://bitnami.com/)

Chuẩn bị môi trường của bạn cho phần này:

```bash timeout=300 wait=30
$ prepare-environment security/sealed-secrets
```

[Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) cung cấp một cơ chế để mã hóa một đối tượng Secret để nó an toàn để lưu trữ - thậm chí là vào một kho lưu trữ công cộng. Một SealedSecret chỉ có thể được giải mã bởi bộ điều khiển đang chạy trong cụm Kubernetes và không ai khác có thể lấy được Secret gốc từ một SealedSecret.

Trong chương này, bạn sẽ sử dụng SealedSecrets để mã hóa các tài liệu YAML liên quan đến Secrets của Kubernetes cũng như có thể triển khai các Secrets đã được mã hóa này lên các cụm EKS của bạn bằng các luồng làm việc thông thường với các công cụ như [kubectl](https://kubernetes.io/docs/reference/kubectl/).
