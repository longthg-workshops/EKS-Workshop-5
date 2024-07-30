---
title: "Áp dụng tiêu chuẩn an toàn"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 8.2 </b>"
---
Như đã thảo luận trong phần [Tiêu chuẩn Bảo mật Pod (PSS)](../pod-security-standards/) giới thiệu, có 3 cấp độ Chính sách được xác định trước, **Privileged** (Đặc quyền), **Baseline** (Cơ sở), và **Restricted** (Hạn chế). Trong khi khuyến nghị cài đặt một PSS Hạn chế, điều này có thể gây ra hành vi không mong muốn ở cấp độ ứng dụng trừ khi được thiết lập đúng cách. Để bắt đầu, khuyến nghị cài đặt một Chính sách Baseline sẽ ngăn chặn các việc thăng cấp Đặc quyền đã biết như Containers truy cập HostProcess, HostPath, HostPorts hoặc cho phép theo dõi lưu lượng chẳng hạn, có thể thiết lập các chính sách cá nhân để hạn chế hoặc không cho phép truy cập đặc quyền đó vào các container.

Một Chính sách Baseline Kyverno sẽ giúp hạn chế tất cả các việc thăng cấp đặc quyền đã biết dưới một chính sách duy nhất, và cũng duy trì và cập nhật Chính sách thường xuyên bổ sung các lỗ hổng mới nhất được tìm thấy vào Chính sách.

Các container Privileged có thể làm gần như mọi thứ mà máy chủ có thể làm và thường được sử dụng trong các ống dẫn CI/CD để cho phép xây dựng và xuất bản hình ảnh Container.
Với lỗi [CVE-2022-23648](https://github.com/containerd/containerd/security/advisories/GHSA-crp2-qrr5-8pq7) hiện đã được sửa, bất kỳ nhân vật xấu nào cũng có thể thoát khỏi container có đặc quyền bằng cách lạm dụng chức năng `release_agent` của Nhóm Kiểm soát để thực thi các lệnh tùy ý trên máy chủ container.

Trong thí nghiệm này, chúng ta sẽ chạy một Pod có đặc quyền trên cụm EKS của chúng ta. Để làm điều đó, thực thi lệnh sau:

```bash
$ kubectl run privileged-pod --image=nginx --restart=Never --privileged
pod/privileged-pod created
$ kubectl delete pod privileged-pod
pod "privileged-pod" deleted
```

Để tránh những khả năng có đặc quyền được thăng cấp như vậy và tránh việc sử dụng không được ủy quyền của các quyền trên, khuyến nghị cài đặt một Chính sách Baseline bằng cách sử dụng Kyverno.

Hồ sơ cơ sở của Tiêu chuẩn Bảo mật Pod là một bộ sưu tập các bước cơ bản và quan trọng nhất có thể được thực hiện để bảo mật Pods. Bắt đầu từ Kyverno 1.8, một hồ sơ hoàn chỉnh có thể được gán cho cụm thông qua một quy tắc duy nhất. Để kiểm tra thêm về các đặc quyền bị chặn bởi Hồ sơ Cơ sở, vui lòng tham khảo [tại đây](https://kyverno.io/policies/#:~:text=Baseline%20Pod%20Security%20Standards,cluster%20through%20a%20single%20rule)

```file
manifests/modules/security/kyverno/baseline-policy/baseline-policy.yaml
```

Lưu ý rằng chính sách trên đang ở chế độ `Enforce`, và sẽ chặn bất kỳ yêu cầu nào để tạo Pod có đặc quyền.

Tiến hành và áp dụng Chính sách Baseline.

```bash
$ kubectl apply -f ~/environment/eks-workshop/modules/security/kyverno/baseline-policy/baseline-policy.yaml
clusterpolicy.kyverno.io/baseline-policy created
```

Bây giờ, hãy thử chạy lại Pod có đặc quyền.

```bash expectError=true
$ kubectl run privileged-pod --image=nginx --restart=Never --privileged
Error from server: admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Pod/default/privileged-pod was blocked due to the following policies 

baseline-policy:
  baseline: |
    Validation rule 'baseline' failed. It violates PodSecurity "baseline:latest": ({Allowed:false ForbiddenReason:privileged ForbiddenDetail:container "privileged-pod" must not set securityContext.privileged=true})
```

Như đã thấy, việc tạo không thành công, vì nó không tuân thủ với Chính sách Baseline của chúng tôi được thiết lập trên Cụm.

### Lưu ý về Chính sách Tự tạo

PSA hoạt động ở cấp độ Pod, nhưng trong thực tế Pods thường được quản lý bởi các bộ điều khiển Pod, như Deployments. Không có chỉ dẫn về các lỗi bảo mật Pod ở cấp độ điều khiển Pod có thể làm cho các vấn đề trở nên phức tạp trong việc sửa lỗi. Chế độ áp dụng của PSA là chế độ duy nhất của PSA ngăn các Pod từ việc được tạo, tuy nhiên việc áp dụng PSA không hoạt động ở cấp độ điều khiển Pod. Để cải thiện trải nghiệm này, khuyến nghị sử dụng các chế độ cảnh báo và kiểm tra của PSA cùng với chế độ áp dụng. Với điều đó, PSA sẽ chỉ ra rằng các tài nguyên điều khiển đang cố gắng tạo các Pod sẽ thất bại với cấp độ PSS được áp dụng.

Sử dụng các giải pháp PaC với Kubernetes đặt ra một thách thức khác là viết và duy trì các chính sách để bao gồm tất cả các tài nguyên khác nhau được sử dụng trong các cụm. Với tính năng [Quy tắc Tự tạo của Kyverno cho Các điều khiển Pod](https://kyverno.io/docs/writing-policies/autogen/), các chính sách Pod sẽ tự động tạo ra các chính sách điều khiển Pod liên quan (Deployment, DaemonSet, v.v.). Tính năng của Kyverno này tăng khả năng thể hiện của các chính sách và giảm bớt công sức để duy trì các chính sách cho các tài nguyên liên quan. cải thiện trải nghiệm người dùng PSA trong đó các tài nguyên điều khiển không bị ngăn cản khỏi tiến triển, trong khi các Pod cơ bản đều là.