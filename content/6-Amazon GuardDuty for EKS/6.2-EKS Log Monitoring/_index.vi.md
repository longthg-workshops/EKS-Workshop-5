---
title: "Giám sát log EKS"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 6.2 </b>"
---

#### Giám sát nhật ký kiểm tra EKS

Khi kích hoạt, Trình Giám sát Nhật ký Kiểm tra EKS ngay lập tức bắt đầu giám sát nhật ký kiểm tra Kubernetes từ các cụm của bạn và phân tích chúng để phát hiện hoạt động có thể độc hại và nghi ngờ. Nó tiêu thụ các sự kiện nhật ký kiểm tra Kubernetes trực tiếp từ tính năng nhật ký điều khiển Amazon EKS thông qua một luồng nhật ký dòng độc lập và trùng lặp.

Trong bài thực hành này, chúng ta sẽ tạo ra một số phát hiện giám sát kiểm tra Kubernetes trong cụm Amazon EKS của bạn, được liệt kê dưới đây.

- `Execution:Kubernetes/ExecInKubeSystemPod`
- `Discovery:Kubernetes/SuccessfulAnonymousAccess`
- `Policy:Kubernetes/AnonymousAccessGranted`
- `Impact:Kubernetes/SuccessfulAnonymousAccess`
- `Policy:Kubernetes/AdminAccessToDefaultServiceAccount`
- `Policy:Kubernetes/ExposedDashboard`
- `PrivilegeEscalation:Kubernetes/PrivilegedContainer`
- `Persistence:Kubernetes/ContainerWithSensitiveMount`


**Phát hiện này cho thấy rằng một lệnh đã được thực thi bên trong một Pod trong Namespace `kube-system` trên Cụm EKS.**

Trước tiên hãy chạy một Pod trong Namespace `kube-system` cung cấp truy cập vào môi trường shell của nó.

```bash
$ kubectl -n kube-system run nginx --image=nginx
$ kubectl wait --for=condition=ready pod nginx -n kube-system
$ kubectl -n kube-system get pod nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          28s
```

Tiếp theo chạy lệnh dưới đây để tạo ra mục phát hiện `Execution:Kubernetes/ExecInKubeSystemPod`:

```bash
$ kubectl -n kube-system exec nginx -- pwd
/
```

Trong vài phút, chúng ta sẽ thấy mục phát hiện `Execution:Kubernetes/ExecInKubeSystemPod` trên [bảng điều khiển Findings của GuardDuty](https://console.aws.amazon.com/guardduty/home#/findings).

![](/images/p6/p62/6.2-1-FindingsMenu.png)

Nếu bạn nhấp vào mục phát hiện này, nó sẽ mở một tab ở bên phải màn hình, hiển thị chi tiết về phát hiện và một giải thích ngắn gọn về nó.

![](/images/p6/p62/6.2-2-FindingDetails.png)

Nó cũng cung cấp cho bạn lựa chọn để điều tra phát hiện bằng cách sử dụng Amazon Detective.

![](/images/p6/p62/6.2-3-DetectiveOnFinding.png)

Một thông tin quan trọng đáng xem xét là **Hành động** được phát hiện, trên mục này (Loại giám sát nhật ký), chúng ta có thể thấy rằng liên quan đến một `KUBERNETES_API_CALL`.

![](/images/p6/p62/6.2-4-Actions.png)

Dọn dẹp Pod gây ra phát hiện:

```bash
$ kubectl -n kube-system delete pod nginx
```

Trong bài thực hành tiếp theo này, chúng ta sẽ cấp quyền quản trị cụm cho một Tài khoản Dịch vụ. Điều này không phải là thực hành tốt vì có thể dẫn đến việc Các Pod được kết nối với Tài khoản Dịch vụ này được khởi chạy không cẩn thận với quyền quản trị, cho phép người dùng có quyền `exec` vào các Pod này, để leo thang và có quyền truy cập không hạn chế vào cụm.

Để mô phỏng điều này, chúng ta cần gán vai trò `cluster-admin` cho Tài khoản Dịch vụ `default` trong không gian tên `default`.

```bash
$ kubectl -n default create rolebinding sa-default-admin --clusterrole cluster-admin --serviceaccount default:default
```

Trong vài phút, bạn sẽ thấy phát hiện `Policy:Kubernetes/AdminAccessToDefaultServiceAccount` trên [bảng điều khiển Phát hiện GuardDuty](https://console.aws.amazon.com/guardduty/home#/findings). Dành chút thời gian để phân tích chi tiết của Finding, Action và Detective Investigation.

![](/images/p6/p62/6.2-5-IAM.png)

Xóa RoleBinding vi phạm bằng cách chạy lệnh sau.

```bash
$ kubectl -n default delete rolebinding sa-default-admin 
```

**Phát hiện này thông báo cho bạn biết rằng bảng điều khiển Cluster EKS của bạn đã được tiếp cận từ internet thông qua dịch vụ Load Balancer. Một bảng điều khiển được tiếp cận khiến giao diện quản lý của cụm của bạn có thể truy cập công khai từ internet và cho phép kẻ tấn công xấu khai thác bất kỳ khe hở xác thực và kiểm soát truy cập nào có thể tồn tại.**

Để mô phỏng điều này, chúng ta cần cài đặt thành phần bảng điều khiển Kubernetes. Chúng tôi sẽ sử dụng phiên bản v2.7.0 của bảng điều khiển, đây là phiên bản tương thích mới nhất với Cluster EKS phiên bản v1.29 dựa trên [ghi chú phát hành](https://github.com/kubernetes/dashboard/releases/tag/v2.7.0).
Sau đó, chúng ta có thể tiếp cận bảng điều khiển từ Internet với loại Dịch vụ `LoadBalancer`, điều này sẽ tạo ra một Load Balancer Mạng (NLB) trong tài khoản AWS của bạn.

Chạy các lệnh sau để cài đặt thành phần bảng điều khiển Kubernetes. Điều này sẽ tạo ra một Namespace mới gọi là `kubernetes-dashboard`, và tất cả các tài nguyên sẽ được triển khai ở đó.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
$ kubectl -n kubernetes-dashboard rollout status deployment/kubernetes-dashboard
$ kubectl -n kubernetes-dashboard get pods
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-64bcc67c9c-tt9vl   1/1     Running   0          66s
kubernetes-dashboard-5c8bd6b59-945zj         1/1     Running   0          66s
```

Bây giờ, hãy sửa đổi Dịch vụ `kubernetes-dashboard` mới tạo thành loại `LoadBalancer`.

```bash
$ kubectl -n kubernetes-dashboard patch svc kubernetes-dashboard -p='{"spec": {"type": "LoadBalancer"}}'
```

Sau vài phút, NLB sẽ được tạo ra và hiển thị một địa chỉ có thể truy cập công khai trong Dịch vụ `kubernetes-dashboard`.

```bash
$ kubectl -n kubernetes-dashboard get svc
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP      172.20.8.169     <none>                                                                    8000/TCP        3m
kubernetes-dashboard        LoadBalancer   172.20.218.132   ad0fbc5914a2c4d1baa8dcc32101196b-2094501166.us-west-2.elb.amazonaws.com   443:32762/TCP   3m1s
```

Nếu bạn quay lại [bảng điều khiển Phát hiện GuardDuty](https://console.aws.amazon.com/guardduty/home#/findings), bạn sẽ thấy phát hiện `Policy:Kubernetes/ExposedDashboard`. Một lần nữa, hãy sử dụng một vài thời gian để phân tích chi tiết của _Phát hiện_, _Hành động_ và _Điều tra phát hiện_.

![](/images/p6/p62/6.2-6-Dashboard.png)

Gỡ cài đặt các thành phần bảng điều khiển Kubernetes bằng cách chạy lệnh sau:

```bash
$ kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Trong lab này, bạn sẽ tạo một container với Security Context `privileged`, có quyền truy cập cấp root trong Namespace `default` của cụm EKS của bạn. Container này có thể truy cập vào một thư mục nhạy cảm từ máy chủ, được gắn kết và truy cập như một thư mục dữ liệu bên trong container của bạn.

Bài tập này sẽ tạo ra hai kết quả khác nhau, `PrivilegeEscalation:Kubernetes/PrivilegedContainer` chỉ ra rằng một container được khởi chạy với quyền `Privileged`, và `Persistence:Kubernetes/ContainerWithSensitiveMount` chỉ ra một đường dẫn máy chủ bên ngoài nhạy cảm được gắn kết bên trong container.

Để mô phỏng kết quả, bạn sẽ sử dụng một tập tin mô tả được cấu hình trước với một số tham số cụ thể đã được đặt trước, `SecurityContext: privileged: true` và cũng các tùy chọn `volume` và `volumeMount`, ánh xạ thư mục máy chủ `/etc` thành `mount` thư mục `Pod` `/host-etc`.

```file
manifests/modules/security/Guardduty/mount/privileged-pod-example.yaml
```

Áp dụng tập tin mô tả được hiển thị trên với lệnh sau:

```bash
$ kubectl apply -f ~/environment/eks-workshop/modules/security/Guardduty/mount/privileged-pod-example.yaml
```
*Pod này chỉ chạy một lần, cho đến khi đạt được trạng thái `Completed`*

Trong vài phút tới, chúng ta sẽ thấy hai kết quả `PrivilegeEscalation:Kubernetes/PrivilegedContainer` và `Persistence:Kubernetes/ContainerWithSensitiveMount` trên [bảng điều khiển Tìm thấy của GuardDuty](https://console.aws.amazon.com/guardduty/home#/findings).

![](/images/p6/p62/6.2-7-Privilege.png)

![](/images/p6/p62/6.2-8-Persistence.png)

Một lần nữa, hãy dành một ít thời gian để phân tích chi tiết _Finding_, _Action_ và _Detective Investigation_.

Dọn dẹp Pod bằng cách chạy lệnh dưới đây:

```bash
$ kubectl delete -f ~/environment/eks-workshop/modules/security/Guardduty/mount/privileged-pod-example.yaml
```