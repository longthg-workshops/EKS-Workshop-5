---
title: "Kích hoạt lớp bảo vệ GuardDuty trên EKS"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 6.1 </b>"
---

Trong lab này, chúng ta sẽ kích hoạt Amazon GuardDuty EKS Protection. Điều này sẽ cung cấp phạm vi phát hiện mối đe dọa cho EKS Audit Log Monitoring và EKS Runtime Monitoring để giúp bảo vệ các cụm của bạn.

EKS Audit Log Monitoring sử dụng các log kiểm tra của Kubernetes để ghi lại các hoạt động theo thứ tự thời gian từ người dùng, ứng dụng sử dụng API Kubernetes và mặt bằng điều khiển để tìm kiếm các hoạt động có thể nghi ngờ.

EKS Runtime Monitoring sử dụng các sự kiện cấp độ hệ điều hành để giúp bạn phát hiện các mối đe dọa tiềm ẩn trong các nút và container Amazon EKS.

**Kích hoạt Amazon GuardDuty qua AWS CLI**

```bash test=false
$ aws guardduty create-detector --enable --features '[{"Name" : "EKS_AUDIT_LOGS", "Status" : "ENABLED"}, {"Name" : "EKS_RUNTIME_MONITORING", "Status" : "ENABLED", "AdditionalConfiguration" : [{"Name" : "EKS_ADDON_MANAGEMENT", "Status" : "ENABLED"}]}]'
{
    "DetectorId": "1qaz0p2wsx9ol3edc8ik4rfv7ujm5tgb6yhn"
}
```

**Kích hoạt Amazon GuardDuty qua AWS Console**

Truy cập [Bảng điều khiển Amazon GuardDuty](https://console.aws.amazon.com/guardduty/home)

Nhấp vào nút **Get Started**.

![](/images/p6/p61/6.1-1-GettingStarted.png)

Nhấp vào **Enable GuardDuty**

![](/images/p6/p61/6.1-2-Enable.png)


Truy cập **EKS Protection** trên menu bên trái và kiểm tra liệu Bảo vệ EKS đã được kích hoạt cho cả hai Audit Logs và Runtime Monitoring.

![](/images/p6/p61/6.1-3-EnableEKS.png)

Kéo xuống mục **Runtime Monitoring Configuration** và nhấp vào _Enable_:
![](/images/p6/p61/6.1-4-EnableRTMonitor.png)

Kéo xuống mục **Automated Agent Configuration** và nhấp vào _Enable_:
![](/images/p6/p61/6.1-5-EnableRTMonitorEKS.png)

Hãy kiểm tra tab **EKS clusters runtime coverage.**.

![](/images/p6/p61/6.1-6-RTCoverageEKS.png)

*Nếu cụm của bạn không xuất hiện trong danh sách Cụm hoặc thống kê phạm vi không hiển thị 1/1 (100%), hãy đợi thêm vài phút để Amazon GuardDuty hoàn thành việc triển khai mô hình giám sát.*

Bạn cũng có thể xác nhận việc triển khai Pod `aws-guardduty-agent` trong Cụm EKS của bạn.

```bash test=false
$ kubectl -n amazon-guardduty get pods                                                                                                                
NAME                        READY   STATUS    RESTARTS   AGE
aws-guardduty-agent-h7qg5   1/1     Running   0          58s
aws-guardduty-agent-hgbsg   1/1     Running   0          58s
aws-guardduty-agent-k7x2b   1/1     Running   0          58s
```

Sau đó, truy cập **Findings** trên menu bên trái. Bạn sẽ thấy rằng chưa có phát hiện nào có sẵn.

![](/images/p6/p61/6.1-7-findings.png)