+++
title = 'API Versioning in ASP.NET Core'
date = 2024-06-08T12:21:44+01:00
draft = false
tags = ["ASP.Net Core"]
+++

API Versioning is a critical aspect of building and maintaining robust and scalable APIs. It allows developers to introduce new features and improvements while maintaining backward compatibility for existing clients. We will look into the rationale behind API versioning, the problems it resolves, and the implementations of various versioning strategies in ASP.NET Core.

# Why API Versioning?

- **Backward Compatibility**: Ensures existing clients continue to function even when new features are added.
- **Controlled Evolution**: Allows incremental and controlled release of new functionalities.
- **Flexibility**: Provides flexibility to make significant changes without breaking existing API contracts.

# Problems Addressed by API Versioning

- **Compatibility Issues**: Avoids breaking changes for existing consumers.
- **Maintenance Complexity**: Simplifies maintenance by clearly separating different versions of APIs.
- **Deprecation Strategy**: Facilitates a planned deprecation strategy, allowing clients to migrate at their own pace.

# API Versioning in ASP.NET Core: Detailed Implementation and Comparison

API versioning is essential for evolving your APIs while maintaining backward compatibility. This guide provides detailed explanations and code snippets for implementing various API versioning strategies in ASP.NET Core, including Query String Parameter Versioning, Media Type (Header) Versioning, and URI Versioning. Additionally, it compares the advantages and disadvantages of these strategies.

## Implementing API Versioning in ASP.NET Core

### Query String Parameter Versioning

Query String Parameter Versioning involves specifying the API version as a query parameter in the URL. This method is straightforward to implement and makes the version explicit in the request URL.

#### Implementation

1. Install the API Versioning Package:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

2. Configure Services:
   In `Startup.cs`, configure the API versioning service:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.ReportApiVersions = true;
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.ApiVersionReader = new QueryStringApiVersionReader("v");
    });
}
```

3. Versioned Controller:
   Create a versioned controller:

```csharp
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet]
    public IActionResult GetV1() => Ok("Version 1");

    [HttpGet, MapToApiVersion("2.0")]
    public IActionResult GetV2() => Ok("Version 2");
}
```

##### Example Request

- Version 1: GET /api/values?v=1.0
- Version 2: GET /api/values?v=2.0

### Media Type (Header) Versioning

Media Type Versioning, also known as Content Negotiation, involves specifying the version in the Accept header of the HTTP request. This method adheres to REST principles by leveraging HTTP headers and keeps the URL clean.

#### Implementation

1. Configure Services:
   In `Startup.cs`, configure the API versioning service:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.ReportApiVersions = true;
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.ApiVersionReader = new MediaTypeApiVersionReader("v");
    });
}
```

2. Versioned Controller:
   Create a versioned controller:

```csharp
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet]
    public IActionResult GetV1() => Ok("Version 1");

    [HttpGet, MapToApiVersion("2.0")]
    public IActionResult GetV2() => Ok("Version 2");
}
```

##### Example Request

- Version 1: GET /api/values with Accept: application/vnd.myapi.v1+json
- Version 2: GET /api/values with Accept: application/vnd.myapi.v2+json

### URI Versioning

URI Versioning incorporates the version directly into the URL path. This method makes the version immediately visible and easy to understand.

#### Implementation

1. Configure Services:
   In `Startup.cs`, configure the API versioning service:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.ReportApiVersions = true;
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.ApiVersionReader = new UrlSegmentApiVersionReader();
    });
}
```

2. Versioned Controller:
   Create a versioned controller:

```csharp
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet]
    public IActionResult GetV1() => Ok("Version 1");

    [HttpGet, MapToApiVersion("2.0")]
    public IActionResult GetV2() => Ok("Version 2");
}
```

#### Example Request

- Version 1: GET /api/v1/values
- Version 2: GET /api/v2/values

# Comparison of API Versioning Strategies

API versioning is crucial for managing changes and ensuring backward compatibility in your APIs. Each versioning strategy has its own advantages and disadvantages.

| Versioning Strategy    | Advantages                                              | Disadvantages                                     |
| ---------------------- | ------------------------------------------------------- | ------------------------------------------------- |
| Query String Parameter | - Simple to implement                                   | - Clutters query string                           |
|                        | - Explicit versioning in the URL                        | - Potential conflicts with other query parameters |
| Media Type (Header)    | - Clean URLs                                            | - More complex to implement and understand        |
|                        | - Adheres to REST principles by leveraging HTTP headers | - Requires clients to set headers correctly       |
| URI Versioning         | - Clear and explicit versioning visible in the URL      | - Adds extra segments to the URL path             |
|                        | - Easy to understand and implement                      | - Not strictly RESTful according to some purists  |

Each versioning strategy offers unique benefits and drawbacks. Here's a brief recap:

- **Query String Parameter Versioning**: Straightforward but can clutter URLs.
- **Media Type (Header) Versioning**: Keeps URLs clean but requires more effort to implement and understand.
- **URI Versioning**: Highly visible and simple to implement but can make URLs longer and more complex.

Choose the strategy that best fits your API's needs and your consumers' preferences. Proper versioning will ensure your API evolves smoothly and remains backward compatible.

# When to Use API Versioning

- When you need to support multiple versions of an API simultaneously.
- When you expect significant changes in the API over time.
- When you want to provide backward compatibility for existing clients.

# When Not to Use API Versioning

- For internal or limited-use APIs where changes can be communicated and managed easily.
- When the API is still in early development and subject to frequent changes.

# Issues and Considerations

- **Client Compatibility**: Ensure that clients can handle different versions.
- **Documentation**: Maintain clear and updated documentation for each version.
- **Testing**: Thoroughly test all versions to avoid breaking changes.
- **Deprecation**: Plan and communicate the deprecation of old versions.

# Best Practices

- **Start with Versioning**: Even if you only have one version, plan for future versions from the start.
- **Use Semantic Versioning**: Follow semantic versioning (e.g., 1.0, 2.0) for clarity.
- **Deprecate Gracefully**: Provide ample notice and support for deprecated versions.
- **Automate Testing**: Use automated tests to ensure compatibility across versions.

# Conclusion

API versioning is an essential practice for developing scalable and maintainable APIs. By understanding and implementing the appropriate versioning strategy in ASP.NET Core, you can ensure that your API evolves smoothly while maintaining compatibility for existing clients. Consider the advantages and disadvantages of each versioning method to choose the best one for your specific scenario.

# References

- https://github.com/dotnet/aspnet-api-versioning
- https://github.com/dotnet/aspnet-api-versioning/wiki
