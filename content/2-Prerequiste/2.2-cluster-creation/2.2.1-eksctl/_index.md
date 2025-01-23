---
title: "Using eksctl"
weight: 1
chapter: false
pre: "<b> 2.2.1 </b>"
---

### How to build a cluster for the lab exercises using the [eksctl tool](https://eksctl.io/). 

**_This is the easiest way to get started, and is recommended for most learners._**

The `eksctl` utility has been pre-installed in your Amazon Cloud9 Environment, so we can immediately create the cluster. This is the configuration that will be used to build the cluster:

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

Based on this configuration `eksctl` will:

- Create a VPC across three availability zones
- Create an EKS cluster
- Create an IAM OIDC provider
- Add a managed node group named `default`
- Configure the VPC CNI to use prefix delegation

Apply the configuration file like so:

```bash
$ export EKS_CLUSTER_NAME=eks-workshop
$ curl -fsSL https://raw.githubusercontent.com/VAR::MANIFESTS_OWNER/VAR::MANIFESTS_REPOSITORY/VAR::MANIFESTS_REF/cluster/eksctl/cluster.yaml | \
envsubst | eksctl create cluster -f -
```

This process will take around 20 minutes.

### Next Steps

Now that the cluster is ready, head to the [Navigating the labs](/docs/introduction/navigating-labs) section or skip ahead to any module in the workshop with the top navigation bar. Once you're completed with the workshop, follow the steps below to clean-up your environment.

### Cleaning Up (steps once you are done with the Workshop)

{{% notice info %}}
The following demonstrates how you will later clean up resources once you are done using the EKS cluster you created in previous steps to complete the modules.  
{{% /notice %}}

Before deleting the Cloud9/VSCode IDE environment we need to clean up the cluster that we set up in previous steps.

First, use `delete-environment` to ensure that the sample application and any left-over lab infrastructure is removed:

```bash
$ delete-environment
```

Next delete the cluster with `eksctl`:

```bash
$ eksctl delete cluster $EKS_CLUSTER_NAME --wait
```