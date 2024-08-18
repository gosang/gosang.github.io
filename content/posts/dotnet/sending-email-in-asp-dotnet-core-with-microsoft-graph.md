+++
title = 'Sending Email in ASP.NET Core With Microsoft Graph'
date = 2024-08-15T16:04:17+01:00
draft = false
tags = ["ASP.Net Core", "Graph", "Email"]
+++

In this blog, we'll look into how to send emails in an ASP.NET Core 8 API using Microsoft Graph. We will cover what Microsoft Graph is, the rationale for using it, key concepts, and implementation details with sample C# code. We'll also discuss the advantages, disadvantages, and best practices for using Microsoft Graph for email services. Additionally, we will explain how to restrict access to a specific mailbox for the application in Azure AD.

# What is Microsoft Graph?

Microsoft Graph is a unified API endpoint that allows developers to access a wide range of Microsoft services and data within Office 365, Windows 10, and Enterprise Mobility + Security. It provides a single API endpoint to connect to resources such as mail, calendar, contacts, documents, and more.

# Why Microsoft Graph

Microsoft Graph offers several benefits for sending emails in ASP.NET Core:

- **Unified Endpoint**: Access various Microsoft services through a single endpoint.
- **Security**: Leverage Microsoft’s robust security infrastructure.
- **Scalability**: Seamlessly scale your application to send large volumes of emails.
- **Integration**: Easily integrate with other Office 365 services and data.

# Key Concepts

## Authentication and Authorization

To use Microsoft Graph, your application must be registered in Azure Active Directory (Azure AD). This registration provides the necessary credentials to authenticate and authorize your application.

## Scopes and Permissions

Microsoft Graph requires permissions (scopes) to access specific resources. For sending emails, you need the Mail.Send permission.

## Access Tokens

After authentication, you obtain an access token, which you include in the HTTP headers of your requests to Microsoft Graph.

# A Sample Graph API Email Implementation in ASP.NET Core 8

## Step 1: Register Your Application in Azure AD

- Go to the Azure portal and navigate to Azure Active Directory.
- Select "App registrations" and register a new application.
- Configure the necessary API permissions (Mail.Send).
- Obtain the Client ID and Client Secret.

## Step 2: Install Required NuGet Packages

Install the Microsoft Graph SDK and Azure Identity library:

```bash
dotnet add package Microsoft.Graph
dotnet add package Azure.Identity
```

## Step 3: Configure Your Application

Add the following configuration settings to your appsettings.json:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "ClientSecret": "your-client-secret"
  }
}
```

## Step 4: Implement the Email Service

Create a service to handle sending emails:

```csharp
using Azure.Identity;
using Microsoft.Graph;

public class EmailService
{
    private readonly GraphServiceClient _graphClient;

    public EmailService(IConfiguration configuration)
    {
        var options = new TokenCredentialOptions
        {
            AuthorityHost = AzureAuthorityHosts.AzurePublicCloud
        };

        var clientSecretCredential = new ClientSecretCredential(
            configuration["AzureAd:TenantId"],
            configuration["AzureAd:ClientId"],
            configuration["AzureAd:ClientSecret"],
            options);

        _graphClient = new GraphServiceClient(clientSecretCredential);
    }

    public async Task SendEmailAsync(EmailSendingRequest request)
    {
        var sendMailRequest = new SendMailPostRequestBody
        {
            Message = new Message
            {
                Subject = request.Subject,
                Body = new ItemBody
                {
                    ContentType = BodyType.Text,
                    Content = request.Content
                },
                ToRecipients =
                [
                    new Recipient
                    {
                        EmailAddress = new EmailAddress
                        {
                            Address = request.Rrecipient
                        }
                    }
                ]
            }
        };

        try
        {
            await _graphClient.Users[mailbox].SendMail.PostAsync(sendMailRequest);
            return true;

        }catch (Exception ex)
        {
            _logger.LogError(ex, "An error occured whilst sending email");
            return false;
        }
    }
}
```

## Step 5: Use the Email Service in Your Controller

Add the email service to your controller to send an email:

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmailController : ControllerBase
{
    private readonly EmailService _emailService;

    public EmailController(EmailService emailService)
    {
        _emailService = emailService;
    }

    [HttpPost]
    [Route("send")]
    public async Task<IActionResult> SendEmail([FromBody] EmailSendingRequest request)
    {
        await _emailService.SendEmailAsync(request.Recipient, request.Subject, request.Body);
        return Ok("Email sent successfully");
    }
}

public class EmailSendingRequest
{
    public string Recipient { get; set; }
    public string Subject { get; set; }
    public string Body { get; set; }
}
```

Now the Email API endpoint can be accessed for sending email.

# Restricting Access to a Specific Mailbox

To restrict access so that the service can only send emails from one specific mailbox, follow these steps:

## Step 1: Create a Mail-Enabled Security Group

- Go to the Azure portal and navigate to Azure Active Directory.
- Select "Groups" and create a new security group.
- Add the specific mailbox (user) to this group.

## Step 2: Assign Application Permissions

- In the Azure portal, go to "App registrations" and select your application.
- Navigate to "API permissions" and select "Add a permission".
- Choose "Microsoft Graph" and then "Application permissions".
- Select the Mail.Send permission and click "Add permissions".

## Step 3: Configure the Application to Use the Specific Mailbox

Navigate to "Certificates & secrets" in your app registration.
Create a new client secret and note it down.
Navigate to "API permissions" and grant admin consent.

## Step 4: Modify the Email Service

Update your EmailService to send emails on behalf of the specific mailbox:

```csharp
// Sample Email Sending Request
public class EmailSendingRequest
{
    public string Sender { get; set; }
    public string Recipient { get; set; }
    public string Subject { get; set; }
    public string Content { get; set; }
}

// Sample Sending Email
public async Task SendEmailAsync(EmailSendingRequest request)
    {
        var sendMailRequest = new SendMailPostRequestBody
        {
            Message = new Message
            {
                Subject = request.Subject,
                Body = new ItemBody
                {
                    ContentType = BodyType.Text,
                    Content = request.Content
                },
                ToRecipients =
                [
                    new Recipient
                    {
                        EmailAddress = new EmailAddress
                        {
                            Address = request.Rrecipient
                        }
                    }
                ]
            }
        };

        try
        {
            await _graphClient.Users[request.Sender].SendMail.PostAsync(sendMailRequest);
            return true;

        }catch (Exception ex)
        {
            _logger.LogError(ex, "An error occured whilst sending email");
            return false;
        }
    }

```

To allow multiple mailboxes for sending email, similar steps should be followed.

# Advantages and Disadvantages

| Advantages                                                        | Disadvantages                                                                              |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Security**: Leverage Microsoft’s security features.             | **Complexity**: Requires configuration and understanding of Microsoft Entra ID (Azure AD). |
| **Integration**: Easily integrate with other Office 365 services. | **Cost**: Usage of Microsoft Graph API can incur costs depending on the volume of emails.  |
| **Scalability**: Efficiently handle large volumes of emails.      | **Rate Limits**: Subject to Microsoft Graph API rate limits.                               |
|                                                                   | **Microsoft mailbox limits**: Subject to Microsoft Mailbox limits if used.                 |

|

# When to Use and When Not to Use

| When to use                                                                      | When not to use                                                             |
| -------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| When you need to integrate deeply with other Microsoft services.                 | When you need a simple, standalone email service without deep integrations. |
| When security and compliance are top priorities.                                 | When you are working with non-Microsoft ecosystems.                         |
| When you are already within the Microsoft ecosystem and need a unified solution. | When cost is a major concern, and you have a high volume of emails to send. |

# Issues, Considerations, and Best Practices

## Issues and Considerations

- **Authentication**: Properly managing OAuth tokens and refreshing them when necessary.
- **Rate Limits**: Be aware of and handle rate limits imposed by Microsoft Graph.
- **Error Handling**: Implement robust error handling and retry logic.

## Best Practices

- **Secure Credentials**: Use Azure Key Vault or other secure methods to store credentials.
- **Logging and Monitoring**: Implement comprehensive logging and monitoring.
- **Batching Requests**: Use batching for sending multiple emails to reduce API calls.
- **Optimize Scopes**: Request only the necessary permissions.

# Conclusion

Using Microsoft Graph to send emails in an ASP.NET Core 8 API provides a secure, scalable, and integrated solution, especially suited for applications within the Microsoft ecosystem.

However, it requires proper setup, understanding of Azure AD, and handling potential complexities like rate limits and authentication.

By following the steps and best practices outlined in this blog, you can effectively implement and manage an email service using Microsoft Graph in your ASP.NET Core applications.

For more detailed information, refer to the official [Microsoft Graph documentation](https://learn.microsoft.com/en-us/graph/) and [Microsoft Entra ID (Azure AD) documentation](https://learn.microsoft.com/en-us/entra/identity/).
