---
title: "Quick tip: Find permissions in RBAC roles easily"
subtitle :  "Tips & Tricks"
description :  "A handy walkthru to speed up the tedious work"
author: "Bas van de Sande"
featuredImage : "/rbac/rbac-feature.png"
date: 2024-01-24T19:00:52+01:00
tags: 
    - Azure
    - RBAC
categories: 
    - technical
    - tips and tricks
draft: false
---

When building infrastructures in Azure, you sometimes come to a point in which you need to add an additional action to a RBAC role in order to access certain Azure resources. In our case we needed to add read permissions on the Front Door activity logs to an internal security administrator role. The way I used to do it, was browing MS-learn to can find a comprehensive list of built-in roles. For me this was a cumberstone process, until today. One of my co-workers - Martin Bouchery - showed me an easy trick to find all RBAC roles in Azure that have a specific action tied to its profile. A walk through...


## Lookup the desired action

First, open up the [List of built-in Azure roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles). 

![ms learn](/rbac/rbac-mslearn1.png)

In my case I'm interested in all roles with "Front door" in the name. I open up "Front Door Profile Reader" role to see what actions are allowed. In my case I'm interested in the action "Microsoft.CDN/profiles/queryloganalyticsmetrics/action".

![ms learn actions](/rbac/rbac-mslearn3.png)

## The old and tedious way

When I have the specific action, I can search in the list of roles for my current role, open it and verify if the particular action is present. In case of a custom role, I need to figure it out using either the portal or the CLI...  _Cumberstone like I said before._

## The quick and easy way

In your subscription, go to Access control (IAM), choose to add a role assignment (for this you be in an owner role on the subscription).

![Azure portal - Go to Access Management (IAM) - Add a role assignment](/rbac/rbac-portal1.png)

The "Add role assignment" pane shows, in there paste the specific action in the search bar. Immediately the RBAC roles show up in a matter of seconds that have the action assigned to them; whether they are built-in or custom roles. 

![In "Add role assingment" - enter the specific action - see the roles](/rbac/rbac-portal2.png)

This works for all kind of actions, e.g.I want to know what roles are entitled to work with `Microsoft.Storage/storageAccounts/blobServices/containers`. **Poof!**

![This works for any action in Azure](/rbac/rbac-portal3.png)

This little trick, might save you a lot of time. **Enjoy!**
