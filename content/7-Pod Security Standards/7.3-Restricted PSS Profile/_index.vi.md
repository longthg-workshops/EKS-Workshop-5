---
title: "Cấu hình PSS bị hạn chế"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 7.3 </b>"
---

Cuối cùng, chúng ta có thể xem xét Restricted profile, đó là chính sách bị hạn chế nặng nhất theo các thực hành tốt nhất hiện tại của Pod hardening. Thêm nhãn vào namespace `assets` để kích hoạt tất cả các chế độ PSA cho Restricted PSS profile:

```kustomization
modules/security/pss-psa/restricted-namespace/namespace.yaml
Namespace/assets
```

Chạy Kustomize để áp dụng thay đổi này để thêm nhãn vào namespace `assets`:

```bash  timeout=180 hook=restricted-namespace
$ kubectl apply -k ~/environment/eks-workshop/modules/security/pss-psa/restricted-namespace
Cảnh báo: các pod hiện tại trong namespace "assets" vi phạm mức thực thi PodSecurity mới "restricted:latest"
Cảnh báo: assets-d59d88b99-flkgp: allowPrivilegeEscalation != false, runAsNonRoot != true, seccompProfile
namespace/assets configured
serviceaccount/assets unchanged
configmap/assets unchanged
service/assets unchanged
deployment.apps/assets unchanged
```

Tương tự như Baseline profile, chúng ta nhận được cảnh báo rằng Deployment của assets đang vi phạm Restricted profile.

```bash
$ kubectl -n assets delete pod --all
pod "assets-d59d88b99-flkgp" deleted
```

Các Pod không được tạo lại:

```bash test=false
$ kubectl -n assets get pod   
Không tìm thấy tài nguyên trong namespace assets.
```

Đầu ra trên cho thấy PSA không cho phép tạo ra các Pod trong Namespace `assets`, vì cấu hình bảo mật của Pod vi phạm Restricted PSS profile. Hành vi này giống như chúng ta đã thấy trước đó ở phần trước.

Trong trường hợp của Restricted profile, thực tế chúng ta cần khóa một số cấu hình bảo mật một cách tích cực để đáp ứng profile đó. Hãy thêm một số điều khiển bảo mật vào cấu hình Pod để tuân thủ với Privileged PSS profile được cấu hình cho namespace `assets`:

```kustomization
modules/security/pss-psa/restricted-workload/deployment.yaml
Deployment/assets
```

Chạy Kustomize để áp dụng những thay đổi này, sau đó chúng ta sẽ tạo lại Deployment:

```bash timeout=180 hook=restricted-deploy-with-changes
$ kubectl apply -k ~/environment/eks-workshop/modules/security/pss-psa/restricted-workload
namespace/assets unchanged
serviceaccount/assets unchanged
configmap/assets unchanged
service/assets unchanged
deployment.apps/assets configured
```

Bây giờ, chạy các lệnh dưới đây để kiểm tra PSA cho phép việc tạo ra Deployment và Pod với các thay đổi trên trong namespace `assets`:

```bash
$ kubectl -n assets  get pod   
NAME                     READY   STATUS    RESTARTS   AGE
assets-8dd6fc8c6-9kptf   1/1     Running   0          3m6s
```

Đầu ra trên cho thấy rằng PSA đã cho phép vì cấu hình bảo mật của Pod tuân thủ Restricted PSS profile.

Lưu ý rằng các quyền bảo mật được nêu trên không phải là danh sách toàn diện các điều khiển được phép trong Restricted PSS profile. Đối với các điều khiển bảo mật chi tiết được phép / không được phép trong mỗi profile PSS, vui lòng tham khảo [tài liệu](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted).