---
title: "Báo cáo & Kiểm tra"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 8.4 </b>"
---

# Báo cáo Chính sách của Kyverno

Kyverno cũng bao gồm một công cụ [Báo cáo Chính sách](https://kyverno.io/docs/policy-reports/) sử dụng định dạng mở được xác định bởi Nhóm làm việc Chính sách Kubernetes và triển khai như tài nguyên tùy chỉnh trong cụm. Kyverno phát ra các báo cáo này khi các hành động nhập cảnh như *TẠO*, *CẬP NHẬT*, và *XÓA* được thực hiện trong cụm, chúng cũng được tạo ra như là kết quả của các quét nền kiểm tra chính sách trên các tài nguyên đã tồn tại.

Cho đến nay trong buổi thực hành, chúng ta đã tạo một số Chính sách cho các quy tắc cụ thể. Khi một tài nguyên được phù hợp với một hoặc nhiều quy tắc theo định nghĩa của chính sách và vi phạm bất kỳ quy tắc nào, một mục sẽ được tạo ra trong báo cáo cho mỗi vi phạm, dẫn đến nhiều mục nếu cùng một tài nguyên phù hợp và vi phạm nhiều quy tắc. Khi các tài nguyên bị xóa, mục của chúng sẽ được loại bỏ khỏi các báo cáo, có nghĩa là Báo cáo Kyverno luôn đại diện cho trạng thái hiện tại của cụm và không ghi lại thông tin lịch sử.

Như đã thấy trước đó, Kyverno có hai loại `validationFailureAction`, chế độ `Kiểm tra` sẽ cho phép tài nguyên được tạo ra và báo cáo hành động trong các Báo cáo Chính sách, hoặc `Bắt buộc` sẽ từ chối việc tạo tài nguyên, nhưng cũng không thêm mục vào các Báo cáo Chính sách. Ví dụ, nếu một Chính sách trong chế độ `Kiểm tra` chứa một quy tắc đơn giản yêu cầu tất cả các tài nguyên phải thiết lập nhãn `CostCenter` và một Pod được tạo ra mà không có nhãn đó, Kyverno sẽ cho phép tạo Pod nhưng ghi lại nó là kết quả `FAIL` trong một Báo cáo Chính sách do vi phạm quy tắc. Nếu Chính sách này cùng chế độ `Bắt buộc`, Kyverno sẽ ngăn chặn ngay lập tức việc tạo tài nguyên và điều này sẽ không tạo ra một mục trong Báo cáo Chính sách, tuy nhiên nếu Pod được tạo ra tuân thủ quy tắc, nó sẽ được báo cáo là `PASS` trong báo cáo. Có thể kiểm tra các hành động bị chặn trong các sự kiện Kubernetes cho namespace nơi hành động được yêu cầu.

Bây giờ, chúng ta sẽ kiểm tra trạng thái tuân thủ của cụm của chúng ta đối với các chính sách chúng ta đã tạo cho đến nay trong buổi thực hành này với một tổng quan về các Báo cáo Chính sách được tạo ra.

```shell
$ kubectl get policyreports -A

NAMESPACE     NAME                             PASS   FAIL   WARN   ERROR   SKIP   AGE
assets        cpol-baseline-policy             3      0      0      0       0      19m
assets        cpol-require-labels              0      3      0      0       0      27m
assets        cpol-restrict-image-registries   3      0      0      0       0      25m
carts         cpol-baseline-policy             6      0      0      0       0      19m
carts         cpol-require-labels              0      6      0      0       0      27m
carts         cpol-restrict-image-registries   3      3      0      0       0      25m
catalog       cpol-baseline-policy             5      0      0      0       0      19m
catalog       cpol-require-labels              0      5      0      0       0      27m
catalog       cpol-restrict-image-registries   5      0      0      0       0      25m
checkout      cpol-baseline-policy             6      0      0      0       0      19m
checkout      cpol-require-labels              0      6      0      0       0      27m
checkout      cpol-restrict-image-registries   6      0      0      0       0      25m
default       cpol-baseline-policy             2      0      0      0       0      19m
default       cpol-require-labels              2      0      0      0       0      13m
default       cpol-restrict-image-registries   1      1      0      0       0      13m
kube-system   cpol-baseline-policy             4      8      0      0       0      19m
kube-system   cpol-require-labels              0      12     0      0       0      27m
kube-system   cpol-restrict-image-registries   0      12     0      0       0      25m
kyverno       cpol-baseline-policy             24     0      0      0

       0      19m
kyverno       cpol-require-labels              0      24     0      0       0      27m
kyverno       cpol-restrict-image-registries   0      24     0      0       0      25m
orders        cpol-baseline-policy             6      0      0      0       0      19m
orders        cpol-require-labels              0      6      0      0       0      27m
orders        cpol-restrict-image-registries   6      0      0      0       0      25m
rabbitmq      cpol-baseline-policy             2      0      0      0       0      19m
rabbitmq      cpol-require-labels              0      2      0      0       0      27m
rabbitmq      cpol-restrict-image-registries   2      0      0      0       0      25m
ui            cpol-baseline-policy             3      0      0      0       0      19m
ui            cpol-require-labels              0      3      0      0       0      27m
ui            cpol-restrict-image-registries   3      0      0      0       0      25m
```

> Đầu ra có thể thay đổi.

Vì chúng ta chỉ làm việc với ClusterPolicies, bạn có thể thấy trong đầu ra trên một số Báo cáo đã được tạo ra trên tất cả các namespaces, như `cpol-verify-image`, `cpol-baseline-policy`, và `cpol-restrict-image-registries` và không chỉ trong namespace `default`, nơi chúng ta đã tạo các tài nguyên để được xác nhận. Bạn cũng có thể nhìn thấy trạng thái của các đối tượng như `PASS`, `FAIL`, `WARN`, `ERROR`, và `SKIP`.

Như đã đề cập trước đó, các hành động bị chặn sẽ tồn tại trong các sự kiện namespace, hãy xem xét những sự kiện đó bằng cách sử dụng lệnh dưới đây.

```shell
$ kubectl get events | grep block
8m         Warning   PolicyViolation   clusterpolicy/restrict-image-registries   Pod default/nginx-public: [validate-registries] fail (blocked); validation error: Unknown Image registry. rule validate-registries failed at path /spec/containers/0/image/
3m         Warning   PolicyViolation   clusterpolicy/restrict-image-registries   Pod default/nginx-public: [validate-registries] fail (blocked); validation error: Unknown Image registry. rule validate-registries failed at path /spec/containers/0/image/
```

> Đầu ra có thể thay đổi.

Bây giờ, hãy xem xét kỹ hơn các Báo cáo Chính sách cho namespace `default` được sử dụng trong các bài thực hành.

```shell
$ kubectl get policyreports
NAME                                           PASS   FAIL   WARN   ERROR   SKIP   AGE
default       cpol-baseline-policy             2      0      0      0       0      19m
default       cpol-require-labels              2      0      0      0       0      13m
default       cpol-restrict-image-registries   1      1      0      0       0      13m
```

Kiểm tra xem cho ClusterPolicy `restrict-image-registries`, chúng ta chỉ có một Báo cáo `FAIL` và một Báo cáo `PASS`. Điều này đã xảy ra vì tất cả các ClusterPolicies được tạo với chế độ `Bắt buộc`, và như đã đề cập, các tài nguyên bị chặn không được báo cáo, cũng như các tài nguyên đã chạy trước đó có thể vi phạm các quy tắc chính sách, đã được loại bỏ.

Pod `nginx`, mà chúng ta để chạy với một hình ảnh có sẵn công khai, là tài nguyên duy nhất còn lại vi phạm chính sách `restrict-image-registries`, và nó được hiển thị trong báo cáo.

Hãy xem chi tiết hơn các vi phạm cho Chính sách này bằng cách mô tả một báo cáo cụ thể. Như được thể hiện trong ví dụ dưới đây, sử dụng lệnh `kubectl describe` cho Báo cáo `cpol-restrict-image-registries` để xem các kết quả xác nhận cho ClusterPolicy `restrict-image-registries`.

```shell
$ kubectl describe policyreport cpol-restrict-image-registries
Name:         cpol-restrict-image-registries
Namespace:    default
Labels:       app.kubernetes.io/managed-by=kyverno
              cpol.kyverno.io/restrict-image-registries=607025
Annotations:  <none>
API Version:  wgpolicyk8s.io/v1alpha2
Kind:         PolicyReport
Metadata:
  Creation Timestamp:  2024-01-18T01:03:40Z
  Generation:          1
  Resource Version:    607320
  UID:                 7abb6c11-9610-4493-ab1e-df94360ce773
Results:
  Message:  validation error: Unknown Image registry. rule validate-registries failed at path /spec/containers/0/image/
  Policy:   restrict-image-registries
  Resources:
    API Version:  v1
    Kind:         Pod
    Name:         nginx
    Namespace:    default
    UID:          dd5e65a9-66b5-4192-89aa-a291d150807d
  Result:         fail
  Rule:           validate-registries
  Scored:         true
  Source:         kyverno
  Timestamp:
    Nanos:    0
    Seconds:  1705539793
  Message:    validation rule 'validate-registries' passed.
  Policy:     restrict-image-registries
  Resources:
    API

 Version:  v1
    Kind:         Pod
    Name:         nginx-ecr
    Namespace:    default
    UID:          e638aad7-7fff-4908-bbe8-581c371da6e3
  Result:         pass
  Rule:           validate-registries
  Scored:         true
  Source:         kyverno
  Timestamp:
    Nanos:    0
    Seconds:  1705539793
Summary:
  Error:  0
  Fail:   1
  Pass:   1
  Skip:   0
  Warn:   0
Events:   <none>
```

Đầu ra trên hiển thị xác nhận chính sách của Pod `nginx` nhận kết quả `fail` và thông báo lỗi xác nhận. Ngược lại, xác nhận chính sách của `nginx-ecr` nhận kết quả `pass`. Theo dõi các báo cáo theo cách này có thể là một gánh nặng đối với các quản trị viên. Kyverno cũng hỗ trợ một công cụ dựa trên giao diện đồ họa cho [Báo cáo Chính sách](https://kyverno.github.io/policy-reporter/core/targets/#policy-reporter-ui). Điều này nằm ngoài phạm vi của bài thực hành này.

Trong bài lab này, bạn đã học cách bổ sung cấu hình PSA/PSS Kubernetes của mình bằng Kyverno. Các Tiêu chuẩn Bảo mật Pod (PSS) và cài đặt Kubernetes trong cây, thực hiện các tiêu chuẩn Bảo mật Pod (PSP), cung cấp các khối xây dựng tốt cho việc quản lý bảo mật pod. Đa số người dùng chuyển từ Chính sách Bảo mật Pod Kubernetes (PSP) nên thành công khi sử dụng các tính năng PSA/PSS.

Kyverno bổ sung trải nghiệm người dùng được tạo ra bởi PSA/PSS, bằng cách tận dụng cài đặt bảo mật pod Kubernetes trong cây và cung cấp một số cải tiến hữu ích cho việc vận hành chính sách. Bạn có thể sử dụng Kyverno để quản lý việc sử dụng nhãn bảo mật pod một cách đúng đắn. Ngoài ra, bạn cũng có thể sử dụng quy tắc `validate.podSecurity` mới của Kyverno để dễ dàng quản lý các tiêu chuẩn bảo mật pod với tính linh hoạt và trải nghiệm người dùng tốt hơn. Và, với Kyverno CLI, bạn có thể tự động hóa việc đánh giá chính sách, phía trước của các cụm của bạn.