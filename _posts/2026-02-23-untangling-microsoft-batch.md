---
layout: post
title:  "Untangling Microsoft Graph's $batch requests in Burp"
date:   2026-02-23 07:00:00 -0400
categories: technical azure
image: /assets/img/untangling-batch/untangling-batch-header.png
---

**tl;dr:** 
- Requests to Microsoft Graph's `$batch` endpoint bundle several API calls into one JSON object. This makes analyzing Azure Portal traffic difficult, since underlying API calls for requests to the `$batch` endpoint are not individually logged.
- This post shares the [graph_batch_parser.py](https://github.com/siigil/burp-extensions/blob/main/graph_batch_parser.py) Burp Suite extension as a way to speed up analysis of `$batch` requests. The extension processes `$batch` requests into a set of synthetic request/response pairs. Requests can then be reviewed in the "Graph Batch" tab, as well as Burp's Site Map.

## Batch what now?
Hands up if you've ever played around with Entra in the Azure Portal, wondering what API calls are made for different actions! 🤚

...or is that just me?

If you ever look at what network traffic the Azure Portal generates under the hood, you'll find that while *some* of Entra's API calls are made to individual endpoints in Microsoft Graph (`graph.microsoft.com`), many more are made using the "batch" endpoints:
- `graph.microsoft.com/v1.0/$batch`
- `graph.microsoft.com/beta/$batch`

[Batch requests are key](https://learn.microsoft.com/en-us/graph/json-batching) to reducing individual requests to the Microsoft Graph API, especially when performing an action that requires multiple API calls. Think of scenarios like fetching all of a user's object details, group memberships, and role assignments; or getting the individual counts of all users, devices, and groups in a tenant. By reducing the number of API calls needed for these actions, an application can serve its content without waiting on dozens of individual API responses.

Each request to the `/$batch` endpoint can contain up to 20 individual API requests, bundled into a single JSON request.

As an example, the `POST` request body below was generated to the `$batch` endpoint by the Azure Portal while browsing user details. It contains requests to both the `/policies/authenticationmethodspolicy` and `/reports/healthmonitoring/alerts` endpoints:

```json
{
  "requests": [
    {
      "id": "46439850-987f-4e92-9533-9eecfe564b82",
      "method": "GET",
      "url": "/policies/authenticationmethodspolicy",
      "headers": {
        "x-ms-command-name": "AuthenticationMethods - GetAuthenticationMethodsPolicy",
        "x-ms-client-request-id": "946edd68-c6ff-4a61-8a26-568f23a87cce",
        "client-request-id": "946edd68-c6ff-4a61-8a26-568f23a87cce",
        "x-ms-client-session-id": "f5b5dc47a5ca4d819edcf0e9e54131fd"
      }
    },
    {
      "id": "d8205284-3f40-45df-b990-50ef26f74564",
      "method": "GET",
      "url": "/reports/healthmonitoring/alerts?%24filter=state+eq+%27active%27+and+createdDateTime+gt+2026-02-12T19%3A27%3A32Z",
      "headers": {
        "Prefer": "include-unknown-enum-members",
        "x-ms-command-name": "ScenarioHealthMonitoringScenario - BK",
        "x-ms-client-request-id": "7864a4ab-9e7c-4203-a9bf-a83fd11ad3f9",
        "client-request-id": "7864a4ab-9e7c-4203-a9bf-a83fd11ad3f9",
        "x-ms-client-session-id": "f5b5dc47a5ca4d819edcf0e9e54131fd"
      }
    }
  ]
}
```

How often are these batch requests used? Looking in Burp Suite, the answer is quite a lot:
![Batch requests are used heavily in the Azure Portal](/assets/img/untangling-batch/1-batch-request-chaos.png)

And _wow_, do I ever hate them!

## Analysis spaghetti
You might ask why I'm opposed to such a helpful endpoint. Isn't better app performance better for everyone?
 
Unfortunately, in this case what's good for developers is difficult for researchers:
1. Batch requests **only log a single request** in Burp's HTTP History tab, do not generate an entry for the API requests _within_ the batch request. This makes it hard to review which individual API requests were made.
2. The requests inside a batch request will **never populate Burp's Site Map**! Only the `$batch` endpoint will be logged for a batch request. This makes it hard to review Microsoft Graph API structure.
3. The `$batch` endpoint supports up to **twenty (20!) requests** in one! That's a lot of scrolling.

So unless going through hundreds of _these_ requests is your idea of a good time...
![Batch requests require several scrolls to review](/assets/img/untangling-batch/2-batch-request-example.png)

...you're not going to have a good time.

## The Graph Batch Parser
I needed to understand how new and existing Azure Portal features work, and didn't want to lose my mind scrolling.

To help with this, I created a Burp Suite extension for my own research. This extension performs the following steps:
1. Parses all  `$batch` requests into a set of equivalent (synthetic) API requests
2. Displays these synthetic requests for easier review in the Graph Batch tab
3. Adds them to Burp's site map, highlighted in cyan

After testing it out and finding it helpful, I'd like to share in case anyone else has pulled their hair out over batch requests before.

You can find it here:
* [graph_batch_parser.py](https://github.com/siigil/burp-extensions/blob/main/graph_batch_parser.py)

![The Graph Batch Parser extension processes Microsoft Graph requests into their equivalent sub-requests](/assets/img/untangling-batch/3-batch-extension-demo.gif)

**Disclaimer:** This extension contains LLM/AI assisted code. That said: I've designed it, put it through reviews, tested its performance, and used it in my own research. Based on my own testing, I think this is in decent shape to share.