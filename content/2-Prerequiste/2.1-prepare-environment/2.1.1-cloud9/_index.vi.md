---
title: "Triển khai môi trường lab trên Cloud9 (Đã ngừng hỗ trợ)"

weight: 1
chapter: false
pre: "<b> 2.1.1 </b>"
---

**_Lưu ý:_** _Hiện tại AWS đã ngừng hỗ trợ Cloud9 cho tài khoản mới. Chúng tôi khuyến khích bạn dùng các giải pháp được nêu ở sau phần này._

#### **Hướng dẫn thiết lập môi trường Cloud9 trong tài khoản AWS của bạn**

Bước đầu tiên là tạo một IDE với mẫu CloudFormation được cung cấp. Cách đơn giản nhất là mở giao diện CloudFormation theo các đường dẫn dưới đây:

Các region sau đã được kiểm tra và đảm bảo. Việc chạy các bài workshop tại các vùng địa lý khác có thể không được đảm bảo:

| Region           | Cloud9 Link                                                                                                                                                                                                                                                                                                                        | VSCode Link (Preview)                                                                                                                                                                                                                                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `us-west2`       | [Launch](https://us-west-2.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0.s3.us-west-2.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)               | [Launch](https://us-west-2.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0.s3.us-west-2.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `eu-west-1`      | [Launch](https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-dub-85e3be25bd827406.s3.eu-west-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)               | [Launch](https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-dub-85e3be25bd827406.s3.eu-west-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `ap-southeast-1` | [Launch](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-sin-694a125e41645312.s3.ap-southeast-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF") | [Launch](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-sin-694a125e41645312.s3.ap-southeast-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF") |

![EKS](../../../../images/2/1/1/00015.png?featherlight=false&width=90pc)

Một cách khác là mở **CloudShell** tại một trong các region trên và chạy các lệnh sau:

```bash test=false
$ wget -q https://raw.githubusercontent.com/aws-samples/eks-workshop-v2/stable/lab/cfn/eks-workshop-ide-cfn.yaml -O eks-workshop-ide-cfn.yaml
$ aws cloudformation deploy --stack-name eks-workshop-ide \
    --template-file ./eks-workshop-ide-cfn.yaml \
    --parameter-overrides RepositoryRef=stable \
    --capabilities CAPABILITY_NAMED_IAM
Waiting for changeset to be created...
Waiting for stack create/update to complete
Successfully created/updated stack - eks-workshop-ide
```

**CloudFormation Stack** sẽ mất khoảng 5 phút để triển khai, và khi hoàn tất, bạn có thể lấy URL cho **Cloud9 IDE** như sau:

```bash test=false
$ aws cloudformation describe-stacks --stack-name eks-workshop-ide \
    --query 'Stacks[0].Outputs[?OutputKey==`Cloud9Url`].OutputValue' --output text
https://us-west-2.console.aws.amazon.com/cloud9/ide/7b05513358534d11afeb7119845c5461?region=us-west-2
```

Mở URL này trong trình duyệt web để truy cập vào **IDE**.

![EKS](../../../../images/2/1/1/00016.png?featherlight=false&width=90pc)

Bạn có thể đóng **CloudShell** ngay bây giờ, tất cả các lệnh tiếp theo sẽ được thực hiện trong phần terminal ở dưới cùng của **Cloud9 IDE**. **AWS CLI** đã được cài đặt sẵn và sẽ nhận các thông tin xác thực được gắn với **Cloud9 IDE**:

```bash test=false
$ aws sts get-caller-identity
```

Bước tiếp theo là tạo một cụm **EKS** để thực hiện các bài workshop. Nội dung này sẽ được trình bày tại _phần 2.2_.