---
author: Nick Frichette
title: Brute Force IAM Permissions
description: Brute force the IAM permissions of a user or role to see what you have access to.
enableEditBtn: true
editBaseURL: https://github.com/Hacking-the-Cloud/hackingthe.cloud/blob/master/content
---
Link to Tool: [GitHub](https://github.com/andresriancho/enumerate-iam)

When attacking AWS you may compromise credentials for an IAM user or role. This can be an excellent step to gain access to other resources, however it presents a problem for us. How do we know what we have access to? Unless these credentials have specific IAM permissions we won't be able to look them up, and we may not have enough context clues from where we found the credentials to know about them. 

This leaves us with basically one option, brute force the permissions. To do this, we will try as many AWS API calls as possible to determine what we have access to. There are several tools to do this however here we will be covering [enumerate-iam](https://github.com/andresriancho/enumerate-iam) by Andrés Riancho.

To use enumerate-iam, simply pull a copy of the tool from GitHub, provide the credentials, and watch the magic happen. All calls by enumerate-iam are non-destructive, meaning only get and list operations are used. This reduces the risk of accidentally deleting something in a client's account.

```
user@host:/enum$ ./enumerate-iam.py --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY --session-token $AWS_SESSION_TOKEN
2020-12-20 18:41:26,375 - 13 - [INFO] Starting permission enumeration for access-key-id "ASIAAAAAAAAAAAAAAAAA"
2020-12-20 18:41:26,812 - 13 - [INFO] -- Account ARN : arn:aws:sts::012345678912:assumed-role/role-b/user-b
2020-12-20 18:41:26,812 - 13 - [INFO] -- Account Id  : 012345678912
2020-12-20 18:41:26,813 - 13 - [INFO] -- Account Path: assumed-role/role-b/user-b
2020-12-20 18:41:27,283 - 13 - [INFO] Attempting common-service describe / list brute force.
2020-12-20 18:41:34,992 - 13 - [INFO] -- codestar.list_projects() worked!
2020-12-20 18:41:35,928 - 13 - [INFO] -- sts.get_caller_identity() worked!
2020-12-20 18:41:36,838 - 13 - [INFO] -- dynamodb.describe_endpoints() worked!
2020-12-20 18:41:38,107 - 13 - [INFO] -- sagemaker.list_models() worked!
```

### Updating APIs
With an attack surface that evolves as rapidly as AWS, we often have to find and abuse newer features. This is one area where enumerate-iam shines. The tool itself has a built in feature to read in new AWS API calls from the JavaScript SDK, and use that information to brute force. After downloading enumerate-iam, perform the following steps to update the API lists.

```
cd enumerate_iam/
git clone https://github.com/aws/aws-sdk-js.git
python generate_bruteforce_tests.py
```

This will create or update a file named bruteforce_tests.py under enumerate-iam.

### OPSEC Considerations
One thing to note is that this tool is very noisy and will generate a ton of CloudTrail logs. This makes it very easy for a defender to spot this activity and lock you out of that role or user. Try other methods of permission enumeration first, or be willing to lose access to these credentials before resorting to brute-force. 

{{< notice success Note >}}
As of 12/20/2020 there is a bug in the AWS API that allows you to [enumerate a variety of permissions without logging to CloudTrail](/aws/enumeration/stealth_perm_enum/). Try that method first, before resorting to brute force so you can stay more stealthy.
{{< /notice >}}