---
title: ASP.NET Core Blazor static files
author: guardrex
description: Learn how to configure and manage static files for Blazor apps.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/12/2024
uid: blazor/fundamentals/static-files
---
# ASP.NET Core Blazor static files

[!INCLUDE[](~/includes/not-latest-version.md)]

This article describes Blazor app configuration for serving static files.

:::moniker range=">= aspnetcore-9.0"

For general information on serving static files with Map Static Assets routing endpoint conventions, see <xref:fundamentals/map-static-files> before reading this article.

:::moniker-end

:::moniker range=">= aspnetcore-10.0"

## Preloaded Blazor framework static assets

In Blazor Web Apps, framework static assets are automatically preloaded using [`Link` headers](https://developer.mozilla.org/docs/Web/HTTP/Reference/Headers/Link), which allows the browser to preload resources before the initial page is fetched and rendered.

In standalone Blazor WebAssembly apps, framework assets are scheduled for high priority downloading and caching early in browser `index.html` page processing when:

* The `OverrideHtmlAssetPlaceholders` MSBuild property in the app's project file (`.csproj`) is set to `true`:

  ```xml
  <PropertyGroup>
    <OverrideHtmlAssetPlaceholders>true</OverrideHtmlAssetPlaceholders>
  </PropertyGroup>
  ```

* The following `<link>` element containing [`rel="preload"`](https://developer.mozilla.org/docs/Web/HTML/Reference/Attributes/rel/preload) is present in the `<head>` content of `wwwroot/index.html`:

  ```html
  <link rel="preload" id="webassembly" />
  ```

:::moniker-end

## Static asset delivery in server-side Blazor apps

:::moniker range=">= aspnetcore-9.0"

Serving static assets is managed by either routing endpoint conventions or a middleware described in the following table.

Feature | API | .NET Version | Description
--- | --- | :---: | ---
Map Static Assets routing endpoint conventions | <xref:Microsoft.AspNetCore.Builder.StaticAssetsEndpointRouteBuilderExtensions.MapStaticAssets%2A> | .NET 9 or later | Optimizes the delivery of static assets to clients.
Static Files Middleware | <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> | All .NET versions | Serves static assets to clients without the optimizations of Map Static Assets but useful for some tasks that Map Static Assets isn't capable of managing.

Map Static Assets can replace <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> in most situations. However, Map Static Assets is optimized for serving the assets from known locations in the app at build and publish time. If the app serves assets from other locations, such as disk or embedded resources, <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> should be used.

Map Static Assets (<xref:Microsoft.AspNetCore.Builder.StaticAssetsEndpointRouteBuilderExtensions.MapStaticAssets%2A>) also replaces calling <xref:Microsoft.AspNetCore.Builder.ComponentsWebAssemblyApplicationBuilderExtensions.UseBlazorFrameworkFiles%2A> in apps that serve Blazor WebAssembly framework files, and explicitly calling <xref:Microsoft.AspNetCore.Builder.ComponentsWebAssemblyApplicationBuilderExtensions.UseBlazorFrameworkFiles%2A> in a Blazor Web App isn't necessary because the API is automatically called when invoking <xref:Microsoft.Extensions.DependencyInjection.WebAssemblyRazorComponentsBuilderExtensions.AddInteractiveWebAssemblyComponents%2A>.

When [Interactive WebAssembly or Interactive Auto render modes](xref:blazor/fundamentals/index#render-modes) are enabled:

* Blazor creates an endpoint to expose the resource collection as a JS module.
* The URL is emitted to the body of the request as persisted component state when a WebAssembly component is rendered into the page.
* During WebAssembly boot, Blazor retrieves the URL, imports the module, and calls a function to retrieve the asset collection and reconstruct it in memory. The URL is specific to the content and cached forever, so this overhead cost is only paid once per user until the app is updated.
* The resource collection is also exposed at a human-readable URL (`_framework/resource-collection.js`), so JS has access to the resource collection for [enhanced navigation](xref:blazor/fundamentals/routing#enhanced-navigation-and-form-handling) or to implement features of other frameworks and third-party components.

Static File Middleware (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) is useful in the following situations that Map Static Assets (<xref:Microsoft.AspNetCore.Builder.StaticAssetsEndpointRouteBuilderExtensions.MapStaticAssets%2A>) can't handle:

* Serving files from disk that aren't part of the build or publish process, for example, files added to the application folder during or after deployment.
* Applying a path prefix to Blazor WebAssembly static asset files, which is covered in the [Prefix for Blazor WebAssembly assets](#prefix-for-blazor-webassembly-assets) section.
* Configuring file mappings of extensions to specific content types and setting static file options, which is covered in the [File mappings and static file options](#file-mappings-and-static-file-options) section.
      
For more information, see <xref:fundamentals/static-files>.

## Deliver assets with Map Static Assets routing endpoint conventions

*This section applies to server-side Blazor apps.*

<!-- UPDATE 10.0 Compiler implementation for tilde/slash-based HREFs. -->

Assets are delivered via the <xref:Microsoft.AspNetCore.Components.ComponentBase.Assets?displayProperty=nameWithType> property, which resolves the fingerprinted URL for a given asset. In the following example, Bootstrap, the Blazor project template app stylesheet (`app.css`), and the [CSS isolation stylesheet](xref:blazor/components/css-isolation) (based on an app's namespace of `BlazorSample`) are linked in a root component, typically the `App` component (`Components/App.razor`):

```razor
<link rel="stylesheet" href="@Assets["bootstrap/bootstrap.min.css"]" />
<link rel="stylesheet" href="@Assets["app.css"]" />
<link rel="stylesheet" href="@Assets["BlazorSample.styles.css"]" />
```

## `ImportMap` component

*This section applies to Blazor Web Apps that call <xref:Microsoft.AspNetCore.Builder.RazorComponentsEndpointRouteBuilderExtensions.MapRazorComponents%2A>.*

The `ImportMap` component (<xref:Microsoft.AspNetCore.Components.ImportMap>) represents an import map element (`<script type="importmap"></script>`) that defines the import map for module scripts. The Import Map component is placed in `<head>` content of the root component, typically the `App` component (`Components/App.razor`).

```razor
<ImportMap />
```

If a custom <xref:Microsoft.AspNetCore.Components.ImportMapDefinition> isn't assigned to an Import Map component, the import map is generated based on the app's assets.

> [!NOTE]
> <xref:Microsoft.AspNetCore.Components.ImportMapDefinition> instances are expensive to create, so we recommended caching them when creating an additional instance.

The following examples demonstrate custom import map definitions and the import maps that they create.

Basic import map:

```csharp
new ImportMapDefinition(
    new Dictionary<string, string>
    {
        { "jquery", "https://cdn.example.com/jquery.js" },
    },
    null,
    null);
```

The preceding code results in the following import map:

```json
{
  "imports": {
    "jquery": "https://cdn.example.com/jquery.js"
  }
}
```

Scoped import map:

```csharp
new ImportMapDefinition(
    null,
    new Dictionary<string, IReadOnlyDictionary<string, string>>
    {
        ["/scoped/"] = new Dictionary<string, string>
        {
            { "jquery", "https://cdn.example.com/jquery.js" },
        }
    },
    null);
```

The preceding code results in the following import map:

```json
{
  "scopes": {
    "/scoped/": {
      "jquery": "https://cdn.example.com/jquery.js"
    }
  }
}
```

Import map with integrity:

```csharp
new ImportMapDefinition(
    new Dictionary<string, string>
    {
        { "jquery", "https://cdn.example.com/jquery.js" },
    },
    null,
    new Dictionary<string, string>
    {
        { "https://cdn.example.com/jquery.js", "sha384-abc123" },
    });
```

The preceding code results in the following import map:

```json
{
  "imports": {
    "jquery": "https://cdn.example.com/jquery.js"
  },
  "integrity": {
    "https://cdn.example.com/jquery.js": "sha384-abc123"
  }
}
```

Combine import map definitions (<xref:Microsoft.AspNetCore.Components.ImportMapDefinition>) with <xref:Microsoft.AspNetCore.Components.ImportMapDefinition.Combine%2A?displayProperty=nameWithType>.

Import map created from a <xref:Microsoft.AspNetCore.Components.ResourceAssetCollection> that maps static assets to their corresponding unique URLs:

```csharp
ImportMapDefinition.FromResourceCollection(
    new ResourceAssetCollection(
    [
        new ResourceAsset(
            "jquery.fingerprint.js",
            [
                new ResourceAssetProperty("integrity", "sha384-abc123"),
                new ResourceAssetProperty("label", "jquery.js"),
            ])
    ]));
```

The preceding code results in the following import map:

```json
{
  "imports": {
    "./jquery.js": "./jquery.fingerprint.js"
  },
  "integrity": {
    "jquery.fingerprint.js": "sha384-abc123"
  }
}
```

## Import map Content Security Policy (CSP) violations

*This section applies to Blazor Web Apps that call <xref:Microsoft.AspNetCore.Builder.RazorComponentsEndpointRouteBuilderExtensions.MapRazorComponents%2A>.*

The `ImportMap` component is rendered as an inline `<script>` tag, which violates a strict [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/Guides/CSP) that sets the `default-src` or `script-src` directive.

For examples of how to address the policy violation with Subresource Integrity (SRI) or a cryptographic nonce, see [Resolving CSP violations with Subresource Integrity (SRI) or a nonce](xref:blazor/security/content-security-policy#resolving-csp-violations-with-subresource-integrity-sri-or-a-cryptographic-nonce).

:::moniker-end

:::moniker range="< aspnetcore-9.0"

Configure Static File Middleware to serve static assets to clients by calling <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> in the app's request processing pipeline. For more information, see <xref:fundamentals/static-files>.

In releases prior to .NET 8, Blazor framework static files, such as the Blazor script, are served via Static File Middleware. In .NET 8 or later, Blazor framework static files are mapped using endpoint routing, and Static File Middleware is no longer used.

:::moniker-end

:::moniker range=">= aspnetcore-10.0"

## Fingerprint client-side static assets in standalone Blazor WebAssembly apps

In standalone Blazor WebAssembly apps during build/publish, the framework overrides placeholders in `index.html` with values computed during build to fingerprint static assets for client-side rendering. A [fingerprint](https://wikipedia.org/wiki/Fingerprint_(computing)) is placed into the `blazor.webassembly.js` script file name, and an import map is generated for other .NET assets.

The following configuration must be present in the `wwwwoot/index.html` file of a standalone Blazor WebAssembly app to adopt fingerprinting:

```html
<head>
    ...
    <script type="importmap"></script>
    ...
</head>

<body>
    ...
    <script src="_framework/blazor.webassembly#[.{fingerprint}].js"></script>
    ...
</body>

</html>
```

In the project file (`.csproj`), the `<OverrideHtmlAssetPlaceholders>` property is set to `true`:

```xml
<PropertyGroup>
  <OverrideHtmlAssetPlaceholders>true</OverrideHtmlAssetPlaceholders>
</PropertyGroup>
```

When resolving imports for JavaScript interop, the import map is used by the browser resolve fingerprinted files.

Any script in `index.html` with the fingerprint marker is fingerprinted by the framework. For example, a script file named `scripts.js` in the app's `wwwroot/js` folder is fingerprinted by adding `#[.{fingerprint}]` before the file extension (`.js`):

```html
<script src="js/scripts#[.{fingerprint}].js"></script>
```

## Fingerprint client-side static assets in Blazor Web Apps

For client-side rendering (CSR) in Blazor Web Apps (Interactive Auto or Interactive WebAssembly render modes), static asset server-side [fingerprinting](https://wikipedia.org/wiki/Fingerprint_(computing)) is enabled by adopting [Map Static Assets routing endpoint conventions (`MapStaticAssets`)](xref:fundamentals/map-static-files), [`ImportMap` component](xref:blazor/fundamentals/static-files#importmap-component), and the <xref:Microsoft.AspNetCore.Components.ComponentBase.Assets?displayProperty=nameWithType> property (`@Assets["..."]`). For more information, see <xref:fundamentals/map-static-files>.

To fingerprint additional JavaScript modules for CSR, use the `<StaticWebAssetFingerprintPattern>` item in the app's project file (`.csproj`). In the following example, a fingerprint is added for all developer-supplied `.mjs` files in the app:

```xml
<ItemGroup>
  <StaticWebAssetFingerprintPattern Include="JSModule" Pattern="*.mjs" 
    Expression="#[.{fingerprint}]!" />
</ItemGroup>
```

When resolving imports for JavaScript interop, the import map is used by the browser resolve fingerprinted files.

:::moniker-end

## Summary of static file `<link>` `href` formats

*This section applies to all .NET releases and Blazor apps.*

The following tables summarize static file `<link>` `href` formats by .NET release.

:::moniker range=">= aspnetcore-6.0"

For the location of `<head>` content where static file links are placed, see <xref:blazor/project-structure#location-of-head-and-body-content>. Static asset links can also be supplied using [`<HeadContent>` components](xref:blazor/components/control-head-content) in individual Razor components.

:::moniker-end

:::moniker range="< aspnetcore-6.0"

For the location of `<head>` content where static file links are placed, see <xref:blazor/project-structure#location-of-head-and-body-content>.

:::moniker-end

### .NET 9 or later

App type | `href` value | Examples
--- | --- | ---
Blazor Web App | `@Assets["{PATH}"]` | `<link rel="stylesheet" href="@Assets["app.css"]" />`<br>`<link href="@Assets["_content/ComponentLib/styles.css"]" rel="stylesheet" />`
Blazor Server&dagger; | `@Assets["{PATH}"]` | `<link href="@Assets["css/site.css"]" rel="stylesheet" />`<br>`<link href="@Assets["_content/ComponentLib/styles.css"]" rel="stylesheet" />`
Standalone Blazor WebAssembly | `{PATH}` | `<link rel="stylesheet" href="css/app.css" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`

### .NET 8.x

App type | `href` value | Examples
--- | --- | ---
Blazor Web App | `{PATH}` | `<link rel="stylesheet" href="app.css" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`
Blazor Server&dagger; | `{PATH}` | `<link href="css/site.css" rel="stylesheet" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`
Standalone Blazor WebAssembly | `{PATH}` | `<link rel="stylesheet" href="css/app.css" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`

### .NET 7.x or earlier

App type | `href` value | Examples
--- | --- | ---
Blazor Server&dagger; | `{PATH}` | `<link href="css/site.css" rel="stylesheet" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`
Hosted Blazor WebAssembly&Dagger; | `{PATH}` | `<link href="css/app.css" rel="stylesheet" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`
Blazor WebAssembly | `{PATH}` | `<link href="css/app.css" rel="stylesheet" />`<br>`<link href="_content/ComponentLib/styles.css" rel="stylesheet" />`

&dagger;Blazor Server is supported in .NET 8 or later but is no longer a project template after .NET 7.  
&Dagger;We recommend updating Hosted Blazor WebAssembly apps to Blazor Web Apps when adopting .NET 8 or later.

:::moniker range=">= aspnetcore-8.0"

## Static Web Asset Project Mode

*This section applies to the `.Client` project of a Blazor Web App.*

The required `<StaticWebAssetProjectMode>Default</StaticWebAssetProjectMode>` setting in the `.Client` project of a Blazor Web App reverts Blazor WebAssembly static asset behaviors back to the defaults, so that the project behaves as part of the hosted project. The Blazor WebAssembly SDK (`Microsoft.NET.Sdk.BlazorWebAssembly`) configures static web assets in a specific way to work in "standalone" mode with a server simply consuming the outputs from the library. This isn't appropriate for a Blazor Web App, where the WebAssembly portion of the app is a logical part of the host and must behave more like a library. For example, the project doesn't expose the styles bundle (for example, `BlazorSample.Client.styles.css`) and instead only provides the host with the project bundle, so that the host can include it in its own styles bundle.

Changing the value (`Default`) of `<StaticWebAssetProjectMode>` or removing the property from the `.Client` project isn't supported.

:::moniker-end

## Static files in non-`Development` environments

*This section applies to server-side static files.*

When running an app locally, static web assets are only enabled in the <xref:Microsoft.Extensions.Hosting.Environments.Development> environment. To enable static files for environments other than <xref:Microsoft.Extensions.Hosting.Environments.Development> during local development and testing (for example, <xref:Microsoft.Extensions.Hosting.Environments.Staging>), call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderExtensions.UseStaticWebAssets%2A> on the <xref:Microsoft.AspNetCore.Builder.WebApplicationBuilder> in the `Program` file.

> [!WARNING]
> Call <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderExtensions.UseStaticWebAssets%2A> for the ***exact environment*** to prevent activating the feature in production, as it serves files from separate locations on disk *other than from the project* if called in a production environment. The example in this section checks for the <xref:Microsoft.Extensions.Hosting.Environments.Staging> environment by calling <xref:Microsoft.Extensions.Hosting.HostEnvironmentEnvExtensions.IsStaging%2A>.

```csharp
if (builder.Environment.IsStaging())
{
    builder.WebHost.UseStaticWebAssets();
}
```

:::moniker range=">= aspnetcore-8.0"

## Prefix for Blazor WebAssembly assets

*This section applies to Blazor Web Apps.*

Use the <xref:Microsoft.AspNetCore.Components.WebAssembly.Server.WebAssemblyComponentsEndpointOptions.PathPrefix?displayProperty=nameWithType> endpoint option to set the path string that indicates the prefix for Blazor WebAssembly assets. The path must correspond to a referenced Blazor WebAssembly application project.

```csharp
endpoints.MapRazorComponents<App>()
    .AddInteractiveWebAssemblyRenderMode(options => 
        options.PathPrefix = "{PATH PREFIX}");
```

In the preceding example, the `{PATH PREFIX}` placeholder is the path prefix and must start with a forward slash (`/`).

In the following example, the path prefix is set to `/path-prefix`:

```csharp
endpoints.MapRazorComponents<App>()
    .AddInteractiveWebAssemblyRenderMode(options => 
        options.PathPrefix = "/path-prefix");
```

:::moniker-end

## Static web asset base path

:::moniker range=">= aspnetcore-8.0"

*This section applies to standalone Blazor WebAssembly apps.*

Publishing the app places the app's static assets, including Blazor framework files (`_framework` folder assets), at the root path (`/`) in published output. The `<StaticWebAssetBasePath>` property specified in the project file (`.csproj`) sets the base path to a non-root path:

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>{PATH}</StaticWebAssetBasePath>
</PropertyGroup>
```

In the preceding example, the `{PATH}` placeholder is the path.

Without setting the `<StaticWebAssetBasePath>` property, a standalone app is published at `/BlazorStandaloneSample/bin/Release/{TFM}/publish/wwwroot/`.

In the preceding example, the `{TFM}` placeholder is the [Target Framework Moniker (TFM)](/dotnet/standard/frameworks).

If the `<StaticWebAssetBasePath>` property in a standalone Blazor WebAssembly app sets the published static asset path to `app1`, the root path to the app in published output is `/app1`.

In the standalone Blazor WebAssembly app's project file (`.csproj`):

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>app1</StaticWebAssetBasePath>
</PropertyGroup>
```

In published output, the path to the standalone Blazor WebAssembly app is `/BlazorStandaloneSample/bin/Release/{TFM}/publish/wwwroot/app1/`.

In the preceding example, the `{TFM}` placeholder is the [Target Framework Moniker (TFM)](/dotnet/standard/frameworks).

:::moniker-end

:::moniker range="< aspnetcore-8.0"

*This section applies to standalone Blazor WebAssembly apps and hosted Blazor WebAssembly solutions.*

Publishing the app places the app's static assets, including Blazor framework files (`_framework` folder assets), at the root path (`/`) in published output. The `<StaticWebAssetBasePath>` property specified in the project file (`.csproj`) sets the base path to a non-root path:

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>{PATH}</StaticWebAssetBasePath>
</PropertyGroup>
```

In the preceding example, the `{PATH}` placeholder is the path.

Without setting the `<StaticWebAssetBasePath>` property, the client app of a hosted solution or a standalone app is published at the following paths:

* In the **:::no-loc text="Server":::** project of a hosted Blazor WebAssembly solution: `/BlazorHostedSample/Server/bin/Release/{TFM}/publish/wwwroot/`
* In a standalone Blazor WebAssembly app: `/BlazorStandaloneSample/bin/Release/{TFM}/publish/wwwroot/`

If the `<StaticWebAssetBasePath>` property in the **:::no-loc text="Client":::** project of a hosted Blazor WebAssembly app or in a standalone Blazor WebAssembly app sets the published static asset path to `app1`, the root path to the app in published output is `/app1`.

In the **:::no-loc text="Client":::** app's project file (`.csproj`) or the standalone Blazor WebAssembly app's project file (`.csproj`):

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>app1</StaticWebAssetBasePath>
</PropertyGroup>
```

In published output:

* Path to the client app in the **:::no-loc text="Server":::** project of a hosted Blazor WebAssembly solution: `/BlazorHostedSample/Server/bin/Release/{TFM}/publish/wwwroot/app1/`
* Path to a standalone Blazor WebAssembly app: `/BlazorStandaloneSample/bin/Release/{TFM}/publish/wwwroot/app1/`

The `<StaticWebAssetBasePath>` property is most commonly used to control the paths to published static assets of multiple Blazor WebAssembly apps in a single hosted deployment. For more information, see <xref:blazor/host-and-deploy/webassembly/multiple-hosted-webassembly>. The property is also effective in standalone Blazor WebAssembly apps.

In the preceding examples, the `{TFM}` placeholder is the [Target Framework Moniker (TFM)](/dotnet/standard/frameworks).

:::moniker-end

## File mappings and static file options

*This section applies to server-side static files.*

:::moniker range=">= aspnetcore-8.0"

To create additional file mappings with a <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> or configure other <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>, use **one** of the following approaches. In the following examples, the `{EXTENSION}` placeholder is the file extension, and the `{CONTENT TYPE}` placeholder is the content type. The namespace for the following API is <xref:Microsoft.AspNetCore.StaticFiles>.

* Configure options through [dependency injection (DI)](xref:blazor/fundamentals/dependency-injection) in the `Program` file using <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>:

  ```csharp
  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  builder.Services.Configure<StaticFileOptions>(options =>
  {
      options.ContentTypeProvider = provider;
  });

  app.UseStaticFiles();
  ```

* Pass the <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> directly to <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> in the `Program` file:

  ```csharp
  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  app.UseStaticFiles(new StaticFileOptions { ContentTypeProvider = provider });
  ```

:::moniker-end

:::moniker range="< aspnetcore-8.0"

To create additional file mappings with a <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> or configure other <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>, use **one** of the following approaches. In the following examples, the `{EXTENSION}` placeholder is the file extension, and the `{CONTENT TYPE}` placeholder is the content type.

* Configure options through [dependency injection (DI)](xref:blazor/fundamentals/dependency-injection) in the `Program` file using <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>:

  ```csharp
  using Microsoft.AspNetCore.StaticFiles;

  ...

  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  builder.Services.Configure<StaticFileOptions>(options =>
  {
      options.ContentTypeProvider = provider;
  });
  ```

  This approach configures the same file provider used to serve the Blazor script. Make sure that your custom configuration doesn't interfere with serving the Blazor script. For example, don't remove the mapping for JavaScript files by configuring the provider with `provider.Mappings.Remove(".js")`.

* Use two calls to <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> in the `Program` file:
  * Configure the custom file provider in the first call with <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>.
  * The second middleware serves the Blazor script, which uses the default static files configuration provided by the Blazor framework.

  ```csharp
  using Microsoft.AspNetCore.StaticFiles;

  ...

  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  app.UseStaticFiles(new StaticFileOptions { ContentTypeProvider = provider });
  app.UseStaticFiles();
  ```

* You can avoid interfering with serving `_framework/blazor.server.js` by using <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> to execute a custom Static File Middleware:

  ```csharp
  app.MapWhen(ctx => !ctx.Request.Path
      .StartsWithSegments("/_framework/blazor.server.js"),
          subApp => subApp.UseStaticFiles(new StaticFileOptions() { ... }));
  ```

:::moniker-end

:::moniker range=">= aspnetcore-8.0"

## Serve files from multiple locations

*The guidance in this section only applies to Blazor Web Apps.*

To serve files from multiple locations with a <xref:Microsoft.Extensions.FileProviders.CompositeFileProvider>:

* Add the namespace for <xref:Microsoft.Extensions.FileProviders?displayProperty=fullName> to the top of the `Program` file of the server project.
* In the server project's `Program` file ***before*** the call to <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>:
  * Create a <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider> with the path to the static assets.
  * Create a <xref:Microsoft.Extensions.FileProviders.CompositeFileProvider> from the <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootFileProvider> and the <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>. Assign the composite file provider back to the app's <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootFileProvider>.

Example:

Create a new folder in the server project named `AdditionalStaticAssets`. Place an image into the folder.

Add the following `using` statement to the top of the server project's `Program` file:

```csharp
using Microsoft.Extensions.FileProviders;
```

In the server project's `Program` file ***before*** the call to <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>, add the following code:

```csharp
var secondaryProvider = new PhysicalFileProvider(
    Path.Combine(builder.Environment.ContentRootPath, "AdditionalStaticAssets"));
app.Environment.WebRootFileProvider = new CompositeFileProvider(
    app.Environment.WebRootFileProvider, secondaryProvider);
```

In the app's `Home` component (`Home.razor`) markup, reference the image with an `<img>` tag:

```razor
<img src="{IMAGE FILE NAME}" alt="{ALT TEXT}" />
```

In the preceding example:

* The `{IMAGE FILE NAME}` placeholder is the image file name. There's no need to provide a path segment if the image file is at the root of the `AdditionalStaticAssets` folder.
* The `{ALT TEXT}` placeholder is the image alternate text.

Run the app.

:::moniker-end

## Additional resources

:::moniker range=">= aspnetcore-8.0"

* <xref:blazor/host-and-deploy/app-base-path>
* [Avoid file capture in a route parameter](xref:blazor/fundamentals/routing#avoid-file-capture-in-a-route-parameter)

:::moniker-end

:::moniker range="< aspnetcore-8.0"

* <xref:blazor/host-and-deploy/app-base-path>
* [Avoid file capture in a route parameter](xref:blazor/fundamentals/routing#avoid-file-capture-in-a-route-parameter)
* <xref:blazor/host-and-deploy/webassembly/multiple-hosted-webassembly>

:::moniker-end
