---
layout: post
title:  "Reference: What's not in Entra ID's Free tier?"
date:   2024-11-07 09:30:00 -0400
categories: technical azure
image: /assets/img/entra-free-tier/entra-free-tier-header.png
---

> This post is a reference sheet.
{: .prompt-info }

*Capturing what features require Entra ID paid licensing.*

# Overview
This post will capture Entra ID's features not covered by "Free" tier licensing (requiring a P1 or P2 license). Comparative documentation of this tier was removed from Entra ID's licensing page earlier this year. So I've been keeping a personal reference page for my own testing!

If you play with Entra ID at all or are on a budget, my hope is that this can serve as a quick-reference sheet for these differences. See "[[#Features requiring P1 & P2 licensing]]" to jump directly to the details.

# Background
Entra Suite was [introduced July 2024](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/microsoft-entra-suite-now-generally-available/ba-p/2520427) as an additional licensing tier of Entra ID. The main emphasis of this change was to highlight several new Zero-Trust features. However, the addition of Suite removed the Free tier pricing comparison for Entra ID.

While this change makes the communication of purchasable licensing simpler, it makes the page more difficult for hobbyists, testers, and those running Entra ID on a budget.

## Licensing page: before & after
For some context, here's a look at the pricing comparison page's new layout:
![Suite Page](/assets/img/entra-free-tier/entra-suite-page.png)
Source: [Microsoft](https://www.microsoft.com/en-us/security/business/microsoft-entra-pricing)

And here's how that same page used to look. Note the "Free" option on the left:
![Free Page](/assets/img/entra-free-tier/entra-free-page.png)
Source: [Wayback Machine](https://web.archive.org/web/20240502014425/https://www.microsoft.com/en-us/security/business/microsoft-entra-pricing)

Each page is followed by a list detailing what features are supported at each tier.

# Features requiring P1 & P2 licensing
Below is a list of Entra ID features requiring P1 or P2 licensing, based on a previous capture of the licensing page ([Wayback Machine](https://web.archive.org/web/20240502014425/https://www.microsoft.com/en-us/security/business/microsoft-entra-pricing)). I've added a couple additional notes or acronyms for personal reference. (If you see any errors below, feel free to ping me and I can add in an update!)

## P1 license required
- Advanced group management (dynamic groups, naming policies, expiration, default classification)
- Multi-tenant organizations
- Self-service password reset/change/unlock (SSPR) with on-premises write-back (Standard SSPR previously listed in Free)
- Self-service sign-in activity search and reporting
- Self-service group management (My Groups)
- Self-service entitlement management (My Access)
- CAP: Conditional access
- CAE: Continuous access evaluation
- Global password protection and management (custom banned passwords, users synchronized from on-premises Active Directory)
- Custom security attributes
- Administrative Unit role assignments
- "Advanced security and usage reports" ([More detail here](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins#license-and-role-requirements))

## P2 license required
*(P2 includes all P1 features.)*
- Risk-based Conditional Access (sign-in risk, user risk)
- Authentication context (step-up authentication)
- Device and application filters for Conditional Access
- Token protection
- Vulnerabilities and risky accounts (Risky Users)
- Risk event investigation
- Access certification & reviews
- Privileged identity management (PIM)
- Basic entitlement management

# Thoughts
New features are exciting, but it's frustrating to loose the comparison to Free tier. While the introduction of Suite heralds a simpler comparison of *paid* licenses, something gets lost in the onboarding of new users: What's available to test before buying?

It makes sense for a business to design around paid features, so I won't comment on what the "right answer" is for designing a page like this. Regardless, I hope if you're among Entra curious you found this helpful!