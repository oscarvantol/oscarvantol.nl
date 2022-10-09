---
date: 2020-09-23
title: Azure App Configuration and Azure Functions
---

Azure App configuration is an amazing service that we have been using in our project to centralize our configuration. If you are unfamiliar with the service I suggest reading [Azure App Configuration: an introduction](https://www.rickvandenbosch.net/blog/azure-app-configuration-an-introduction/) by [@rickvdbosch](https://twitter.com/rickvdbosch).

We were really happy to move our apis to App Configuration in combination with Azure Key Vault references, this meant that we centralized our configuration, removed duplication and pushed all the secrets to Azure Key Vault. Next to that we have the extra benefit of the other features that it's offering like feature flags.

## Sounds simple, let's do it!

The logical next step after updating the configuration setup for all the apis was to do the same for our Azure Functions. We have a lot of them and they contain a lot of configuration, but an Azure Function is not the same as an Asp.NET core api. This is where the journey starts for today.

The first step was to fire up google and search around for Azure App Configuration in combination with Azure Functions, this led me to the official [Microsoft documentation](https://docs.microsoft.com/en-us/azure/azure-app-configuration/quickstart-azure-functions-csharp#connect-to-an-app-configuration-store). 
The documentation shows following example:

```
private static IConfiguration Configuration { set; get; }

static Function1()
{
    var builder = new ConfigurationBuilder();
    builder.AddAzureAppConfiguration(Environment.GetEnvironmentVariable("ConnectionString"));
    Configuration = builder.Build();
}
```

## Meh...
Okay, this will work but it doesn't feel as slick as the setup that I had in the apis. I really wanted a setup the is similar to the one in the webapps. I had a lot of functions to upgrade and they all are already depending on either an existing IConfiguration implementation or the IOptions pattern.


## FunctionsStartup

In Azure Functions you cannot really get into the HostBuilder setup like in a webapp, the closest thing available is implementing a 'FunctionsStartup'. Well in most of the functions I already had that available because we are setting up our dependency injection here. If you want more background on this topic check the [documentation](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection) or my [examples](https://github.com/oscarvantol/examples-azure-functions) on GitHub.

The biggest issue is that the configuration is already setup by the time the code in the 'Configure' method is called. My plan was to create a new configuration and replace it in the service collection. 

```
    var configBuilder = new ConfigurationBuilder();
    // Add env vars to the new config
    configBuilder.AddEnvironmentVariables();

    // Replace IConfiguration
    builder.Services.RemoveAll<IConfiguration>();
    builder.Services.AddSingleton<IConfiguration>(configBuilder.Build());
```

> **_Update:_**  After publishing the initial version of this post I got a tip from [Anthony Chu](https://twitter.com/nthonyChu) to use [IFunctionsConfigurationBuilder ](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection#customizing-configuration-sources) instead. This new recommended way that I somehow missed in my quest makes this so much more clean! To be able to use this you need at least version 1.1.0-preview1 of the [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/) package installed. Since I have absolutely no fear of preview packages that is not a problem.

> **_Another update:_** The [Microsoft.Azure.Functions.Extensions 1.1.0](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/) package is now out of preview. 

A bit of tweaking and testing later I ended up with the following implementation. This Adds **Azure App Configuration** and **Azure Key Vault** to be able to leverage the Key Vault References. What you can also see in here that it only requires an endpoint to App Configuration because we are using Azure Identity's Default Credential to authentication. You can also use a connection string, but the objective was to remove the secrets from configuration.


```
using System;
using Azure.Identity;
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.Azure;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection.Extensions;

[assembly: FunctionsStartup(typeof(AppConfigurationExample.StartUp))]
namespace AppConfigurationExample
{
    public class StartUp : FunctionsStartup
    {
        public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
        {
            //get the original configuration
            var tmpConfig = builder.GetContext().Configuration;

            // create a new configurationbuilder and add appconfiguration
            builder.ConfigurationBuilder.AddAzureAppConfiguration((options) =>
            {
                var defaultAzureCredential = GetDefaultAzureCredential();

                options.Connect(new Uri(tmpConfig["AppConfigUrl"]), defaultAzureCredential)
                // also setup key vault for key vault references
                    .ConfigureKeyVault(kvOptions =>
                    {
                        kvOptions.SetCredential(defaultAzureCredential);
                    });

                // configure appconfiguation features you want;
                // options.UseFeatureFlags();
                // options.Select(KeyFilter.Any, LabelFilter.Null);
            });

        }

        public override void Configure(IFunctionsHostBuilder builder)
        {
            // Setup DI
            //builder.Services.AddScoped<>();
            //builder.Services.AddTransient<>();
            //builder.Services.AddSingleton<>();
        }

        private DefaultAzureCredential GetDefaultAzureCredential() => new DefaultAzureCredential(new DefaultAzureCredentialOptions
        {
            //be explicit about this to prevent frustration
            ExcludeManagedIdentityCredential = false,
            ExcludeAzureCliCredential = false,
            ExcludeSharedTokenCacheCredential = true,
            ExcludeVisualStudioCodeCredential = true,
            ExcludeInteractiveBrowserCredential = true,
            ExcludeEnvironmentCredential = true,
            ExcludeVisualStudioCredential = true
        });

    }
}

```

## Consuming the configuration
This part was already implemented in my functions, so I didn't have to do anything here. One option is to inject IConfiguration in a non status class containing a function like here:

```
    public class Function1
    {
        readonly IConfiguration _configuration;

        public Function1(IConfiguration configuration)
        {
           _configuration = configuration;
        }

        [FunctionName("Function1")]
        public async Task<string> Run([HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req)
        {
            return _configuration["greetingsconfigkey"];
        }
    }
```

The better way would be to use the **IOptions** pattern as shown in the next example:

**Inside Configure method in StartUp**
```
        builder.Services.AddOptions<ExampleSettingsConfig>()
         .Configure<IConfiguration>((configSection, configuration) =>
         {
                configuration.GetSection("ExampleTestSettings").Bind(configSection);
         });
```
**The Function constructor**
```
        public Function1(IOptions<ExampleSettingsConfig> exampleOptions)
        {
            _config = exampleOptions.Value;
        }
```

> **But there is a warning...**
>This implementation has some limitations described [here](https://github.com/Azure/azure-functions-host/issues/4464#issuecomment-513017446). The configuration needed for the binding triggers cannot be moved to App Configuration because it is used outside your Azure Function code by the platform. If you would have a Service Bus trigger, the scale controller that will determine if it needs to spin up hosts for your functions needs access to the queue or subscription you are listening to. I am sure this has the attention of the team responsible, but I can imagine it is not an easy problem to solve.

## What else did we get?
Next to the details options you have in the Azure App Configuration setup like setting up label filters you also get the Feature Flags feature. You can enable this by calling 'options.UseFeatureFlags()' in the setup.

## Sharing is caring

**Example code**
I added a simple example implementation on GitHub for anyone to do whatever they want with it. You can find it [here](https://github.com/oscarvantol/examples-azure-functions/tree/master/AppConfigurationExample).



[oscarvantol.nl](https://oscarvantol.nl) 
