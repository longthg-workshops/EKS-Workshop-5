---
title: "Hạn chế Image Registry"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 8.3 </b>"
---


Sử dụng hình ảnh container từ các nguồn không rõ trên các Cụm EKS của bạn, có thể không được quét để phát hiện Các Lỗ Hổng và Công Bố (CVE), đại diện cho một yếu tố nguy cơ đối với tổng thể an ninh của môi trường của bạn. Khi lựa chọn nguồn hình ảnh container, bạn cần đảm bảo rằng chúng có nguồn gốc từ Các Đăng Ký Đáng Tin Cậy, nhằm giảm thiểu nguy cơ tiếp xúc và khai thác các lỗ hổng. Một số tổ chức lớn cũng có Hướng Dẫn Về An Ninh giới hạn việc sử dụng hình ảnh container từ hệ thống Image Registry riêng được lưu trữ của họ.

Trong phần này, bạn sẽ thấy làm thế nào Kyverno có thể giúp bạn chạy các tải công việc container an toàn bằng cách hạn chế các Image Registry có thể được sử dụng trong cụm của bạn.

Như đã thấy trong các phòng thí nghiệm trước đó, bạn có thể chạy Pods với các hình ảnh từ bất kỳ đăng ký nào có sẵn, vì vậy hãy chạy một Pod mẫu bằng cách sử dụng đăng ký mặc định trỏ đến `docker.io`.

```bash
$ kubectl run nginx --image=nginx

NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          47s

$ kubectl describe pod nginx | grep Image
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:4c0fdaa8b6341bfdeca5f18f7837462c80cff90527ee35ef185571e1c327beac
```

Trong trường hợp này, chỉ là một hình ảnh cơ sở `nginx` được kéo từ registry công khai. Một kẻ tấn công có thể kéo bất kỳ hình ảnh có lỗ hổng nào và chạy trên Cụm EKS, khai thác tài nguyên được cấp trong cụm.

Tiếp theo, như một thực hành tốt nhất, bạn sẽ xác định một chính sách sẽ hạn chế việc sử dụng bất kỳ Image Registry không được ủy quyền nào và chỉ phụ thuộc vào Các Đăng Ký Đáng Tin Cậy được chỉ định.

Trong phòng thí nghiệm này, bạn sẽ sử dụng [Amazon ECR Public Gallery](https://public.ecr.aws/) làm Đăng Ký Đáng Tin Cậy, chặn bất kỳ container nào sử dụng Hình Ảnh được lưu trữ trong các đăng ký khác để chạy. Dưới đây là một chính sách Kyverno mẫu để hạn chế việc kéo hình ảnh cho trường hợp sử dụng này.

```file
manifests/modules/security/kyverno/images/restrict-registries.yaml
```

> Phần trên không hạn chế việc sử dụng InitContainers hoặc Ephemeral Containers đến kho lưu trữ được tham chiếu.

Áp dụng chính sách trên bằng lệnh dưới đây.

```bash
$ kubectl apply -f ~/environment/eks-workshop/modules/security/kyverno/images/restrict-registries.yaml

clusterpolicy.kyverno.io/restrict-image-registries created
```

Hãy thử chạy một Pod mẫu khác bằng cách sử dụng hình ảnh mặc định từ Đăng Ký Công Cộng.

```bash expectError=true
$ kubectl run nginx-public --image=nginx

Error from server: admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Pod/default/nginx-public was blocked due to the following policies 

restrict-image-registries:
  validate-registries: 'validation error: Unknown Image registry. rule validate-registries
    failed at path /spec/containers/0/image/'
```

Pod không thể chạy và hiển thị một đầu ra cho biết việc Tạo Pod đã bị chặn do Chính Sách Kyverno đã được tạo trước đó của chúng tôi.

Bây giờ hãy thử chạy một Pod mẫu bằng cách sử dụng Hình Ảnh `nginx` được lưu trữ trong Đăng Ký Đáng Tin Cậy, đã được xác định trước trong Chính Sách (public.ecr.aws).

```bash
$ kubectl run nginx-ecr --image=public.ecr.aws/nginx/nginx
pod/nginx-public created
```

Pod đã được tạo thành công!

Bạn đã thấy cách bạn có thể chặn Hình Ảnh từ các đăng ký công cộng để chạy trên Cụm EKS của bạn và chỉ hạn chế Các Kho Lưu Trữ Hình Ảnh được phép. Một người có thể tiến xa hơn, và chỉ cho phép các kho lưu trữ riêng tư làm Thực Hành Bảo Mật Tốt Nhất.

> Đừng gỡ bỏ các Pods đang chạy được tạo trong nhiệm vụ này vì chúng ta sẽ sử dụng chúng cho phòng thí nghiệm tiếp theo.