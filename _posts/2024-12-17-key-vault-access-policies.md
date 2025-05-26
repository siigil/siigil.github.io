---
layout: post
title:  "Escalating privileges to read secrets with Azure Key Vault access policies"
date:   2024-12-17 06:30:00 -0400
categories: technical azure
image: /assets/img/external/escalating-privileges-to-read-secrets.png
---

Azure Key Vault Contributors are not allowed access to Key Vault data. But did you know they can still gain access to Key Vault keys, secrets, and certificates when a key vault is using access policies?

I recently published a post explaining this on Datadog Security Labs:
- ["Escalating privileges to read secrets with Azure Key Vault access policies"](https://securitylabs.datadoghq.com/articles/escalating-privileges-to-read-secrets-with-azure-key-vault-access-policies/)

The permission this relates to has been well documented in previous research. What's interesting here is that an Azure built-in role that should _not_ have access to Key Vault data included this permission. This risk was not previously called out in Microsoft documentation, but has been [recently updated to state it more clearly](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/security#key-vault-contributor).

Hope you find it interesting!