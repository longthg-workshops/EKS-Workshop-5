---
title: "Security Context"
weight: 10
chapter: false
pre: "<b> 1.10 </b>"
---

### Container Security
 ```
 $ docker run --user=1001 ubuntu sleep 3600
 $ docker run -cap-add MAC_ADMIN ubuntu
 ```
 
 ![csec](/images/p1/p1-10/csec.PNG)
 
### Kubernetes Security
You may choose to configure the security settings at a container level or at a pod level.

 ![ksec](/images/p1/p1-10/ksec.PNG)

###  Security Context

#### Pod-level security
Add security context on the container and a field called `**securityContext**` under the spec section.

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

#### Container-level security
  
- Set the same context at the container level, then move the whole section under container section.
  
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

To add capabilities use the `**capabilities**` option

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
**_Note:_** Capabilities is can only be apply on the container level, not on the pod level.
  
#### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/