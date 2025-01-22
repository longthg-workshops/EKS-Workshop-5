---
title: "Deploy Cloud9 Lab Environment (Deprecated - Not recommended)"

weight: 1
chapter: false
pre: "<b> 2.1.1 </b>"
---

**_Attention:_** Cloud 9 has been deprecated and may not be supported for new accounts. We recommend moving to our suggested alternative solutions.


### How to set up the environment to run the labs in your own AWS account.

The first step is to create an IDE with the provided CloudFormation templates. You have the choice between using AWS Cloud9 or a browser-accessible instance of VSCode that will run on an EC2 instance in your AWS account.

These instructions have been tested in the following AWS regions and are not guaranteed to work in others without modification:

| Region           | Cloud9 Link                                                                                                                                                                                                                                                                                                                        | VSCode Link (Preview)                                                                                                                                                                                                                                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `us-west2`       | [Launch](https://us-west-2.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0.s3.us-west-2.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)               | [Launch](https://us-west-2.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0.s3.us-west-2.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `eu-west-1`      | [Launch](https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-dub-85e3be25bd827406.s3.eu-west-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)               | [Launch](https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-dub-85e3be25bd827406.s3.eu-west-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `ap-southeast-1` | [Launch](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-sin-694a125e41645312.s3.ap-southeast-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF") | [Launch](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-sin-694a125e41645312.s3.ap-southeast-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF") |

Alternatively, you can open CloudShell in the mentioned region and run the following command:

```bash test=false
$ wget -q https://raw.githubusercontent.com/aws-samples/eks-workshop-v2/stable/lab/cfn/eks-workshop-ide-cfn.yaml -O eks-workshop-ide-cfn.yaml
$ aws cloudformation deploy --stack-name eks-workshop-ide \
    --template-file ./eks-workshop-ide-cfn.yaml \
    --parameter-overrides RepositoryRef=stable \
    --capabilities CAPABILITY_NAMED_IAM
Waiting for changeset to be created...
Waiting for stack create/update to complete
Successfully created/updated stack - eks-workshop-ide
```

The stack may take a few minutes to complete. When the stack creation is done, you can run the followin command in CloudShell to get the IDE's URL:

```bash
aws cloudformation describe-stacks --stack-name eks-workshop-ide \
    --query 'Stacks[0].Outputs[?OutputKey==`Cloud9Url`].OutputValue' --output text

https://us-west-2.console.aws.amazon.com/cloud9/ide/7b05513358534d11afeb7119845c5461?region=us-west-2
```

Open the URL in your browser to use Cloud9.

![EKS](../../../images/2/1/1/00016.png?featherlight=false&width=90pc)

You can now close **CloudShell**, as all the following commands will be done in the **Cloud9** Terminal. The **AWS CLI** is also install and will retrieve authentication information attached to the IDE. You can run the following command to check it:

```bash test=false
$ aws sts get-caller-identity
```

The next step is to create an **EKS** cluster for the workshop. This will be in part _2.2_.
