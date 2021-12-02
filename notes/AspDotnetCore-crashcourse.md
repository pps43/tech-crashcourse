# ASP.NET CORE

![](../res/aspdotnetcore.png)

## hosting

### [web host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1)


### [generic host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-3.1)



## web (UI) app
### server rendered
- [Razor Pages](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0&tabs=visual-studio)
- [MVC](https://docs.microsoft.com/en-us/aspnet/core/mvc/overview?view=aspnetcore-6.0)
- [Blazor](https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-6.0) (Server)

### client rendered
- [Blazor](https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-6.0) (WebAssembly)


## web api

## realtime web app
- [SingalR](https://docs.microsoft.com/en-us/aspnet/core/signalr/introduction?view=aspnetcore-3.1)

SingalR uses `WebSockets` underneath whenever possible. Or it will fallback to `Server-Sent Events` or `Long Polling`.

More info on `Long Polling`, `WebSockets`, see [zhihu](https://zhuanlan.zhihu.com/p/23467317)

## proto
- [http/3](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/http3?view=aspnetcore-6.0)
- [grpc](https://docs.microsoft.com/en-us/aspnet/core/grpc/?view=aspnetcore-6.0)
- [WebSockets](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/websockets?view=aspnetcore-6.0)

`http/3` uses a new protocol called `QUIC`(built on UDP) than TCP in `http/2` or `http1.1`. It's still evolving. Also see [official blog: HTTP/3 support in .NET 6](https://devblogs.microsoft.com/dotnet/http-3-support-in-dotnet-6/)

`grpc` can work with .NET in 2 ways. Before .NET Core 3.0 (2019) released, `Grpc.Core` is the only way, calling grpc's [`C-Core`](https://grpc.io/blog/grpc-stacks/). Now the new way is [`grpc-dotnet`](https://grpc.io/blog/grpc-on-dotnetcore/), implement grpc in c#.

`WebSockets` is used in `SingalR` as first choice, but also you can use raw WebSockets.

## data format
- protobuf
- json
- [MessagePack](https://docs.microsoft.com/en-us/aspnet/core/signalr/messagepackhubprotocol?view=aspnetcore-6.0)

`MessagePack` is used in `SingalR` hubs, to communicate between clients and servers.

## database access
- EF core

## deploy
- [IIS](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/?view=aspnetcore-3.1)
- [Nginx](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-3.1)
- [Apache](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-apache?view=aspnetcore-3.1)
- [Docker](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/?view=aspnetcore-3.1)
- [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-6.0)
- Azure App Service

## security

## test tools