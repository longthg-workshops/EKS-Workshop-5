---
title: "Ngữ cảnh bảo mật"
weight: 10
chapter: false
pre: "<b> 1.10 </b>"
---

### Bảo mật Container
 ```
 $ docker run --user=1001 ubuntu sleep 3600
 $ docker run -cap-add MAC_ADMIN ubuntu
 ```
 
 ![csec](/images/p1/p1-10/csec.PNG)
 
### Bảo mật Kubernetes
Bạn có thể chọn cấu hình các thiết lập bảo mật ở mức container hoặc mức pod.

 ![ksec](/images/p1/p1-10/ksec.PNG)

### Ngữ cảnh Bảo mật

#### Bảo mật cấp Pod
Thêm ngữ cảnh bảo mật vào pod, môi trường này được gọi là **`securityContext`** dưới phần spec.

  ```yaml
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

#### Bảo mật cấp container
  
- Đặt cùng một ngữ cảnh ở mức container, sau đó di chuyển toàn bộ phần dưới mục container.
  
  ```yaml
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

```yaml
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
**_Lưu ý:_** Capabilities chỉ áp dụng ở cấp container, không áp dụng ở cấp pod.
  
#### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/