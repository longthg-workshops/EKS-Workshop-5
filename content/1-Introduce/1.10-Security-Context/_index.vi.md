---
title: "Security-Context"
date: "`r Sys.Date()`"
weight: 10
chapter: false
pre: "<b> 1.10 </b>"
---

#### Security-Context

#### Bảo mật Ngữ cảnh
  
Trong phần này, chúng ta sẽ xem xét về bảo mật ngữ cảnh

#### Bảo mật Container
 ```
 $ docker run --user=1001 ubuntu sleep 3600
 $ docker run -cap-add MAC_ADMIN ubuntu
 ```
 
 ![csec](/images/p1/p1-10/csec.PNG)
 
#### Bảo mật Kubernetes
- Bạn có thể chọn cấu hình các thiết lập bảo mật ở mức container hoặc mức pod.

 ![ksec](/images/p1/p1-10/ksec.PNG)

#### Ngữ cảnh Bảo mật
- Để thêm ngữ cảnh bảo mật vào container và một trường gọi là **`securityContext`** dưới phần spec.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    securityContext:
      runAsUser: 1000
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
  ```
  ![sxc1](/images/p1/p1-10/sxc1.PNG)
  
- Để đặt cùng một ngữ cảnh ở mức container, sau đó di chuyển toàn bộ phần dưới mục container.
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
  ```

  
- Để thêm khả năng sử dụng tùy chọn **`capabilities`**
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities: 
          add: ["MAC_ADMIN"]
  ```
  ![cap](/images/p1/p1-10/cap.PNG)
  
  
#### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/