---
title: "Network-Policies (Chính sách Mạng)"
weight: 11
chapter: false
pre: "<b> 1.11 </b>"
---

### Lưu lượng vào và ra

Lưu lượng đi qua một máy chủ web frontend cho người dùng một máy chủ ứng dụng phục vụ API phía sau và một máy chủ cơ sở dữ liệu.

![traffic](/images/1/11/traffic.PNG)

- Có hai loại lưu lượng

  - Lưu lượng vào (Ingress)

  - Lưu lượng ra (Egress)

![ing1](/images/1/11/ing1.PNG)

![ing2](/images/1/11/ing2.PNG)

### Bảo mật Mạng

Bảo mật mạng là một khía cạnh quan trọng của bất kỳ cuộc triển khai nào trên Kubernetes, vì nó đảm bảo rằng dữ liệu được truyền trong cụm được bảo vệ khỏi truy cập trái phép, chặn hoặc sửa đổi. Bạn sẽ thấy mình cần một hình thức kiểm soát khả năng hiển thị và truy cập của các pod của mình đối với các pod khác và/hoặc các thực thể bên ngoài.

![nsec](/images/1/11/nsec.PNG)

### Chính sách Mạng

Chính sách mạng (_NetworkPolicy_) là một cấu trúc lấy ứng dụng làm trung tâm, cho phép bạn xác định cách một pod giao tiếp với các thực thể mạng khác trên toàn bộ mạng. Mỗi chính sách mạng được áp dụng vào kết nối đến một pod, tại mỗi đầu hoặc cả hai đầu, và không liên quan tới các kết nối khác.

![npol](/images/1/11/npol.PNG)

![npol1](/images/1/11/npol1.PNG)

#### Bộ chọn Chính sách Mạng

Nằm ở mục Selector. Chức năng này cho phép chọn các pod được áp dụng chính sách mạng dựa trên các nhãn (label) của chúng.

![npolsec](/images/1/11/npolsec.PNG)

#### Quy tắc Chính sách Mạng

Hình ảnh sau hiển thị chính sách chỉ cho phép truy cập từ một pod xác định đến cổng 3306.

![npol2](/images/1/11/npol2.PNG)

#### Tạo chính sách mạng

1. Để tạo một chính sách mạng

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

2. Áp dụng chính sách mạng vào cụm Kubernetes.

```bash
$ kubectl create -f policy-definition.yaml
```

![npol3](/images/1/11/npol3.PNG)

![npol4](/images/1/11/npol4.PNG)

### Lưu ý

Một số giải pháp mạng Kubernetes hỗ trợ chính sách mạng, một số thì không.

![note1](/images/1/11/note1.PNG)

### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/
- https://kodekloud.com/topic/developing-network-policies/