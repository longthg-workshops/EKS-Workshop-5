---
title: "Quản lý khoá niêm phong"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 5.3.4 </b>"
---

#### Quản lý khoá niêm phong

Để giải mã dữ liệu được mã hóa bên trong một SealedSecret, bạn cần sử dụng khóa niêm phong được quản lý bởi bộ điều khiển. Có thể có tình huống khi bạn cố gắng khôi phục trạng thái ban đầu của một cụm sau một thảm họa hoặc bạn muốn tận dụng quy trình GitOps để triển khai các tài nguyên Kubernetes, bao gồm SealedSecrets, từ một kho lưu trữ Git và tạo một cụm EKS mới. Bộ điều khiển được triển khai trong cụm EKS mới phải sử dụng cùng một khóa niêm phong để có thể mở niêm phong các SealedSecrets.

Chạy lệnh sau để lấy khóa niêm phong từ cụm. Trong môi trường sản xuất, việc sử dụng Kubernetes RBAC để cấp quyền cho một tập hợp hạn chế các khách hàng thực hiện thao tác này được coi là một thực hành tốt.

```bash
$ kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml \
  > /tmp/master-sealing-key.yaml
```

Để kiểm tra hoạt động, hãy xóa Secret chứa khóa niêm phong và tái chế bộ điều khiển Sealed Secrets:

```bash
$ kubectl delete secret -n kube-system -l sealedsecrets.bitnami.com/sealed-secrets-key
$ kubectl -n kube-system delete pod -l name=sealed-secrets-controller
$ kubectl wait --for=condition=Ready --timeout=30s pods -l name=sealed-secrets-controller -n kube-system
```

Bây giờ, chúng ta sẽ kiểm tra nhật ký của bộ điều khiển. Lưu ý rằng nó không thể giải mã SealedSecret:

```bash
$ kubectl logs deployment/sealed-secrets-controller -n kube-system
[...]
2022/11/18 22:47:42 Updating catalog/catalog-sealed-db
2022/11/18 22:47:43 Error updating catalog/catalog-sealed-db, giving up: no key could decrypt secret (password, username, endpoint, name)
E1118 22:47:43.030178       1 controller.go:175] no key could decrypt secret (password, username, endpoint, name)
2022/11/18 22:47:43 Event(v1.ObjectReference{Kind:"SealedSecret", Namespace:"catalog", Name:"catalog-sealed-db", UID:"a6705e6f-72a1-43f5-8c0b-4f45b9b6f5fb", APIVersion:"bitnami.com/v1alpha1", ResourceVersion:"519192", FieldPath:""}): type: 'Warning' reason: 'ErrUnsealFailed' Failed to unseal: no key could decrypt secret (password, username, endpoint, name)
```

Điều này xảy ra vì chúng ta đã xóa khóa niêm phong, làm cho bộ điều khiển tạo ra một khóa mới khi nó khởi động. Điều này thực tế làm cho tất cả các tài nguyên SealedSecret của chúng ta không thể truy cập được bởi bộ điều khiển này. May mắn thay, chúng ta đã trước đó lưu nó vào `/tmp/master-sealing-key.yaml` để có thể tạo lại nó trong cụm EKS:

```bash
$ kubectl apply -f /tmp/master-sealing-key.yaml
$ kubectl -n kube-system delete pod -l name=sealed-secrets-controller
$ kubectl wait --for=condition=Ready --timeout=30s pods -l name=sealed-secrets-controller -n kube-system
```

Kiểm tra nhật ký một lần nữa sẽ thấy rằng lần này bộ điều khiển đã nhận được khóa niêm phong mà chúng ta đã khôi phục và đã mở niêm phong `catalog-sealed-db` của chúng ta:

```bash
$ kubectl logs deployment/sealed-secrets-controller -n kube-system
[...]
2022/11/18 22:52:51 Updating catalog/catalog-sealed-db
2022/11/18 22:52:51 Event(v1.ObjectReference{Kind:"SealedSecret", Namespace:"catalog", Name:"catalog-sealed-db", UID:"a6705e6f-72a1-43f5-8c0b-4f45b9b6f5fb", APIVersion:"bitnami.com/v1alpha1", ResourceVersion:"519192", FieldPath:""}): type: 'Normal' reason: 'Unsealed' SealedSecret unsealed successfully
```

Tệp `/tmp/master-sealing-key.yaml` chứa cặp khóa công khóa riêng được tạo bởi bộ điều khiển. Nếu tệp này bị lộ, tất cả các biểu hiện SealedSecret có thể được mở niêm phong và thông tin nhạy cảm được mã hóa mà chúng lưu trữ sẽ được tiết lộ. Do đó, tệp này phải được bảo vệ bằng cách cấp quyền truy cập ít nhất đặc quyền. Đối với hướng dẫn bổ sung về các chủ đề như làm mới khóa niêm phong và quản lý khóa niêm phong thủ công, vui lòng tham khảo [tài liệu](https://github.com/bitnami-labs/sealed-secrets#secret-rotation).

Một lựa chọn để bảo mật khóa niêm phong là lưu nội dung của tệp `/tmp/master-sealing-key.yaml` dưới dạng tham số SecureString trong AWS Systems Manager Parameter Store. Tham số có thể được bảo vệ bằng cách sử dụng một khóa quản lý khách hàng KMS (CMK) và bạn có thể sử dụng Chính sách khóa để hạn chế tập hợp các nguyên tắc IAM nào có thể sử dụng khóa này để lấy tham số. Bên cạnh đó, bạn cũng có thể kích hoạt tự động làm mới của CMK này trong KMS.

Lưu ý rằng các tham số tier tiêu chuẩn hỗ trợ một giá trị tham số tối đa là 4096 ký tự. Do đó, với kích thước của tệp master.yaml, bạn sẽ phải lưu nó dưới dạng tham số trong tier Advanced.