---
title: "Securing your EKS Cluster"
weight: 1
chapter: false
---

# Securing your EKS Cluster

Security and Compliance is a shared responsibility between AWS and the customer. AWS is responsible for protecting the infrastructure that runs all of the services offered in the AWS Cloud, known as **_Security OF the Cloud_**. This infrastructure is composed of the hardware, software, networking, and facilities that run AWS Cloud services. The customer’s responsibility is determined by the AWS Cloud services they select. This determines the amount of configuration work the customer must perform as part of their security responsibilities, known as **_Security IN the Cloud_**.

![SRM](images/home/0001-Shared_Responsibility_Model.png)

For AWS, the **_Shared Responsibility Model_** can be describe as below:

- **Security of the cloud:** – AWS is responsible for protecting the infrastructure that runs AWS services in the AWS Cloud. For Amazon EKS, AWS is responsible for the Kubernetes control plane, which includes the control plane nodes and etcd database. Third-party auditors regularly test and verify the effectiveness of our security as part of the AWS compliance programs. To learn about the compliance programs that apply to Amazon EKS, see AWS Services in Scope by Compliance Program.

- **Security in the cloud:** – Your responsibility includes the following areas.

    - The security configuration of the data plane, including the configuration of the security groups that allow traffic to pass from the Amazon EKS control plane into the customer VPC

    - The configuration of the nodes and the containers themselves

    - The node’s operating system (including updates and security patches)

    - Other associated application software:

        - Setting up and managing network controls, such as firewall rules

        - Managing platform-level identity and access management, either with or in addition to IAM

-   The sensitivity of your data, your company’s requirements, and applicable laws and regulations

Depending on whether **_Self-Managed Node Groups_**, **_Manage Node Groups_**, or **_Fargate_** is used, AWS may shares more or less responsibility for your security:

![Self-Managed](images/home/0002-eks-self.jpg)

![AWS-Managed](images/home/0003-eks-managed.jpg)

![Fargate](images/home/0004-eks-fargate.jpg)

In this workshop, we'll explore various aspects of Amazon EKS related to security. To learn more about security with EKS refer to the [Amazon EKS Best Practices Guide for Security](https://aws.github.io/aws-eks-best-practices/security/docs/).