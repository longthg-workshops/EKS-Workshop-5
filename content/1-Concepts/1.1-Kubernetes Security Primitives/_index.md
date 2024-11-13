---
title: "Kubernetes Security Primitives"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

### Introduction

Too types of decisions need to be made:

- Who can access ?
- What can they do ?

![KubeSec](../../images/0001/0002.png?featherlight=false&width=90pc)

From there, we have the following processes:

- **Authentication:** Decides who can access the server API - Defined with authentication mechanisms.

- **Authorization:** After being granted access, what are they allowed to do ? - Defined with authorization mechanisms.

### TLS Certificates

All communication with the cluster, between the various components such as the ETCD Cluster, kube-controller-manager, scheduler, api server, as well as those running on the working nodes such as the kubelet and kubeproxy is secured using TLS encryption.

![EKS](../../images/0001/0003.png?featherlight=false&width=90pc)

### Network policies

Network policies determine how applications in the cluster communicate. They allow you to specify which pods can and cannot communicate with each other, as well as with the outside world.

### Examples of host security policies

- Block password-based access
- Only allow SSH access using keys

![EKS](../../images/0001/0001.png?featherlight=false&width=90pc)