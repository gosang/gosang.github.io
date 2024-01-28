+++
title = 'Securing ASP.NET Core Web API With OAuth 2.0 in Microsoft Entra ID'
date = 2024-01-28T14:22:05Z
draft = false
tags = ["ASP.NET Core", "OAuth 2.0"]
+++

OAuth 2.0 is a widely adopted protocol for securing web applications and APIs. It provides a standardized way for third-party applications to obtain limited access to a user's resources without exposing their credentials. In this blog post, we will explore the fundamentals of OAuth 2.0 and how to implement it to secure an ASP.NET Core Web API using Microsoft Entra ID.

# Understanding OAuth 2.0

## What is OAuth 2.0?

OAuth 2.0 is an open standard authorization framework that enables secure, delegated access to resources. It allows a user to grant a third-party application limited access to their resources, without sharing their credentials.

## Why OAuth 2.0 and Problem Resolution

The primary goal of OAuth 2.0 is to provide a secure and standardized way for applications to access resources on behalf of users. It resolves the issue of sharing passwords between different services and enhances security by using access tokens.

# Implementing OAuth 2.0 with Microsoft Entra ID

## Scenario: E-commerce System

Consider an e-commerce system with ASP.NET Core Web API backend and a React frontend. The API is built using CQRS MediatR pattern, FluentValidation for input validation, AutoMapper for object mapping, Entity Framework Core for database interactions, and Testcontainers with xUnit for unit tests.

## OAuth 2.0 Flow

OAuth 2.0 defines several authorization flows that cater to different use cases. Two prominent flows are the Authorization Code Flow and the Client Credentials Flow. Let's delve into the details of each and explore their respective use cases.

### OAuth 2.0 Authorization Code Flow

To implement the Authorization Code Flow, use the OpenID Connect middleware in your authentication configuration. This flow is suitable for scenarios where user interaction is required, such as a web application.

### OAuth 2.0 Client Credentials Flow

For scenarios where a client application needs to access resources without user involvement, use the Client Credentials Flow. In your API, configure the client application with the appropriate client ID and secret.

### Setting Up OAuth 2.0 Registration in Microsoft Entra ID

To implement OAuth 2.0 Authorization Code Flow and OAuth 2.0 Client Credentials Flow with Microsoft Entra ID, you need to register your Web API and Client Application in the Azure AD B2C portal. Below are the succinct steps for both scenarios.

#### OAuth 2.0 Authorization Code Flow

##### Step 1: Register the Web API

1. Go to the `Azure portal` and navigate to Azure AD B2C.
2. In the left-hand menu, select "App registrations."
3. Click on "New registration" and fill in the required information:
   - `Name`: Enter a name for your API.
   - `Supported account types`: Choose "Accounts in any identity provider or organizational directory."
   - `Redirect URI`: Add the redirect URI for your application.
4. After registration, note the "Application (client) ID" and "Directory (tenant) ID" for configuration.
5. Under the API section, click on "Expose an API" and set the Application ID URI.
6. Add a Scope under the "Scopes" section, e.g., `access_as_user`.

##### Step 2: Register the Client Application

1. In the Azure AD B2C portal, go to "App registrations."
2. Click on "New registration" and provide the necessary details:
   - `Name`: Enter a name for your client application.
   - `Supported account types`: Choose "Accounts in any identity provider or organizational directory."
   - `Redirect URI`: Add the redirect URI for your application.
3. After registration, note the "Application (client) ID" and "Directory (tenant) ID" for configuration.
4. Under the "Authentication" section, configure the following:
   - `Implicit grant and hybrid flows`: Select "ID token" and "Access token."
5. Under the "API permissions" section, add the permissions required, including the Scope defined in the Web API.

#### OAuth 2.0 Client Credentials Flow

##### Step 1: Register the Web API

1. Follow Steps 1-5 from the Authorization Code Flow to register your Web API.
2. Under the "API permissions" section, add the necessary permissions required for your API.

##### Step 2: Register the Client Application

1. Follow Steps 1-3 from the Authorization Code Flow to register your Client Application.
2. After registration, note the "Application (client) ID" and "Directory (tenant) ID" for configuration.
3. Under the "Authentication" section, configure the following:
   - `Treat application as a public client`: Set it to "Yes."
4. Under the "Certificates & secrets" section, generate a new client secret and note it for configuration.
5. Under the "API permissions" section, add the necessary permissions required for your API.

These configurations will enable your ASP.NET Core Web API to validate tokens and authorize requests.

By registering your Web API and Client Application in Microsoft Entra ID, you enable secure OAuth 2.0 flows. The provided steps ensure proper configuration for both Authorization Code Flow and Client Credentials Flow, allowing your applications to authenticate and authorize seamlessly.

#### Configuration in ASP.NET Core

In your ASP.NET Core project, use the obtained values to configure OAuth 2.0 in appsettings.json or directly in code:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "<Your Tenant Domain>",
    "TenantId": "<Directory (tenant) ID>",
    "ClientId": "<Application (client) ID>",
    "CallbackPath": "/signin-oidc"
  },
  "ApiScope": "api://<Web API Application ID URI>/access_as_user"
}
```

## Implementing OAuth 2.0 with Microsoft Entra ID

Securing your ASP.NET Core Web API with OAuth 2.0 using Microsoft Entra ID involves several steps to configure authentication and authorization. Microsoft Entra ID is a solution that integrates with Azure AD B2C to provide secure and seamless identity management. Here's a succinct guide on how to implement OAuth 2.0 in your .NET Core API.

### Sample .NET Core API Project

#### Step 1: Install required NuGet packages

```bash
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
dotnet add package Microsoft.Identity.Web
```

#### Step 2: Configure OAuth 2.0 with Entra ID

In your `Startup.cs` file, configure authentication services:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Other configurations...

    services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))
    .EnableTokenAcquisitionToCallDownstreamApi(new string[] { Configuration["ApiScope"] })
    .AddInMemoryTokenCaches();

services.AddAuthentication(Configuration.GetSection("ApiAuth:Scheme"))
    .AddJwtBearer(Configuration.GetSection("ApiAuth"))
    .AddMicrosoftIdentityWebApi(Configuration.GetSection("AzureAd"))
    .EnableTokenAcquisitionToCallDownstreamApi(new string[] { Configuration["ApiScope"] })
    .AddInMemoryTokenCaches();

    // Other configurations...
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Other configurations...

    app.UseAuthentication();
    app.UseAuthorization();

    // Other configurations...
}
```

#### Step 3: Secure API Controller

In your API controller, use the `[Authorize]` attribute to enforce authentication:

```csharp
[Authorize]
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    // Controller logic here
}
```

Or use the `[Authorize]` attribute for user-based access and `[AuthorizeForScopes]` for client credentials-based access:

```csharp
[Authorize]
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    // Controller logic for user-based access

    [AuthorizeForScopes(Scopes = new[] { "api://<client-id>/access_as_user" })]
    [HttpGet("admin")]
    public IActionResult AdminEndpoint()
    {
        // Controller logic for client credentials-based access
    }
}
```

Now, your API is protected, and only authenticated users can access the `ProductsController`.

# Advantages and Disadvantages

## Advantages

- **Security**: OAuth 2.0 enhances security by eliminating the need to share credentials between services.
- **Standardization**: Being an open standard, OAuth 2.0 provides a consistent and widely accepted way of implementing authorization.

## Disadvantages

- **Complexity**: Implementing OAuth 2.0 can be complex, especially for beginners.
- **Token Management**: Proper token management is crucial, and improper handling can lead to security vulnerabilities.

# Issues and Considerations

- **Token Expiry**: Regularly refresh tokens to avoid expiration issues.
- **Scope Definition**: Clearly define and limit the scopes of access tokens.
- **Logging and Monitoring**: Implement robust logging and monitoring to track unauthorized access attempts.

# When to Use OAuth 2.0

Use OAuth 2.0 when:

- **Third-Party Access**: You need to allow third-party applications to access user resources.
- **Standardization is Essential**: You want to follow a widely accepted standard for authorization.

# When Not to Use OAuth 2.0

Avoid OAuth 2.0 when:

- **Simplicity is Preferred**: For simple scenarios, where the overhead of OAuth 2.0 is not justified.
- **Internal Access Only**: If all consumers of the API are internal services with no need for external authorization.

# Best Practices

- **Use HTTPS**: Always use HTTPS to encrypt communication.
- **Token Validation**: Implement thorough validation of access tokens.
- **Keep Secrets Secure**: Safeguard client secrets and other sensitive information.
- **Regular Updates**: Stay updated with the latest OAuth 2.0 specifications and security best practices.

# Conclusion

Implementing OAuth 2.0 with Microsoft Entra ID enhances the security of your ASP.NET Core Web API, especially in the context of an e-commerce system. While it adds complexity, the advantages of standardization and improved security outweigh the challenges. Follow best practices, be mindful of considerations, and OAuth 2.0 will be a robust solution for securing your API.

# References:

[Microsoft OAuth 2.0](https://learn.microsoft.com/en-us/entra/architecture/auth-oauth2)
[Microsoft Identity Web](https://learn.microsoft.com/en-us/entra/msal/dotnet/microsoft-identity-web/)
[OAuth 2.0 Specification](https://oauth.net/2/)
