---
title: "Workshop Structure"
weight: 3
chapter: false
pre: "<b> 2.3 </b>"
---

Let’s review how to navigate this web site and the content provided.

### Structure

The content of this workshop is made up of:

1. Individual lab exercises
2. Supporting content that explains concepts related to the labs

{{% notice note %}}
You should start each lab from the first page. Starting in the middle of a lab will cause unpredictable behavior.
{{% /notice %}}

Once you have accessed the VSCode IDE, we recommend you use the **+** button and select **New Terminal** to open a new full screen terminal window.

![EKS](/images/part2/3/00013.png?featherlight=false&width=90pc)

This will open a new tab with a fresh terminal.

![EKS](/images/part2/3/00014.png?featherlight=false&width=90pc)

You may also close the small terminal at the bottom if you wish.

### Terminal commands

Most of the interaction you will do in this workshop will be done with terminal commands, which you can either manually type or copy/paste to the VSCode IDE terminal. You will see this terminal commands displayed like this:

```bash test=false
$ echo "This is an example command"
```

Hover your mouse over `echo "This is an example command"` and click to copy that command to your clipboard.

You will also come across commands with sample output like this:

```bash test=false
$ date
Fri Aug 30 12:25:58 MDT 2024
```

Using the 'click to copy' function will only copy the command and ignore the sample output.

Another pattern used in the content is presenting several commands in a single terminal:

```bash test=false
$ echo "This is an example command"
This is an example command
$ date
Fri Aug 30 12:26:58 MDT 2024
```

In this case you can either copy each command individually or copy all of the commands using the clipboard icon in the top right of the terminal window. Give it a shot!

### Resetting your EKS cluster

In the event that you accidentally configure your cluster in a way that is not functioning you have been provided with a mechanism to reset your EKS cluster as best we can which can be run at any time. Simply run the command `prepare-environment` and wait until it completes. This may take several minutes depending on the state of your cluster when it is run.
