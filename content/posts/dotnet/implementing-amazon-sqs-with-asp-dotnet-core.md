+++
title = 'Implementing Amazon SQS with ASP.NET Core 8 API'
date = 2024-08-25T12:13:31+01:00
draft = false
tags = ["ASP.NET Core", "AWS", "SES", "Email"]
+++

Amazon Simple Queue Service (SQS) is a fully managed message queuing service provided by AWS. It enables the decoupling of microservices, distributed systems, and serverless applications by allowing them to communicate asynchronously. This blog post will walk you through the concepts, rationale, and practical implementation of Amazon SQS using ASP.NET Core 8 API. We'll also discuss the advantages, disadvantages, best practices, and scenarios where SQS is suitable or not.

# What is Amazon SQS?

## Overview

Amazon SQS is a message queuing service that allows you to send, store, and receive messages between software components, making it easier to build scalable and decoupled systems. SQS supports two types of queues:

- **Standard Queue**: Provides maximum throughput, best-effort ordering, and at-least-once delivery. However, messages might be delivered out of order or more than once.
- **FIFO (First-In-First-Out) Queue**: Ensures messages are processed exactly once, in the order they were sent. This type of queue is ideal when the order of operations is critical.

## Key Concepts

- **Queue**: A temporary storage location for messages. You send messages to the queue and receive them from it.
- **Message**: The data sent between different components via SQS.
- **Producer**: The component that sends messages to the queue.
- **Consumer**: The component that retrieves and processes messages from the queue.
- **Visibility Timeout**: The time during which a message is invisible to other consumers after being retrieved from the queue.
- **Dead-Letter Queue (DLQ)**: A secondary queue used to store messages that could not be processed successfully after multiple attempts.

## Why Use Amazon SQS?

Amazon SQS is an excellent choice for applications that require:

- **Decoupling**: Separate the components of a distributed system to reduce dependencies and increase reliability.
- **Scalability**: Handle varying workloads without the need for manual intervention.
- **Fault Tolerance**: Queue messages for later processing if downstream components are temporarily unavailable.

# Implementing Amazon SQS with ASP.NET Core 8 API

## Prerequisites

Before diving into the implementation, ensure you have the following:

- An AWS account with IAM permissions to use SQS.
- .NET SDK 8 installed.
- An ASP.NET Core 8 project created.

## Step 1: Install AWS SDK for .NET

First, install the AWS SDK for .NET, which includes the Amazon SQS package.

```bash
dotnet add package AWSSDK.SQS
```

## Step 2: Configure AWS SQS in ASP.NET Core

Next, configure the AWS SQS client in your ASP.NET Core 8 application. You can do this in the `Program.cs` file.

```csharp
using Amazon.SQS;
using Amazon.Extensions.NETCore.Setup;

var builder = WebApplication.CreateBuilder(args);

// Add AWS services to the container.
builder.Services.AddDefaultAWSOptions(builder.Configuration.GetAWSOptions());
builder.Services.AddAWSService<IAmazonSQS>();

var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

Ensure that your `appsettings.json` includes the necessary AWS credentials:

```json
{
  "AWS": {
    "Region": "us-east-1",
    "Profile": "default"
  }
}
```

## Step 3: Send a Message to SQS

Create a service to handle sending messages to the SQS queue.

```csharp
public class SqsService
{
    private readonly IAmazonSQS _sqsClient;
    private readonly string _queueUrl;

    public SqsService(IAmazonSQS sqsClient, IConfiguration configuration)
    {
        _sqsClient = sqsClient;
        _queueUrl = configuration["AWS:QueueUrl"];
    }

    public async Task SendMessageAsync(string message)
    {
        var sendMessageRequest = new SendMessageRequest
        {
            QueueUrl = _queueUrl,
            MessageBody = message
        };

        var response = await _sqsClient.SendMessageAsync(sendMessageRequest);

        if (response.HttpStatusCode != HttpStatusCode.OK)
        {
            throw new Exception("Failed to send message to SQS");
        }
    }
}
```

## Step 4: Receive and Process Messages from SQS

Now, create a method to receive and process messages from the queue.

```csharp
public async Task ReceiveMessagesAsync()
{
    var receiveMessageRequest = new ReceiveMessageRequest
    {
        QueueUrl = _queueUrl,
        MaxNumberOfMessages = 10,
        WaitTimeSeconds = 5
    };

    var response = await _sqsClient.ReceiveMessageAsync(receiveMessageRequest);

    foreach (var message in response.Messages)
    {
        try
        {
            // Process the message
            Console.WriteLine($"Processing message: {message.Body}");

            // Delete the message after processing
            var deleteMessageRequest = new DeleteMessageRequest
            {
                QueueUrl = _queueUrl,
                ReceiptHandle = message.ReceiptHandle
            };

            await _sqsClient.DeleteMessageAsync(deleteMessageRequest);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error processing message: {ex.Message}");
            // Handle the exception (e.g., send to DLQ)
        }
    }
}
```

## Step 5: Integrate SQS Service in Your API

Finally, integrate the SQS service into your API controller.

```csharp
[ApiController]
[Route("api/[controller]")]
public class MessagesController : ControllerBase
{
    private readonly SqsService _sqsService;

    public MessagesController(SqsService sqsService)
    {
        _sqsService = sqsService;
    }

    [HttpPost]
    public async Task<IActionResult> SendMessage([FromBody] string message)
    {
        await _sqsService.SendMessageAsync(message);
        return Ok("Message sent to SQS.");
    }

    [HttpGet]
    public async Task<IActionResult> ReceiveMessages()
    {
        await _sqsService.ReceiveMessagesAsync();
        return Ok("Messages received and processed.");
    }
}
```

# Advantages of Implementing Amazon SQS with ASP.NET Core 8 API

| Advantage                                                                                                        | Disadvantage                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scalability**: SQS automatically scales to handle large volumes of messages.                                   | **Latency**: SQS introduces some latency due to the decoupling nature of message queuing.                                                            |
| **Reliability**: Messages are stored redundantly across multiple availability zones, ensuring high availability. | **Complexity**: Managing message queues, especially when dealing with dead-letter queues and visibility timeouts, can add complexity to your system. |
| **Decoupling**: SQS allows you to decouple your services, making the system more resilient to failures.          | **Limited Message Size**: SQS messages are limited to 256 KB, which might require additional handling for larger payloads.                           |
| **Cost-Effective**: You pay only for the usage, making it cost-effective for many applications.                  |                                                                                                                                                      |

# When to Use Amazon SQS with ASP.NET Core 8 API

| When to use                                                                                                                    | When not to use                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **Asynchronous Processing**: When tasks can be processed asynchronously, such as sending emails or processing background jobs. | **Real-Time Processing**: When low latency is critical, such as in real-time gaming or trading applications.                      |
| **Decoupling Services**: When you want to reduce dependencies between different services or components in your system.         | **Small Applications**: For simple applications, the overhead of managing SQS might outweigh its benefits.                        |
| **Rate Limiting**: When you need to throttle the number of requests processed by a service.                                    | **Stateful Applications**: When the state needs to be maintained across services, direct communication might be more appropriate. |

# Issues, Considerations, and Best Practices

## Issues

- **Message Duplication**: SQS might deliver a message more than once, requiring idempotent message processing.
- **Visibility Timeout**: Incorrect configuration can lead to message loss or multiple processes handling the same message.
- **Error Handling**: Messages that fail to process should be routed to a dead-letter queue to avoid blocking the processing pipeline.

## Best Practices

- **Use Dead-Letter Queues**: Set up DLQs to handle messages that fail to process after multiple attempts.
- **Implement Idempotency**: Ensure that your message processing logic is idempotent to handle message duplication gracefully.
- **Monitor and Alert**: Use AWS CloudWatch to monitor your queues and set up alerts for unusual activity, such as an increase in message age or DLQ traffic.

# Conclusion

Amazon SQS is a powerful tool for building scalable, decoupled applications, and integrating it with ASP.NET Core 8 is relatively straightforward. However, it's essential to understand its strengths and limitations to use it effectively. By following best practices and carefully considering your application's requirements, you can leverage SQS to build robust and reliable systems.

# References

- [AWS SQS Documentation](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)

- [AWS SDK for .NET documentation](https://aws.amazon.com/developer/language/net/)
