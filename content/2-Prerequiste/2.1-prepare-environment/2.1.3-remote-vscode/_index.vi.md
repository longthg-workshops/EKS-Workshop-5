---
title: "Triển khai VSCode Server trên EC2 và truy cập bằng VSCode trên máy của bạn"

weight: 3
chapter: false
pre: "<b> 2.1.3 </b>"
---

### **Tạo Key Pair**
{{% notice note %}}
Nếu bạn đã tạo và lưu sẵn Key Pair, bạn có thể bỏ qua bước này và chuyển sang bước tiếp theo.
{{% /notice %}}

Đầu tiên, bạn cần vào [bảng điều khiển EC2](console.aws.amazon.com/ec2/home). Tại đây, ở thanh điều hướng bên trái, chọn _Key Pairs_. Ở giao diện _Key Pairs_, chọn _Create Key Pair_.

![](../../../../images/2/1/3/001.jpg)

Tại giao diện **_Create Key Pair_**:
- Tại mục _Name_, nhập tên tùy ý cho Key Pair.
- Tiếp đến, chọn một trong hai loại khóa và định dạng _.pem_.
- Nhấn nút _Create key pair_ ở cuối (màu cam) để tiến hành tạo khóa.

![](../../../../images/2/1/3/002.jpg?width=50pc)

Lưu file private key trên máy tính. Khóa này sẽ được dùng để gắn vào EC2 được tạo ở bước kế tiếp.

![](../../../../images/2/1/3/003.jpg?width=50pc)

Hãy đảm bảo chỉ có bạn mới có thể truy cập vào file _.pem_.

- Với Linux, sử dụng lệnh:
```bash
chmod 400 <đường dẫn tới file pem>
```

- Với Windows, đảm bảo quyền truy cập của bạn như sau:

![](../../../../images/2/1/3/004.jpg?width=50pc)

### **Tạo CloudFormation Stack**
Tải tệp template tại link này.

Truy cập giao diện [CloudFormation](console.aws.amazon.com/cloudformation/home)

![](../../../../images/2/1/3/010.jpg)

- Chọn **_Create Stack_**.

Nhập tên cho **_Stack Name_** và **_Environment Name_**.

![](../../../../images/2/1/3/011.jpg?width=70pc)

Với **_SshKeyName_**, nhập tên của Key Pair đã tạo ở bước trước đó.

![](../../../../images/2/1/3/009.jpg?width=70pc)

Nhấn Next qua các bước và đợi đến khi Stack được tạo xong.

![](../../../../images/2/1/3/008.jpg?width=50pc)

### **Cấu hình kết nối SSH trên máy tính của bạn**
Tại giao diện của CloudFormation stack đã tạo, chuyển sang mục _Output_. Tìm khóa _IdeInstancePublicIP_, giá trị của khóa này chính là địa chỉ cần kết nối đến.

![](../../../../images/2/1/3/007.jpg?width=70pc)

Thêm vào file `C:\Users\<Tên người dùng>\.ssh\config` (nếu bạn đang dùng Windows) hoặc `~/.ssh/config` (nếu bạn đang dùng Linux):

```
Host remote-connection
    HostName <public ip của EC2>
    User ec2-user
    IdentityFile "<đường dẫn đến file pem đã lưu>"
```

### **Kết nối VSCode đến EC2**
- Cài đặt tiện ích Remote-SSH để hỗ trợ việc kết nối vối SSH Host

![](../../../../images/2/1/3/014.png)

- Sau khi cài đặt tiện ích, bạn sẽ nhìn thấy icon ở phía dưới bên trái của màn hình.

- Ấn vào icon để mở bảng chọn kết nối. Chọn “Connect to Host…”

![](../../../../images/2/1/3/015.png)

- Chọn “Add New SSH Host” để tạo một SSH Host mới

![](../../../../images/2/1/3/016.png)

- Chọn `C:\Users\<tên người dùng>\.ssh\config` để mở file config 

- Sau khi thiết lập xong, bạn ấn vào icon để mở bạng chọn và điều hướng tới “remote-connection”

![](../../../../images/2/1/3/017.png)

- Tiếp theo, bạn chọn Platform details “Linux” và chọn “Continue”

![](../../../../images/2/1/3/018.png)

Nếu thành công, một cửa sổ VSCode mới sẽ mở lên và bạn sẽ được truy cập vào EC2. Chọn **_"File > Open"_** để mở thư mục

![](../../../../images/2/1/3/019.jpg?width=50pc)

Chọn thư mục _**/home/\< tên người dùng \>/environment**_

![](../../../../images/2/1/3/020.jpg?width=50pc)