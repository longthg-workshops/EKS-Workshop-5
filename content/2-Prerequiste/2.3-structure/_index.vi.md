---
title: "Cấu trúc workshop"
weight: 3
chapter: false
pre: "<b> 2.3 </b>"
---

### Giới thiệu về cấu trúc các bài lab

Nội dung của hội thảo này bao gồm:

1. Các bài tập thực hành cá nhân
2. Nội dung hỗ trợ giải thích các khái niệm liên quan đến các bài thực hành

Các bài tập thực hành được thiết kế một cách sao cho bạn có thể chạy bất kỳ modules nào như một bài tập độc lập.

{{% notice note %}}
Bạn nên bắt đầu mỗi bài lab từ trang đầu tiên. Bắt đầu ở giữa một bài lab sẽ gây ra các biểu hiện không đoán trước được.
{{% /notice %}}

### Cloud9 IDE

Sau khi bạn đã truy cập vào Cloud9 IDE, chúng tôi khuyến nghị bạn sử dụng nút **+** và chọn **New Terminal** để mở một cửa sổ terminal mới toàn màn hình.

![EKS](/images/part2/3/00013.png?featherlight=false&width=90pc)


Điều này sẽ mở một tab mới với một terminal mới.

![EKS](/images/part2/3/00014.png?featherlight=false&width=90pc)

Bạn cũng có thể đóng terminal nhỏ ở dưới nếu bạn muốn.

### Các lệnh Terminal

Hầu hết các tương tác mà bạn sẽ thực hiện trong hội thảo này sẽ được thực hiện bằng các lệnh terminal, mà bạn có thể gõ thủ công hoặc sao chép/dán vào terminal Cloud9 IDE. Bạn sẽ thấy các lệnh terminal được hiển thị như sau:

```bash test=false
$ echo "Đây là một ví dụ lệnh"
```

Di chuột qua `echo "Đây là một ví dụ lệnh"` và nhấp vào biểu tượng để sao chép lệnh đó vào clipboard của bạn.

Bạn cũng sẽ gặp phải các lệnh có kết quả mẫu như sau:

```bash test=false
$ kubectl get nodes
NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-42-10-104.us-west-2.compute.internal   Ready    <none>   6h      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-10-210.us-west-2.compute.internal   Ready    <none>   6h      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-198.us-west-2.compute.internal   Ready    <none>   6h      vVAR::KUBERNETES_NODE_VERSION
```

Giữ chuột, kéo qua lệnh cần sao chép và nhấn Ctrl+C để sao chép. Hãy thử xem!

### Thiết lập lại EKS cluster của bạn

Trong trường hợp bạn cấu hình cluster theo cách khiến nó không hoạt động, bạn được cung cấp một cơ chế để thiết lập lại EKS cluster của mình sao cho tốt nhất có thể, có thể chạy bất kỳ lúc nào. Đơn giản chỉ cần chạy lệnh `prepare-environment` và đợi cho đến khi nó hoàn tất. Điều này có thể mất vài phút tùy thuộc vào trạng thái của cluster của bạn khi chạy lệnh này.
