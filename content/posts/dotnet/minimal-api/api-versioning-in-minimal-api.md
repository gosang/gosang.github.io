+++
title = 'API Versioning in Minimal APIs with ASP.NET Core 10'
date = 2026-02-07T11:56:56Z
draft = true
+++

Minimal APIs in ASP.NET Core are designed to be fast, lightweight, and expressive. As your API evolves, you’ll inevitably need to introduce changes such as new fields, new behavior, or breaking changes. That’s where API versioning comes in.

What API versioning is and why you need it

- Core concepts and approaches in ASP.NET Core 10
- How to implement versioning in a Minimal API with concise C# samples
- Pros, cons, and when to use (or avoid) versioning
- Issues, considerations, and best practices

## What Is API Versioning?

API versioning is the practice of managing changes to your API in a way that doesn’t break existing clients. Instead of changing behavior silently, you introduce a new version of the API while continuing to support older ones for a period of time.

Example:

- GET /api/v1/products → old behavior
- GET /api/v2/products → new behavior

## Why Version Your API?

- Backward compatibility: Clients shouldn’t break when you ship new features.
- Controlled evolution: You can introduce breaking changes safely.
- Clear contracts: Each version represents a stable, documented contract.
- Independent release cadence: You can improve v2 without touching v1 users.

## Key Versioning Approaches

ASP.NET Core supports several versioning styles:

| Approach     | Example                          | Notes                  |
| ------------ | -------------------------------- | ---------------------- |
| URL path     | `/api/v1/products`               | Most common & explicit |
| Query string | `/api/products?api-version=1.0`  | Easy to implement      |
| Header-based | `api-version: 1.0`               | Clean URLs             |
| Media type   | `Accept: application/json;v=1.0` | Advanced, REST-pure    |

In Minimal APIs, URL segment + API Versioning package is the most practical and readable choice.

## Key Concepts in ASP.NET Core 10 Versioning

Using the `Asp.Versioning.*` packages:

- ApiVersion – Represents a version (e.g., 1.0, 2.0)
- IApiVersionReader – Determines how the version is read (URL, header, etc.)
- ApiVersionSet – Groups endpoints by version
- MapToApiVersion – Maps a route to a specific version
- ReportApiVersions – Adds version info to responses

## Implementing API Versioning in a Minimal API (ASP.NET Core 10)

1. Create a Minimal API project

```bash
dotnet new web -n VersionedMinimalApi
cd VersionedMinimalApi
```

2. Install Versioning Packages

```bash
dotnet add package Asp.Versioning.Http
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

3. Configure Versioning in `Program.cs`

```csharp
using Asp.Versioning;
using Asp.Versioning.Builder;

var builder = WebApplication.CreateBuilder(args);

// Add API Versioning
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // adds headers: api-supported-versions
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});

var app = builder.Build();
```

4. Define Versioned Endpoints

```csharp
var apiVersionSet = app.NewApiVersionSet()
    .HasApiVersion(new ApiVersion(1, 0))
    .HasApiVersion(new ApiVersion(2, 0))
    .ReportApiVersions()
    .Build();

// V1
app.MapGet("/api/v{version:apiVersion}/products", (ApiVersion version) =>
{
    return Results.Ok(new[]
    {
        new { Id = 1, Name = "Laptop", Price = 1000 }
    });
})
.WithApiVersionSet(apiVersionSet)
.MapToApiVersion(1.0);

// V2
app.MapGet("/api/v{version:apiVersion}/products", (ApiVersion version) =>
{
    return Results.Ok(new[]
    {
        new { Id = 1, Name = "Laptop", Price = 1000, Currency = "USD", InStock = true }
    });
})
.WithApiVersionSet(apiVersionSet)
.MapToApiVersion(2.0);

app.Run();

```

5. Try It

- GET /api/v1/products → old response
- GET /api/v2/products → new response

You’ve now implemented versioning in a Minimal API with ASP.NET Core 10.

## Advantages of API Versioning in Minimal APIs

✅ Safe evolution of APIs
✅ No breaking changes for existing clients
✅ Clear contracts per version
✅ Supports long-term API stability
✅ Enables gradual migration

## Disadvantages

❌ More code & maintenance
❌ More testing complexity
❌ Documentation overhead
❌ Risk of supporting old versions too long
