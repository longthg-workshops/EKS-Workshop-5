---
title: "Sealed Secret cho Kubernetes"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 5.3.1 </b>"
---

#### Sealed Secret cho Kubernetes

Sealed Secrets bao gồm hai phần:

- Một controller phía cụm
- Một CLI phía khách gọi là `kubeseal`

Khi controller khởi động, nó tìm kiếm một cặp khóa riêng tư/công khai trên toàn cụm và tạo một cặp khóa RSA 4096 bit mới nếu không tìm thấy. Khóa riêng được lưu trữ trong một đối tượng Secret trong cùng không gian tên với controller (mặc định là kube-system). Phần khóa công cộng của nó được công khai cho bất kỳ ai muốn sử dụng SealedSecrets với cụm này.

Trong quá trình mã hóa, mỗi giá trị trong Secret gốc được mã hóa theo đối xứng bằng AES-256 bằng một khóa phiên được tạo ngẫu nhiên. Khóa phiên sau đó được mã hóa không đối xứng với khóa công cộng của controller bằng SHA256 và tên không gian/tên gốc của Secret làm tham số đầu vào. Đầu ra của quá trình mã hóa là một chuỗi được xây dựng như sau:
độ dài (2 byte) của khóa phiên đã được mã hóa + khóa phiên đã được mã hóa + Secret đã được mã hóa

Khi một tài nguyên tùy chỉnh SealedSecret được triển khai lên cụm Kubernetes, controller sẽ nhận nó, mở khóa nó bằng khóa riêng và tạo một tài nguyên Secret. Trong quá trình giải mã, không gian tên/tên của SealedSecret được sử dụng lại làm tham số đầu vào. Điều này đảm bảo rằng SealedSecret và Secret được chặt chẽ liên kết với cùng một không gian tên và tên.

Công cụ CLI đồng hành, kubeseal, được sử dụng để tạo một định nghĩa tài nguyên tùy chỉnh SealedSecret (CRD) từ một định nghĩa tài nguyên Secret bằng cách sử dụng khóa công cộng. kubeseal có thể giao tiếp với controller thông qua máy chủ API Kubernetes và lấy khóa công cộng cần thiết để mã hóa một Secret trong thời gian chạy. Khóa công cộng cũng có thể được tải xuống từ controller và lưu trữ cục bộ để sử dụng offline.

Các SealedSecrets có thể có ba phạm vi sau:

- **strict (mặc định)**: Secret phải được niêm phong với chính xác cùng tên và không gian tên. Các thuộc tính này trở thành một phần của dữ liệu đã mã hóa và do đó, việc thay đổi tên và/hoặc không gian tên sẽ dẫn đến "lỗi giải mã".
- **namespace-wide**: Secret đã niêm phong có thể được đổi tên tự do trong một không gian tên cụ thể.
- **cluster-wide**: Secret có thể được mở niêm phong trong bất kỳ không gian tên nào và có thể được đặt bất kỳ tên nào.