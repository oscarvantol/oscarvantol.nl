---
title: Playwright, NUnit, Page Object Models and Dependencies
date: 2022-07-11
---

# 

In my job I work with multiple teams that next to 'regular' developers also include Test Automation experts. It is very common to see them using a tool like Selenium. A while ago one of the teams needed to reboot a part of their project and we decided to start fresh.

## Fresh and clean
I remembered seeing a [Playwright](https://playwright.dev) demo and we started to investigate if it was something for us. As a C# developer I was pleased with the clean Async implementation and the clear documentation pages. The TA folks were open to explore the tool after I created a demo including a working pipeline running headless tests quickly in a hosted agent in Azure DevOps.

## Follow the guidance
In the official documentation you can see how to do most things. What the [documentation](https://playwright.dev/dotnet/docs/test-runners) also states is to use NUnit, as a developer I would normally use XUnit as my go to but let's not be stubborn for once as the docs state there are some issues.

```c#
[Parallelizable(ParallelScope.Self)]
public class Tests : PageTest
{
    [Test]
    public async Task ShouldAdd()
    {
        int result = await Page.EvaluateAsync<int>("() => 7 + 3");
        Assert.AreEqual(10, result);
    }
}
```

## Demos are simple...
Once the team was implementing real scenarios we instantly needed a bit more thinking around like configuration, secrets and connections to other apis. Remember we wanted to keep it fresh and clean! As a developer being very comfortable with the ASP.NET Core setup the answer to some of those issues are obvious.

## Configuration
The older projects were based on .NET Framework and were still using the ```System.Configuration.Configurationmanager```. To quickly get configuration values within the test class we used ```ConfigurationBuilder```.

```c#
var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json") //make sure the file is copied to output
    .AddEnvironmentVariables()
    // Add any other configuration sources
    .Build();
```

## Secrets
In any real test you are problably using different accounts to login per test case. We really did not want any secrets like passwords in configuration files and although you can use secret vars in pipelines it did not solve it for local development. Also here we implemented the same solution as in our 'normal' dev work. This means **Azure Key Vault**, not only that but in combination with **Managed Identity**. If you are logged in with the Azure Cli and have access to the Key Vault the following example magically works.

```c#
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
// ---
var keyVaultUri = new Uri(configuration["keyvaultUrl"]);
var tokenCredential = new DefaultAzureCredential();
var secretClient = new SecretClient(keyVaultUri, tokenCredential);

var secret = await secretClient.GetSecretAsync("SomeSecretName");
```

## Page Object Models
For test automation it is pretty common to use the POM structure to separate and group interaction per page. This means that you create a class per page to hide implementation from your actual unit test and with that also create some reusability. This can have as effect that the test code will be littered with **new** statements passing in everything needed inside the POM.

## Dependencies
Yes you called it, shouldn't we do some kind of dependency management here? If we would want to keep this clean we want to just get the models, configuration and things like the Secret Client when we need them. 

*We were going for test that look a bit like this:*

```c#
[Test]
public async Task CheckPlatforms()
{
    await Page.GotoAsync(Configuration["BaseUrl"]);
    var homePage = GetService<HomePage>();

    await homePage.OpenPlatformPopup();
    var platFormPopUp = GetService<PlatformPopUp>();

    Assert.IsTrue(await platFormPopUp.HasSpotify());
}
```

There are many ways to set this up, the following example creates a base class called ```TestStartup``` that inherits PageTest so that the actual test class can inherit TestStartup instead of PageTest. It exposes IConfiguration and a GetService method, the Services (POMs and other dependencies) are added on a ServiceCollection inside the TestStartup class.

[TestStartup.cs](https://github.com/oscarvantol/playwright-nunit-di/blob/main/PlayNuDi/TestStartup.cs)
```c#
public class TestStartUp : PageTest
{
    private IServiceProvider _serviceProvider;
    private IConfiguration _configuration;
    private TokenCredential _tokenCredential;

    public TestStartUp()
    {
        _configuration = BuildConfiguration();
        _serviceProvider = BuildServices();
        _tokenCredential = new DefaultAzureCredential();
    }

    private IConfiguration BuildConfiguration()
    {
        var builder = new ConfigurationBuilder()
           .AddJsonFile("appsettings.json")
           .AddJsonFile("appsettings.dev.json", optional: true);
        return builder.Build();
    }

    private IServiceProvider BuildServices()
    {
        var services = new ServiceCollection();
        services.AddSingleton(_configuration);
        services.AddSingleton<SecretClient>(s =>
           new(new Uri(_configuration["keyvaultUrl"]), _tokenCredential));
        services.AddTransient(page => Page);
        services.AddTransient<HomePage>();
        services.AddTransient<PlatformPopUp>();

        return services.BuildServiceProvider();
    }

    public IConfiguration Configuration => _configuration;
    public T GetService<T>() => _serviceProvider.GetService<T>() ?? throw new NullReferenceException("Do not forget to register the service.");
}
```

## Resources

- This [repo on GitHub](https://github.com/oscarvantol/playwright-nunit-di) hosts the example code to make this work. In the future I'll add some other implementation examples that are useful for us.
- [https://playwright.dev/](https://playwright.dev/dotnet/)
- [Betatalks #52 - Automated testing with Playwright, NUnit and Azure Pipelines. ](https://www.youtube.com/watch?v=buORqcx9GrI)


[oscarvantol.nl](https://oscarvantol.nl) 
