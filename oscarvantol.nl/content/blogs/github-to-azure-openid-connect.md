---
draft: true
title: Publish from GitHub to Azure the nice way

---


I work a lot with Azure and Azure DevOps and just for the record, I still love Azure DevOps! Even though I don't think Azure DevOps will suddenly dissapear with a snap I do think the future is in GitHub. At the moment of writing this I am preparing a workshop day to go through the current GitHub features from the ISV perspective. This means that in that training we will go through every feature in GitHub that could replace the one you are using right now in Azure DevOps. 

## Deploying to Azure

Lets say you want to build an app and deploy it to Azure with a GitHub Action. There are multiple ways to do that, from an Azure DevOps perspective you might be used to setup a Service Connection to your Azure Subscription or maybe to a resource group. In GitHub there is a similar construction to do this, you can create a service principle and generate a secret in Azure and copy the nescesary context over to the GitHub repository's secrets. Some of the drawbacks of having an actual password like secret are that you need to pass the secret around, it can be used from everywhere and that the secret expires.

## The nice way

There is a similar construction that does not use an actual secret but it builds a trust with your repository so that the token credential can be generated when your pipeline (action) is running. This setup is called: **OpenID Connect with an Azure service principal using a Federated Identity Credential**. The idea behind this is that on one hand you have an App Registration with a Service Principal in Azure that has access to a resource group or subscription. On the other hand you have a GitHub Action that only requests a token using the **ApplicationClientId**, **TenantId** and **SubscriptionId**. 

_But how does Azure decide to supply that token with just three guids and without some kind of real secret?_

For this we need at least one **Federated Identity Credential**, this FIC is defined in Azure and references your GitHub repo. You can setup three kinds of Federated Identity Credentials for GitHub, `branch`, `pull request` and `environment`.

## What are the steps?


### 1. Create the application and the service principal
We are going to use the Azure Cli to create an app and a service principal. When you create the app you will get an appId, this will be input for the service principal. Creating your Service Principal will return objectId, you need this in the next step.

```
# create the app, this will output the appId
#
> az ad app create --display-name my-app-name1 --query appId
"36790de3-0a90-4a0d-b867-0bc5fad4c16d"

# create the service principal, this will output the objectId
#
> az ad sp create --id 36790de3-0a90-4a0d-b867-0bc5fad4c16d --query objectId 
"6e62903a-6ef7-4e96-bb3c-f0b15d1b1bce"

```


### 2. Grant permissions to Azure Resources
```
# assign a role to the service principal
#
az role assignment create --role contributor --subscription <put-your-subscription-id-here> --assignee-object-id 6e62903a-6ef7-4e96-bb3c-f0b15d1b1bce --assignee-principal-type ServicePrincipal

```

### 3. Create a Federated Identity Credential

### 4. Save the 'secrets' in GitHub

### 5. Connect to Azure in a GitHub Action

## This is a lot of stuff
This is quite a recipe and you need to do this for all the repositories you work with for at least one but maybe multiple target environments. For this reason I created an example all in one script and shared it [here](). It's a powershell script that you should be able to run while being logged in to the correct tenant with the Azure Cli and even sets the Secrets in GitHub using the GitHub Cli. I hope this helps a bit!

[oscarvantol.nl](https://oscarvantol.nl) 
