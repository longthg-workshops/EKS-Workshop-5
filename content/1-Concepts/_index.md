---
title: "Concepts"
weight: 1
chapter: false
pre: "<b> 1. </b>"
---

### Kubernetes
#### Kubernetes overview
Kubernetes is a portable, extensible, open source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

The name Kubernetes originates from Greek, meaning helmsman or pilot. K8s as an abbreviation results from counting the eight letters between the "K" and the "s". Google open-sourced the Kubernetes project in 2014. Kubernetes combines over 15 years of Google's experience running production workloads at scale with best-of-breed ideas and practices from the community.

![Kubernetes](../images/home/kubernetes.webp?width=70pc)

### Amazon Elastic Kubernetes Service (EKS)
Amazon Elastic Kubernetes Service (Amazon EKS) is a managed service that eliminates the need to install, operate, and maintain your own Kubernetes control plane on Amazon Web Services (AWS).

![EKS](../images/home/EKS.png?width=90pc)

#### Features of Amazon EKS
The following are key features of Amazon EKS:

##### **Secure networking and authentication**
Amazon EKS integrates your Kubernetes workloads with AWS networking and security services. It also integrates with AWS Identity and Access Management (IAM) to provide authentication for your Kubernetes clusters.

##### **Easy cluster scaling**
Amazon EKS enables you to scale your Kubernetes clusters up and down easily based on the demand of your workloads. Amazon EKS supports horizontal Pod autoscaling based on CPU or custom metrics, and cluster autoscaling based on the demand of the entire workload.

##### **Managed Kubernetes experience**
You can make changes to your Kubernetes clusters using eksctl, AWS Management Console, AWS Command Line Interface (AWS CLI), the API, kubectl, and Terraform.

##### **High availability**
Amazon EKS provides high availability for your control plane across multiple Availability Zones.

##### **Integration with AWS services**
Amazon EKS integrates with other AWS services, providing a comprehensive platform for deploying and managing your containerized applications. You can also more easily troubleshoot your Kubernetes workloads with various observability tools.