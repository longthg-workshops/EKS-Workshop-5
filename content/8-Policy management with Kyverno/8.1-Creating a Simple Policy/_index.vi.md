---
title: "Tạo một chính sách đơn giản"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 8.1 </b>"
---

Để hiểu về Chính sách Kyverno, chúng ta sẽ bắt đầu bài thực hành của mình với một yêu cầu nhãn Pod đơn giản. Như bạn có thể biết, Nhãn trong Kubernetes có thể được sử dụng để gắn thẻ các đối tượng và tài nguyên trong Cụm.

Dưới đây là một chính sách mẫu yêu cầu một Nhãn `CostCenter`.

```file
manifests/modules/security/kyverno/simple-policy/require-labels-policy.yaml
```

Kyverno có 2 loại tài nguyên Chính sách, **ClusterPolicy** được sử dụng cho Tài nguyên Toàn cụm và **Policy** được sử dụng cho Tài nguyên Thuộc vùng. Ví dụ trên hiển thị một ClusterPolicy. Dành một ít thời gian để đào sâu và kiểm tra các chi tiết dưới đây trong cấu hình.

- Dưới phần spec của Chính sách, có một thuộc tính `validationFailureAction` nó cho biết cho Kyverno nếu tài nguyên đang được xác nhận nên được cho phép nhưng được báo cáo `Audit` hoặc bị chặn `Enforce`. Mặc định là Audit, ví dụ được đặt thành Enforce.
- `rules` là một hoặc nhiều quy tắc để được xác nhận.
- Tuyên bố `match` thiết lập phạm vi của những gì sẽ được kiểm tra. Trong trường hợp này, nó là bất kỳ tài nguyên `Pod` nào.
- Tuyên bố `validate` cố gắng kiểm tra tích cực những gì được xác định. Nếu tuyên bố, khi so sánh với tài nguyên yêu cầu, là đúng, nó được phép. Nếu sai, nó bị chặn.
- `message` là điều gì sẽ được hiển thị cho người dùng nếu quy tắc này không vượt qua xác nhận.
- Đối tượng `pattern` định nghĩa mẫu nào sẽ được kiểm tra trong tài nguyên. Trong trường hợp này, nó đang tìm kiếm `metadata.labels` với `CostCenter`.

Chính sách Ví dụ ở trên sẽ chặn bất kỳ việc tạo Pod nào không có nhãn `CostCenter`.

Tạo chính sách bằng lệnh sau.

```bash
$ kubectl apply -f ~/environment/eks-workshop/modules/security/kyverno/simple-policy/require-labels-policy.yaml

clusterpolicy.kyverno.io/require-labels created
```

Tiếp theo, hãy xem các Pod đang chạy trong namespace `ui`, chú ý các nhãn đã áp dụng.

```bash
$ kubectl -n ui get pods --show-labels
NAME                  READY   STATUS    RESTARTS   AGE   LABELS
ui-67d8cf77cf-d4j47   1/1     Running   0          9m    app.kubernetes.io/component=service,app.kubernetes.io/created-by=eks-workshop,app.kubernetes.io/instance=ui,app.kubernetes.io/name=ui,pod-template-hash=67d8cf77cf
```

Kiểm tra Pod đang chạy không có Nhãn được yêu cầu và Kyverno không chấm dứt nó, điều này xảy ra vì như đã thấy trước đó, Kyverno hoạt động như một `AdmissionController` và sẽ không can thiệp vào các tài nguyên đã tồn tại trong cụm.

Tuy nhiên, nếu bạn xóa Pod đang chạy, nó sẽ không thể được tạo lại vì nó không có Nhãn được yêu cầu. Hãy tiến hành xóa Pod đang chạy trong không gian tên `ui`.

```bash
$ kubectl -n ui delete pod --all
pod "ui-67d8cf77cf-d4j47" deleted
$ kubectl -n ui get pods
No resources found in ui namespace.
```

Như đã đề cập, Pod không được tạo lại, hãy thử cưỡng chế triển khai của `ui`.

```bash expectError=true
$ kubectl -n ui rollout restart deployment/ui
error: failed to patch: admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Deployment/ui/ui was blocked due to the following policies 

require-labels:
  autogen-check-team: 'validation error: Label ''CostCenter'' is required to deploy
    the Pod. rule autogen-check-team failed at path /spec/template/metadata/labels/CostCenter/'
```

Cuộc triển khai thất bại với webhook thẩm định từ chối yêu cầu do Chính sách Kyverno `require-labels`.

Bạn cũng có thể kiểm tra thông báo lỗi này mô tả việc triển khai `ui`, hoặc hiển thị `sự kiện` trong không gian tên `ui`.

```bash
$ kubectl -n ui describe deployment ui
...
Events:
  Type     Reason             Age                From                   Message
  ----     ------             ----               ----                   -------
  Warning  PolicyViolation    12m (x2 over 9m)   kyverno-scan           policy require-labels/autogen-check-team fail: validation error: Label 'CostCenter' is required to deploy the Pod. rule autogen-check-team failed at path /spec/template/metadata/labels/CostCenter/

$ kubectl -n ui get events | grep PolicyViolation
9m         Warning   PolicyViolation     pod/ui-67d8cf77cf-hvqcd    policy require-labels/check-team fail: validation error: Label 'CostCenter' is required to deploy the Pod. rule check-team failed at path /metadata/labels/CostCenter/
9m         Warning   PolicyViolation     replicaset/ui-67

d8cf77cf   policy require-labels/autogen-check-team fail: validation error: Label 'CostCenter' is required to deploy the Pod. rule autogen-check-team failed at path /spec/template/metadata/labels/CostCenter/
9m         Warning   PolicyViolation     deployment/ui              policy require-labels/autogen-check-team fail: validation error: Label 'CostCenter' is required to deploy the Pod. rule autogen-check-team failed at path /spec/template/metadata/labels/CostCenter/
```

Bây giờ thêm nhãn yêu cầu `CostCenter` vào `ui` Deployment, sử dụng bản vá Kustomization dưới đây.

```kustomization
modules/security/kyverno/simple-policy/ui-labeled/deployment.yaml
Deployment/ui
```

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/security/kyverno/simple-policy/ui-labeled
namespace/ui unchanged
serviceaccount/ui unchanged
configmap/ui unchanged
service/ui unchanged
deployment.apps/ui configured
$ kubectl -n ui rollout status deployment/ui
$ kubectl -n ui get pods --show-labels
NAME                  READY   STATUS    RESTARTS   AGE   LABELS
ui-5498685db8-k57nk   1/1     Running   0          60s   CostCenter=IT,app.kubernetes.io/component=service,app.kubernetes.io/created-by=eks-workshop,app.kubernetes.io/instance=ui,app.kubernetes.io/name=ui,pod-template-hash=5498685db8
```

Như bạn có thể thấy, webhook thẩm định đã xác nhận chính sách và Pod được tạo ra với Nhãn chính xác `CostCenter=IT`!

### Quy tắc Chuyển đổi

Trong các ví dụ trên, bạn đã kiểm tra cách Chính sách Xác nhận hoạt động trong hành vi mặc định của chúng được xác định trong `validationFailureAction`. Tuy nhiên, Kyverno cũng có thể được sử dụng để quản lý các quy tắc Chuyển đổi trong Chính sách, để sửa đổi bất kỳ Yêu cầu API nào để đáp ứng hoặc bắt buộc các yêu cầu cụ thể trên các tài nguyên Kubernetes. Việc biến đổi tài nguyên xảy ra trước khi xác nhận, vì vậy các quy tắc xác nhận sẽ không mâu thuẫn với các thay đổi thực hiện bởi phần mutation.

Dưới đây là một Chính sách mẫu có một quy tắc biến đổi được xác định, sẽ được sử dụng để tự động thêm nhãn mặc định `CostCenter=IT` của chúng tôi vào bất kỳ `Pod` nào.

```file
manifests/modules/security/kyverno/simple-policy/add-labels-mutation-policy.yaml
```

Chú ý phần `mutate`, dưới `spec` của ClusterPolicy.

Hãy tiến hành và tạo Chính sách trên bằng lệnh sau.

```bash
$ kubectl apply -f  ~/environment/eks-workshop/modules/security/kyverno/simple-policy/add-labels-mutation-policy.yaml

clusterpolicy.kyverno.io/add-labels created
```

Để xác nhận Webhook Chuyển đổi, lần này hãy triển khai lại `assets` Deployment mà không chỉ định rõ một nhãn:

```bash
$ kubectl -n assets rollout restart deployment/assets
deployment.apps/assets restarted
$ kubectl -n assets rollout status deployment/assets
deployment "assets" successfully rolled out
```

Xác nhận nhãn `CostCenter=IT` được tự động thêm vào Pod để đáp ứng yêu cầu chính sách, dẫn đến việc tạo Pod thành công ngay cả khi Deployment không có nhãn được chỉ định:

```bash
$ kubectl -n assets get pods --show-labels 
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
assets-bb88b4789-kmk62   1/1     Running   0          25s   CostCenter=IT,app.kubernetes.io/component=service,app.kubernetes.io/created-by=eks-workshop,app.kubernetes.io/instance=assets,app.kubernetes.io/name=assets,pod-template-hash=bb88b4789
```

Cũng có thể biến đổi các tài nguyên hiện có trong Cụm Amazon EKS của bạn bằng các Chính sách Kyverno sử dụng các tham số `patchStrategicMerge` và `patchesJson6902` trong Chính sách Kyverno của bạn.

Đó chỉ là một ví dụ đơn giản về các nhãn cho các Pod của chúng tôi với các quy tắc Xác nhận và Chuyển đổi. Điều này có thể được áp dụng vào các tình huống khác nhau như hạn chế Hình ảnh từ các kho lưu trữ không xác định, thêm Dữ liệu vào Bản đồ Cấu hình, Tolerations và nhiều hơn nữa. Trong các bài thực hành tiếp theo, bạn sẽ đi qua một số trường hợp sử dụng nâng cao hơn.