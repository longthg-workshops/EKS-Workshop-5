---
title: "Bảo mật cụm EKS của bạn"
weight: 1
chapter: false
---

# Bảo mật cụm EKS của bạn

Tính bảo mật và Sự tuân thủ là trách nhiệm chung giữa AWS và khách hàng. AWS chịu trách nhiệm bảo vệ cơ sở hạ tầng bên dưới tất cả các dịch vụ được cung cấp trên đám mây AWS, được gọi là **_Bảo mật CHO Đám mây_**. Cơ sở hạ tầng này bao gồm phần cứng, phần mềm, mạng và các cơ sở bên dưới dùng để chạy các dịch vụ AWS Cloud. Trách nhiệm của khách hàng được xác định bởi các dịch vụ AWS mà họ chọn. Nó quyết định lượng công việc cấu hình khách hàng phải thực hiện như một phần trách nhiệm bảo mật của họ, được gọi là **_Bảo mật TRONG Đám mây_**.

![SRM](../images/home/0001-Shared_Responsibility_Model.png)

Đối với EKS, **_Mô hình Chia sẻ trách nhiệm (Shared Responsibility Model)_** có thể được miêu tả như sau:

- **Bảo mật cho đám mây:** - AWS chịu trách nhiệm bảo vệ cơ sở hạ tầng dùng cho việc vận hành các dịch vụ AWS. Đối với Amazon EKS, AWS chịu trách nhiệm về tầng điều khiển Kubernetes, bao gồm các nút tính toán thuộc tầng điều khiển và cơ sở dữ liệu **_etcd_**. Các kiểm toán viên bên thứ ba thường xuyên kiểm tra và xác minh tính hiệu quả của bảo mật của chúng tôi như một phần của các chương trình tuân thủ AWS. Để tìm hiểu về các chương trình tuân thủ áp dụng cho Amazon EKS, hãy xem Dịch vụ AWS trong phạm vi theo Chương trình tuân thủ.

- **Bảo mật trong đám mây:** - Trách nhiệm của bạn bao gồm các lĩnh vực sau.

    - Cấu hình bảo mật của tầng dữ liệu, bao gồm cấu hình các nhóm bảo mật (security group) cho phép lưu lượng truyền từ tầng điều khiển Amazon EKS vào VPC của khách hàng

    - Cấu hình các node cũng như các container

    - Hệ điều hành của node (bao gồm các bản cập nhật và bản vá bảo mật)

    - Phần mềm ứng dụng liên quan khác:

        - Thiết lập và quản lý các biện pháp kiểm soát mạng, chẳng hạn như các quy tắc tường lửa

        - Quản lý danh tính và quản lý quyền truy cập ở cấp nền tảng, bao gồm cả trong và ngoài phạm vi IAM

    - Mức độ nhạy cảm của dữ liệu của bạn, các yêu cầu của công ty bạn và luật pháp và quy định hiện hành

Tùy thuộc vào việc sử dụng **_Self-Managed Node Groups_** (nhóm nút tính toán do người dùng quản lý), **_Manage Node Groups_** (nhóm nút do AWS quản lý), hay **_Fargate_**, phần trách nhiệm của AWS đối với bảo mật của bạn có thể ít hoặc nhiều hơn:

![Self-Managed](../images/home/0002-eks-self.jpg)

![AWS-Managed](../images/home/0003-eks-managed.jpg)

![Fargate](../images/home/0004-eks-fargate.jpg)

Trong workshop này, chúng ta sẽ tìm hiểu một số mặt liên quan tới bảo mật Amazon EKS. Để tìm hiểu thêm về bảo mật trên EKS, hãy tham khảo [Amazon EKS Best Practices Guide for Security](https://aws.github.io/aws-eks-best-practices/security/docs/).