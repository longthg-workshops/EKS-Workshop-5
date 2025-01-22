---
title: "Deploying VSCode on EC2 and access it from your local VSCode"

weight: 3
chapter: false
pre: "<b> 2.1.3 </b>"
---
### **Create your Key Pair**
{{% notice note %}}
If you already have a key pair, you can skip to the next step.
{{% /notice %}}

First, you need to enter the [EC2 Console](console.aws.amazon.com/ec2/home). On the left navigator bar, choose _Key Pairs_. On the _Key Pairs_ interface, choose _Create Key Pair_.

![](../../../images/2/1/3/001.jpg)

On **_Create Key Pair_**:
- At _Name_, enter any name for your key pair.
- Next, choose a key type and the _.pem_ format.
- Click _Create key pair_ at the bottom to proceed to create your key pair.

![](../../../images/2/1/3/002.jpg?width=50pc)

Save your private key in your local computer and use it later for the lab stack creation.

![](../../../images/2/1/3/003.jpg?width=50pc)

Ensure that only you can access the _.pem_ file.

- On Linux, use the following command:
```bash
chmod 400 <đường dẫn tới file pem>
```

- On Windows, ensure the file access permission is as follow:

![](../../../images/2/1/3/004.jpg?width=50pc)

### **Create your CloudFormation Stack**
Dowload the template [here](https://raw.githubusercontent.com/longthg-workshops/eks-workshop-v2-fork/main/lab/cfn/ec2-workshop-local-ide-cfn.yaml).

Enter the [CloudFormation](console.aws.amazon.com/cloudformation/home) console.

![](../../../images/2/1/3/010.jpg)

- Choose **_Create Stack_**.

Enter **_Stack Name_**s.

![](../../../images/2/1/3/011.jpg?width=70pc)

At **_SshKeyName_**, enter the newly created Key Pair (from the previous step).

![](../../../images/2/1/3/009.jpg?width=70pc)

Click _Next_ through all steps and wait for the stack to complete.

![](../../../images/2/1/3/008.jpg?width=50pc)

### **Configure SSH connection on your computer**
On the CloudFormation Stack UI, get to _Output_. Find the key _IdeInstancePublicIP_, the value is the address of your VSCode server instance.

![](../../../images/2/1/3/007.jpg?width=70pc)

Add the following lines `C:\Users\<username>\.ssh\config` (on Windows) hoặc `~/.ssh/config` (on Linux):

```
Host remote-connection
    HostName <EC2 public ip>
    User ec2-user
    IdentityFile "<path-to-pem>"
```

### **Connect your VSCode with your EC2 Instance**
- Install the Remote-SSH addon for your local VSCode

![](../../../images/2/1/3/014.png)

- After installing you'll see the icon on the lower left corner of the screen

- Click the icon to open the Connection command pallete. Choose “Connect to Host…”

![](../../../images/2/1/3/015.png)

- Choose “Add New SSH Host”

![](../../../images/2/1/3/016.png)

- Choose your config file 

- After configuring, click the icon to open the pallette and choose “remote-connection”

![](../../../images/2/1/3/017.png)

Next, choose Platform details “Linux”, then “Continue”

![](../../../images/2/1/3/018.png)

Choose **_"File > Open"_** to open your environment folder

![](../../../images/2/1/3/019.jpg?width=50pc)

Choose the folder _**/home/\< username \>/environment**_

![](../../../images/2/1/3/020.jpg?width=50pc)