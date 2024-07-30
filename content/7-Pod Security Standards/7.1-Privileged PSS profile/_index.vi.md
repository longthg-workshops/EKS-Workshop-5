---
title: "Cấu hình PSS được cấp quyền"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 7.1 </b>"
---

#### Quan sát hồ sơ PSS

Chúng ta sẽ bắt đầu quan sát hồ sơ Privileged (PSS) bằng cách quan sát hồ sơ Privileged, là hồ sơ linh hoạt nhất và cho phép việc nâng cao đặc quyền đã biết.

Kể từ phiên bản Kubernetes 1.23, theo mặc định, tất cả các chế độ PSA (tức là enforce, audit và warn) đều được kích hoạt cho hồ sơ PSS Privileged ở cấp độ cluster. Điều đó có nghĩa, theo mặc định, PSA cho phép triển khai (Deployments) hoặc các Pod với hồ sơ PSS Privileged (tức là không có bất kỳ hạn chế nào) trên tất cả các không gian tên (namespace). Các thiết lập mặc định này cung cấp ít ảnh hưởng đến các cluster và giảm thiểu tác động tiêu cực đối với các ứng dụng. Như chúng ta sẽ thấy, nhãn Namespace có thể được sử dụng để lựa chọn vào các thiết lập hạn chế hơn.

Bạn có thể kiểm tra xem không có nhãn PSA nào được thêm vào không gian tên `assets`, theo mặc định:

```bash
$ kubectl describe ns assets 
Name:         assets
Labels:       app.kubernetes.io/created-by=eks-workshop
              kubernetes.io/metadata.name=assets
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

Như bạn thấy, không gian tên `assets` không có bất kỳ nhãn PSA nào được đính kèm.

Hãy cũng kiểm tra xem Triển khai (Deployment) và Pod đang chạy hiện tại trong không gian tên `assets`.

```bash
$ kubectl -n assets get deployment
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
assets   1/1     1            1           5m24s
$ kubectl -n assets get pod
NAME                     READY   STATUS    RESTARTS   AGE
assets-ddb8f87dc-8z6l9   1/1     Running   0          5m24s
```

YAML cho Pod `assets` sẽ cho chúng ta thấy cấu hình bảo mật hiện tại:

```bash
$ kubectl -n assets get deployment assets -o yaml | yq '.spec.template.spec'
containers:
  - envFrom:
      - configMapRef:
          name: assets
    image: public.ecr.aws/aws-containers/retail-store-sample-assets:0.4.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 3
      httpGet:
        path: /health.html
        port: 8080
        scheme: HTTP
      periodSeconds: 3
      successThreshold: 1
      timeoutSeconds: 1
    name: assets
    ports:
      - containerPort: 8080
        name: http
        protocol: TCP
    resources:
      limits:
        memory: 128Mi
      requests:
        cpu: 128m
        memory: 128Mi
    securityContext:
      capabilities:
        drop:
          - ALL
      readOnlyRootFilesystem: false
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
      - mountPath: /tmp
        name: tmp-volume
dnsPolicy: ClusterFirst
restartPolicy: Always
schedulerName: default-scheduler
securityContext: {}
serviceAccount: assets
serviceAccountName: assets
terminationGracePeriodSeconds: 30
volumes:
  - emptyDir:
      medium: Memory
    name: tmp-volume
```

Trong cấu hình bảo mật Pod trên, `securityContext` là nil ở cấp độ Pod. Ở cấp độ container, `securityContext` được cấu hình để loại bỏ tất cả các khả năng Linux và `readOnlyRootFilesystem` được đặt thành false. Việc triển khai và Pod đã chạy cho thấy rằng PSA (được cấu hình cho hồ sơ PSS Privileged theo mặc định) đã cho phép cấu hình bảo mật Pod trên.

Nhưng các điều kiện bảo mật khác mà PSA này cho phép là gì? Để kiểm tra điều đó, hãy thêm một số quyền hạn khác vào cấu hình bảo mật Pod trên và kiểm tra xem PSA có vẫn cho phép nó hay không trong không gian tên `assets`. Cụ thể, hãy thêm các cờ `privileged` và `runAsUser:0` vào Pod trên, điều này có nghĩa là nó có thể truy cập vào tài nguyên máy chủ mà thường được yêu cầu cho các công việc như các agent giám sát và các sidecar của mạng dịch vụ, và cũng được phép chạy dưới dạng người dùng `root`:

```kustomization
modules/security/pss-psa/privileged-workload/deployment.yaml
Deployment/assets
```

Chạy Kustomize để áp dụng các thay đổi trên và kiểm tra xem PSA có cho phép Pod với các quyền hạn bảo mật trên hay không.

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/security/pss-psa/privileged-workload
namespace/assets unchanged
serviceaccount/assets unchanged
configmap/assets unchanged
service/assets unchanged
deployment.apps/assets configured
```

Hãy kiểm tra xem Triển khai và Pod có được tạo lại với các quyền hạn bảo mật trên trong không gian tên `assets` không

```bash
$ kubectl -n assets get pod
NAME                      READY   STATUS    RESTARTS   AGE
assets-64c49f848b-gmrt

t   1/1     Running   0          9s

$ kubectl -n assets exec $(kubectl -n assets get pods -o name) -- whoami
root
```

Điều này cho thấy rằng chế độ PSA mặc định được kích hoạt cho hồ sơ PSS Privileged là phù hợp và cho phép các Pod yêu cầu quyền hạn bảo mật nâng cao nếu cần.

Lưu ý rằng các quyền hạn bảo mật trên không phải là danh sách toàn diện của các điều khiển cho phép trong hồ sơ PSS Privileged. Để biết chi tiết về các điều khiển bảo mật được phép/từ chối dưới mỗi hồ sơ PSS, hãy tham khảo [tài liệu](https://kubernetes.io/docs/concepts/security/pod-security-standards/).