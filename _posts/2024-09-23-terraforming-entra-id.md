---
layout: post
title:  "Terraforming Entra ID Test Environments"
date:   2024-09-22 09:30:00 -0400
categories: technical azure
image: /assets/img/terraforming-entra-id/terraforming-entra-id-header.png
---

*Quick notes on Terraform + Entra ID.*

# On quickly building labs
I recently needed a quick Entra ID test environment to better understand groups, role assignments, and [administrative units](https://securitylabs.datadoghq.com/articles/abusing-entra-id-administrative-units/). Several great tools exist for this purpose! However, I realized that these tools were too complex for my needs.

While complex solutions exist that may fulfill a purpose, it's nice to create something simple that does *exactly* what's needed - especially if you're new to a topic.

Building your own solution also helps create small steps into new technical terrain. Besides the obvious experience benefit of building in code, it's easier to debug by starting with smaller problems like this (see "[Building Up to Hands On](https://kknowl.es/posts/hands-on/#technique-3-projects)" for more of my thoughts here!).

Since existing tools were too complex for my use case, I made a handful of simple Terraform files to meet my needs.

# Terraform + Entra ID!
In the interest of sharing my notes from this exercise, I've added these test environment files to a project on GitHub:
- [https://github.com/siigil/entra-id-terraform](https://github.com/siigil/entra-id-terraform)

The repository is organized in a series of files that will all be applied when you run Terraform from this directory. Files are focused on individual topics to make it easier to understand each concept.

You may find `05-administrative-units.tf` particularly interesting if you're curious to experiment more with restricted management and hidden membership administrative units. (Though you can also do this with [Stratus](https://stratus-red-team.cloud/attack-techniques/entra-id/)!)

Hope you find something useful here, or enjoy playing with these files to build your own environment!