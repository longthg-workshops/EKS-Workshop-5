---
title: "Chuẩn bị môi trường"
weight: 1
chapter: false
pre: "<b> 2.1 </b>"
---

### **Tạo CloudFormation Stack**
Tải tệp template tại [link này](https://github.com/aws-samples/eks-workshop-v2/releases/download/release-snapshot-631eaeb7/ide-cfn.yaml).

Truy cập giao diện [CloudFormation](console.aws.amazon.com/cloudformation/home)

![](/images/2/1/3/010.jpg)

- Chọn **_Create Stack_**.

Nhập tên cho **_Stack Name_** và **_Environment Name_**.

![](/images/2/1/3/011.jpg?width=70pc)

Thay đổi các tham số cho phù hợp hoặc giữ nguyên.

Nhấn Next qua các bước và đợi đến khi Stack được tạo xong.

![](/images/2/1/3/008.jpg?width=50pc)

Stack sẽ mất từ 4-5 phút để tạo. Sau khi stack hoàn thành, chuyển qua tab output để lấy thông tin truy cập vào IDE.

### **Truy cập VSCode IDE trên trình duyệt**

_Khóa đầu ra `IdeUrl` chứa URL cần nhập vào trình duyệt của bạn để truy cập IDE. `IdePasswordSecret` chứa liên kết đến khóa AWS Secrets Manger chứa mật khẩu được tạo cho IDE._

- Để nhận mật khẩu, mở URL `IdePasswordSecret` và nhấn nút **Retrieve**:

![secretsmanager retrieve](/images/2/1/2/vscode-password-retrieve.webp)

- Mật khẩu để bạn sao chép sẽ hiện lên:

![cloudformation outputs](/images/2/1/2/vscode-password-visible.webp)

- Mở URL `IdeUrl`lên và để truy cập trang web IDE. Bạn sẽ nhận thông báo yêu cầu mật khẩu

![cloudformation outputs](/images/2/1/2/vscode-password.webp)

Sau khi nhập mật khẩu, bạn sẽ truy cập được vào màn hình ban đầu của VSCode.

![cloudformation outputs](/images/2/1/2/vscode-splash.webp)


{{% notice note %}}
Trong trường hợp CloudFormation báo lỗi không tạo được CloudFront Distribution do chưa xác thực tài khoản, bạn có thể yêu cầu nâng giới hạn tài nguyên CloudFront Distribution [tại đây](https://support.console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase).
![](/images/2/1/2/quota-failed-01.jpg?featherlight=false&width=30pc)
Tại mục _Service_, chọn "CloudFront Distribution".
![](/images/2/1/2/quota-inc-01.jpg?featherlight=false&width=90pc)
Tại mục _Request 1_, chọn _Quota_ là _Web Distributions per Account_ và đặt giá trị cao hơn giới hạn trước đó của bạn (nếu tài khoản của bạn mới được tạo và bạn chưa dùng CloudFront Distribution trên tài khoản, bạn có thể điền mọi giá trị dương)
![](/images/2/1/2/quota-inc-02.jpg?featherlight=false&width=90pc)
Tại mục _Case description_, chép mã lỗi chi tiết tại bước tạo CloudFront Distribution và dán vào khung, kèm miêu tả của bạn về lỗi gặp phải.
![](/images/2/1/2/quota-inc-03.jpg?featherlight=false&width=90pc)
Sau khi thực hiện xong, chọn phương thức liên hệ. Sau đó nhấn _Submit_ ở cuối trang.
![](/images/2/1/2/quota-inc-04.jpg?featherlight=false&width=90pc)
Hãy chờ một thời gian, theo dõi case và tích cực trao đổi để được hỗ trợ. Việc chờ xác thực tài khoản va nâng hạn mức
{{% /notice %}}


Mở URL này trong trình duyệt web để truy cập vào **IDE**.

![EKS](/images/2/1/2/vsc-web.png?featherlight=false&width=90pc)

Bạn có thể đóng **CloudShell** ngay bây giờ, tất cả các lệnh tiếp theo sẽ được thực hiện trong phần terminal ở dưới cùng của **Cloud9 IDE**. **AWS CLI** đã được cài đặt sẵn và sẽ nhận các thông tin xác thực được gắn với **Cloud9 IDE**:

```bash test=false
$ aws sts get-caller-identity
```

Bước tiếp theo là tạo một cụm **EKS** để thực hiện các bài workshop. Nội dung này sẽ được trình bày tại [phần 2.2](../../../../2.2-cluster-creation/)
