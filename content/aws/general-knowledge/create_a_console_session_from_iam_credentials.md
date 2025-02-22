---
author_name: Nick Frichette
title: Create a Console Session from IAM Credentials
description: "How to use IAM credentials to create an AWS Console session."
---

Link to Tool: [aws-vault](https://github.com/99designs/aws-vault)  
Alternative Tool: [leapp](https://github.com/Noovolari/leapp)

When performing an AWS assessment you will likely encounter IAM credentials. These credentials can be used with the [AWS CLI](/aws/general-knowledge/using_stolen_iam_credentials/#working-with-the-keys) or other tooling to query the AWS API. 

While this can be useful, sometimes you just can't beat clicking around the console. If you have IAM credentials, there is a way that you can spawn an AWS Console session using a tool such as [aws-vault](https://github.com/99designs/aws-vault). This can make certain actions much easier rather than trying to remember the specific flag name for the AWS CLI.

!!! Note
    If you are using temporary IAM credentials (ASIA...), for example, from an EC2 instance, you do not need to have any special IAM permissions to do this. If you are using long-term credentials (AKIA...), you need to have either [sts:GetFederationToken](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/get-federation-token.html) or [sts:AssumeRole](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/assume-role.html) permissions. This is to generate the temporary credentials you will need.

!!! Tip
    If you are attempting to avoid detection, this technique is **not** recommended. Aside from the suspicious `ConsoleLogin` CloudTrail log, and the odd user-agent (Why is the IAM role associated with the CI/CD server using a Firefox user-agent string?), you will also generate a ton of CloudTrail logs.

To start, [export the relevant environment variables](/aws/general-knowledge/using_stolen_iam_credentials/#working-with-the-keys) for the IAM credentials you have. Next, [install aws-vault](https://github.com/99designs/aws-vault#installing).

From here, perform the following commands depending on the type of credentials you have.

## User Credentials

For long-term credentials (Those starting with AKIA), there is an extra step that must be completed first. You will need to generate temporary credentials to retrieve the sign in token. To do this, we will make use of [sts:GetFederationToken](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/get-federation-token.html). As an alternative, [sts:AssumeRole](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sts/assume-role.html) can also be used.

```
aws sts get-federation-token --name blah
```

This will return temporary IAM credentials that you can use with the next step.

## STS Credentials

For short-term credentials (Those starting with ASIA), you can run the following command:

```
aws-vault login
```

!!! Tip
    If you'd like to generate a link without it automatically opening a new tab in your browser you can use the `-s` flag and it will be printed to stdout.

To learn more about custom identity broker access to the AWS Console please see the [official documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_enable-console-custom-url.html).