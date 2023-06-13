---
title: "Note to self: fix invalid GitHub credentials in VSCode"
subtitle :  "Getting started"
description :  "Troubleshooting Credentials are suddenly invalid"
author: "Bas van de Sande"
date: 2023-06-13T20:58:18+01:00
featuredImage :  "/ghcreds/ghcreds-feature.png"
tags :  
    - GitHub 
    - Credentials
categories : 
    - technical
draft :  false
---

From time to time I start working on projects for new customers. As soon as I start I receive an invitation from Github to join the customer's Github organization. Most of the organization have a SAML based single sign on (SSO). 
After joining the organization using my personal Github account, I'm able to work on the repositories on which I have access. So far so good... 

Then the moment comes that I need to clone the repository, which can be a hassle from time-to-time. Normally I clone the repository using VSCode. After cloning the repository, I start working by adding my own features. 

From time to times, when I want to pull code or push my feature branch to the origin, I get confronted with an error message: 

```
remote: The organization has enabled or enforced SAML SSO. To access
remote: this repository, you must re-authorize the OAuth Application GitHub for VSCode.
fatal: unable to access 'https://github.com/': The requested URL returned error: 403
```

This error indicates that the application needs to be re-authorized in GitHub. For some reason I keep forgetting on how to do that. As a note to my self I describe the steps involved.
- Go to github.com and open the Settings menu in your profile.
- Click on "Applications" under "Integrations" and navigate to the tab "Authorized OAuth Apps".

![Applications](/ghcreds/ghcreds-applications.png)

-  Click the ellipse button `...` of the specific application (e.g. VSCode) and choose "Revoke".

![Revoke](/ghcreds/ghcreds-revoke.png)

- Confirm that the authorization of the application needs to be revoked by clicking the button.

![Confirm Revoke](/ghcreds/ghcreds-revoke-confirm.png)


Open an command line (powershell or bash), navigate to your local repo folder and enter the command:

```
git config --global credential.helper manager-core
```

Re-run `git pull` and follow the pop-up instructions to authenticate in a browser (which happened automatically for me with SSO).

After these steps git commands started to work for me again.