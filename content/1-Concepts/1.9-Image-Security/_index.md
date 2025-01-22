---
title: "Image Security"
weight: 9
chapter: false
pre: "<b> 1.9 </b>"
---

In this section we will take a look at image security

### Cấu hình Image cho pod

See the following yaml structure as a reference:
   
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
```
  
### Private Registry 

- To log in the registry

```bash
$ docker login private-registry.io
```

- Run the application using the image available at the private registry

```bash
$ docker run private-registry.io/apps/internal-app
```

- To pass the credentials to the docker untaged on the worker node for that we first create a secret object with credentials in it.

```bash
$ kubectl create secret docker-registry regcred \
  --docker-server=private-registry.io \ 
  --docker-username=registry-user \
  --docker-password=registry-password \
  --docker-email=registry-user@org.com
```

- We then specify the secret inside our pod definition file under the imagePullSecret section 
  
```yaml
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
  
### Further reading
- https://kubernetes.io/docs/concepts/containers/images/