---
title: AWS accounts
description: Configure your infrastructure so Octopus can deploy infrastructure to AWS and run scripts against the AWS CLI.
position: 20
---

To deploy infrastructure to AWS, you can define an AWS account in Octopus.

Octopus manages the AWS credentials used by the AWS steps.

The AWS account is either a pair of access and secret keys, or the credentials are retrieved from the IAM role assigned to the instance that is executing the deployment.

## Create an AWS account

AWS steps can use an Octopus managed AWS account for authentication.

1. Navigate to **{{Infrastructure,Accounts}}**, click the **ADD ACCOUNT** and select **AWS Account**.
1. Add a memorable name for the account.
1. Provide a description for the account.
1. Enter the **Access Key** and the secret **Key**.

See the [AWS documentation](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) for instructions to create the access and secret keys.

5. Click the **SAVE AND TEST** to save the account and verify the credentials are valid.

:::hint
AWS steps can also defer to the IAM role assigned to the instance that hosts the Octopus Server for authentication. In this scenario there is no need to create the AWS account.
:::

## AWS account variables

You can access your AWS account from within projects through a variable of type **AWS Account Variable**. Learn more about [AWS Account Variables](/docs/projects/variables/aws-account-variables.md)

## Using AWS Service roles for an EC2 instance

AWS allows you to assign a role to an EC2 instance, referred to as an [AWS service role for an EC2 instance](https://g.octopushq.com/AwsDocsRolesTermsAndConcepts), and that role can be accessed to generate the credentials that are used to deploy AWS resources and run scripts.

All AWS steps execute on a worker. By default, that will be the [built-in worker](/docs/infrastructure/workers/index.md#built-in-worker) in the Octopus Server. As such, Octopus Server itself would need to be run on an EC2 instance with an IAM role applied to take advantage of this feature.

If you use [external workers](/docs/infrastructure/workers/index.md#external-workers) which are their own EC2 instances, they can have their own IAM roles that apply when running AWS steps.

:::hint
When using the IAM role assigned to either the built-in worker or external worker EC2 instances, there is no need to create an AWS account in Octopus.
:::

## Manually using AWS account details in a step

A number of steps in Octopus use the AWS account directly. For example, in the CloudFormation steps, you define the AWS account variable that will be used to execute the template deployment, and the step will take care of passing along the access and secret keys defined in the account.

It is also possible to use the keys defined in the AWS account manually, such as in script steps.

First, add the AWS Account as a variable. In the screenshot below, the account has been assigned to the **AWS Account** variable.

The **OctopusPrintVariables** has been set to true to print the variables to the output logs. This is a handy way to view the available variables that can be consumed by a custom script. You can find more information on debugging variables at [Debug problems with Octopus variables](/docs/support/debug-problems-with-octopus-variables.md).

![Variables](variables.png "width=500")

When running a step, the available variables will be printed to the log. In this example, the following variables are shown:

```
[AWS Account] = 'amazonwebservicesaccount-aws-account'
[AWS Account.AccessKey] = 'ABCDEFGHIJKLONOPQRST'
[AWS Account.SecretKey] = '********'
```

**AWS Account.AccessKey** is the access key associated with the AWS account, and **AWS Account.SecretKey** is the secret key. The secret key is hidden as asterisks in the log because it is a sensitive value, but the complete key is available to your script.

You can then use these variables in your scripts or other step types. For example, the following PowerShell script would print the access key to the console.

```
Write-Host "$($OctopusParameters["AWS Account.AccessKey"])"
```

## Known AWS connection issue

If you are experiencing SSL/TLS connection errors when connecting to AWS from your Octopus Server, you may be missing the **Amazon Root CA** on your Windows Server. The certificates can be downloaded from the [Amazon Trust Repository](https://www.amazontrust.com/repository/).

## AWS deployments

Learn more about [AWS deployments](/docs/deployments/aws/index.md).
