---
title: "Sử dụng Terraform"
weight: 2
chapter: false
pre: "<b> 2.2.2 </b>"
---

#### Xây dựng một cụm cho các bài thực hành Lab sử dụng [Hashicorp Terraform](https://developer.hashicorp.com/terraform).

Đây nhằm mục đích phục vụ cho những người học đã quen với việc làm việc với hạ tầng mã nguồn mở của Terraform.

CLI **terraform** đã được cài đặt sẵn trong Môi trường Amazon Cloud9 của bạn, vì vậy chúng ta có thể ngay lập tức tạo ra cụm. Hãy xem qua các tập tin cấu hình Terraform chính sẽ được sử dụng để xây dựng cụm và hạ tầng hỗ trợ của nó.

#### Hiểu các tập tin cấu hình Terraform

Tập tin `providers.tf` cấu hình các nhà cung cấp Terraform mà sẽ cần để xây dựng hạ tầng. Trong trường hợp này, của chúng ta sử dụng các nhà cung cấp **aws**, **kubernetes** và **helm**:

```
provider "aws" {
  default_tags {
    tags = local.tags
  }
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 4.67.0"
    }
  }

  required_version = ">= 1.4.2"
}
```

Tập tin `main.tf` thiết lập một số nguồn dữ liệu Terraform để chúng ta có thể lấy thông tin tài khoản AWS và region hiện đang được sử dụng, cũng như một số thẻ mặc định:

```
locals {
  tags = {
    created-by = "eks-workshop-v2"
    env        = var.cluster_name
  }
}
```

Cấu hình `vpc.tf` sẽ đảm bảo hạ tầng VPC của chúng ta được tạo ra:

```
locals {
  private_subnets = [for k, v in local.azs : cidrsubnet(var.vpc_cidr, 3, k + 3)]
  public_subnets  = [for k, v in local.azs : cidrsubnet(var.vpc_cidr, 3, k)]
  azs             = slice(data.aws_availability_zones.available.names, 0, 3)
}

data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.1"

  name = var.cluster_name
  cidr = var.vpc_cidr

  azs                   = local.azs
  public_subnets        = local.public_subnets
  private_subnets       = local.private_subnets
  public_subnet_suffix  = "SubnetPublic"
  private_subnet_suffix = "SubnetPrivate"

  enable_nat_gateway   = true
  create_igw           = true
  enable_dns_hostnames = true
  single_nat_gateway   = true

  # Manage so we can name
  manage_default_network_acl    = true
  default_network_acl_tags      = { Name = "${var.cluster_name}-default" }
  manage_default_route_table    = true
  default_route_table_tags      = { Name = "${var.cluster_name}-default" }
  manage_default_security_group = true
  default_security_group_tags   = { Name = "${var.cluster_name}-default" }

  public_subnet_tags = merge(local.tags, {
    "kubernetes.io/role/elb" = "1"
  })
  private_subnet_tags = merge(local.tags, {
    "karpenter.sh/discovery"          = var.cluster_name
    "kubernetes.io/role/internal-elb" = "1"
  })

  tags = local.tags
}
```

Cuối cùng, tập tin `eks.tf` chỉ định cấu hình cụm EKS của chúng tôi, bao gồm một Nhóm Node Quản lý:

```
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name                             = var.cluster_name
  cluster_version                          = var.cluster_version
  cluster_endpoint_public_access           = true
  enable_cluster_creator_admin_permissions = true

  cluster_addons = {
    vpc-cni = {
      before_compute = true
      most_recent    = true
      configuration_values = jsonencode({
        env = {
          ENABLE_POD_ENI                    = "true"
          ENABLE_PREFIX_DELEGATION          = "true"
          POD_SECURITY_GROUP_ENFORCING_MODE = "standard"
        }
        nodeAgent = {
          enablePolicyEventLogs = "true"
        }
        enableNetworkPolicy = "true"
      })
    }
  }

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  create_cluster_security_group = false
  create_node_security_group    = false

  eks_managed_node_groups = {
    default = {
      instance_types           = ["m5.large"]
      force_update_version     = true
      release_version          = var.ami_release_version
      use_name_prefix          = false
      iam_role_name            = "${var.cluster_name}-ng-default"
      iam_role_use_name_prefix = false

      min_size     = 3
      max_size     = 6
      desired_size = 3

      update_config = {
        max_unavailable_percentage = 50
      }

      labels = {
        workshop-default = "yes"
      }
    }
  }

  tags = merge(local.tags, {
    "karpenter.sh/discovery" = var.cluster_name
  })
}
```

#### Tạo môi trường workshop với Terraform

Đối với cấu hình đã cho, **terraform** sẽ tạo ra Môi trường Workshop với các bước sau:
- Tạo một VPC qua ba AZ
- Tạo ra một cụm EKS
- Tạo một nhà cung cấp IAM OIDC
- Thêm một nhóm node quản lý có tên là `default`
- Cấu hình VPC CNI để sử dụng ủy quyền tiền tố

Tải các tập tin Terraform về:

```bash
$ mkdir -p ~/environment/terraform; cd ~/environment/terraform
$ curl --remote-name-all https://raw.githubusercontent.com/aws-samples/eks-workshop-v2/stable/cluster/terraform/{main.tf,variables.tf,providers.tf,vpc.tf,eks.tf}
```

Chạy các lệnh Terraform sau để triển khai môi trường workshop của bạn.

```bash
$ terraform init
$ terraform apply -var="cluster_name=$EKS_CLUSTER_NAME" -auto-approve
```

Thường mất khoảng 20-25 phút để hoàn thành. Sau khi cụm được tạo ra, chạy lệnh này để sử dụng cụm cho các bài thực hành Lab:

```bash
$ use-cluster $EKS_CLUSTER_NAME
```

#### Bước Tiếp Theo

Bây giờ cụm đã sẵn sàng, hãy đi đến [Bắt Đầu](/docs/introduction/getting-started) hoặc nhảy qua bất kỳ mô-đun nào trong workshop với thanh điều hướng hàng đầu. Khi bạn hoàn thành với workshop, làm theo các bước dưới đây để dọn dẹp môi trường của bạn.

#### Dọn dẹp

Phần sau đây sẽ hướng dẫn cách dọn dẹp các tài nguyên sau khi bạn đã hoàn thành các bài thực hành mong muốn. Những bước này sẽ xóa hết tất cả hạ tầng được cung cấp.

Trước khi xóa môi trường Cloud9, chúng ta cần dọn dẹp cụm mà chúng ta đã thiết lập ở trên.

Đầu tiên, sử dụng `delete-environment` để đảm bảo rằng ứng dụng mẫu và bất kỳ hạ tầng Lab nào còn sót lại đều được loại bỏ:

```bash
$ delete-environment
```

Tiếp theo, xóa cụm với **terraform**:

```bash
$ cd ~/environment/terraform
$ terraform destroy -var="cluster_name=$EKS_CLUSTER_NAME" -auto-approve
```
