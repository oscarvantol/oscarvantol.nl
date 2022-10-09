---
date: 2021-03-27
title: Azure Functions in .NET 5 and beyond
aliases:
    - /blog-azure-functions-in-.NET5
---
In November of 2020 .NET 5 got released, the promise of "one-dotnet". The 'core' addition has been dropped and Full Framework evolution has stopped at the 4.8 version. This does not mean Full Framework support ends, this is tied to the support of windows therefore you can expect an eternity of patches. But when you are building something new now you should use .NET 5 or .NET core 3.1 because of its long term support. 

Right from the release of .NET 5 we could start using it in for example Azure App Service but for Azure Functions this was not the case. As a heavy user of Azure Functions this topic had my interest and I followed the development a bit and in this post you can read what I got away from this.

![alt text](/assets/blog-af5/azure-functions-logo-color-raster.png "Azure Functions")

## Why wasn't it there already?
The reason that this is a bit more challenging is because the way the .NET model for Azure Function is implemented. The functions that you create will build a library that is loaded dynamically and hosted inside a running process. That means that that hosted process determines what version of .NET running. Azure Functions V1 was running .NET Full Framework and from V2 this was .NET core. The design of Azure Functions originates from Azure WebJobs and as you can imagine these libraries have some history by now. This makes it a lot of effort to get every next version of .NET to work, but also to get everything on the Azure side working side by side with a lot of different versions of .NET.

## What's different?
By the end of 2020 the Azure Functions team [reported](https://techcommunity.microsoft.com/t5/apps-on-azure/net-5-support-on-azure-functions/ba-p/1973055) that they would start building a new model for .NET, the "out-of-process" or "isolated" model. This model means that the application you build is no longer loaded inside the host but it is running by itself and communicating with the Functions runtime. This model is similar to the way all other (non .NET) languages run in Azure Functions.

## What's new?
Early March 2021 this [article](https://techcommunity.microsoft.com/t5/apps-on-azure/net-on-azure-functions-roadmap/ba-p/2197916) appeared and we got a 1.0 version of the new ["isolated process"](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide) model, the classic model is now referred to as the [class library](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-class-library?tabs=v2%2Ccmd) model.

The loose coupling means less risk on dependency conflicts that you might have ran into in the past. The complete separation would also mean that you could start using future preview releases. The startup of your application is similar to the model in ASP.NET using the hostbuilder where you can handle your configuration, dependency injection and almost identical middleware support.

## What's the catch?
It will definitely take some time to upgrade your functions, the leap might be even bigger than going from v1 to v2. The new model makes a lot of sense, it feels clean and light weight but is also not as mature. Although the triggers that we know are mostly available, in the current version some features like rich types are not available yet. The main branch is moving pretty fast and apis are being opened up for all kinds extensibility, be sure to monitor that.

 If we are talking about missing features, the elephant in the room is most definitely Durable Functions. The design of DF rely on the tight coupling that we no longer have. Other languages also have an implementation of DF and is on the roadmap to get this back but this will take some time. In November there is a LTS release (.NET 6) where the class library model still needs to be supported. After the .NET 6 release the team will focus on getting DF in by .NET 7.

## What should I do?
The new model is the future of Azure Functions for .NET and I would recommend getting some experience with it, but be aware it is not ready yet for all workloads. Early adopters and teams that thrive on the edge of chaos will be fine with juggling different models at the same time. 
But if you and your team tend to move a bit more carefully or if you rely on Durable Functions don't worry, you still have years of support ahead going from .NET core 3.1 to .NET 6. 

## Show me some code!

The new model is a rewrite so you won't be needing to take on the known dependencies for Azure Functions. If you look at the .csproj of a new Azure Function project you will see multiple small dependencies starting with *Microsoft.Azure.Functions.Worker*.

```
  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.Functions.Worker" Version="1.0.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.CosmosDB" Version="3.0.9" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.EventGrid" Version="2.1.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.EventHubs" Version="4.2.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Http" Version="3.0.12" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Kafka" Version="3.2.1" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.RabbitMQ" Version="1.0.0-beta" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.ServiceBus" Version="4.2.1" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.SignalRService" Version="1.2.2" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Storage" Version="4.0.4" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Timer" Version="4.0.1" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Warmup" Version="4.0.1" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Sdk" Version="1.0.1" OutputItemType="Analyzer" />
    <PackageReference Include="System.Net.NameResolution" Version="4.3.0" />
  </ItemGroup>

```

### The startup
The startup with control over configuration and dependency injection was a late addition to the old model, next to that it was not mandatory. If you were using this setup it would look something like this:

```
[assembly: FunctionsStartup(typeof(StartUp))]
namespace MyFunctions
{
    public class StartUp : FunctionsStartup
    {
        public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
        {
            //Add Azure AppConfiguration
            //
        }

        public override void Configure(IFunctionsHostBuilder builder)
        {
           builder.Services
                AddSuffHere();          
        }
    }
}
```

In the new model you need a main entry point where you need to create a HostBuilder, because it is c#9 you can even do it in a top level statement.

```
using ExampleFunction;
using Microsoft.Azure.Functions.Worker.Configuration;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Hosting;

var host = new HostBuilder()
    .ConfigureAppConfiguration(c =>
    {
        c.AddCommandLine(args);
        //c.AddEnvironmentVariables();
    })
    .ConfigureFunctionsWorkerDefaults(app =>
    {
        //app.UseMiddleware<ExampleMiddleware>();
    })
    .ConfigureServices(s =>
    {
        //s.AddSuffHere();
    })
    .Build();

await host.RunAsync();
```

### The function
The functions themselves seem pretty familiar but they definitly have some differences. First thing you probably won't notice is that the attribute on the function is *[Function]* and not *[FunctionName]*. 
Next thing is that you cannot just inject in the function ILogger anymore, it is available through an optional *FunctionContext*.

```
        [Function(nameof(HttpFunction1))]
        public async Task<string> HttpFunction1([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = "test")] HttpRequestData req, FunctionContext functionContext)
        {
            var log = functionContext.GetLogger<HttpFunction>();            
            log.LogInformation("You called the trigger!");

            return "Hello world";
        }
```

The biggest change is the way output bindings are changed and not clear from the example above, if you want to use multiple outputs you need to define them in a class and use it a a return type. This is illustrated in the example below that I have borrowed from the official [repository](https://github.com/Azure/azure-functions-dotnet-worker/blob/main/samples/FunctionApp/Function3/Function3.cs).

```
    public static class Function3
    {
        [Function("Function3")]
        public static MyOutputType Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequestData req,
            FunctionContext context)
        {
            var response = req.CreateResponse(HttpStatusCode.OK);
            response.WriteString("Success!");

            return new MyOutputType()
            {
                Name = "some name",
                HttpResponse = response
            };
        }
    }

    public class MyOutputType
    {
        [QueueOutput("functionstesting2", Connection = "AzureWebJobsStorage")]
        public string Name { get; set; }

        public HttpResponseData HttpResponse { get; set; }
    }
```

### Getting started yourself

If you want to get your hands dirty, go to this [readme](https://github.com/Azure/azure-functions-dotnet-worker/blob/main/README.md) on GitHub. In the same repository you can find a lot of [Samples](https://github.com/Azure/azure-functions-dotnet-worker/tree/main/samples) that the team created. 

Next to that you can also check out my [examples](https://github.com/oscarvantol/azure-functions-dotnet5-examples).

> Be sure to check the [Known issues](https://github.com/Azure/azure-functions-dotnet-worker/wiki/Known-issues) prevent frustration.

As things evolve I will post updates here.

Package updates:
- Microsoft.Azure.Functions.Worker => 1.1.0
- Microsoft.Azure.Functions.Worker.Sdk => 1.0.2


[oscarvantol.nl](https://oscarvantol.nl)
