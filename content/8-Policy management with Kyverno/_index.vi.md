---
title: "Policy management with Kyverno"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b> 8. </b>"
---

### Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=30
$ prepare-environment security/kyverno
```

Điều này sẽ thực hiện các thay đổi sau đây cho môi trường lab của bạn:

Cài đặt các add-on Kubernetes sau trong cụm EKS:

- Quản lý Chính sách Kyverno
- Các Chính sách Kyverno
- Báo cáo Chính sách

Bạn có thể xem Terraform áp dụng các thay đổi này [tại đây](https://github.com/aws-samples/eks-workshop-v2/tree/main/manifests/modules/security/kyverno/.workshop/terraform).
:::

Khi các container được sử dụng rộng rãi trong môi trường sản xuất, các nhóm DevOps, Bảo mật và Nền tảng cần một giải pháp để hợp tác và quản lý Việc quản trị và [Chính sách-dưới-dạng-Mã (PaC)](https://aws.github.io/aws-eks-best-practices/security/docs/pods/#policy-as-code-pac). Điều này đảm bảo rằng tất cả các nhóm khác nhau đều có thể có cùng một nguồn tin cậy trong việc đảm bảo bảo mật, cũng như sử dụng cùng một "ngôn ngữ" cơ bản khi mô tả các nhu cầu cá nhân của họ.

Kubernetes theo bản chất của nó được thiết kế để là một công cụ để xây dựng và điều phối, điều này có nghĩa là nó thiếu các rào cản được xác định trước. Để cung cấp cho các nhà xây dựng một cách để kiểm soát bảo mật, Kubernetes cung cấp (bắt đầu từ phiên bản 1.23) [Pod Security Admission (PSA)](https://kubernetes.io/docs/concepts/security/pod-security-admission/), một bộ kiểm soát đăng ký tích hợp mà thực hiện các điều khiển bảo mật được mô tả trong [Tiêu chuẩn Bảo mật Pod (PSS)](https://kubernetes.io/docs/concepts/security/pod-security-standards/), được kích hoạt mặc định trong Dịch vụ Amazon Elastic Kubernetes (EKS).

### Kyverno là gì

[Kyverno](https://kyverno.io/) (tiếng Hy Lạp có nghĩa là "quản trị") là một công cụ đặc biệt được thiết kế cho Kubernetes. Đó là một dự án của Cloud Native Computing Foundation (CNCF) cho phép các nhóm hợp tác và thực thi Chính sách-dưới-dạng-Mã.

Bộ động lực chính của Kyverno tích hợp với máy chủ API Kubernetes như [Bộ kiểm soát Đăng ký Động](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/), cho phép các chính sách **biến đổi** và **xác nhận** các yêu cầu API Kubernetes đến, đảm bảo tuân thủ với các quy tắc được xác định trước trước khi dữ liệu được lưu trữ và cuối cùng được áp dụng vào cụm.

Kyverno cho phép các nguồn tài nguyên Kubernetes mô tả theo cách khai báo viết bằng YAML, không cần học ngôn ngữ chính sách mới, và kết quả có sẵn dưới dạng các nguồn tài nguyên Kubernetes và sự kiện.

Các chính sách Kyverno có thể được sử dụng để **xác nhận**, **biến đổi**, và **tạo ra** cấu hình tài nguyên, cũng như **xác nhận** chữ ký hình ảnh và giấy chứng nhận, cung cấp tất cả các khối xây dựng cần thiết cho việc thực thi các tiêu chuẩn bảo mật chuỗi cung ứng phần mềm hoàn chỉnh.

### Cách làm việc của Kyverno

Như đã đề cập ở trên, Kyverno chạy như một Bộ kiểm soát Đăng ký Động trong một Cụm Kubernetes. Kyverno nhận các cuộc gọi webhook HTTP xác nhận và biến đổi từ máy chủ API Kubernetes và áp dụng các chính sách phù hợp để trả lại kết quả thực thi chính sách hoặc từ chối các yêu cầu. Nó cũng có thể được sử dụng để Kiểm tra các yêu cầu và theo dõi tư duy Bảo mật của môi trường trước khi thực thi.

Biểu đồ dưới đây hiển thị kiến trúc logic cấp cao của Kyverno.

![KyvernoArchitecture](assets/ky-arch.png)

Hai thành phần chính là Máy chủ Webhook & Bộ điều khiển Webhook. **Máy chủ Webhook** xử lý các yêu cầu AdmissionReview đến từ máy chủ API Kubernetes và gửi chúng đến Engine để xử lý. Nó được cấu hình động bởi **Bộ điều khiển Webhook** theo dõi các chính sách được cài đặt và sửa đổi các webhook để yêu cầu chỉ các tài nguyên phù hợp với các chính sách đó.

---

Trước khi tiếp tục với các lab, xác thực các tài nguyên Kyverno được cung cấp bởi tập lệnh `prepare-environment`.

```bash
$ kubectl -n kyverno get all
NAME                                                           READY   STATUS      RESTARTS   AGE
pod/kyverno-admission-controller-594c99487b-wpnsr              1/1     Running     0          8m15s
pod/kyverno-background-controller-7547578799-ltg7f             1/1     Running     0          8m15s
pod/kyverno-cleanup-admission-reports-28314690-6vjn4           0/1     Completed   0          3m20s
pod/kyverno-cleanup-cluster-admission-reports-28314690-2jjht   0/1     Completed   0          3m20s
pod/kyverno-cleanup-controller-79575cdb59-mlbz2                1/1     Running     0          8m15s
pod/kyverno-reports-controller-8668db7759-zxjdh                1/1     Running     0          8m15s
pod/policy-reporter-57f7dfc766-n48qk                           1/1     Running     0          7m53s

NAME                                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/kyverno-background-controller-metrics   ClusterIP   172.20.42.104    <none>        8000/TCP   8m16s
service/kyverno-cleanup-controller              ClusterIP   172.20.25.127    <none>        443/TCP    8m16s
service/kyverno-cleanup-controller-metrics      ClusterIP   172.20.184.34    <none>        8000/TCP   8m16s
service/kyverno-reports-controller-metrics      ClusterIP   172.20.84.109    <none>        8000/TCP   8m16s
service/kyverno-svc                             ClusterIP   172.20.157.100   <none>        443/TCP    8m16s
service/kyverno-svc-metrics                     ClusterIP   172.20.36.168    <none>        8000/TCP   8m16s
service/policy-reporter                         ClusterIP   172.20.175.164   <none>        8080/TCP   7m53s

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kyverno-admission-controller    1/1     1            1           8m16s
deployment.apps/kyverno-background-controller   1/1     1            1           8m16s
deployment.apps/kyverno-cleanup-controller      1/1     1            1           8m16s
deployment.apps/kyverno-reports-controller      1/1     1            1           8m16s
deployment.apps/policy-reporter                 1/1     1            1           7m53s

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/kyverno-admission-controller-594c99487b    1         1         1       8m16s
replicaset.apps/kyverno-background-controller-7547578799   1         1         1       8m16s
replicaset.apps/kyverno-cleanup-controller-79575cdb59      1         1         1       8m16s
replicaset.apps/kyverno-reports-controller-8668db7759      1         1         1       8m16s
replicaset.apps/policy-reporter-57f7dfc766                 1         1         1       7m53s

NAME                                                      SCHEDULE       SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/kyverno-cleanup-admission-reports           */10 * * * *   False     0        3m20s           8m16s
cronjob.batch/kyverno-cleanup-cluster-admission-reports   */10 * * * *   False     0        3m20s           8m16s

NAME                                                           COMPLETIONS   DURATION   AGE
job.batch/kyverno-cleanup-admission-reports-28314690           1/1           13s        3m20s
job.batch/kyverno-cleanup-cluster-admission-reports-28314690   1/1           10s        3m20s
```