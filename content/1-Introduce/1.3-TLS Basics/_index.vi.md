---
title: "TLS cơ bản"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 1.3 </b>"
---

#### TLS cơ bản

#### Chứng chỉ (TLS Basics)

- Chứng chỉ được sử dụng để đảm bảo sự tin tưởng giữa 2 bên trong quá trình giao dịch.

- Ví dụ: khi một người dùng cố gắng truy cập máy chủ web, các chứng chỉ tls đảm bảo rằng giao tiếp giữa họ được mã hóa.

![EKS](/images/0002/0001.png?featherlight=false&width=90pc)

#### Mã hóa đối xứng
Đây là một cách mã hóa an toàn, nhưng nó sử dụng cùng một khóa để mã hóa và giải mã dữ liệu và khóa phải được trao đổi giữa người gửi và người nhận, có nguy cơ một hacker có thể truy cập vào khóa và giải mã dữ liệu.

![EKS](/images/0002/0002.png?featherlight=false&width=90pc)

#### Mã hóa không đối xứng
Thay vì sử dụng một khóa duy nhất để mã hóa và giải mã dữ liệu, mã hóa không đối xứng sử dụng một cặp khóa, một khóa riêng tư và một khóa công khai.

![EKS](/images/0002/0003.png?featherlight=false&width=90pc)

![EKS](/images/0002/0004.png?featherlight=false&width=90pc)


![EKS](/images/0002/0005.png?featherlight=false&width=90pc)


![EKS](/images/0002/0006.png?featherlight=false&width=90pc)


#### Làm thế nào để xem xét một chứng chỉ và xác minh tính hợp lệ của nó?

- Ai đã ký và cấp phát chứng chỉ.
- Nếu bạn tạo ra chứng chỉ, sau đó bạn sẽ phải tự ký chứng chỉ đó; điều này được biết đến là chứng chỉ tự ký.

![EKS](/images/0002/0007.png?featherlight=false&width=90pc)

#### Làm thế nào để tạo chứng chỉ hợp lệ? Làm thế nào để có được chứng chỉ của bạn được ký bởi một người có thẩm quyền?

Đó là phần Nhà cung cấp chứng chỉ (Certificate Authority - CA) sẽ làm giúp bạn. Một số CA phổ biến là Symantec, DigiCert, Comodo, GlobalSign v.v.

![EKS](/images/0002/0008.png?featherlight=false&width=90pc)

![EKS](/images/0002/0009.png?featherlight=false&width=90pc)

![EKS](/images/0002/00010.png?featherlight=false&width=90pc)

#### Public Key Infrastructure

![EKS](/images/0002/00011.png?featherlight=false&width=90pc)

#### Certificates naming convention

![EKS](/images/0002/00012.png?featherlight=false&width=90pc)