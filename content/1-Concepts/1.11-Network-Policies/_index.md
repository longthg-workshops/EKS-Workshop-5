---
title: "Network-Policies"
weight: 11
chapter: false
pre: "<b> 1.11 </b>"
---

### Ingress and Egress

Trafic flowing through a webserver serving frontend to users an app server serving backend API and a database server

![traffic](/images/1/11/traffic.PNG)
  
- There are two types of traffic
  - Ingress

  - Egress

![ing1](/images/1/11/ing1.PNG)

![ing2](/images/1/11/ing2.PNG)
  
### Network Security

Network security is a critical aspect of any Kubernetes deployment, as it ensures that data transmitted within the cluster is protected against unauthorized access, interception, or modification. You will find yourself in need of a form of control over the visibility and access of your pods to other pods and/or outside entities.

![nsec](/images/1/11/nsec.PNG)
  
### Network Policies

A NetworkPoliciy is an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" over the network. A Network Policy apply to a connection with a pod on one or both ends, and are not relevant to other connections.

![npol](/images/1/11/npol.PNG)

![npol1](/images/1/11/npol1.PNG)
  
#### Network Policy Selectors

Select pods that will have the policies applied to based on their labels.
  
![npolsec](/images/1/11/npolsec.PNG)
  
#### Network Policy Rules

The following image shows a policy that only allow access from a specified pod to port 3306.

![npol2](/images/1/11/npol2.PNG)
  
#### Create a network policy
 
1. To create a network policy

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

2. Apply the policy to the Kubernetes cluster

```bash
$ kubectl create -f policy-definition.yaml
```
  
![npol3](/images/1/11/npol3.PNG)

![npol4](/images/1/11/npol4.PNG)
  
### Note
 
Some network solutions for Kubernetes supports network policies, while others don't.

![note1](/images/1/11/note1.PNG)


### Further reading
- https://kodekloud.com/topic/developing-network-policies/
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/