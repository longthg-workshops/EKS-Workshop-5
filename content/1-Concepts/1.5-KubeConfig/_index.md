---
title: "KubeConfig"
weight: 5
chapter: false
pre: "<b> 1.5 </b>"
---

In this section, we will take a look at kubeconfig in kubernetes

- Client uses the certificate file and key to query the kubernetes Rest API for a list of pods using curl.

- You can specify the same using kubectl

- We can move these information to a configuration file called kubeconfig. And the specify this file as the kubeconfig option in the command.
  
```
$ kubectl get pods --kubeconfig config
```
  
### The Kubeconfig File
The kubeconfig file has 3 sections
  - Clusters

  - Contexts

  - USers
  
To view the current file being used
```
$ kubectl config view
```
You can specify the kubeconfig file with kubectl config view with "--kubeconfig" flag
```
$ kubectl config veiw --kubeconfig=my-custom-config
```
  
How do you update your current context? Or change the current context

```bash
$ kubectl config use-context <context-name>
```
**Example:**
```bash
$ kubectl config use-context prod-user@production
```
  
- kubectl config help

```
$ kubectl config -h
```
 
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config