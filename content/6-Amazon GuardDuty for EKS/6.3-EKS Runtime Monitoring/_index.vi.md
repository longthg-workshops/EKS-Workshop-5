---
title: "EKS Runtime Monitoring"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 6.3 </b>"
---

#### Giám sát EKS Runtime

Giám sát runtime (EKS Runtime Monitoring) cung cấp khả năng phát hiện mối đe dọa trong runtime cho các nút và container Amazon EKS. Nó sử dụng agent bảo mật GuardDuty (phần mở rộng EKS) để cung cấp khả năng hiển thị runtime trong các công việc EKS cụ thể, ví dụ như truy cập tệp, thực thi quy trình, tăng quyền và kết nối mạng nhận dạng các container cụ thể có thể bị xâm nhập.

Khi bạn kích hoạt Giám sát Thời Gian Chạy EKS, GuardDuty có thể bắt đầu giám sát các sự kiện thời gian chạy trong cụm EKS của bạn. Nếu cụm EKS của bạn không có agent bảo mật được triển khai tự động thông qua GuardDuty hoặc thủ công, GuardDuty sẽ không thể nhận các sự kiện thời gian chạy của các cụm EKS của bạn, có nghĩa là agent phải được triển khai trên các nút EKS trong các cụm EKS của bạn. Bạn có thể chọn GuardDuty để quản lý agent bảo mật tự động hoặc bạn có thể quản lý việc triển khai và cập nhật agent bảo mật thủ công.

Trong bài thực hành này, chúng ta sẽ tạo ra một số phát hiện thời gian chạy EKS trong cụm Amazon EKS của bạn, được liệt kê dưới đây.

- `Execution:Runtime/NewBinaryExecuted`
- `CryptoCurrency:Runtime/BitcoinTool.B!DNS`
- `Execution:Runtime/NewLibraryLoaded`
- `DefenseEvasion:Runtime/FilelessExecution`

Dấu hiệu này cho thấy một container đã cố gắng thực hiện việc đào tiền điện tử bên trong một Pod.

Để mô phỏng dấu hiệu này, chúng ta sẽ chạy một Pod hình ảnh `ubuntu` trong không gian tên `default` và từ đó chạy một vài lệnh để mô phỏng việc tải xuống một quy trình đào tiền điện tử.

Chạy lệnh dưới đây để bắt đầu Pod:

```bash
$ kubectl run crypto -n other --image ubuntu --restart=Never --command -- sleep infinity
$ kubectl wait --for=condition=ready pod crypto -n other
```

Tiếp theo, chúng ta có thể sử dụng `kubectl exec` để chạy một loạt các lệnh bên trong Pod. Đầu tiên, hãy cài đặt tiện ích `curl`:

```bash
$ kubectl exec crypto -n other -- bash -c 'apt update && apt install -y curl'
```

Tiếp theo, hãy tải xuống quy trình đào tiền điện tử nhưng ghi đầu ra vào `/dev/null`:

```bash test=false
$ kubectl exec crypto -n other -- bash -c 'curl -s -o /dev/null http://us-east.equihash-hub.miningpoolhub.com:12026 || true && echo "Done!"'
```

Những lệnh này sẽ kích hoạt ba phát hiện khác nhau trên [bảng điều khiển Phát hiện GuardDuty](https://console.aws.amazon.com/guardduty/home#/findings).

Phát hiện đầu tiên là `Execution:Runtime/NewBinaryExecuted` liên quan đến việc cài đặt gói `curl` thông qua công cụ APT.

![](/images/p6/p63/6.3-1-NewBinExec.png)

Hãy xem xét kỹ các chi tiết của phát hiện này, bởi vì chúng liên quan đến giám sát thời gian chạy của GuardDuty, hiển thị thông tin cụ thể về thời gian chạy, ngữ cảnh và quy trình.

Phát hiện thứ hai và ba liên quan đến các phát hiện `CryptoCurrency:Runtime/BitcoinTool.B!DNS`. Lưu ý một lần nữa rằng các chi tiết phát hiện mang lại thông tin khác nhau, lần này hiển thị hành động `DNS_REQUEST`, và **Bằng chứng tình báo Mối đe dọa**.

![](/images/p6/p63/6.3-2-DNS.png)