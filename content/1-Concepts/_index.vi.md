---
title: "Một số khái niệm"
weight: 1
chapter: false
pre: "<b> 1. </b>"
---

### Kubernetes
#### Tổng quan về Kubernetes

Kubernetes là một nền tảng nguồn mở, có tính cơ động, có thể mở rộng để quản lý các ứng dụng được đóng gói và các service, giúp thuận lợi trong việc cấu hình và tự động hoá việc triển khai ứng dụng. Kubernetes là một hệ sinh thái lớn và phát triển nhanh chóng. Các dịch vụ, sự hỗ trợ và công cụ có sẵn rộng rãi.

Tên gọi Kubernetes có nguồn gốc từ tiếng Hy Lạp, có ý nghĩa là người lái tàu hoặc hoa tiêu. Google mở mã nguồn Kubernetes từ năm 2014. Kubernetes xây dựng dựa trên một thập kỷ rưỡi kinh nghiệm mà Google có được với việc vận hành một khối lượng lớn workload trong thực tế, kết hợp với các ý tưởng và thực tiễn tốt nhất từ cộng đồng.

![Kubernetes](../../images/home/kubernetes.webp?width=70pc)

### Amazon Elastic Kubernetes Service (EKS)
Amazon Elastic Kubernetes Service (Amazon EKS) là một dịch vụ được quản lý giúp loại bỏ nhu cầu cài đặt, vận hành và bảo trì lớp điều khiển Kubernetes của riêng bạn trên Amazon Web Services (AWS).

![EKS](../../images/home/EKS.png?width=90pc)

#### Các tính năng của Amazon EKS
Sau đây là các tính năng chính của Amazon EKS:

##### **Xây dựng mạng và xác thực an toàn**
Amazon EKS tích hợp khối công việc của bạn trên Kubernetes với các dịch vụ mạng và bảo mật của AWS. Nó cũng tích hợp với AWS Identity and Access Management (IAM) để cung cấp xác thực cho các cụm Kubernetes của bạn.

##### **Dễ dàng mở rộng cụm**
Amazon EKS cho phép bạn dễ dàng chỉnh quy mô các cụm Kubernetes của mình lên và xuống dựa trên nhu cầu của khối lượng công việc của bạn. Amazon EKS hỗ trợ tự động mở rộng Pod theo chiều ngang dựa trên CPU hoặc số liệu tùy chỉnh và tự động mở rộng cụm dựa trên nhu cầu của toàn bộ khối lượng công việc.

##### **Trải nghiệm Kubernetes được quản lý**
Bạn có thể thực hiện các thay đổi đối với các cụm Kubernetes của mình bằng eksctl, AWS Management Console, AWS Command Line Interface (AWS CLI), API, kubectl và Terraform.

##### **Tính khả dụng cao**
Amazon EKS cung cấp tính khả dụng cao cho lớp điều khiển của bạn trên nhiều Vùng khả dụng.

##### **Tích hợp với các dịch vụ AWS**
Amazon EKS tích hợp với các dịch vụ AWS khác, cung cấp nền tảng toàn diện để triển khai và quản lý các ứng dụng được chứa trong container của bạn. Bạn cũng có thể dễ dàng khắc phục sự cố khối lượng công việc Kubernetes của mình hơn bằng nhiều công cụ quan sát khác nhau.