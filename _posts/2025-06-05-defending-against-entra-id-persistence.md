---
layout: post
title:  "Persistence is Futile: Defending against Entra ID persistence"
date:   2025-06-04 09:00:00 -0400
categories: technical azure 
image: /assets/img/defending-persistence/defending-persistence-header.png
---

I recently presented "Persisting Unseen: Attacker Methods of Infesting Entra ID" at [RSAC's virtual Cloud Security seminar](https://www.rsaconference.com/library/virtual%20seminar/hds19-cloud-security). This session introduced some methods attackers may use now or in the near future to maintain access to Entra ID (formerly Azure AD) once they've obtained a privileged foothold.

This post serves two purposes:
- **A follow-up** to my session that you can easily share with your team. The links below will get started reviewing, hardening, and monitoring these areas of Entra ID!
- **A standalone** list of recommended reading on securing Entra ID against some more well-known persistence techniques.

You can find the slides for my session here:
- [Persisting Unseen: Attacker Methods of Infesting Entra ID](/assets/pdf/2025_RSAC-Cloud-Seminar_Entra-ID-Persistence.pdf)

This post is by no means a *comprehensive* guide, and is meant more as an *introductory* guide. The reading and resources on this list focus on curated reading in three core areas you can begin to monitor and secure.

For those who feel they've got the below covered, there's even more reading and blogs I look to in "[What Next?](#what-next)".

## Logging & Monitoring
Let's start by mentioning the most important reference for investigating Entra ID activity:
- [Microsoft, "Microsoft Entra audit log categories and activities"](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/reference-audit-activities) 

This page is as good as a source of truth as it gets for Entra ID Audit logging. Microsoft includes all categories and Activity names for Entra ID actions on this page. 

Keep this list on hand if you ever need to reference the likely Activity name for an action. While there may be several Activity names for an action based on how it's performed, this list is the best to start from.

For more detail on core logging requirements of Entra ID, check out this guide from Invictus IR:
- [Invictus IR, "Cloud Incident Readiness: Key logs for cloud incidents](https://www.invictus-ir.com/news/cloud-incident-readiness-key-logs-for-cloud-incidents)

## Role Assignments
<img src="/assets/img/defending-persistence/securing-roles.png">
 *Source: "Persisting Unseen: Attacker Methods of Infesting Entra ID"*

### Background
Role assignments are a core function of Entra ID, allowing users to take privileged actions within the identity management side of Entra, Azure, and M365.

For the purposes of how role assignments can be misused, it's good to get a grasp of how they work, along with privileged built-in roles and permissions. From there, you'll want to get comfortable with assignment nuances: Privileged Identity Management (PIM) and administrative units (AUs) are great examples of this.

On core Entra ID role concepts:
- [Microsoft, "Understand roles in Microsoft Entra ID"](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/concept-understand-roles)
- [Microsoft, "Microsoft Entra built-in roles"](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference)
- [Microsoft, "Privileged roles and permissions in Microsoft Entra ID"](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/privileged-roles-permissions)

On nuances and assignment types:
- [Trimarc Security, "Demystifying Privileged Identity Management - Part 1"](https://www.hub.trimarcsecurity.com/post/demystifying-privileged-identity-management-part-1)
- [Microsoft, "Administrative units in Microsoft Entra ID"](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/administrative-units)
- [Microsoft, "Elevate access to manage all Azure subscriptions and management groups"](https://learn.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin?tabs=azure-portal%2Centra-audit-logs) (How Entra ID Global Administrators are able to gain the Azure User Access Administrator role)

Microsoft Graph permissions weren't covered in this talk, but you can find some of my favorite reading on them in "[What Next?](#what-next)".

### Attacks
*Basic* use of privileged roles to persist is fairly straightforward: If you're an admin, you can take privileged actions. But don't let this distract you! There's still unexpected ways that roles can be used.

For some examples, the Entra Portal conceals certain privileged roles not intended for direct assignment. Certain properties of AUs can also conceal role assignments.

On concealing role assignments:
- [Andy Robbins, "The Most Dangerous Entra Role You’ve (Probably) Never Heard Of"](https://specterops.io/blog/2024/02/16/the-most-dangerous-entra-role-youve-probably-never-heard-of/)
- [Katie Knowles, "Hidden in Plain Sight: Abusing Entra ID Administrative Units for Sticky Persistence"](https://securitylabs.datadoghq.com/articles/abusing-entra-id-administrative-units/)

### Defense
To inspect and defend against privileged role misuse, you'll want to ensure you're reviewing **all** role assignments. Even those that don't show in the Portal, or that are provided through PIM.

Some resources on doing this right, **heavy emphasis** on Vasil Michev's blog on the topic:
- **[Vasil Michev, "Reporting on Entra ID directory role assignments (including PIM)"](https://www.michev.info/blog/post/5958/reporting-on-entra-id-directory-role-assignments-including-pim)**
- [Microsoft, "List Microsoft Entra role assignments"](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/view-assignments)
- [Microsoft, "Privileged Identity Management documentation"](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/)

### Detection
Monitor role assignments, and updates to custom roles. With all the ways roles can be assigned, this is more event names than you'd think!

A good list to start with:
- "Add member to role"
- "Add eligible member to role"
- "Add scoped member to role"
- "Add member to role scoped over Restricted Management Administrative Unit"
- "Add member to role completed (PIM activation)"
- "Update role definition"

Additionally, monitor assignments of hidden and privileged roles:
Hidden Roles:
- Partner Tier2 Support
- Partner Tier1 Support
- Directory Synchronization Accounts

Privileged Roles:
- Privileged roles [as classified by Microsoft](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/privileged-roles-permissions) (if your organization is not so large that this is unfeasible)
- Custom roles that your team considers high-sensitivity, or have privileged permissions

## Applications
<img src="/assets/img/defending-persistence/securing-apps.png">
*Source: "Persisting Unseen: Attacker Methods of Infesting Entra ID"*

### Background
Entra ID's application model, especially in the context of identities, is notoriously difficult to understand. If the application model doesn't make sense at first, don't be discouraged! The more you work with it, the easier it will be for this to "just make sense".

Applications are split between two types of objects, app registrations and service principals. They can hold traditional Entra ID roles, as well as application and delegated permissions more classically associated with apps.

At a basic level, you should know that:
- **App registrations** define Entra's OAuth applications, and live in a single Entra ID tenant. When the application represented by the app registration is added to any Entra ID tenant, a service principal is created using some of the app registration's properties (such as what application permissions to request) to represent the application.
- **Service principals (SPs)** represent the application in a local Entra ID tenant. An SP is created in each tenant that has added the application. SPs associated with app registrations are sometimes more specifically referred to as **enterprise applications**. The SP holds permissions assignments that have been granted to the application.

Credentials (e.g. secrets, certificates) can be added **independently** to:
- **The app registration.** App registration credentials can be used to authenticate to **any** tenant that has installed the application.
- **The SP.** SP credentials can be used to authenticate to **just** the tenant where that SP lives.

In **both cases**, once authentication is complete, the ID of the SP will be used in the local tenant as the primary actor completing an action.

For more depth on this:
- [John Savill, "Azure AD App Registrations, Enterprise Apps and Service Principals"](https://www.youtube.com/watch?v=WVNvoiA_ktw&t=1200s)
- [Microsoft, "Application model"](https://learn.microsoft.com/en-us/entra/identity-platform/application-model)
- A security view: [Semperis, "UnOAuthorized: Privilege Elevation Through Microsoft Applications"](https://www.semperis.com/blog/unoauthorized-privilege-elevation-through-microsoft-applications/) (in section "Multitenant applications in Entra ID")

For now, we just need to grasp that:
1. Application identities can have privileges, 
2. Application identities can have credentials added to them,
3. Applications only need a credential to authenticate (no second factor).

### Attacks
An attacker with the "Application Administrator", "Cloud Application Administrator", or Owner role over a target application can add credentials to authenticate as an application and use its identity and permissions:
- If permissions are over the SP, they can "hijack" the application's identity within the SP's home tenant.
- If permissions are over an app registration, they can "hijack" the application in **any** tenant the application is installed in, so long as they know the tenant ID of their target ([easy to find!](https://aadinternals.com/osint/)).

More on these attacks:
- [Dirk-jan Mollema, "Taking over default application permissions as Application Admin"](https://dirkjanm.io/azure-ad-privilege-escalation-application-admin/)
- [Eric Woodruff, "UnOAuthorized: Privilege Elevation Through Microsoft Applications"](https://www.semperis.com/blog/unoauthorized-privilege-elevation-through-microsoft-applications/)
- [Microsoft, "Microsoft Actions Following Attack by Nation State Actor Midnight Blizzard"](https://msrc.microsoft.com/blog/2024/01/microsoft-actions-following-attack-by-nation-state-actor-midnight-blizzard/)
- [Lior Sonntag, "Midnight Blizzard attack on Microsoft corporate environment"](https://www.wiz.io/blog/midnight-blizzard-microsoft-breach-analysis-and-best-practices)
- [Microsoft, "Threat actor consent phishing campaign abusing the verified publisher process"](https://msrc.microsoft.com/blog/2023/01/threat-actor-consent-phishing-campaign-abusing-the-verified-publisher-process/)

### Defense
Lucky for us, preventing credentials on applications has gotten much easier. The past couple years have produced great resources, and new settings to defend applications.

Resources on protecting your applications from malicious credentials:
- **[Vasil Michev, "Script to review and remove service principal credentials"](https://www.michev.info/blog/post/5894/script-to-review-and-remove-service-principal-credentials)**
- **[Microsoft, "How to configure app instance property lock for your applications"](https://learn.microsoft.com/en-us/entra/identity-platform/howto-configure-app-instance-property-locks)**
- [Microsoft, "Security best practices for application properties in Microsoft Entra ID"](https://learn.microsoft.com/en-us/entra/identity-platform/security-best-practices-for-app-registration#certificates-and-secrets)
- [Daniel Bradley, "How to block the creation of Client Secrets on Entra applications"](https://ourcloudnetwork.com/how-to-block-the-creation-of-client-secrets-on-entra-applications/)

Additionally, you should ensure settings are in place to prevent OAuth consent phishing:
- [Microsoft, "Configure user consent settings"](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-user-consent?pivots=portal#configure-user-consent-settings)
- [Microsoft, "Protect against consent phishing"](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/protect-against-consent-phishing)
- [Microsoft, "Investigate and remediate risky OAuth apps"](https://learn.microsoft.com/en-us/defender-cloud-apps/investigate-risky-oauth)

### Detection
To monitor applications, focus on new credentials added to applications, as well as new permissions and applications. Focus on high-risk permissions assignments first if these events are too much for your environment.

Monitor credentials added to app registrations and SPs:
- "Update application - Certificates and secrets management"
- "Add service principal credentials"

Monitor app permissions assignment:
- "Add app role assignment to service principal"
- "Consent to application"
- "Add delegated permission grant"

## Authentication
<img src="/assets/img/defending-persistence/securing-auth.png">
*Source: "Persisting Unseen: Attacker Methods of Infesting Entra ID"*

### Background
How many ways can a user sign in? For Entra ID, there's a **lot**. It's good to get a feel for general authentication methods, and what they qualify for in Microsoft's model of what's considered "strong authentication".

Conditional Access Policies (CAPs) define what authentication is allowed by which users in which context. When combined with authentication methods, they define the overall user authentication strength of your Entra ID tenant.

On core authentication methods and conditional access:
- [Microsoft, "What authentication and verification methods are available in Microsoft Entra ID?"](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods)
- [Microsoft, "What is Conditional Access?"](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)

While not applicable to human identity authentication, it's important to keep in mind that application identities (SPs) will authenticate through a separate set of workflows. More on this is available in "[What Next?](#what-next)".

#### Device Code Authentication
Microsoft has currently deployed a CAP set to report-only mode, titled "Block Device Code Flows". This policy will likely be enforced by default in the near future, thus my not touching on this in more detail! It's still a great idea to enforce a block of this method now if you haven't already.

You can read more about this risk and blocking it here:
- On the policy: [Microsoft, "Block authentication flows with Conditional Access policy"](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-block-authentication-flows)
- On the risk: [Microsoft, "Storm-2372 conducts device code phishing campaign"](https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/)

### Attacks
With MFA in place for most identities, attacks against authentication are now more than password updates. While this is great for raising the bar on initial access, it can make understanding the myriad of techniques difficult to get through.

Some good starting resources on this are:
- Manipulating CAPs: [Microsoft, "Octo Tempest crosses boundaries to facilitate extortion, encryption, and destruction"](https://www.microsoft.com/en-us/security/blog/2023/10/25/octo-tempest-crosses-boundaries-to-facilitate-extortion-encryption-and-destruction/)
- Adding new MFA methods: [CISA, "Scattered Spider"](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-320a)
- Using an 8 character Temporary Access Pass (TAP) to access user accounts with strong authentication: [Daniel Heinsen, "I’d TAP That Pass"](https://posts.specterops.io/id-tap-that-pass-8f79fff839ac)
- Adding FIDO2 tokens to access user accounts: [Max Rozendaal, "Abusing FIDO2 passkeys to take over Global Administrators in Entra ID"](https://www.secura.com/services/information-technology/vapt/what-can-be-pentested/cloud-pentesting/abusing-fido2-passkeys)

### Defense
Ensure you understand how your users are authenticating, and which are exceptions to these policies, is your best defense. This will allow you to review and clean up users who may have been exempted from CAP.

It's also a good idea to block authentication flows you're not familiar with. This could include Device Code Authentication, MFA via SMS, or (if you're not using it) the TAP method discussed above. This limits the variety of authentication methods an attacker is likely to target.

To start on these reviews:
- [Microsoft, "Use access reviews to manage users excluded from Conditional Access policies"](https://learn.microsoft.com/en-us/entra/id-governance/conditional-access-exclusion)
- **[Microsoft, "Block authentication flows with Conditional Access policy"](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-block-authentication-flows)**

### Detection
With many methods of authentication, monitoring authentication misuse takes many events. Below is a subset of what you should look into.

These will likely need tuning based on what's typical to your environment! How users authenticate in an environment can vary a lot between organizations.

Monitor CAP modification:
- "Update Conditional Access policy"
- "Delete Conditional Access policy"
- "Update named location"
- "Update security defaults"

Monitor changes to user authentication methods:
- "Admin registered security info"
- "Admin deleted security info"
- "Reset password"
- "User registered security info"
- "Admin started password reset"
- "Admin registered temporary access pass method for user"

Monitor TAP Sign-in Logs:
- "Sign-in activity" where "authenticationDetails.authenticationMethod" is "Temporary Access Pass"

Monitor Device Code Sign-in Logs:
- "Sign-in activity" where "properties.authenticationProtocol" is "deviceCode"

## What Next?

### Microsoft Graph Permissions
Microsoft Graph permissions are a type of application permission that grant applications the ability to perform privileged actions within Entra ID and M365, through the Microsoft Graph API. These are crucial to review as a follow-up to reading on [[#Applications]], and grant additional permissions similar to those granted by [[#Role Assignments]].

More on Microsoft Graph permissions:
- [Microsoft, "Overview of Microsoft Graph permissions"](https://learn.microsoft.com/en-us/graph/permissions-overview?tabs=http)
- [Microsoft, "Microsoft Graph permissions reference"](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [Merill Fernando, "Microsoft Graph Permissions Explorer"](https://graphpermissions.merill.net/permission/)
- [Emilien Socchi, "Application permissions tiering"](https://github.com/emiliensocchi/azure-tiering/tree/main/Microsoft%20Graph%20application%20permissions)

### Voices to Follow
You may also want to follow what's new at this stage. I've included some voices I frequently look to below: They may help guide your journey from here.

#### Newsletter
If you want just one resource to start with:
Merill Fernando's [Entra News](https://entra.news/) newsletter provides quality, weekly updates on Entra ID. The newsletter's section on security is especially helpful!

**NOTE:** The lists below are provided in ***alphabetical order***, not importance. (It's also possible I've left someone off by accident!) Regardless, I hope this gets you started.

#### Individuals
Individual and personal blogs that frequently focus on Entra ID and Azure security:
- Brad Wyatt, [https://www.thelazyadministrator.com/](https://www.thelazyadministrator.com/)
- Christian Bortone, [https://xybytes.com/](https://xybytes.com/)
- Dirk-jan Mollema, [https://dirkjanm.io/](https://dirkjanm.io/)
- Dr Nestori Syynimaa, [https://aadinternals.com/](https://aadinternals.com/)
- Emilien Socchi, [https://www.emiliensocchi.io/](https://www.emiliensocchi.io/)
- Jeffrey Appel, [https://jeffreyappel.nl/](https://jeffreyappel.nl/)
- Lina Lau,[https://www.inversecos.com/](https://www.inversecos.com/)
- Marius Solbakken, [https://goodworkaround.com/](https://goodworkaround.com/)
- Merill Fernando, [https://blog.merill.net/](https://blog.merill.net/)
- Rudy Ooms, [https://call4cloud.nl/](https://call4cloud.nl/)
- Sapir Federovsky, [https://sapirxfed.com/blog-posts/](https://sapirxfed.com/blog-posts/)
- Thomas Naunheim, [https://www.cloud-architekt.net/blog/](https://www.cloud-architekt.net/blog/)
- Vasil Michev, [https://www.michev.info/](https://www.michev.info/)

#### Companies
Company blogs that frequently focus on Entra ID and Azure security:
- Binary Security, [https://www.binarysecurity.no/posts/](https://www.binarysecurity.no/posts/)
- Datadog, [https://securitylabs.datadoghq.com/](https://securitylabs.datadoghq.com/) (**Disclosure:** Datadog is my employer 🐕)
- FalconForce, [https://falconforce.nl/blogs/](https://falconforce.nl/blogs/)
- Invictus IR, [https://www.invictus-ir.com/news](https://www.invictus-ir.com/news)
- NetSPI, [https://www.netspi.com/blog/technical-blog/](https://www.netspi.com/blog/technical-blog/)
- Permiso, [https://permiso.io/blog](https://permiso.io/blog)
- SecureWorks, [https://www.secureworks.com/research](https://www.secureworks.com/research?q=azure)
- SpecterOps, [https://specterops.io/blog/category/research/](https://specterops.io/blog/category/research/)
- TrustOnCloud, [https://trustoncloud.com/blog/](https://trustoncloud.com/blog/)
- Wiz, [https://www.wiz.io/blog/tag/research](https://www.wiz.io/blog/tag/research)