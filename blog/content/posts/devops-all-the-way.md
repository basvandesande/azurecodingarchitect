---
title : "DevOps All the Way"
subtitle : "DevOps makes the world go round"
description : "Working in DevOps, means that your blog has to go DevOps all the way. In this article I describe how to setup an Azure DevOps CI/CD pipeline."
author : "Bas van de Sande"
date : 2021-07-30T09:00:06+02:00
featuredImage : "/devops/devops-feature.png"
tags : 
  - devops
  - hugo
  - pipeline
categories : 
  - technology
draft :  false
---

In [Practice what you preach]({{< ref "/posts/practice-what-you-preach" >}} "Practice what you Preach") I wrote that I decided to host my blog on the Azure platform, using Hugo as content management system, Blob Storage and Azure CDN for hosting the blog. The only thing lacking is a workflow of writing content, approving it and publishing it on the web. 

As I work for a [company](https://www.xpirit.com) that specialized on Azure and DevOps, I decided to take the plunge and go DevOps all the way. Doing some googling, I found out that there were some great tutorials out there to use an Azure DevOps CI/CD pipeline in combination with Hugo to compile and publish the blog. As a starter I followed this [writing](https://medium.com/@kurtmkurtm/setting-up-a-blog-with-the-hugo-framework-on-azure-blob-storage-12605609a90) by kurtmkurtm.

There was however one part of the instruction I wasn't happy with. In the description the author created a CI pipeline using a yaml pipeline, the CD release pipeline was setup by clicking it together. In a DevOps world you have to automate as much as possible to make it repeatable in an easy way (building, testing, deploying). Personally, I consider a CI/CD pipeline as a part of the code you write, thus it needs to be in code (and not in a hybrid fashion).

What I did, was adding stages to the original pipeline. One stage for build and one stage for deployment.

```yaml
#
# Azure pipeline to build and publishe a hugo blog release
# github.com/kurtmkurtm
# modified by Bas van de Sande
#

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
- group: blogvars
   
stages:
- stage: Generateblog
  displayName: Generate Blog
  jobs:
  - job: 
    displayName: Generate Blog
    steps:
    - script: |
        wget -c https://github.com/gohugoio/hugo/releases/download/v$(hugo.version)/hugo_$(hugo.version)_Linux-64bit.deb   
      displayName: 'Download HUGO'

    - script: 'sudo dpkg -i hugo_$(hugo.version)_Linux-64bit.deb'
      displayName: 'Install HUGO'

    - script: |
        cd $(blog.path)
        hugo  --log -v
      displayName: 'Generate Blog'

    - task: CopyFiles@2
      displayName: 'Copy Blog'
      inputs:
        SourceFolder: '$(blog.path)/public'
        Contents: '**'
        TargetFolder: '$(Build.ArtifactStagingDirectory)'

    - task: PublishBuildArtifacts@1
      displayName: 'Publish Artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: PubllishBlog
  displayName: Publish Blog
  jobs:
  - job:
    displayName: Publish Blog
    steps:
    - task: DownloadBuildArtifacts@0
      displayName: Download Artifacts
      inputs:
        buildType: 'current'
        downloadType: 'specific'
        itemPattern: '**'
        downloadPath: '$(System.ArtifactsDirectory)'
    
    - task: AzureCLI@2
      displayName: Copy web to Blob storage
      inputs:
        azureSubscription: '$(azureSubscription)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: az storage blob upload-batch -d '$web' -s $(System.ArtifactsDirectory)/drop  --account-name $(storageAccountName) --account-key $(storageAccountKey)
```

In order to make the pipeline reusable, I took out all literals and placed them into variables. These variables are maintained in the Azure DevOps Library section below.

![Pipeline variables](/devops/devops-all-the-way.png)

Thus by implementing the automated build and deployment pipeline, every merge into the main branch results into an automated deployment. Mission accomplished!

![#success](/devops/buildsucceeded.png)


