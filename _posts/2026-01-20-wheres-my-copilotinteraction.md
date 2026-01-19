---
layout: post
title:  "Dude, where's my CopilotInteraction?"
date:   2026-01-19 07:00:00 -0400
categories: technical azure 
image: /assets/img/asst/wheres-my-copilotinteraction-header.png
---

**tl;dr:** This post documents requirements for logging the `CopilotInteraction` event, as well as some caveats of when it isn't logged. This event is key for Copilot audit trails. An App Insights alternative for logging interactions in Copilot Studio is also provided.

## Background

In August 2025, I started looking into Copilot Studio. This led to some really interesting discoveries, like [using Copilot Studio's agents for OAuth phishing](https://securitylabs.datadoghq.com/articles/cophish-using-microsoft-copilot-studio-as-a-wrapper).

Of all my investigations into Copilot Studio, I found `CopilotInteraction` the most confusing. These logs capture user interactions with Copilot, including M365 files accessed by a user during a Copilot session. Given how important these logs are for audit purposes, I was surprised that they required additional licensing (Exchange) and were not always logged.

**Disclaimer:** This post is not an official licensing guide, just a reflection of my personal experience with Copilot Studio. I've tried my best to ensure it's accurate. That said, there may be additional clarifications I'm unaware of.

### Additional reading

This post will not cover the contents of `CopilotInteraction` events. To understand the contents of these logs, I recommend reading the following articles:

- Microsoft, [Copilot interaction events overview](https://learn.microsoft.com/en-us/office/office-365-management-api/copilot-schema)  
- Nikki Chapple, [How to Investigate Microsoft 365 Copilot Interactions](https://nikkichapple.com/investigating-microsoft-365-copilot-interactions/)  
- Zack Korman, [Copilot Broke Your Audit Log, but Microsoft Won’t Tell You](https://pistachioapp.com/blog/copilot-broke-your-audit-log)

## Requirements for logging Copilot Studio interactions

In short, for `CopilotInteraction` to be logged in a given interaction, the following must be true:

1. The Copilot service (e.g. Copilot Studio agent) **must authenticate users** through the "Authenticate with Microsoft" option.  
2. The user accessing the Copilot service **must be authenticated** (not anonymous) when interacting with the agent.  
3. The user **must have an Exchange license**.

A Copilot Studio agent configured without authentication does not generate `CopilotInteraction` events. Additionally, a user authenticated through Microsoft but without an assigned Exchange license will not generate these events.

### Wait, why?

These requirements are due to Copilot interactions logging similarly to Microsoft Teams, where logs are [stored in the user's Exchange inbox](https://nikkichapple.com/investigating-microsoft-365-copilot-interactions/). No logs will be stored without an Exchange license for an interacting user.

Similarly, the user needs to be identified to the [channel](https://learn.microsoft.com/en-us/microsoft-copilot-studio/publication-fundamentals-publish-channels) the Copilot is used through for any logs to be stored. For this reason, only [Microsoft integrated services](https://learn.microsoft.com/en-us/office/office-365-management-api/copilot-schema#audit-copilot-schema-definitions) (logged as `AppHost`) such as Bing, Teams, Outlook, and other applications that can use Microsoft's integrated authentication will log the `CopilotInteraction` event.

### Security impact

`CopilotInteraction` logs provide an audit trail for M365 file access in scenarios such as incident response through the `AccessedResources` property. Other [file access logs are not always kept](https://pistachioapp.com/blog/copilot-broke-your-audit-log#footnote-1) in the apps that accessed files are from, such as SharePoint or OneDrive. 

Without Copilot interactions, a security team will not be able to review what files a user accessed during their session.

## Ensuring CopilotInteraction is logged

To ensure Copilot interactions are logged, you'll need to enable Exchange licensing for all interacting users, and make sure Copilot Studio agents use Microsoft authentication.

Consider App Insights logging for key Copilot Studio agents if this isn't possible. This will not provide the same simple audit trail of M365 file access, but will log user interactions.

### Enable Exchange licensing for all Copilot Studio users

Ensure an Exchange license or other Microsoft 365 license (e.g. E1, E3, E5) is [assigned in the Microsoft 365 admin center](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/assign-licenses-to-users) for all users that require `CopilotInteraction` logging.

Copilot interactions will **not** be captured for users that do not have an Exchange inbox.

### Enable Microsoft authentication for all Copilot Studio agents

For all agents that require `CopilotInteraction` logging, ensure that the "Authenticate with Microsoft" option is selected in the agent's settings under "Security → Authentication". This will limit use of the agent to [Microsoft 365 application channels](https://learn.microsoft.com/en-us/office/office-365-management-api/copilot-schema#audit-copilot-schema-definitions), which are able to capture interaction logs. Please reach out to Microsoft if you require clarification for a specific channel.

Channels that do not support the "Authenticate with Microsoft" option, such as the preview "Demo website" channel, will not generate `CopilotInteraction` logs. Consider App Insights logging in this scenario.

### Consider App Insights for all Copilot Studio agents

Copilot Studio activity logs can also be captured with App Insights. This creates a separate monitoring source from `CopilotInteraction` events that can track conversation metadata, with an option for full message text. Consider the volume of logging full message text before enabling this option.

These logs do *not* include an `AccessedResources` property, but can still provide useful insight during incident review. 

Details on configuring App Insights for Copilot Studio can be found in [Microsoft's guide](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-bot-framework-composer-capture-telemetry).

## Closing thoughts

Given that Copilot Studio is structured a part of Microsoft 365, in hindsight the way `CopilotInteraction` logs work isn't any worse than Teams. But it falls into the same data residency frustrations that can limit incident response and detection. It's crucial to be aware of these caveats when investigating Copilot Studio.

On top of this, Copilot logging introduces a new problem: What if users that aren't licensed for Exchange regularly need to use agents? This can make life hard for security teams.

I hope this post has at least clarified the basics of logging Copilot interactions, and some of the the risks of not logging them.

Reach out if you have questions\! I'm no licensing expert, but happy to share what I know.