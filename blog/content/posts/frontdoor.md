---
title: "A nightmare on FrontDoor in Bicep..."
subtitle :  "Inconsistent documentation and imcompatible versions"
description :  "Troubleshooting getting Azure Frontdoor up and running in Bicep"
author: "Bas van de Sande"
featuredImage : "/frontdoor/frontdoor-feature.png"
date: 2023-07-18T22:36:57+02:00
tags: 
    - Azure
    - Frontdoor
    - Bicep
categories: 
    - troubleshooting
    - technical
    - development
draft: false
---

For some reason I seem to attract the most exotic scenarios when it comes to building Infrastructure as Code (IaC). This time it was no different. For a client I'm working on building an environment in which disaster recovery is a top priority. Besides being zone redundant, the client requires region redundancy as well, in case a complete Azure region gets wiped out by a disaster or a combination of disasters... My imagination gets triggered and I envision all kinds of apocalyptic movie scenarios.    

When it comes to region redundancy Mirosoft states that Azure regions should have a distance in between of at elast 300 miles (480 km). In this case my client is building a redundant infrastructure in both West Europe and in its region pair North Europe. In case both regions are wiped out at the same time, we have other problems and Azure won't be one of them :P

## The scenario:
The client has a number of websites that are accessible from over the internet. Instead of exposing the websites directly to the internet, the websites are tucked away behind an hardened Azure Application Gateway. The application gateway is responsible for redirecting the web traffic to the corresponding app services. All traffic is encrypted using SSL, meaning that a keyvault is used to store the SSL certificates.
This infrastructure needs to be available in both regions, in which we consider West Europe as the primary region and North Europe as the secondary. 

In our scenario, we don't want customers having to use different URL's for accessing websites in West Europe or in North Europe. They all should listen to the same URLs. To facilitate this, Azure has an offering called FrontDoor, a technology which emerged from the Xbox Live era. This is where the pain starts...

## Different technologies
It turns out that the FrontDoor concept is basically two different technologies: Frontdoor Classic and Frontdoor Standard/Premium. The object models of the versions are very different and internally there is a plethora of versions. Versions that turn out to be incompatible with each other.

When you want to build a FrontDoor using Bicep, always use `Microsoft.Cdn/profiles`. It turns out that the modern frontdoor is a CDN (Content Delivery Network). If you see a Names like `Microsoft.Network/frontDoors` then run.... This is the old expensive Frontdoor Classic; which is five times more expensive than the Standard offering.

## Microsoft.Cdn is the way to go!
In the FrontDoor Standard/Premium, the problems start when you are following the Bicep examples that are out there. None of the examples out there with a (secured) KeyVault is working.  It turns out that the different versions of the bicep definitions are incompatible with each other. 

Each time I ran my GitHub workflow in order to deploy the Frontdoor, I ended up with the following exception.

_**BadRequest: Expected secret source id to be in the format '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup Name}/providers/Microsoft.KeyVault/vaults/{keyVaultName}/ certificates/{secretName}/{secretVersion - if applicable}**_


No matter what I tried, looking up existing resources, manually creating the the resourceId string. Each time I was confronted with the same exception. It drove me mad.

![error](/frontdoor/frontdoor-error.png)

It turns out that between the different versions of the frontdoor bicep model there is a number of incompatibilities. One of them involves retrieving secrets from a keyvault in order to be injected into Azure Frontdoor. 

For me the following combination turned out to work: KeyVault `Microsoft.KeyVault/vaults@2022-07-01` and in order to create the certifcate in the FrontDoor : FrontDoor secrets  `Microsoft.Cdn/profiles/secrets@2021-06-01`.  


**The Bicep below is a good starting point to:** 
- build an Azure FrontDoor Standard
- Using an User Managed Identity
- Who has permissions to manage certificates in a keyvaults and read secrets from it
- And is able to bind SSL certificates to the FrontDoor  

From here on, I can set up the origin group, to which I can add the origins with their respective priority. 

In my case, I want that all traffic will be routed to West Europe (priority 1) and in case of a non responding region to North Europe (priority 2). When up and running I can enforce the Application Gateway access to be FrontDoor only, by setting a rule to accept incoming connections only from Front Door...

Happy times ahead! 


```c#
@description('The name of the Front Door endpoint to create. This must be globally unique.')
param endpointName string = '${frontdoorProfileName}-${uniqueString(resourceGroup().id)}'

@description('The name of the SKU to use when creating the Front Door profile.')
@allowed([
  'Standard_AzureFrontDoor'
  'Premium_AzureFrontDoor'
])
param skuName string = 'Standard_AzureFrontDoor'

@description('The name of the resource group that contains the key vault with custom domain\'s certificate.')
param certificateKeyVaultResourceGroupName string = resourceGroup().name

@description('The name of the Key Vault that contains the custom domain\'s certificate.')
param certificateKeyVaultName string

@description('The name of the Key Vault secret that contains the custom domain\'s certificate.')
param certificateKeyVaultSecretName string

@description('The version of the Key Vault secret that contains the custom domain\'s certificate. Set the value to an empty string to use the latest version.')
param certificateKeyVaultSecretVersion string = ''


@description('Name of the frontdoor.')
param frontdoorProfileName string

@allowed([
  'northeurope'
  'westeurope'
])
@description('The location of resources.')
param location string 

var identityname = '${frontdoorProfileName}_id'

// Create the user identity and assign the identity rights to the existing keyvault
resource frontdooridentity 'Microsoft.ManagedIdentity/userAssignedIdentities@2018-11-30' = {
  name: identityname
  location: location
}

resource existingCertificatesKeyVault 'Microsoft.KeyVault/vaults@2023-02-01' existing = {
  name: certificateKeyVaultName
}

var certificatesOfficerRoleDefinitionName='a4417e6f-fecd-4de8-b567-7b0420556985'
var secretsUserRoleDefinitionName='4633458b-17de-408a-b874-0445c86b69e6'

resource certificatesOfficerRoleDefinition 'Microsoft.Authorization/roleDefinitions@2022-04-01' existing = {
  scope: subscription()
  name: certificatesOfficerRoleDefinitionName
}

resource secretsUserRoleDefinition 'Microsoft.Authorization/roleDefinitions@2022-04-01' existing = {
  scope: subscription()
  name: secretsUserRoleDefinitionName
}

resource certificatesOfficerRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  scope: existingCertificatesKeyVault
  name: guid(existingCertificatesKeyVault.id, frontdooridentity.id, certificatesOfficerRoleDefinition.id)
  properties:{
    roleDefinitionId: certificatesOfficerRoleDefinition.id
    principalId: frontdooridentity.properties.principalId
    principalType: 'ServicePrincipal'
  }
}

resource secretsUserRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  scope: existingCertificatesKeyVault
  name: guid(existingCertificatesKeyVault.id, frontdooridentity.id, secretsUserRoleDefinition.id)
  properties:{
    roleDefinitionId: secretsUserRoleDefinition.id
    principalId: frontdooridentity.properties.principalId
    principalType: 'ServicePrincipal'
  }
}

resource profile 'Microsoft.Cdn/profiles@2022-11-01-preview' = {
  name: frontdoorProfileName
  location: 'global'
  sku: {
    name: skuName
  }
  identity:{
    type:'SystemAssigned, UserAssigned'
    userAssignedIdentities:{
      '${frontdooridentity.id}': {}
    }
  }
}

resource endpoint 'Microsoft.Cdn/profiles/afdEndpoints@2020-09-01' = {
  name: endpointName
  parent: profile
  location: 'global'
  properties: {
    originResponseTimeoutSeconds: 240
    enabledState: 'Enabled'
  }
}

resource keyVault 'Microsoft.KeyVault/vaults@2022-07-01' existing = {
  scope: resourceGroup(certificateKeyVaultResourceGroupName)
  name: certificateKeyVaultName

  resource secret 'secrets' existing = {
    name: certificateKeyVaultSecretName
  }
}

resource secret 'Microsoft.Cdn/profiles/secrets@2021-06-01' = {
  name: certificateKeyVaultSecretName
  parent: profile
  properties: {
    parameters: {
      type: 'CustomerCertificate'
      useLatestVersion: (certificateKeyVaultSecretVersion == '')
      secretVersion: certificateKeyVaultSecretVersion
      secretSource: {
        id: keyVault::secret.id
      }
    }
  }
}

....

```




