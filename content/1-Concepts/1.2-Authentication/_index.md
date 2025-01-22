---
title: "Authentication"
weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---

### Accounts

![EKS](../../../images/0001/0004.png?featherlight=false&width=90pc)

1. Different users can access the cluster security, managed by self-managed applications within the cluster they access.

![EKS](../../../images/0001/0005.png?featherlight=false&width=90pc)

2. So, we have with 2 types of users:

- Humans: e.g. Admins and Developers

- Robots: like processes/services or applications that require access to the cluster.

![EKS](../../../images/0001/0006.png?featherlight=false&width=90pc)

3. The entirety of user access management is done by the apiserver. All requests go through the apiserver.

![EKS](../../../images/0001/0007.png?featherlight=false&width=90pc)

### Authentication Mechanisms

There are different authentication mechanisms that are configurable.

![EKS](../../../images/0001/0008.png?featherlight=false&width=90pc)

#### Basic Authentication Mechanisms

![EKS](../../../images/0001/0009.png?featherlight=false&width=90pc)

#### Kube-apiserver Configuration

- If you setup via kubeadm, update the kube-apiserver.yaml template file with the option(s).

![EKS](../../../images/0001/00010.png?featherlight=false&width=90pc)

#### Authenticate User with API

- To authenticate with basic credentials when accessing the API server, specify the username and password in a curl command.

```
$ curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
```

![EKS](../../../images/0001/00011.png?featherlight=false&width=90pc)


We can add more columns in the user-details.csv file to assign users to specific groups.

#### References

- https://kubernetes.io/docs/reference/access-authn-authz/authentication/