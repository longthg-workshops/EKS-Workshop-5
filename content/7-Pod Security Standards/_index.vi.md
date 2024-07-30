---
title: "Tiêu chuẩn an toàn cho pod"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b> 7. </b>"
---

#### Chuẩn bị môi trường của bạn cho phần này:

```bash timeout=300 wait=30
$ prepare-environment security/pss-psa
```

:::

Việc áp dụng Kubernetes một cách an toàn bao gồm việc ngăn chặn các thay đổi không mong muốn vào các cụm. Các thay đổi không mong muốn có thể làm gián đoạn hoạt động của cụm, hành vi công việc và thậm chí làm nguy hiểm tính toàn vẹn của môi trường. Giới thiệu Pods thiếu cấu hình bảo mật chính xác là một ví dụ về một thay đổi không mong muốn vào cụm. Để kiểm soát bảo mật Pods, Kubernetes cung cấp [Pod Security Policy / PSP](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) tài nguyên. PSPs chỉ định một bộ cài đặt bảo mật mà Pods phải đáp ứng trước khi chúng có thể được tạo hoặc cập nhật trong một cụm. Tuy nhiên, từ phiên bản Kubernetes 1.21, PSPs đã bị loại bỏ và được lên lịch loại bỏ hoàn toàn trong phiên bản Kubernetes 1.25.

Trong Kubernetes, PSPs đang được thay thế bằng [Pod Security Admission / PSA](https://kubernetes.io/docs/concepts/security/pod-security-admission/), một bộ kiểm soát thẩm định tích hợp triển khai các điều khiển bảo mật được mô tả trong [Pod Security Standards / PSS](https://kubernetes.io/docs/concepts/security/pod-security-standards/). Kể từ phiên bản Kubernetes 1.23, PSA và PSS đều đã đạt được trạng thái tính năng beta và được kích hoạt mặc định trong Dịch vụ Kubernetes Đàn Elastic của Amazon (EKS).

#### Tiêu Chuẩn Bảo Mật Pod (PSS) và Kiểm Soát Thẩm Định Bảo Mật Pod (PSA)

Theo tài liệu Kubernetes, PSS "xác định ba chính sách khác nhau để phủ rộng phổ bảo mật. Các chính sách này tích lũy và biến đổi từ rất cho phép đến hạn chế rất cao."

Các cấp độ chính sách được xác định như sau:

* **Đặc Quyền:** Chính sách không giới hạn (không an toàn), cung cấp mức độ quyền rộng nhất có thể. Chính sách này cho phép các sự nâng cấp đặc quyền đã biết. Đây là sự vắng mặt của một chính sách. Điều này phù hợp cho các ứng dụng như các đại lý nhật ký, CNIs, các trình điều khiển lưu trữ và các ứng dụng phổ biến khác cần truy cập đặc quyền.
* **Cơ Bản:** Chính sách hạn chế tối thiểu ngăn chặn các sự nâng cấp đặc quyền đã biết. Cho phép cấu hình Pods mặc định (được chỉ định tối thiểu). Chính sách cơ bản ngăn chặn việc sử dụng hostNetwork, hostPID, hostIPC, hostPath, hostPort, khả năng thêm quyền Linux, cùng với một số hạn chế khác.
* **Hạn Chế:** Chính sách hạn chế nặng, tuân thủ các thực hành tốt nhất hiện tại về cứng cụm Pod. Chính sách này kế thừa từ cấu hình cơ bản và thêm các hạn chế khác như không thể chạy dưới dạng root hoặc root-group. Các chính sách hạn chế có thể ảnh hưởng đến khả năng hoạt động của một ứng dụng. Chúng chủ yếu được dành cho các ứng dụng quan trọng về bảo mật.

Bộ kiểm soát thẩm định PSA triển khai các điều khiển, được mô tả bởi các chính sách PSS, qua ba chế độ hoạt động được liệt kê dưới đây.

* **thực thi:** Vi phạm chính sách sẽ khiến Pods bị từ chối.
* **kiểm tra:** Vi phạm chính sách sẽ gây ra việc thêm chú thích kiểm tra vào sự kiện được ghi lại trong [bản ghi kiểm tra](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/), nhưng nó vẫn được phép.
* **cảnh báo:** Vi phạm chính sách sẽ gây ra một cảnh báo phù hợp với người dùng, nhưng vẫn được phép.

### Kiểm Soát Thẩm Định Bảo Mật Pods (PSA) tích hợp sẵn

Từ phiên bản Kubernetes 1.23, Cổng tính năng PodSecurity [gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) được kích hoạt mặc định trong Dịch vụ Kubernetes Đàn Elastic của Amazon. Các [cài đặt mặc định của PSS và PSA](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-admission-controller/#configure-the-admission-controller) cho phiên bản Kubernetes 1.23 từ nguồn

 gốc cũng được sử dụng cho Dịch vụ Kubernetes Đàn Elastic của Amazon, như được liệt kê dưới đây.

> *Cổng tính năng PodSecurity đang ở phiên bản Beta (apiVersion: v1beta1) trên Kubernetes v1.23 và v1.24, và trở thành Có Sẵn Một Cách Chính Thức (GA,  apiVersion: v1) trong Kubernetes v1.25.*

```yaml
    defaults:
      enforce: "privileged"
      enforce-version: "latest"
      audit: "privileged"
      audit-version: "latest"
      warn: "privileged"
      warn-version: "latest"
    exemptions:
      # Mảng các tên người dùng xác thực được miễn.
      usernames: []
      # Mảng các tên lớp thời gian chạy được miễn.
      runtimeClasses: []
      # Mảng các không gian tên được miễn.
      namespaces: []
```

Các cài đặt trên cấu hình kịch bản toàn cụm như sau:

* Không có các miễn trừ PSA được cấu hình khi khởi động máy chủ API Kubernetes.
* Hồ sơ PSS Đặc quyền được cấu hình theo mặc định cho tất cả các chế độ PSA, và được đặt thành các phiên bản mới nhất.

#### Nhãn Kiểm Soát Thẩm Định Bảo Mật Pods (PSA) cho Namespaces

Dựa trên cấu hình mặc định trên, bạn phải cấu hình các hồ sơ PSS cụ thể và các chế độ PSA tại cấp độ Kubernetes Namespace, để lựa chọn Namespaces vào Pod security được cung cấp bởi PSA và PSS. Bạn có thể cấu hình Namespaces để xác định chế độ kiểm soát thẩm định bạn muốn sử dụng cho bảo mật Pods. Với [nhãn Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels), bạn có thể chọn một trong các cấp độ PSS được xác định trước mà bạn muốn sử dụng cho Pods trong một Namespace cụ thể. Nhãn bạn chọn xác định hành động mà PSA thực hiện nếu phát hiện ra vi phạm tiềm ẩn. Như dưới đây, bạn có thể cấu hình bất kỳ hoặc tất cả các chế độ, hoặc thậm chí đặt một cấp độ khác nhau cho các chế độ khác nhau. Đối với mỗi chế độ, có hai nhãn có thể xác định chính sách được sử dụng.

```
# Nhãn cấp độ theo từng chế độ chỉ ra cấp độ chính sách được áp dụng cho chế độ đó.
#
# CHẾ ĐỘ phải là một trong các giá trị `enforce`, `audit`, hoặc `warn`.
# CẤP ĐỘ phải là một trong các giá trị `privileged`, `baseline`, hoặc `restricted`.
*pod-security.kubernetes.io/<CHẾ ĐỘ>*: <CẤP ĐỘ>

# Tùy chọn: nhãn phiên bản theo chế độ có thể được sử dụng để ghim chính sách vào
# phiên bản đã được gửi kèm với một phiên bản nhỏ Kubernetes cụ thể (ví dụ: v1.24).
#
# CHẾ ĐỘ phải là một trong các giá trị `enforce`, `audit`, hoặc `warn`.
# PHIÊN BẢN phải là một phiên bản Kubernetes hợp lệ, hoặc `latest`.
*pod-security.kubernetes.io/<CHẾ ĐỘ>-version*: <PHIÊN BẢN>
```

Dưới đây là một ví dụ về cấu hình Namespaces PSA và PSS có thể được sử dụng cho kiểm tra. Lưu ý rằng chúng tôi không bao gồm nhãn tùy chọn chế độ phiên bản PSA. Chúng tôi đã sử dụng cài đặt toàn cụm, mới nhất, được cấu hình mặc định. Bằng cách gỡ bỏ các nhãn mong muốn, bên dưới, bạn có thể kích hoạt các chế độ PSA và các hồ sơ PSS mà bạn cần cho Namespaces tương ứng của bạn.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: psa-pss-test-ns
  labels:
    # pod-security.kubernetes.io/enforce: privileged
    # pod-security.kubernetes.io/audit: privileged
    # pod-security.kubernetes.io/warn: privileged

    # pod-security.kubernetes.io/enforce: baseline
    # pod-security.kubernetes.io/audit: baseline
    # pod-security.kubernetes.io/warn: baseline

    # pod-security.kubernetes.io/enforce: restricted
    # pod-security.kubernetes.io/audit: restricted
    # pod-security.kubernetes.io/warn: restricted

```

#### Xác Thực Kiểm Soát Thẩm Định

Trong Kubernetes, một Kiểm Soát Thẩm Định là một phần mã nguồn mở được viết để ngăn chặn các yêu cầu đến máy chủ API Kubernetes trước khi chúng được lưu trữ vào etcd, và được sử dụng để thực hiện các thay đổi trên cụm. Kiểm Soát Thẩm Định có thể là loại biến đổi, xác thực, hoặc cả hai. Việc triển khai PSA là một bộ kiểm soát thẩm định xác thực, và nó kiểm tra các yêu cầu mô tả Pods

 đến để đảm bảo tuân thủ với các PSS cụ thể đã được chỉ định.

Trong luồng dưới đây, [kiểm soát thẩm định động biến đổi và xác thực](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/), được gọi là webhooks kiểm soát thẩm định, được tích hợp vào luồng yêu cầu máy chủ API Kubernetes, thông qua webhooks. Các webhooks gọi ra các dịch vụ, được cấu hình để phản hồi với một số loại yêu cầu máy chủ API nhất định. Ví dụ, bạn có thể sử dụng webhooks để cấu hình các kiểm soát thẩm định động để xác thực rằng các container trong một Pod đang chạy dưới dạng người dùng không phải root, hoặc các container được cung cấp từ các kho lưu trữ đáng tin cậy.

![](/images/p7/7.0-1-k8s-admission-controllers.png)

#### Sử Dụng PSA và PSS

PSA thực thi các chính sách được mô tả trong PSS, và các chính sách PSS xác định một bộ hồ sơ bảo mật Pods. Trong biểu đồ dưới đây, chúng tôi mô tả cách PSA và PSS hoạt động cùng nhau, với Pods và Namespaces, để xác định các hồ sơ bảo mật Pods và áp dụng kiểm soát thẩm định dựa trên các hồ sơ đó. Như thấy trong biểu đồ dưới đây, các chế độ thực thi PSA và các chính sách PSS được xác định dưới dạng nhãn trong Namespaces đích.

![](/images/p7/7.0-2-using-pss-psa.png)
