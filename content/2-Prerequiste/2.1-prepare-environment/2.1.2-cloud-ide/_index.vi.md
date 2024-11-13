---
title: "Triển khai VSCode dưới dạng web host trên EC2 và CloudFront"

weight: 2
chapter: false
pre: "<b> 2.1.2 </b>"
---

**_Lưu ý:_** _Hiện tại AWS sắp ngừng cung cấp Cloud9. Chúng tôi hiện tại đang tìm các giải pháp thay thế sớm nhất có thể._

#### **Hướng dẫn thiết lập môi trường chạy lab trong tài khoản AWS của bạn**

Bước đầu tiên là tạo một IDE với mẫu CloudFormation được cung cấp. Mở **CloudShell** tại một trong các region _us-west-2_, _us-east-1_ hoặc _ap-southeast-1_ và chạy các lệnh sau:

```bash test=false
$ wget -q https://raw.githubusercontent.com/longthg-workshops/eks-workshop-v2-fork/cloud-ide/lab/cfn/ec2-workshop-cloud-ide-cfn.yaml -O eks-workshop-ide-cfn.yaml
$ aws cloudformation deploy --stack-name eks-workshop-ide \
    --template-file ./eks-workshop-ide-cfn.yaml \
    --parameter-overrides RepositoryRef=stable \
    --capabilities CAPABILITY_NAMED_IAM
Waiting for changeset to be created...
Waiting for stack create/update to complete
Successfully created/updated stack - eks-workshop-ide
```

Một cách khác là sử dụng các link nhanh sau:

Use the AWS CloudFormation quick-create links below to launch the desired template in the appropriate AWS region.

| Region           | Link       (Preview)                                                                                                                                                                                                                                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `us-west2`       | [Launch](https://us-west-2.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0.s3.us-west-2.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `eu-west-1`      | [Launch](https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-dub-85e3be25bd827406.s3.eu-west-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `ap-southeast-1` | [Launch](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-sin-694a125e41645312.s3.ap-southeast-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF") |

_Ba region trên đã được kiểm thử và đảm bảo. Để chạy lab tại các region khác, bạn có thể cần thực hiện một số chỉnh sửa._

Stack sẽ mất từ 4-5 phút để tạo. Sau khi stack hoàn thành, chuyển qua tab output để lấy thông tin truy cập vào IDE:

_Khóa đầu ra `IdeUrl` chứa URL cần nhập vào trình duyệt của bạn để truy cập IDE. `IdePasswordSecret` chứa liên kết đến khóa AWS Secrets Manger chứa mật khẩu được tạo cho IDE._

- Để nhận mật khẩu, mở URL `IdePasswordSecret` và nhấn nút **Retrieve**:

![secretsmanager retrieve](../../../../images/2/1/2/vscode-password-retrieve.webp)

- Mật khẩu để bạn sao chép sẽ hiện lên:

![cloudformation outputs](../../../../images/2/1/2/vscode-password-visible.webp)

- Mở URL `IdeUrl`lên và để truy cập trang web IDE. Bạn sẽ nhận thông báo yêu cầu mật khẩu

![cloudformation outputs](../../../../images/2/1/2/vscode-password.webp)

Sau khi nhập mật khẩu, bạn sẽ truy cập được vào màn hình ban đầu của VSCode.

![cloudformation outputs](../../../../images/2/1/2/vscode-splash.webp)


{{% notice note %}}
Trong trường hợp CloudFormation báo lỗi không tạo được CloudFront Distribution do chưa xác thực tài khoản, bạn có thể yêu cầu nâng giới hạn tài nguyên CloudFront Distribution [tại đây](https://support.console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase).
![](../../../../images/2/1/2/quota-failed-01.jpg?featherlight=false&width=30pc)
Tại mục _Service_, chọn "CloudFront Distribution".
![](../../../../images/2/1/2/quota-inc-01.jpg?featherlight=false&width=90pc)
Tại mục _Request 1_, chọn _Quota_ là _Web Distributions per Account_ và đặt giá trị cao hơn giới hạn trước đó của bạn (nếu tài khoản của bạn mới được tạo và bạn chưa dùng CloudFront Distribution trên tài khoản, bạn có thể điền mọi giá trị dương)
![](../../../../images/2/1/2/quota-inc-02.jpg?featherlight=false&width=90pc)
Tại mục _Case description_, chép mã lỗi chi tiết tại bước tạo CloudFront Distribution và dán vào khung, kèm miêu tả của bạn về lỗi gặp phải.
![](../../../../images/2/1/2/quota-inc-03.jpg?featherlight=false&width=90pc)
Sau khi thực hiện xong, chọn phương thức liên hệ. Sau đó nhấn _Submit_ ở cuối trang.
![](../../../../images/2/1/2/quota-inc-04.jpg?featherlight=false&width=90pc)
Hãy chờ một thời gian, theo dõi case và tích cực trao đổi để được hỗ trợ. Việc chờ xác thực tài khoản va nâng hạn mức
{{% /notice %}}


Mở URL này trong trình duyệt web để truy cập vào **IDE**.

![EKS](../../../../images/2/1/2/vsc-web.png?featherlight=false&width=90pc)

Bạn có thể đóng **CloudShell** ngay bây giờ, tất cả các lệnh tiếp theo sẽ được thực hiện trong phần terminal ở dưới cùng của **Cloud9 IDE**. **AWS CLI** đã được cài đặt sẵn và sẽ nhận các thông tin xác thực được gắn với **Cloud9 IDE**:

```bash test=false
$ aws sts get-caller-identity
```

Bước tiếp theo là tạo một cụm **EKS** để thực hiện các bài workshop. Nội dung này sẽ được trình bày tại [phần 2.2](../../../../2.2-cluster-creation/)
