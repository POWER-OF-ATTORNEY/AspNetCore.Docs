---
title: ASP.NET Core Blazor environments
author: guardrex
description: Learn about environments in Blazor, including how to set the environment.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/12/2024
uid: blazor/fundamentals/environments
---
# ASP.NET Core Blazor environments

[!INCLUDE[](~/includes/not-latest-version.md)]

This article explains how to configure and read the [environment](xref:fundamentals/environments) in a Blazor app.

When running an app locally, the environment defaults to `Development`. When the app is published, the environment defaults to `Production`.

<!-- UPDATE 10.0 The underlying problem with app settings filename 
                 case sensitivity is tracked for 10.0 by ...
                 https://github.com/dotnet/aspnetcore/issues/25152 -->
                
We recommend the following conventions:

* Always use the "`Development`" environment name for local development. This is because the ASP.NET Core framework expects exactly that name when configuring the app and tooling for local development runs of an app.

* For testing, staging, and production environments, always publish and deploy the app. You can use any environment naming scheme that you wish for published apps, but always use app setting file names with casing of the environment segment that exactly matches the environment name. For staging, use "`Staging`" (capital ":::no-loc text="S":::") as the environment name, and name the app settings file to match (`appsettings.Staging.json`). For production, use "`Production`" (capital ":::no-loc text="P":::") as the environment name, and name the app settings file to match (`appsettings.Production.json`).

## Set the environment

The environment is set using any of the following approaches:

:::moniker range=">= aspnetcore-10.0"

* Blazor Web App or Blazor Server: Use any of the approaches described in <xref:fundamentals/environments> for general ASP.NET Core apps.
* Any Blazor app: [Blazor start configuration](#set-the-client-side-environment-via-blazor-startup-configuration)
* Standalone Blazor WebAssembly: `<WasmApplicationEnvironmentName>` property

On the client for a Blazor Web App, the environment is determined from the server via an HTML comment that developers don't interact with:

```html
<!--Blazor-WebAssembly:{"environmentName":"Development", ...}-->
```

For a standalone Blazor WebAssembly app, set the environment with the `<WasmApplicationEnvironmentName>` property in the app's project file (`.csproj`). The following example sets the `Staging` environment:

```xml
<WasmApplicationEnvironmentName>Staging</WasmApplicationEnvironmentName>
```

The default environments are `Development` for build and `Production` for publish.

:::moniker-end

:::moniker range=">= aspnetcore-8.0 < aspnetcore-10.0"

* Blazor Web App or Blazor Server: Use any of the approaches described in <xref:fundamentals/environments> for general ASP.NET Core apps.
* Any Blazor app:
  * [Blazor start configuration](#set-the-client-side-environment-via-blazor-startup-configuration)
  * [Azure App Service](#set-the-environment-for-azure-app-service)
* Blazor WebAssembly: [`Blazor-Environment` header](#set-the-client-side-environment-via-header)

On the client for a Blazor Web App, the environment is determined from the server via a middleware that communicates the environment to the browser via a header named `Blazor-Environment`. The header sets the environment when the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHost> is created in the client-side `Program` file (<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.CreateDefault%2A?displayProperty=nameWithType>).

For a standalone Blazor WebAssembly app running locally, the development server adds the `Blazor-Environment` header with the environment name obtained from the hosting environment. The hosting environment sets the environment from the `ASPNETCORE_ENVIRONMENT` environment variable established by the project's `Properties/launchSettings.json` file. The default value of the environment variable in a project created from the Blazor WebAssembly project template is `Development`. For more information, see the [Set the client-side environment via header](#set-the-client-side-environment-via-header) section.

:::moniker-end

:::moniker range="< aspnetcore-8.0"

* Blazor Server: Use any of the approaches described in <xref:fundamentals/environments> for general ASP.NET Core apps.
* Blazor Server or Blazor WebAssembly:
  * [Blazor start configuration](#set-the-client-side-environment-via-blazor-startup-configuration)
  * [Azure App Service](#set-the-environment-for-azure-app-service)
* Blazor WebAssembly: [`Blazor-Environment` header](#set-the-client-side-environment-via-header)

On the client of a hosted Blazor WebAssembly app, the environment is determined from the server via a middleware that communicates the environment to the browser via a header named `Blazor-Environment`. The header sets the environment when the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHost> is created in the client-side `Program` file (<xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.CreateDefault%2A?displayProperty=nameWithType>).

For a standalone Blazor WebAssembly app running locally, the development server adds the `Blazor-Environment` header with the environment name obtained from the hosting environment. The hosting environment sets the environment from the `ASPNETCORE_ENVIRONMENT` environment variable established by the project's `Properties/launchSettings.json` file. The default value of the environment variable in a project created from the Blazor WebAssembly project template is `Development`. For more information, see the [Set the client-side environment via header](#set-the-client-side-environment-via-header) section.

:::moniker-end

For app's running locally in development, the app defaults to the `Development` environment. Publishing the app defaults the environment to `Production`.

:::moniker range="< aspnetcore-5.0"

For general guidance on ASP.NET Core app configuration, see <xref:fundamentals/environments>. For server-side app configuration with static files in environments other than the <xref:Microsoft.Extensions.Hosting.Environments.Development> environment during development and testing (for example, <xref:Microsoft.Extensions.Hosting.Environments.Staging>), see <xref:blazor/fundamentals/static-files#static-files-in-non-development-environments>.

:::moniker-end

## Set the client-side environment via Blazor startup configuration

The following example starts Blazor in the `Staging` environment if the hostname includes `localhost`. Otherwise, the environment is set to its default value.

:::moniker range=">= aspnetcore-8.0 < aspnetcore-10.0"

Blazor Web App:

:::moniker-end

:::moniker range=">= aspnetcore-8.0"

```html
<script src="{BLAZOR SCRIPT}" autostart="false"></script>
<script>
  if (window.location.hostname.includes("localhost")) {
    Blazor.start({
      webAssembly: {
        environment: "Staging"
      }
    });
  } else {
    Blazor.start();
  }
</script>
```

**In the preceding example, the `{BLAZOR SCRIPT}` placeholder is the Blazor script path and file name.** For the location of the script, see <xref:blazor/project-structure#location-of-the-blazor-script>.

> [!NOTE]
> For Blazor Web Apps that set the `webAssembly` > `environment` property in `Blazor.start` configuration, it's wise to match the server-side environment to the environment set on the `environment` property. Otherwise, prerendering on the server will operate under a different environment than rendering on the client, which results in arbitrary effects. For general guidance on setting the environment for a Blazor Web App, see <xref:fundamentals/environments>.

:::moniker-end

:::moniker range=">= aspnetcore-8.0 < aspnetcore-10.0"

Standalone Blazor WebAssembly:

:::moniker-end

:::moniker range="< aspnetcore-10.0"

```html
<script src="{BLAZOR SCRIPT}" autostart="false"></script>
<script>
  if (window.location.hostname.includes("localhost")) {
    Blazor.start({
      environment: "Staging"
    });
  } else {
    Blazor.start();
  }
</script>
```

:::moniker-end

**In the preceding example, the `{BLAZOR SCRIPT}` placeholder is the Blazor script path and file name.** For the location of the script, see <xref:blazor/project-structure#location-of-the-blazor-script>.

:::moniker range="< aspnetcore-10.0"

Using the `environment` property overrides the environment set by the [`Blazor-Environment` header](#set-the-client-side-environment-via-header).

The preceding approach sets the client's environment without changing the `Blazor-Environment` header's value, nor does it change the server project's console logging of the startup environment for a Blazor Web App that has adopted global Interactive WebAssembly rendering.

:::moniker-end

To log the environment to the console in either a standalone Blazor WebAssembly app or the `.Client` project of a Blazor Web App, place the following C# code in the `Program` file after the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHost> is created with <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.CreateDefault%2A?displayProperty=nameWithType> and before the line that builds and runs the project (`await builder.Build().RunAsync();`):

```csharp
Console.WriteLine(
    $"Client Hosting Environment: {builder.HostEnvironment.Environment}");
```

For more information on Blazor startup, see <xref:blazor/fundamentals/startup>.

:::moniker range="< aspnetcore-10.0"

## Set the client-side environment via header

Blazor WebAssembly apps can set the environment with the `Blazor-Environment` header. Specifically, the response header must be set on the `_framework/blazor.boot.json` file, but there's no harm setting the header on file server responses for other Blazor file requests or the entire Blazor deployment.

Although the Blazor framework issues the header name in kebab case with mixed letter case (`Blazor-Environment`), you're welcome to use all-lower or all-upper kebab case (`blazor-environment`, `BLAZOR-ENVIRONMENT`).

For local development runs with Blazor's built-in development server, you can control the value of the `Blazor-Environment` header by setting the value of the `ASPNETCORE_ENVIRONMENT` environment variable in the project's `Properties/launchSettings.json` file. When running locally with the development server, the order of precedence for determining the app's environment is [`Blazor.start` configuration (`environment` key)](#set-the-client-side-environment-via-blazor-startup-configuration) > `Blazor-Environment` response header (`blazor.boot.json` file) > `ASPNETCORE_ENVIRONMENT` environment variable (`launchSettings.json`). You can't use the `ASPNETCORE_ENVIRONMENT` environment variable (`launchSettings.json`) approach for a deployed Blazor WebAssembly app. The technique only works with the development server on local runs of the app.

### IIS

In the following example for IIS, the custom header (`Blazor-Environment`) is added to the published `web.config` file. The `web.config` file is located in the `bin/Release/{TARGET FRAMEWORK}/publish` folder, where the `{TARGET FRAMEWORK}` placeholder is the target framework:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    ...
    <httpProtocol>
      <customHeaders>
        <add name="Blazor-Environment" value="Staging" />
      </customHeaders>
    </httpProtocol>
  </system.webServer>
</configuration>
```

> [!NOTE]
> To use a custom `web.config` file for IIS that isn't overwritten when the app is published to the `publish` folder, see <xref:blazor/host-and-deploy/webassembly/iis#use-of-a-custom-webconfig>.

### Nginx

For Nginx servers, use the `add_header` directive from the `ngx_http_headers_module`:

```
http {
    server {
        ...
        location / {
            ...
            add_header Blazor-Environment "Staging";
        }
    }
}
```

For more information, see the following resources:

* [Nginx documentation](http://nginx.org/docs/http/ngx_http_headers_module.html)
* <xref:blazor/host-and-deploy/webassembly/nginx>

### Apache

For Apache servers, use the `Header` directive from the `mod_headers` module:

```
<VirtualHost *:80>
    ...
    Header set Blazor-Environment "Staging"
    ...
</VirtualHost>
```

For more information, see the following resources:

* [Apache documentation (search the latest release for "`mod_headers`")](https://httpd.apache.org/docs/)
* <xref:blazor/host-and-deploy/webassembly/apache>

### Set the environment for Azure App Service

<!-- UPDATE 10.0 The underlying problem with app settings filename 
                 case sensitivity is tracked for 10.0 by ...
                 https://github.com/dotnet/aspnetcore/issues/25152 -->

For a standalone Blazor WebAssembly app, you can set the environment manually via [start configuration](#set-the-client-side-environment-via-blazor-startup-configuration) or the [`Blazor-Environment` header](#set-the-client-side-environment-via-header).

For a server-side app, set the environment via an `ASPNETCORE_ENVIRONMENT` app setting in Azure:

1. ***Confirm that the casing of environment segments in app settings file names match their environment name casing exactly***. For example, the matching app settings file name for the `Staging` environment is `appsettings.Staging.json`. If the file name is `appsettings.staging.json` (lowercase "`s`"), the file isn't located, and the settings in the file aren't used in the `Staging` environment.

1. For Visual Studio deployment, confirm that the app is deployed to the correct deployment slot. For an app named `BlazorAzureAppSample`, the app is deployed to the `Staging` deployment slot.

1. In the Azure portal for the environment's deployment slot, set the environment with the `ASPNETCORE_ENVIRONMENT` app setting. For an app named `BlazorAzureAppSample`, the staging App Service Slot is named `BlazorAzureAppSample/Staging`. For the `Staging` slot's configuration, create an app setting for `ASPNETCORE_ENVIRONMENT` with a value of `Staging`. **Deployment slot setting** is enabled for the setting.

When requested in a browser, the `BlazorAzureAppSample/Staging` app loads in the `Staging` environment at `https://blazorazureappsample-staging.azurewebsites.net`.

When the app is loaded in the browser, the response header collection for `blazor.boot.json` indicates that the `Blazor-Environment` header value is `Staging`.

App settings from the `appsettings.{ENVIRONMENT}.json` file are loaded by the app, where the `{ENVIRONMENT}` placeholder is the app's environment. In the preceding example, settings from the `appsettings.Staging.json` file are loaded.

:::moniker-end

## Read the environment in a Blazor WebAssembly app

Obtain the app's environment in a component by injecting <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> and reading the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.Environment> property.

`ReadEnvironment.razor`:

```razor
@page "/read-environment"
@using Microsoft.AspNetCore.Components.WebAssembly.Hosting
@inject IWebAssemblyHostEnvironment Env

<h1>Environment example</h1>

<p>Environment: @Env.Environment</p>
```

:::moniker range=">= aspnetcore-8.0"

## Read the environment client-side in a Blazor Web App

Assuming that prerendering isn't disabled for a component or the app, a component in the `.Client` project is prerendered on the server. Because the server doesn't have a registered <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> service, it isn't possible to inject the service and use the service implementation's host environment extension methods and properties during server prerendering. Injecting the service into an Interactive WebAssembly or Interactive Auto component results in the following runtime error:

> :::no-loc text="There is no registered service of type 'Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment'.":::

To address this, create a custom service implementation for <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> on the server. Add the following class to the server project.

`ServerHostEnvironment.cs`:

```csharp
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
using Microsoft.AspNetCore.Components;

public class ServerHostEnvironment(IWebHostEnvironment env, NavigationManager nav) : 
    IWebAssemblyHostEnvironment
{
    public string Environment => env.EnvironmentName;
    public string BaseAddress => nav.BaseUri;
}
```

In the server project's `Program` file, register the service:

```csharp
builder.Services.TryAddScoped<IWebAssemblyHostEnvironment, ServerHostEnvironment>();
```

At this point, the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> service can be injected into an interactive WebAssembly or interactive Auto component and used as shown in the [Read the environment in a Blazor WebAssembly app](#read-the-environment-in-a-blazor-webassembly-app) section.

The preceding example can demonstrate that it's possible to have a different server environment than client environment, which isn't recommended and may lead to arbitrary results. When setting the environment in a Blazor Web App, it's best to match server and `.Client` project environments. Consider the following scenario in a test app:

* Implement the client-side (`webassembly`) environment property with the `Staging` environment via `Blazor.start`. See the [Set the client-side environment via startup configuration](#set-the-client-side-environment-via-blazor-startup-configuration) section for an example.
* Don't change the server-side `Properties/launchSettings.json` file. Leave the `environmentVariables` section with the `ASPNETCORE_ENVIRONMENT` environment variable set to `Development`.

You can see the value of the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.Environment?displayProperty=nameWithType> property change in the UI.

When prerendering occurs on the server, the component is rendered in the `Development` environment:

> **:::no-loc text="Environment:":::** :::no-loc text="Development":::

When the component is rerendered just a second or two later, after the Blazor bundle is downloaded and the .NET WebAssembly runtime activates, the values change to reflect that the client is operating in the `Staging` environment on the client:

> **:::no-loc text="Environment:":::** :::no-loc text="Staging":::

The preceding example demonstrates why we recommend setting the server environment to match the client environment for development, testing, and production deployments.

For more information, see the [Client-side services fail to resolve during prerendering](xref:blazor/components/render-modes#client-side-services-fail-to-resolve-during-prerendering) section of the *Render modes* article, which appears later in the Blazor documentation.

:::moniker-end

## Read the client-side environment during startup

During startup, the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder> exposes the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment> through the <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostBuilder.HostEnvironment> property, which enables environment-specific logic in host builder code.

In the `Program` file:

```csharp
if (builder.HostEnvironment.Environment == "Custom")
{
    ...
};
```

The following convenience extension methods provided through <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions> permit checking the current environment for `Development`, `Production`, `Staging`, and custom environment names:

* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsDevelopment%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsProduction%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsStaging%2A>
* <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.WebAssemblyHostEnvironmentExtensions.IsEnvironment%2A>

In the `Program` file:

```csharp
if (builder.HostEnvironment.IsStaging())
{
    ...
};

if (builder.HostEnvironment.IsEnvironment("Custom"))
{
    ...
};
```

The <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> property can be used during startup when the <xref:Microsoft.AspNetCore.Components.NavigationManager> service isn't available.

## Additional resources

* <xref:blazor/fundamentals/startup>
* <xref:fundamentals/environments>
* [Blazor samples GitHub repository (`dotnet/blazor-samples`)](https://github.com/dotnet/blazor-samples) ([how to download](xref:blazor/fundamentals/index#sample-apps))
