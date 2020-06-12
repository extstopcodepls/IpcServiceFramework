| CI build | Stable build |
|----------|--------------|
|[![Build Status](https://dev.azure.com/jacques-kang/IpcServiceFramework/_apis/build/status/IpcServiceFramework%20CI?branchName=develop)](https://dev.azure.com/jacques-kang/IpcServiceFramework/_build/latest?definitionId=9&branchName=develop)|[![Build Status](https://dev.azure.com/jacques-kang/IpcServiceFramework/_apis/build/status/IpcServiceFramework?branchName=master)](https://dev.azure.com/jacques-kang/IpcServiceFramework/_build/latest?definitionId=14&branchName=master)|

# IpcServiceFramework

A .NET Core 3.1 based lightweight framework for efficient inter-process communication.
Named pipeline and TCP support out-of-the-box, extensible with other protocols.

## NuGet packages
 - JKang.IpcServiceFramework.Core [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Core.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Core)
 - JKang.IpcServiceFramework.Client [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client)
 - JKang.IpcServiceFramework.Client.NamedPipe [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.NamedPipe.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.NamedPipe)
 - JKang.IpcServiceFramework.Client.Tcp [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.Tcp.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.Tcp)
 - JKang.IpcServiceFramework.Client.Tcp [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.Tcp.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Client.Tcp)
 - JKang.IpcServiceFramework.Hosting [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Hosting.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Hosting)
 - JKang.IpcServiceFramework.Hosting.NamedPipe [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Hosting.NamedPipe.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Hosting.NamedPipe)
 - JKang.IpcServiceFramework.Hosting.Tcp [![NuGet version](https://badge.fury.io/nu/JKang.IpcServiceFramework.Hosting.Tcp.svg)](https://badge.fury.io/nu/JKang.IpcServiceFramework.Hosting.Tcp)


## Usage

 1. Create an interface as service contract and package it in an assembly to be referenced by server and client applications, for example:

    ```csharp
    public interface IInterProcessService
    {
        string ReverseString(string input);
    }
    ```

 1. Implement the service in server application, for example:
 
    ```csharp
    class InterProcessService : IInterProcessService
    {
        public string ReverseString(string input)
        {
            char[] charArray = input.ToCharArray();
            Array.Reverse(input.ToCharArray());
            return new string(charArray);
        }
    }
    ```

 1. Install the following NuGet packages in server application:

    ```powershell
    > Install-Package Microsoft.Extensions.Hosting
    > Install-Package JKang.IpcServiceFramework.Hosting.NamedPipe
    ```

 1. Register the service implementation and configure IPC endpoint(s):

    ```csharp
    class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureServices(services =>
                {
                    services.AddScoped<IInterProcessService, InterProcessService>();
                })
                .ConfigureIpcHost(builder =>
                {
                    // configure IPC endpoints
                    builder.AddNamedPipeEndpoint<IInterProcessService>(pipeName: "my-pipe");
                })
                .ConfigureLogging(builder =>
                {
                    // optionally configure logging
                    builder.SetMinimumLevel(LogLevel.Information);
                });
    }
    ```

 1. Install the following NuGet package in client application:

    ```powershell
    > Install-Package JKang.IpcServiceFramework.Client.NamedPipe
    ```

 1. Invoke the server

    ```csharp
    // register IPC clients
    ServiceProvider serviceProvider = new ServiceCollection()
        .AddNamedPipeIpcClient<IInterProcessService>("client1", pipeName: "pipeinternal")
        .BuildServiceProvider();

    // resolve IPC client factory
    IIpcClientFactory<IInterProcessService> clientFactory = serviceProvider
        .GetRequiredService<IIpcClientFactory<IInterProcessService>>();

    // create client
    IIpcClient<IInterProcessService> client = clientFactory.CreateClient("client1");

    string output = await client.InvokeAsync(x => x.ReverseString(input));
    ```

## FAQs


