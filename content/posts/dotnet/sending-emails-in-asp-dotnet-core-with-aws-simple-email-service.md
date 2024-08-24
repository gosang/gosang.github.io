+++
title = 'Sending Emails in ASP.NET Core With AWS Simple Email Service (SES)'
date = 2024-08-24T15:47:20+01:00
draft = false
tags = ["ASP.NET Core", "AWS", "SES", "Email"]
+++

Sending emails is a common feature in modern web applications, whether it's for sending notifications, newsletters, or confirmations. In ASP.NET, integrating email services can be done using various methods. AWS Simple Email Service (SES) is a reliable, scalable, and cost-effective solution for sending emails. In this blog post, we'll look into how to integrate AWS SES with an ASP.NET Core 8 API using the AWS SDK for .NET.

# What is AWS Simple Email Service (SES)?

AWS Simple Email Service (SES) is a cloud-based email sending service designed to help businesses send transactional, marketing, or bulk emails. SES integrates seamlessly with other AWS services and provides a cost-effective solution for managing email infrastructure.

## Key Features:

Scalability: AWS SES can handle large volumes of email, making it suitable for businesses of all sizes.
Reliability: SES offers high deliverability rates and integrates with Amazon’s infrastructure.
Security: SES supports DKIM, SPF, and DMARC, ensuring emails are not marked as spam.
Cost-Effectiveness: You only pay for what you use, making it budget-friendly for startups and enterprises alike.

# Why Use AWS SES with ASP.NET Core?

Using AWS SES in an ASP.NET Core application allows developers to leverage the cloud's power for sending emails without managing complex email infrastructure. This integration offers:

- **Reduced Complexity**: Simplifies email management by offloading it to a managed service.
- **High Deliverability**: Increases the chances that your emails land in inboxes rather than spam folders.
- **Flexibility**: Can be used for various email types, including transactional, promotional, and bulk emails.

# Implementing AWS SES in ASP.NET Core 8 API

## Prerequisites

- An AWS account with SES enabled.
- Verified email addresses in SES.
- An ASP.NET Core 8 API project.
- AWS SDK for .NET installed in your project.

## Step 1: Install AWS SDK for .NET

First, install the AWS SDK for .NET using NuGet. You can do this via the Package Manager Console:

```bash
Install-Package AWSSDK.SimpleEmail
```

Or via the .NET CLI:

```bash
dotnet add package AWSSDK.SimpleEmail
```

## Step 2: Configure AWS SES in ASP.NET Core

In your `appsettings.json` file, add your AWS credentials and SES region:

```json
{
  "AwsSes": {
    "Profile": "default",
    "Region": "us-east-1",
    "AccessKey": "your-access-key",
    "SecretKey": "your-secret-key"
  }
}
```

Ensure your credentials are securely stored, such as using AWS Secrets Manager or environment variables.

## Step 3: Create the Email Service

Create a service that handles sending emails. Start by defining an interface IEmailService:

```csharp
public interface IEmailService
{
    Task SendEmailAsync(string toEmail, string subject, string body);
}
```

Now, implement the IEmailService interface using AWS SES:

```csharp
using Amazon;
using Amazon.SimpleEmail;
using Amazon.SimpleEmail.Model;

public class EmailService : IEmailService
{
    private readonly IAmazonSimpleEmailService _sesClient;

    public EmailService(IAmazonSimpleEmailService sesClient)
    {
        _sesClient = sesClient;
    }

    public async Task SendEmailAsync(string toEmail, string subject, string body)
    {
        var sendRequest = new SendEmailRequest
        {
            Source = "your-verified-email@example.com",
            Destination = new Destination
            {
                ToAddresses = new List<string> { toEmail }
            },
            Message = new Message
            {
                Subject = new Content(subject),
                Body = new Body
                {
                    Html = new Content(body)
                }
            }
        };

        try
        {
            var response = await _sesClient.SendEmailAsync(sendRequest);
            Console.WriteLine($"Email sent! Message ID: {response.MessageId}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"The email wasn't sent. Error message: {ex.Message}");
        }
    }
}
```

## Step 4: Configure Dependency Injection

In your `Program.cs`, configure the dependency injection for the EmailService:

```csharp
using Amazon.SimpleEmail;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDefaultAWSOptions(builder.Configuration.GetAWSOptions());
builder.Services.AddAWSService<IAmazonSimpleEmailService>();
builder.Services.AddScoped<IEmailService, EmailService>();

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();

```

## Step 5: Using the Email Service in a Controller

You can now inject the IEmailService into your controllers and use it to send emails:

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmailController : ControllerBase
{
    private readonly IEmailService _emailService;

    public EmailController(IEmailService emailService)
    {
        _emailService = emailService;
    }

    [HttpPost("send")]
    public async Task<IActionResult> SendEmail([FromBody] EmailRequest request)
    {
        await _emailService.SendEmailAsync(request.ToEmail, request.Subject, request.Body);
        return Ok("Email sent successfully!");
    }
}

public class EmailRequest
{
    public string ToEmail { get; set; }
    public string Subject { get; set; }
    public string Body { get; set; }
}
```

# Advantages and Disadvantages of AWS SES in ASP.NET Core

| Advantages                                                                                          | Disadvantages                                                                                                    |
| --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Scalability**: Easily scales with your application's email demands.                               | **Complexity**: May require initial setup complexity, especially for small projects.                             |
| **High Deliverability**: Improves the chances of your emails being delivered to inboxes.            | **Email Sending Limits**: SES has sending limits that you might need to increase for higher volume requirements. |
| **Security**: Supports various security protocols to protect your emails from being marked as spam. | **Regional Availability**: SES is not available in all AWS regions.                                              |
| **Cost-Effective**: Pay-as-you-go pricing model helps manage costs effectively.                     |                                                                                                                  |

# When to Use AWS SES

| When to use                                                                                                          | When not to use                                                                                               |
| -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **High Volume Emails**: When your application needs to send a large volume of emails reliably.                       | **Minimal Email Needs**: For small-scale applications with minimal email requirements, SES might be overkill. |
| **Transactional Emails**: For sending important transactional emails like account activations, password resets, etc. | **Complex Email Campaigns**: For advanced email marketing campaigns, you may need more specialized tools.     |
| **Budget-Conscious Solutions**: When you need a cost-effective solution that scales with usage.                      |                                                                                                               |

# Issues, Considerations, and Best Practices

## Issues and Considerations

- **Email Deliverability**: Despite SES's high deliverability, always test your emails to ensure they are not marked as spam.
- **Sending Limits**: Monitor your sending limits and request limit increases if necessary.
- **Cost Management**: Regularly review your usage to ensure your costs are within budget.

## Best Practices

- **Use Environment Variables**: Store your AWS credentials securely using environment variables or AWS Secrets Manager.
- **Monitor Email Logs**: Use CloudWatch to monitor email sending logs for any issues.
- **Verify Email Addresses and Domains**: Always verify the email addresses and domains you intend to send from.

# Conclusion

Integrating AWS Simple Email Service with ASP.NET Core 8 is a powerful way to manage email delivery in your web applications. With the AWS SDK for .NET, sending emails becomes streamlined, reliable, and scalable. While AWS SES offers numerous benefits, it is essential to weigh its advantages against potential drawbacks to determine if it is the right fit for your application.

By following the steps outlined in this blog, you can set up and start using AWS SES in your ASP.NET Core API efficiently. Always keep best practices in mind to ensure secure, reliable, and cost-effective email delivery.

# References

- [AWS SES Documentation](https://docs.aws.amazon.com/ses/)
- [AWS SDK for .NET documentation](https://aws.amazon.com/developer/language/net/)
