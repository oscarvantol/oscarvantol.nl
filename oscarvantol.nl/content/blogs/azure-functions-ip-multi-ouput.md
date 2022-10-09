---
date: 2021-04-14
title: Multiple output bindings in Azure Functions (isolated process model)
---

The new Azure Functions Isolated Process Model is a bit different than the one you might be used to. If you are not familiar with the new model you should first read this: [2021-03-27 Azure Functions in .NET 5 and beyond](https://dev.to/oscarvantol/azure-functions-in-net-5-and-beyond-26d6)

![alt text](https://oscarvantol.nl/assets/blog-af5/azure-functions-logo-color-raster.png "Azure Functions")

In the new isolated model the output binding for a simple http function is similar to the old class library model, the return value of your method **is** the http response. If you need additional output bindings things did change, a lot! You can no longer specify binding attributes on the arguments (like IAsyncCollector) of your function. If you want to use multiple output bindings for your function in the new model, this is also done through the return value of the function method. 

## Return all the output bindings
But how does this work? The idea is pretty straight forward, you create a class with properties for all the output values and decorate them with output binding attributes. Here is an example borrowed from the official [Samples](https://github.com/Azure/azure-functions-dotnet-worker/tree/main/samples).


```
    public static class MultiOutput
    {
        [Function("MultiOutput")]
        public static MyOutputType Run([HttpTrigger(AuthorizationLevel.Anonymous, "get")] HttpRequestData req,
            FunctionContext context)
        {
            var response = req.CreateResponse(HttpStatusCode.OK);
            response.WriteString("Success!");

            string myQueueOutput = "some output";

            return new MyOutputType()
            {
                Name = myQueueOutput,
                HttpResponse = response
            };
        }
    }

    public class MyOutputType
    {
        [QueueOutput("myQueue")]
        public string Name { get; set; }

        public HttpResponseData HttpResponse { get; set; }
    }
```
[source](https://github.com/Azure/azure-functions-dotnet-worker/blob/main/samples/Extensions/MultiOutput/MultiOutput.cs)

## Noisy

As you can see the function **MultiOutput** returns a type **MyOutputType** and the property **Name** is decorated with a **QueueOutputAttribute**. Although this works and was clear to me, I could not help but have the feeling this setup is a bit noisy. Because of that I started playing around with some of the new features of c# to see if I could create a cleaner solution. First thought I had was to use tuples as a return value but this is not a viable solution at this moment, also the result definition with the attributes would make it weird.

Next thought was RECORDS, yes the new record type that helps us get to the holy grail called immutability. Creating a record instead of a good old class has a lot of cool advantages but it does not make this problem go away. To make the record definition smaller we can use **positional records**, this means that in one line of code you define an immutable class with properties, backing fields and a constructor that expects all the values. This together with a **target-typed new expression** will do the trick, right? My next problem appeared, how to put the attribute on the property? After some Bingle magic I ended up in the [language spec](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/records#properties) which had the answer, you can prefix the attribute name with **property:** and it will generate the attribute on the property. 

## End result
My proof of concept looked like this:

```
    public class MultiOutputFunction
    {
        public record MultiOutputResult(HttpResponseData Response, [property: BlobOutput("multioutput/example.txt")] string BlobContent);

        [Function(nameof(MultiOutput))]
        public async Task<MultiOutputResult> MultiOutput([HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "MultiOutput")] HttpRequestData req)
        {
            var response = req.CreateResponse();
            response.WriteString("Hello world");

            return new(response, DateTime.UtcNow.ToString());
        }
    }
```
[source](https://github.com/oscarvantol/azure-functions-dotnet5-examples/blob/main/ExampleFunction/MultiOutputFunction.cs)

> Please be aware this is just little old me playing around with some code, it could be that the Azure Functions Team is thinking about different directions and might not recommend this approach. 


[oscarvantol.nl](https://oscarvantol.nl)
