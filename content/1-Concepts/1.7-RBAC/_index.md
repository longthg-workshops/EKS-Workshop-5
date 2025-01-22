---
title: "Role-Based Access Control"
weight: 7
chapter: false
pre: "<b> 1.7 </b>"
---

**Role-based access control (RBAC)** is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

## How do we create a role?

- Each rule in a role has 3 sections

  - apiGroups

  - resources

  - verbs
- create the role with kubectl command
  ```
  $ kubectl create -f developer-role.yaml
  ```

## Link the user to the newly created role.

For this we create another object called **`RoleBinding`**. This role binding object links a user object to a role.

Create a role binding using kubectl command

```
$ kubectl create -f devuser-developer-binding.yaml
```
- Also note that the roles and role bindings fall under the scope of namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
name: developer
rules:
- apiGroups: [""] # "" indicates the core API group
resources: ["pods"]
verbs: ["get", "list", "update", "delete", "create"]
- apiGroups: [""]
resources: ["ConfigMap"]
verbs: ["create"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: devuser-developer-binding
subjects:
- kind: User
name: dev-user # "name" is case sensitive
apiGroup: rbac.authorization.k8s.io
roleRef:
kind: Role
name: developer
apiGroup: rbac.authorization.k8s.io
```
![rbac1](/images/1/7/0001.PNG)
  

## View RBAC
  
- To list roles
  ```
  $ kubectl get roles
  ```
- To list rolebindings
  ```
  $ kubectl get rolebindings
  ```
- To describe role 
  ```
  $ kubectl describe role developer
  ```
  
  ![rbac2](/images/1/7/0002.PNG)
    
- To describe rolebinding
  ```
  $ kubectl describe rolebinding devuser-developer-binding
  ```
  
  ![rbac3](/images/1/7/0003.PNG)
  
**What if you being a user would like to see if you have access to a particular resource in the cluster ?**
### Check Access permission

You can use the kubectl auth command

```bash
$ kubectl auth can-i create deployments
```
```bash
$ kubectl auth can-i delete nodes
```
```bash
$ kubectl auth can-i create deployments --as dev-user
```
```bash
$ kubectl auth can-i create pods --as dev-user
```
```bash
$ kubectl auth can-i create pods --as dev-user --namespace test
```
  
![rbac5](/images/1/7/0004.PNG)
  
## Resource Names
- Note on resource names we just saw how you can provide access to users for resources like pods within the namespace.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
name: developer
rules:
- apiGroups: [""] # "" indicates the core API group
resources: ["pods"]
verbs: ["get", "update", "create"]
resourceNames: ["blue", "orange"]
```  
![rbac4](/images/1/7/0005.PNG)
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities