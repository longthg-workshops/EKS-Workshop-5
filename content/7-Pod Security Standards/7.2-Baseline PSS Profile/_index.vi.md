---
title: "Cấu hình PSS cơ bản"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 7.2 </b>"
---

#### Hạn chế quyền cho Pod trong Kubernetes

Sẽ thế nào nếu chúng ta muốn hạn chế quyền mà một Pod có thể yêu cầu ? Chẳng hạn như quyền `privileged` mà chúng ta cung cấp cho Pod `assets` trong phần trước (có thể nguy hiểm, cho phép một kẻ tấn công truy cập vào tài nguyên của máy chủ bên ngoài của container).

Chính sách Baseline PSS là một chính sách hạn chế tối thiểu ngăn chặn việc tăng quyền biết trước. Hãy thêm nhãn vào `assets` Namespace để kích hoạt nó:

```kustomization
modules/security/pss-psa/baseline-namespace/namespace.yaml
Namespace/assets
```

Chạy Kustomize để áp dụng thay đổi này và thêm nhãn vào namespace `assets`:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/security/pss-psa/baseline-namespace
Cảnh báo: các pod hiện có trong namespace "assets" vi phạm mức thực thi mới của PodSecurity "baseline:latest"
Cảnh báo: assets-64c49f848b-gmrtt: privileged
namespace/assets được cấu hình
serviceaccount/assets không thay đổi
configmap/assets không thay đổi
service/assets không thay đổi
deployment.apps/assets không thay đổi
```

Bạn có thể thấy ở trên rằng chúng ta đã nhận được một cảnh báo rằng các Pod trong `assets` Deployment vi phạm Baseline PSS, được cung cấp bởi nhãn Namespace `pod-security.kubernetes.io/warn`. Bây giờ, hãy tái chế các Pods trong `assets` Deployment:

```bash
$ kubectl -n assets delete pod --all
```

Hãy kiểm tra xem các Pods có đang chạy không:

```bash
$ kubectl -n assets get pod   
Không tìm thấy tài nguyên trong namespace assets.
```

Như bạn có thể thấy, không có Pods nào đang chạy do nhãn Namespace `pod-security.kubernetes.io/enforce`, nhưng chúng ta không biết ngay lập tức lý do tại sao. Khi sử dụng độc lập, các chế độ PSA có các phản ứng khác nhau dẫn đến các trải nghiệm người dùng khác nhau. Chế độ thực thi ngăn chặn việc tạo Pods nếu các đặc tả Pod tương ứng vi phạm hồ sơ PSS được cấu hình. Tuy nhiên, trong chế độ này, các đối tượng Kubernetes không phải là Pod tạo ra Pods, chẳng hạn như Deployments, sẽ không bị ngăn chặn khỏi việc áp dụng vào cụm, ngay cả khi các đặc tả Pod trong đó vi phạm hồ sơ PSS được áp dụng. Trong trường hợp này, Deployment được áp dụng trong khi các Pods bị ngăn chặn khỏi việc áp dụng.

Chạy lệnh dưới đây để kiểm tra tình trạng điều kiện của tài nguyên Deployment:

```bash
$ kubectl get deployment -n assets assets -o yaml | yq '.status'
- lastTransitionTime: "2022-11-24T04:49:56Z"
  lastUpdateTime: "2022-11-24T05:10:41Z"
  message: ReplicaSet "assets-7445d46757" has successfully progressed.
  reason: NewReplicaSetAvailable
  status: "True"
  type: Progressing
- lastTransitionTime: "2022-11-24T05:10:49Z"
  lastUpdateTime: "2022-11-24T05:10:49Z"
  message: 'pods "assets-67d5fc995b-8r9t2" is forbidden: violates PodSecurity "baseline:latest": privileged (container "assets" must not set securityContext.privileged=true)'
  reason: FailedCreate
  status: "True"
  type: ReplicaFailure
- lastTransitionTime: "2022-11-24T05:10:56Z"
  lastUpdateTime: "2022-11-24T05:10:56Z"
  message: Deployment does not have minimum availability.
  reason: MinimumReplicasUnavailable
  status: "False"
  type: Available
```

Trong một số tình huống, không có biểu hiện ngay lập tức rằng đối tượng Deployment được áp dụng thành công phản ánh việc tạo Pods thất bại. Các đặc tả Pod vi phạm không tạo ra Pods. Kiểm tra tài nguyên Deployment với `kubectl get deploy -o yaml ...` sẽ tiết lộ tin nhắn từ các Pod(s) thất bại trong `.status.conditions` như đã thấy trong quá trình kiểm tra của chúng tôi ở trên.

Trong cả hai chế độ PSA kiểm tra và cảnh báo, các hạn chế Pod không ngăn chặn việc tạo và khởi chạy các Pods vi phạm. Tuy nhiên, trong các chế độ này, các chú thích kiểm tra của audit trên các sự kiện nhật ký kiểm tra API server và cảnh báo đối với các khách hàng API server (ví dụ: kubectl) được kích hoạt. Điều này xảy ra khi Pods, cũng như các đối tượng tạo ra Pods, chứa đặc tả Pod vi phạm PSS.

Bây giờ, hãy sửa tài nguyên Deployment `assets` để nó chạy bằng cách loại bỏ cờ `privileged`:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/security/pss-psa/baseline-workload
namespace/assets không thay đổi
serviceaccount/assets không

 thay đổi
configmap/assets không thay đổi
service/assets không thay đổi
deployment.apps/assets được cấu hình
```

Lần này chúng tôi không nhận được cảnh báo nên hãy kiểm tra xem các Pods có đang chạy không, và chúng tôi có thể xác nhận rằng nó không chạy dưới quyền `root` nữa:

```bash
$ kubectl -n assets get pod   
NAME                      READY   STATUS    RESTARTS   AGE
assets-864479dc44-d9p79   1/1     Running   0          15s

$ kubectl -n assets exec $(kubectl -n assets get pods -o name) -- whoami
nginx
```

Vì chúng tôi đã khắc phục lỗi Pod chạy trong chế độ `privileged` nên bây giờ nó được phép chạy dưới hồ sơ Baseline.
