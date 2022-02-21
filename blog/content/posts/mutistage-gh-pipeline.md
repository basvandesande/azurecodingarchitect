+++
title = "Multi-staged GitHub Pipeline"
subtitle = "Working with GitHub pipelines"
description = "When working with GitHub pipelines I missed a feature that I needed. Approvals. This article describes a workaround in simpler plans."
author="Bas van de Sande"
date = 2021-09-11T19:46:30+02:00
featuredImage = "/multistage/ms-feature.png"
tags = ["github","workaround","teams"]
categories = ["technical"]
draft = false
+++

At the moment I'm working on some infrastructure pipelines to build and deploy Azure infrastructure for one of our clients. For me a cool project as I'm learning to work with technologies such as Bicep and GitHub actions. In this project we use GitHub actions for the CI/CD process. In this process every time a pull request is merged in the main branch, an automated build, test and deployment process is triggered.

![#multistage](/multistage/ms1.png)

In this project we have three environments:
- test
the deployment to this environment should be automatically after each build and merge into main.
- acceptance
the deployment to this environment should only take place after an approval.
- production
the deployment to this environment should only take place after an approval.

Unfortunately, the stage approvals are only part of the enterprise license, and not of the paid team license we use. I was slightly disappointed and during last weekend I got this little idea...

**How about simulating the approval process?**

GitHub actions supports "Inputs". Inputs are values that have to be set when triggering a workflow. In case a worklow is triggered automatically, the default values are being used. In case the workflow is triggered manually, the user is presented with a popup dialog in which the user has to fill in the values.

This gives the user the option to deploy a build to other environments as well. In the example below, I have the option to either deploy to the acceptance and/or production environment; once the build and deploy to test succeeded.

![#multistage](/multistage/ms3.png)

However  In the flow presented above, you cannot deploy to production if you don't deploy to acceptance. To overcome this issue, the yaml needs to be altered, that the deployment for both the acceptance and deployment stage depend on a succeeding deployment in the test environment. 

![#multistage](/multistage/ms2.png)

To do this, the yaml script has to be setup like this:

```yml

name: my multi staged hub infra deployment

# Set environment variables that are being used in the workflows
env:
  LOCATION: WestEurope
  CLI_VERSION: 2.21.0

on:
  push:
    branches: [ master ]
    paths:
    - 'Infra/Modules/**'
    - 'Infra/Deployments/hub/**'
    - '.github/workflows/hub.yml'
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:
    inputs:
      acceptance:
        description: 'Deploy to Acceptance (yes/no)?'
        required: true
        default: 'no'
      production:
        description: 'Deploy to production (yes/no)?'
        required: true
        default: 'no'
jobs:
  build-hub:
    name: Build Hub
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed') || github.event_name == 'repository_dispatch' || github.event_name == 'workflow_dispatch'

    runs-on: ubuntu-latest
    steps:
      - name: Checkout files
        uses: actions/checkout@v2
 
      ...

  test-provision-hub:
    name: TEST - Provision Hub
    needs: build-hub
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v2
        with:
          name: drop
          path: src
  
     ...

  acc-provision-hub:
    name: ACC - Provision Hub 
    if: github.event.inputs.acceptance == 'yes'
    environment: acc   # defined environment with protection rules added
    needs: test-provision-hub
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v2
        with:
          name: drop
          path: src
  
      ...

  prod-provision-hub:
    name: PROD - Provision Hub 
    if: github.event.inputs.production == 'yes'
    environment: prod  # defined environment with protection rules added
    needs: test-provision-hub
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v2
        with:
          name: drop
          path: src
  
      ...

```

For now this workaround will do fine, however I'm not so happy with the omission of the multi stage approval in the GitHub Teams subscription. For that reason I contacted a GitHub representative and had a little chat with him. He told me that the focus within GitHub is the Enterprise edition. He advised me to contact GitHub support for this matter. I guess that is going to be my next step.