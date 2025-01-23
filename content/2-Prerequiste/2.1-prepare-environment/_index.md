---
title: "Environment Preperation"
weight: 1
chapter: false
pre: "<b> 2.1 </b>"
---

{{% notice info %}}
Provisioning this workshop environment in your AWS account will create resources and **there will be cost associated with them**. The cleanup section provides a guide to remove them, preventing further charges.
{{% /notice %}}

This section outlines how to set up the environment to run the labs in your own AWS account.

The first step is to create an IDE with the provided CloudFormation templates. You have the choice between using AWS Cloud9 or a browser-accessible instance of VSCode that will run on an EC2 instance in your AWS account.

### **Create your CloudFormation Stack**
Dowload the template [here](https://github.com/aws-samples/eks-workshop-v2/releases/download/release-snapshot-631eaeb7/ide-cfn.yaml).

Enter the [CloudFormation](console.aws.amazon.com/cloudformation/home) console.

![](/images/2/1/3/010.jpg)

- Choose **_Create Stack_**.

Enter **_Stack Name_**s.

![](/images/2/1/3/011.jpg?width=70pc)

Change the parameters as you see fit or keep it as it is.

Click **_Next_** through all steps and wait for the stack to complete.

![](/images/2/1/3/008.jpg?width=50pc)

The CloudFormation stack will take roughly 5 minutes to deploy, and once completed you can retrieve information required to continue from the **Outputs** tab:

![cloudformation outputs](/images/2/1/2/vscode-outputs.webp)

### **Access your VSCode environment**

The `IdeUrl` output contains the URL to enter in your browser to access the IDE. The `IdePasswordSecret` contains a link to an AWS Secrets Manger secret that contains a generated password for the IDE.

To retrieve the password open that URL and click the **Retrieve** button:

![secretsmanager retrieve](/images/2/1/2/vscode-password-retrieve.webp)

The password will then be available for you to copy:

![cloudformation outputs](/images/2/1/2/vscode-password-visible.webp)

Open the IDE URL provided and you will be prompted for the password:

![cloudformation outputs](/images/2/1/2/vscode-password.webp)

After submitting your password you will be presented with the initial VSCode screen:

![cloudformation outputs](/images/2/1/2/vscode-splash.webp)

{{% notice note %}}
In case CloudFormation failed to create a CloudFront Distribution resource due to your account not being verified, you can request limit increase for CloudFront Distribution [here](https://support.console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase).
![](/images/2/1/2/quota-failed-01.jpg?featherlight=false&width=30pc)
At _Service_, choose "CloudFront Distribution".
![](/images/2/1/2/quota-inc-01.jpg?featherlight=false&width=90pc)
At _Request 1_, choose _Web Distributions per Account_ for _Quota_ set a higher value than your previous value (if you're using a new account, just enter any positive number)
![](/images/2/1/2/quota-inc-02.jpg?featherlight=false&width=90pc)
At _Case description_, copy the detailed error return from CloudStack, at the CloudFront Distribution creation step, and paste it, then enter a brief description of your problem.
![](/images/2/1/2/quota-inc-03.jpg?featherlight=false&width=90pc)
After that, choose a contact method, then click _Submit_ at the bottom.
![](/images/2/1/2/quota-inc-04.jpg?featherlight=false&width=90pc)
Now you wait for up to a few days for the Support center to work on your case.
{{% /notice %}}