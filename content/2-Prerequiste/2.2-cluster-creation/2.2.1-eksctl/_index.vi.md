---
title: "Sử dụng eksctl"
weight: 1
chapter: false
pre: "<b> 2.2.1 </b>"
---

#### Xây dựng một cluster cho các bài thực hành lab sử dụng công cụ [eksctl](https://eksctl.io/). 

_**Đây là cách dễ nhất để bắt đầu, và được khuyến nghị cho hầu hết các người học.**_

Tiện ích **eksctl** đã được cài đặt sẵn trong Môi trường của bạn, vì vậy chúng ta có thể ngay lập tức tạo ra cluster. Đây là cấu hình sẽ được sử dụng để xây dựng cluster:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
availabilityZones:
  - ${AWS_REGION}a
  - ${AWS_REGION}b
  - ${AWS_REGION}c
metadata:
  name: ${EKS_CLUSTER_NAME}
  region: ${AWS_REGION}
  version: "1.30"
  tags:
    karpenter.sh/discovery: ${EKS_CLUSTER_NAME}
    created-by: eks-workshop-v2
    env: ${EKS_CLUSTER_NAME}
iam:
  withOIDC: true
vpc:
  cidr: 10.42.0.0/16
  clusterEndpoints:
    privateAccess: true
    publicAccess: true
addons:
  - name: vpc-cni
    version: 1.16.0
    configurationValues: '{"env":{"ENABLE_PREFIX_DELEGATION":"true", "ENABLE_POD_ENI":"true", "POD_SECURITY_GROUP_ENFORCING_MODE":"standard"},"enableNetworkPolicy": "true", "nodeAgent": {"enablePolicyEventLogs": "true"}}'
    resolveConflicts: overwrite
managedNodeGroups:
  - name: default
    desiredCapacity: 3
    minSize: 3
    maxSize: 6
    instanceType: m5.large
    privateNetworking: true
    releaseVersion: "1.30.0-20240625"
    updateConfig:
      maxUnavailablePercentage: 50
    labels:
      workshop-default: "yes"
```

Dựa trên cấu hình này, **eksctl** sẽ:
- Tạo một VPC trên ba AZ
- Tạo một cluster EKS
- Tạo một nhà cung cấp IAM OIDC
- Thêm một nhóm node quản lý có tên là `default`
- Cấu hình VPC CNI để sử dụng việc ủy quyền tiền tố

Áp dụng tệp cấu hình như sau:

```bash
$ export EKS_CLUSTER_NAME=eks-workshop
$ curl -fsSL https://raw.githubusercontent.com/aws-samples/eks-workshop-v2/stable/cluster/eksctl/cluster.yaml | \
envsubst | eksctl create cluster -f -
```

Thường mất khoảng 20 phút. Khi cluster được tạo, chạy lệnh sau để sử dụng cluster cho các bài thực hành lab:

```bash
$ use-cluster $EKS_CLUSTER_NAME
```

#### Bước Tiếp Theo

Bây giờ cluster đã sẵn sàng, hãy đi đến [Bắt đầu](/docs/introduction/getting-started) module hoặc nhảy tới bất kỳ module nào trong workshop với thanh điều hướng trên cùng. Khi bạn hoàn thành workshop, làm theo các bước dưới đây để dọn dẹp môi trường của bạn.

#### Dọn Dẹp (các bước sau khi bạn hoàn thành Workshop)

{{% notice info %}}
Dưới đây là cách bạn sẽ dọn dẹp tài nguyên sau này khi bạn đã sử dụng xong Cluster EKS bạn đã tạo trong các bước trước để hoàn thành các module.  
{{% /notice %}}

Trước khi xóa môi trường VSCode, chúng ta cần dọn dẹp cluster mà chúng ta đã thiết lập trong các bước trước đó.

Đầu tiên sử dụng `delete-environment` để đảm bảo rằng ứng dụng mẫu và bất kỳ cơ sở hạ tầng lab còn lại nào được loại bỏ:

```bash
$ delete-environment
```

Tiếp theo, xóa cluster bằng **eksctl**:

```bash
$ eksctl delete cluster $EKS_CLUSTER_NAME --wait
```
