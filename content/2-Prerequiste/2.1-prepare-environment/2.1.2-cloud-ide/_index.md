---
title: "Deploy VSCode as a web app with EC2 and CloudFront"

weight: 2
chapter: false
pre: "<b> 2.1.2 </b>"
---


{{% notice info %}}
Provisioning this workshop environment in your AWS account will create resources and **there will be cost associated with them**. The cleanup section provides a guide to remove them, preventing further charges.
{{% /notice %}}

This section outlines how to set up the environment to run the labs in your own AWS account.

The first step is to create an IDE with the provided CloudFormation templates. You have the choice between using AWS Cloud9 or a browser-accessible instance of VSCode that will run on an EC2 instance in your AWS account.

Use the AWS CloudFormation quick-create links below to launch the desired template in the appropriate AWS region.

| Region           | Link       (Preview)                                                                                                                                                                                                                                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `us-west2`       | [Launch](https://us-west-2.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-pdx-f3b3f9f1a7d6a3d0.s3.us-west-2.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `eu-west-1`      | [Launch](https://eu-west-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-dub-85e3be25bd827406.s3.eu-west-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-vscode-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF)         |
| `ap-southeast-1` | [Launch](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home#/stacks/quickcreate?templateUrl=https://ws-assets-prod-iad-r-sin-694a125e41645312.s3.ap-southeast-1.amazonaws.com/39146514-f6d5-41cb-86ef-359f9d2f7265/eks-workshop-ide-cfn.yaml&stackName=eks-workshop-ide&param_RepositoryRef=VAR::MANIFESTS_REF") |

These instructions have been tested in the AWS regions listed above and are not guaranteed to work in others without modification.

{{% notice info %}}
The nature of the workshop material means that the IDE EC2 instance requires broad IAM permissions in your account, for example creating IAM roles. Before continuing please review the IAM permissions that will be provided to the IDE instance in the CloudFormation template. \
We are continuously working to optimize the IAM permissions. Please raise a [GitHub issue](https://github.com/aws-samples/eks-workshop-v2/issues) with any suggestions for improvement.
{{% /notice %}}

Now select that tab that corresponds to the IDE that you have installed.

Scroll to the bottom of the screen and acknowledge the IAM notice:

![acknowledge IAM](../../../images/2/1/2/acknowledge-iam.webp)

Then click the **Create stack** button:

![Create Stack](../../../images/2/1/2/create-stack.webp)

The CloudFormation stack will take roughly 5 minutes to deploy, and once completed you can retrieve information required to continue from the **Outputs** tab:

![cloudformation outputs](../../../images/2/1/2/vscode-outputs.webp)

The `IdeUrl` output contains the URL to enter in your browser to access the IDE. The `IdePasswordSecret` contains a link to an AWS Secrets Manger secret that contains a generated password for the IDE.

To retrieve the password open that URL and click the **Retrieve** button:

![secretsmanager retrieve](../../../images/2/1/2/vscode-password-retrieve.webp)

The password will then be available for you to copy:

![cloudformation outputs](../../../images/2/1/2/vscode-password-visible.webp)

Open the IDE URL provided and you will be prompted for the password:

![cloudformation outputs](../../../images/2/1/2/vscode-password.webp)

After submitting your password you will be presented with the initial VSCode screen:

![cloudformation outputs](../../../images/2/1/2/vscode-splash.webp)

{{% notice note %}}
In case CloudFormation failed to create a CloudFront Distribution resource due to your account not being verified, you can request limit increase for CloudFront Distribution [here](https://support.console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase).
![](../../../images/2/1/2/quota-failed-01.jpg?featherlight=false&width=30pc)
At _Service_, choose "CloudFront Distribution".
![](../../../images/2/1/2/quota-inc-01.jpg?featherlight=false&width=90pc)
At _Request 1_, choose _Web Distributions per Account_ for _Quota_ set a higher value than your previous value (if you're using a new account, just enter any positive number)
![](../../../images/2/1/2/quota-inc-02.jpg?featherlight=false&width=90pc)
At _Case description_, copy the detailed error return from CloudStack, at the CloudFront Distribution creation step, and paste it, then enter a brief description of your problem.
![](../../../images/2/1/2/quota-inc-03.jpg?featherlight=false&width=90pc)
After that, choose a contact method, then click _Submit_ at the bottom.
![](../../../images/2/1/2/quota-inc-04.jpg?featherlight=false&width=90pc)
Now you wait for up to a few days for the Support center to work on your case.
{{% /notice %}}