---
layout: post
title: ".NET 7 containers"
description: ""
category: 
tags: dotnet, docker, webapi, api, csharp, windows, microsoft, deployment
---

# Built-in .NET SDK container support

On August 25th 2022, Microsoft [announced built-in container support](https://devblogs.microsoft.com/dotnet/announcing-builtin-container-support-for-the-dotnet-sdk/) added to the latest 
.NET 7 preview. It's incredibly simple. You just add a package reference and publish with a pre-made publish profile, and suddenly you have a Docker container. I wanted to test this out 
and play with both Docker Compose, since I hadn't used that yet.

My development machine is running Windows 11, Docker Desktop configured for Linux containers. Development was done with .NET 7.0.100-preview.7.22377.5.

# Creating the API

The `dotnet new` tool will create a brand new WebApi project if you aren't working with an existing one. In my case, since I wanted to play with Docker Compose, I needed to build something 
that would rely on another service that I run in a different container. It also needed to be simple enough to explain in a blog post. I settled on a simple cache API, providing an HTTP 
frontend to Redis.

```
PS C:\dev> dotnet new webapi -n SimpleCacheApi
The template "ASP.NET Core Web API" was created successfully.

Processing post-creation actions...
Restoring C:\dev\SimpleCacheApi\SimpleCacheApi.csproj:
  Determining projects to restore...
  Restored C:\dev\SimpleCacheApi\SimpleCacheApi.csproj (in 161 ms).
Restore succeeded.


PS C:\dev> cd .\SimpleCacheApi\
PS C:\dev\SimpleCacheApi>
```

The container support is provided by the [Microsoft.NET.Build.Containers](https://www.nuget.org/packages/Microsoft.NET.Build.Containers) package. Add that to your project through whatever 
method you prefer. I simply added it from the `dotnet` CLI.

```
PS C:\dev\SimpleCacheApi> dotnet add package Microsoft.NET.Build.Containers
  Determining projects to restore...
  Writing C:\Users\ross\AppData\Local\Temp\tmpAF9C.tmp
info : Installed Microsoft.NET.Build.Containers 0.1.8 from https://api.nuget.org/v3/index.json with content hash xhWyycoiI+AtKX6gbIb/XvNSajOr3+CKcYTSklTsNcMwXCNsWWMEC9+1si+0/mLM5GT+RFmaIokmdJDltYA9Ew==.
info : Package 'Microsoft.NET.Build.Containers' is compatible with all the specified frameworks in project 'C:\dev\SimpleCacheApi\SimpleCacheApi.csproj'.
info : PackageReference for package 'Microsoft.NET.Build.Containers' version '0.1.8' added to file 'C:\dev\SimpleCacheApi\SimpleCacheApi.csproj'.
info : Generating MSBuild file C:\dev\SimpleCacheApi\obj\SimpleCacheApi.csproj.nuget.g.props.
info : Generating MSBuild file C:\dev\SimpleCacheApi\obj\SimpleCacheApi.csproj.nuget.g.targets.
info : Writing assets file to disk. Path: C:\dev\SimpleCacheApi\obj\project.assets.json
log  : Restored C:\dev\SimpleCacheApi\SimpleCacheApi.csproj (in 389 ms).
PS C:\dev\SimpleCacheApi>
```

Because I am going to rely on Redis as my backend cache, I also needed to install [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis).

```
PS C:\dev\SimpleCacheApi> dotnet add package StackExchange.Redis
  Determining projects to restore...
  Writing C:\Users\ross\AppData\Local\Temp\tmpD3C1.tmp
info : Installed Microsoft.NETCore.Platforms 5.0.0 from https://api.nuget.org/v3/index.json with content hash VyPlqzH2wavqquTcYpkIIAQ6WdenuKoFN0BdYBbCWsclXacSOHNQn66Gt4z5NBqEYW0FAPm5rlvki9ZiCij5xQ==.
info : Installed System.Windows.Extensions 5.0.0 from https://api.nuget.org/v3/index.json with content hash c1ho9WU9ZxMZawML+ssPKZfdnrg/OjR3pe0m9v8230z3acqphwvPJqzAkH54xRYm5ntZHGG1EPP3sux9H3qSPg==.
info : Installed Pipelines.Sockets.Unofficial 2.2.2 from https://api.nuget.org/v3/index.json with content hash Bhk0FWxH1paI+18zr1g5cTL+ebeuDcBCR+rRFO+fKEhretgjs7MF2Mc1P64FGLecWp4zKCUOPzngBNrqVyY7Zg==.
info : Installed System.IO.Pipelines 5.0.1 from https://api.nuget.org/v3/index.json with content hash qEePWsaq9LoEEIqhbGe6D5J8c9IqQOUuTzzV6wn1POlfdLkJliZY3OlB0j0f17uMWlqZYjH7txj+2YbyrIA8Yg==.
info : Installed System.Configuration.ConfigurationManager 5.0.0 from https://api.nuget.org/v3/index.json with content hash aM7cbfEfVNlEEOj3DsZP+2g9NRwbkyiAv2isQEzw7pnkDg9ekCU2m1cdJLM02Uq691OaCS91tooaxcEn8d0q5w==.
info : Installed System.Diagnostics.PerformanceCounter 5.0.0 from https://api.nuget.org/v3/index.json with content hash kcQWWtGVC3MWMNXdMDWfrmIlFZZ2OdoeT6pSNVRtk9+Sa7jwdPiMlNwb0ZQcS7NRlT92pCfmjRtkSWUW3RAKwg==.
info : Installed Microsoft.Win32.SystemEvents 5.0.0 from https://api.nuget.org/v3/index.json with content hash Bh6blKG8VAKvXiLe2L+sEsn62nc1Ij34MrNxepD2OCrS5cpCwQa9MeLyhVQPQ/R4Wlzwuy6wMK8hLb11QPDRsQ==.
info : Installed System.Security.Permissions 5.0.0 from https://api.nuget.org/v3/index.json with content hash uE8juAhEkp7KDBCdjDIE3H9R1HJuEHqeqX8nLX9gmYKWwsqk3T5qZlPx8qle5DPKimC/Fy3AFTdV7HamgCh9qQ==.
info : Installed StackExchange.Redis 2.6.48 from https://api.nuget.org/v3/index.json with content hash T0rLGogyT6Zny+IMrDx1Z8r4nA3B0C7EVo5SHNjzT4ndOn9aGKe5K7KTVx0y41WaWmfSWpaX7HrPl0tfZ4zuUw==.
info : Installed System.Security.Cryptography.ProtectedData 5.0.0 from https://api.nuget.org/v3/index.json with content hash HGxMSAFAPLNoxBvSfW08vHde0F9uh7BjASwu6JF9JnXuEPhCY3YUqURn0+bQV/4UWeaqymmrHWV+Aw9riQCtCA==.
info : Installed System.Security.AccessControl 5.0.0 from https://api.nuget.org/v3/index.json with content hash dagJ1mHZO3Ani8GH0PHpPEe/oYO+rVdbQjvjJkBRNQkX4t0r1iaeGn8+/ybkSLEan3/slM0t59SVdHzuHf2jmw==.
info : Installed Microsoft.Win32.Registry 5.0.0 from https://api.nuget.org/v3/index.json with content hash dDoKi0PnDz31yAyETfRntsLArTlVAVzUzCIvvEDsDsucrl33Dl8pIJG06ePTJTI3tGpeyHS9Cq7Foc/s4EeKcg==.
info : Installed System.Drawing.Common 5.0.0 from https://api.nuget.org/v3/index.json with content hash SztFwAnpfKC8+sEKXAFxCBWhKQaEd97EiOL7oZJZP56zbqnLpmxACWA8aGseaUExciuEAUuR9dY8f7HkTRAdnw==.
info : Installed System.Security.Principal.Windows 5.0.0 from https://api.nuget.org/v3/index.json with content hash t0MGLukB5WAVU9bO3MGzvlGnyJPgUlcwerXn1kzBRjwLKixT96XV0Uza41W49gVd8zEMFu9vQEFlv0IOrytICA==.
info : Package 'StackExchange.Redis' is compatible with all the specified frameworks in project 'C:\dev\SimpleCacheApi\SimpleCacheApi.csproj'.
info : PackageReference for package 'StackExchange.Redis' version '2.6.48' added to file 'C:\dev\SimpleCacheApi\SimpleCacheApi.csproj'.
info : Writing assets file to disk. Path: C:\dev\SimpleCacheApi\obj\project.assets.json
log  : Restored C:\dev\SimpleCacheApi\SimpleCacheApi.csproj (in 2.13 sec).
PS C:\dev\SimpleCacheApi>
```

## Coding the API

The API itself is very minimal since I only built it to test out the container support, so there wasn't much I had to do. After removing all of the default WeatherForecast stuff, I only had 
two files that needed to be updated.

First, the Redis connection needs to be made available to the `IServiceCollection` by adding the following lines into `Program.cs`:

```csharp
using StackExchange.Redis;

// Add services to the container.
var mux = ConnectionMultiplexer.Connect("redis");
builder.Services.AddSingleton<IConnectionMultiplexer>(mux);
```

I also commented out the call to `UseHttpsRedirection()`.

Wanting this API to be as simple as possible, I also didn't want to deal with JSON. To make that work, I needed to [make a controller action receive text/plain content as a string](https://peterdaugaardrasmussen.com/2020/02/29/asp-net-core-how-to-make-a-controller-endpoint-that-accepts-text-plain/). See [TextPlainInputFormatter.cs](https://github.com/rnelson/SimpleCacheApi/blob/main/TextPlainInputFormatter.cs) and [Program.cs:10](https://github.com/rnelson/SimpleCacheApi/blob/d88db189fbe4ffeecac4d6c9b99e9e41ee66fc10/Program.cs#L10).

Next, `Controllers/CacheController.cs` was added to be the interface between the user and Redis.

```csharp
using System.Diagnostics.CodeAnalysis;
using Microsoft.AspNetCore.Mvc;
using StackExchange.Redis;

namespace SimpleCacheApi.Controllers;

[ApiController]
[Route("[controller]")]
public class CacheController : ControllerBase
{
    private readonly IConnectionMultiplexer _redis;

    public CacheController(IConnectionMultiplexer redis)
    {
        _redis = redis;
    }

    [HttpGet("read/{key}")]
    public async Task<IActionResult> ReadCache(string key)
    {
        var db = _redis.GetDatabase();
        var exists = await db.KeyExistsAsync(key);
        if (!exists) return NotFound();
        return Ok(await db.StringGetAsync(key).ToString());
    }

    [HttpPost("write/{key}")]
    public async Task<IActionResult> WriteCache(string key, [FromBody] string value)
    {
        var db = _redis.GetDatabase();
        var exists = await db.KeyExistsAsync(key);
        if (exists) return Conflict($"{key} already exists; use PUT");
        return Ok(await db.StringSetAsync(key, value));
    }

    [HttpPut("update/{key}")]
    public async Task<IActionResult> UpdateCache(string key, [FromBody] string value)
    {
        var db = _redis.GetDatabase();
        var exists = await db.KeyExistsAsync(key);
        if (!exists) return NotFound();
        return Ok(await db.StringSetAsync(key, value));
    }

    [HttpDelete("delete/{key}")]
    public async Task<IActionResult> DeleteCache(string key)
    {
        var db = _redis.GetDatabase();
        var exists = await db.KeyExistsAsync(key);
        if (!exists) return NotFound();
        return Ok(await db.KeyDeleteAsync(key));
    }
}

```

# Containers

## Publishing the API

Assuming you have a Docker daemon already running on your machine, you can create your container by simply doing a `dotnet publish` that uses the `DefaultContainer` publish profile. The initial 
release of the package only supports Linux on amd64 for the containers, so you also need to specify the OS and architecture in your publish command.

```
C:\dev\SimpleCacheApi>dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer
MSBuild version 17.4.0-preview-22368-02+c8492483a for .NET
  Determining projects to restore...
  All projects are up-to-date for restore.
C:\Program Files\dotnet\sdk\7.0.100-preview.7.22377.5\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.RuntimeIdentifierInf
erence.targets(219,5): message NETSDK1057: You are using a preview version of .NET. See: https://aka.ms/dotnet-support-
policy [C:\dev\SimpleCacheApi\SimpleCacheApi.csproj]
  SimpleCacheApi -> C:\dev\SimpleCacheApi\bin\Debug\net7.0\linux-x64\SimpleCacheApi.dll
  SimpleCacheApi -> C:\dev\SimpleCacheApi\bin\Debug\net7.0\linux-x64\publish\
C:\Users\ross\.nuget\packages\microsoft.net.build.containers\0.1.8\build\Microsoft.NET.Build.Containers.targets(45,9):
warning CONTAINER001: ContainerImageName was not a valid container image name, it was normalized to simplecacheapi [C:\
dev\SimpleCacheApi\SimpleCacheApi.csproj]
  Pushed container 'simplecacheapi:1.0.0' to registry 'docker://'

C:\dev\SimpleCacheApi>
```

### Configuring Compose

The `docker-compose.yml` file for this project is very simple. It adds two services, the API (with a name matching the container name that `dotnet publish` used) that exposes port 5010 on the host 
to 80 on the container and a Redis container set up with all of its defaults.

```yaml
version: "3"
services:
  web:
    image: "simplecacheapi:1.0.0"
    ports:
      - "5010:80"
  redis:
    image: "redis:alpine"
```

### Putting it all together

To start both containers, simply run `docker compose up`.

```
C:\dev\SimpleCacheApi>set DOCKER_BUILDKIT=0
C:\dev\SimpleCacheApi>set COMPOSE_DOCKER_CLI_BUILD=0
C:\dev\SimpleCacheApi>docker compose up
[+] Running 2/0
 - Container simplecacheapi-redis-1  Created                                                                       0.0s
 - Container simplecacheapi-web-1    Recreated                                                                     0.0s
Attaching to simplecacheapi-redis-1, simplecacheapi-web-1
simplecacheapi-redis-1  | 1:C 27 Aug 2022 18:47:48.784 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
simplecacheapi-redis-1  | 1:C 27 Aug 2022 18:47:48.784 # Redis version=7.0.4, bits=64, commit=00000000, modified=0, pid=1, just started
simplecacheapi-redis-1  | 1:C 27 Aug 2022 18:47:48.784 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.784 * monotonic clock: POSIX clock_gettime
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.784 * Running mode=standalone, port=6379.
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.784 # Server initialized
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.785 * Loading RDB produced by version 7.0.4
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.785 * RDB age 17 seconds
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.785 * RDB memory usage when created 1.08 Mb
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.785 * Done loading RDB, keys loaded: 0, keys expired: 0.
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.785 * DB loaded from disk: 0.000 seconds
simplecacheapi-redis-1  | 1:M 27 Aug 2022 18:47:48.785 * Ready to accept connections
simplecacheapi-web-1    | info: Microsoft.Hosting.Lifetime[14]
simplecacheapi-web-1    |       Now listening on: http://[::]:80
simplecacheapi-web-1    | info: Microsoft.Hosting.Lifetime[0]
simplecacheapi-web-1    |       Application started. Press Ctrl+C to shut down.
simplecacheapi-web-1    | info: Microsoft.Hosting.Lifetime[0]
simplecacheapi-web-1    |       Hosting environment: Production
simplecacheapi-web-1    | info: Microsoft.Hosting.Lifetime[0]
simplecacheapi-web-1    |       Content root path: /app
```

### Testing the API

Now that we have Redis running (I don't have it installed on my desktop), we can test out the API. Because I use [JetBrains' Rider](https://www.jetbrains.com/rider/) as my IDE and have the 
[HTTP Client plugin](https://www.jetbrains.com/help/rider/2022.2/Http_client_in__product__code_editor.html) installed, I was able to make a single plaintext file that tests the API out for me:

```
### Try to get the fruit before we've done anything
GET http://localhost:5010/cache/read/fruit
Accept: text/plain

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 404, "Response status is not 404");
    });
%}

### Add a fruit
POST http://localhost:5010/cache/write/fruit
Content-Type: text/plain

apple

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 200, "Response status is not 200");
    });
%}

### Get that fruit; we should see "apple"
GET http://localhost:5010/cache/read/fruit
Accept: text/plain

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.body === "apple", "Response is not 'apple'");
    });
%}

### Update the fruit
PUT http://localhost:5010/cache/update/fruit
Accept: text/plain
Content-Type: text/plain

banana

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 200, "Response status is not 200");
    });
%}

### Get that fruit; we should see "banana" this time
GET http://localhost:5010/cache/read/fruit
Accept: text/plain

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.body === "banana", "Response is not 'banana'");
    });
%}

### All done, delete it
DELETE http://localhost:5010/cache/delete/fruit
Accept: text/plain

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 200, "Response status is not 200");
    });
%}

### Try to get the fruit again. It should be a 404 now
GET http://localhost:5010/cache/read/fruit
Accept: text/plain

> {%
    client.test("Request executed successfully", function() {
        client.assert(response.status === 404, "Response status is not 404");
    });
%}

###
```

