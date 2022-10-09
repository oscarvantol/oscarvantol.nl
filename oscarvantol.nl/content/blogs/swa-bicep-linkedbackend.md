---
date: 2022-10-09
title: Using Bicep to link external Apis to Environments in Azure Static Web Apps
aliases:
  - /blog-swa-bicep-linkedbackend
---

I was preparing a talk called **Azure Static Web Apps and the Api Multiverse** in which I wanted to hook up external apis instead of the managed functions. The first thing I wanted to do is have a front-end that I could deploy that calls some kind of api on the backend. I did not want to create the Static Web App 'the magic way' but I wanted to start with IaC, so for this I created a bicep file.

``` bicep
param location string = resourceGroup().location

resource swamv_ui 'Microsoft.Web/staticSites@2022-03-01' = {
  name: 'swamv'
  location: location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
  properties: {
    stagingEnvironmentPolicy: 'Enabled'
    allowConfigFileUpdates: true
    provider: 'Custom'
    enterpriseGradeCdnStatus: 'Disabled'
  }
}

```

## Build and deploy

Next up I created a simple WASM Blazor app from the template and started a build/deployment pipeline. In this pipeline the code is built and the Azure CLI is used to create the resource using Bicep. Now for publishing the app to the Static Web App resource, a way to do a push deployment is using the SWA cli. This publish command needs a deployment token, you could save it to some secret variable in the pipeline but I decided to just query it because we have de Azure CLI here anyway. 


``` yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: |
    cd swamv-ui
    dotnet publish -o bin/publish
  displayName: 'build blazor app'

- task: AzureCLI@2
  displayName: 'Create SWA (IaC)'
  inputs:
    azureSubscription: 'REDACTED'
    scriptType: 'pscore'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az deployment group create -g swamv-rg -f swamv.bicep

- task: AzureCLI@2
  displayName: 'Deploy Blazor app to SWA'
  inputs:
    azureSubscription: 'REDACTED'
    scriptType: 'pscore'
    scriptLocation: 'inlineScript'
    inlineScript: |
      npm install -g @azure/static-web-apps-cli
      $token = az staticwebapp secrets list --name swamv --query "properties.apiKey"
      swa deploy -d $token -a ./swamv-ui/bin/publish/wwwroot/ --env production
```

## Linking an api with Bicep
I wanted to hook up multiple apis in the demo I'm doing, the first one is a simple Azure Function. In the Azure Portal you can just link a backend by clicking a button and choosing your backend from a dropdown. If you want to define the link in Bicep it is its own resource. You need three components, the Static WebApp which we already created, a resource (reference) to the backend you want to connect and you describe the link resource itself.

``` bicep

resource swamv_backend_azure_functions 'Microsoft.Web/sites@2021-03-01' existing = {
  name: 'funcapiapp'
  scope: resourceGroup('functions-api-app-rg')
}

resource swamv_ui_backend_functions 'Microsoft.Web/staticSites/linkedBackends@2022-03-01' = {
  parent: swamv_ui
  name: 'swamv_backend_functions'
  
  properties: {
    backendResourceId: swamv_backend_azure_functions.id
    region: 'westeurope'
  }
}
```

## Linking an api to an other Environment

But in my talk I wanted to show multiple different backends, I found out if you redefine the link it seems that Azure will not allow you to deploy it because it is trying to add multiple backends. 

I decided to use different environments (which basically are staging slots) for the other backends, I was a bit confused because the linked backend resource did not have any field in there where you can specify the Environment. 

**But wait!**  I did not specify any environment in the Bicep, I just published directly using the SWA cli and it auto created them. As it turns out the environments are sub resources and called 'Builds' because of 'history' and naming is hard?

``` bicep

resource swamv_backend_appservice 'Microsoft.Web/sites@2022-03-01' existing = {
  name: 'swamv-appservice-api'
  scope: resourceGroup('swamv-appservice-api')
}

resource swamv_ui_environment_appservice 'Microsoft.Web/staticSites/builds@2022-03-01' existing = {
  parent: swamv_ui
  name: 'appservice'
}

resource swamv_ui_linkedbackend_appservice 'Microsoft.Web/staticSites/builds/linkedBackends@2022-03-01' = {
  name: 'swamv_backend_appservice'
  parent: swamv_ui_environment_appservice
  properties: {
    backendResourceId: swamv_backend_appservice.id
    region: 'westeurope'
  }
}
```

In this last part you can see that the bicep is referencing an exsiting app service resource called ```swamv-appservice-api```, in this case it is using an existing environment resource ```(build)``` called **appservice** and it is creating a link between the two in the defined resource called ```swamv_ui_linkedbackend_appservice```.

[oscarvantol.nl](https://oscarvantol.nl) 