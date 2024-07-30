
---
title: "Image Security"
date: "`r Sys.Date()`"
weight: 9
chapter: false
pre: "<b> 1.9 </b>"
---

#### Image Security

Trong phần này, chúng ta sẽ xem xét về Image Security

#### Image Security
   
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: nginx
  ```
  
  
#### Private Registry 

- Để đăng nhập vào registry
  ```
  $ docker login private-registry.io
  ```
- Chạy ứng dụng bằng hình ảnh có sẵn tại registry riêng tư
  ```
  $ docker run private-registry.io/apps/internal-app
  ```
  
  
- Để truyền thông tin xác thực cho docker không được gắn nhãn trên nút làm việc, trước tiên chúng ta tạo một đối tượng bí mật chứa thông tin xác thực.
  ```bash
  $ kubectl create secret docker-registry regcred \
    --docker-server=private-registry.io \ 
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry-user@org.com
  ```
- Sau đó, chúng ta chỉ định bí mật trong tệp định nghĩa pod của chúng ta trong phần imagePullSecret
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
    imagePullSecrets:
    - name: regcred
  ```

  
  #### Tài Liệu Tham Khảo K8s
  - https://kubernetes.io/docs/concepts/containers/images/