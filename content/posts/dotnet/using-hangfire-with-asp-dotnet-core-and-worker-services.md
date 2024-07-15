+++
title = 'Using Hangfire with ASP.NET Core and Worker Services'
date = 2024-07-15T11:05:40+01:00
draft = false
tags = [".NET Core", "Scheduling"]
+++

In modern web development, background tasks are essential for offloading time-consuming operations, such as sending emails, processing files, or performing scheduled maintenance. Hangfire is a robust library for handling background jobs in .NET applications. This blog will look into using Hangfire with ASP.NET Core and worker services, covering its concepts, implementation, and best practices.

# What is Hangfire?

Hangfire is an open-source library that simplifies the creation and management of background jobs in .NET applications. It allows you to perform fire-and-forget, delayed, recurring, and continuation tasks with ease.

## Main Concepts

- **Background Jobs**: Tasks that run in the background, such as sending emails or processing files.
- **Recurring Jobs**: Jobs that run on a scheduled basis, like cron jobs.
- **Delayed Jobs**: Jobs that are scheduled to run after a specific delay.
- **Continuation Jobs**: Jobs that are triggered after the completion of a parent job.

## Why Use Hangfire?

- **Easy Integration**: Hangfire integrates seamlessly with ASP.NET Core applications.
- **Persistent Storage**: It uses a variety of storage options like SQL Server, Redis, and more, ensuring job persistence.
- **Dashboard**: A built-in dashboard for monitoring job execution and status.
- **Scalability**: Supports distributed systems and can scale horizontally.

On the other hand, it brings:

- **Resource Intensive**: Requires a database for storing jobs, which can add overhead.
- **Complexity**: May introduce complexity for simple background tasks that do not require persistent storage or monitoring.

# Implementing Hangfire with ASP.NET Core and Worker Services

## Prerequisites

- .NET Core SDK (version 8.0 or later)
- An ASP.NET Core project

## Step-by-Step Implementation

### 1. Install Hangfire NuGet Package

```bash
dotnet add package Hangfire
dotnet add package Hangfire.AspNetCore
```

### 2. Configure Hangfire in Startup.cs

```csharp
using Hangfire;
using Hangfire.SqlServer;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Add Hangfire services
        services.AddHangfire(configuration => configuration
            .SetDataCompatibilityLevel(CompatibilityLevel.Version_170)
            .UseSimpleAssemblyNameTypeSerializer()
            .UseRecommendedSerializerSettings()
            .UseSqlServerStorage(Configuration.GetConnectionString("HangfireConnection"), new SqlServerStorageOptions
            {
                CommandBatchMaxTimeout = TimeSpan.FromMinutes(5),
                SlidingInvisibilityTimeout = TimeSpan.FromMinutes(5),
                QueuePollInterval = TimeSpan.Zero,
                UseRecommendedIsolationLevel = true,
                UsePageLocksOnDequeue = true,
                DisableGlobalLocks = true
            }));

        // Add the processing server as IHostedService
        services.AddHangfireServer();
    }

    public void Configure(IApplicationBuilder app, IBackgroundJobClient backgroundJobs, IRecurringJobManager recurringJobs)
    {
        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
            endpoints.MapHangfireDashboard();
        });

        backgroundJobs.Enqueue(() => Console.WriteLine("Hello, Hangfire!"));
    }
}
```

### 3. Create a Worker Service

```csharp
public class BackgroundWorker : BackgroundService
{
    private readonly IBackgroundJobClient _backgroundJobClient;
    private readonly IRecurringJobManager _recurringJobManager;

    public BackgroundWorker(IBackgroundJobClient backgroundJobClient, IRecurringJobManager recurringJobManager)
    {
        _backgroundJobClient = backgroundJobClient;
        _recurringJobManager = recurringJobManager;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // Example of a fire-and-forget job
        _backgroundJobClient.Enqueue(() => SendEmail());

        // Example of a recurring job
        _recurringJobManager.AddOrUpdate("daily-job", () => PerformDailyTask(), Cron.Daily);

        await Task.CompletedTask;
    }

    public void SendEmail()
    {
        // Logic for sending an email
        Console.WriteLine("Sending email...");
    }

    public void PerformDailyTask()
    {
        // Logic for a daily task
        Console.WriteLine("Performing daily task...");
    }
}
```

### 4. Register the Worker Service

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            })
            .ConfigureServices(services =>
            {
                services.AddHostedService<BackgroundWorker>();
            });
}
```

# When to Use Hangfire

- **Long-Running Tasks**: Ideal for tasks that take a significant amount of time to complete.
- **Scheduled Tasks**: Perfect for recurring jobs that need to run on a schedule.
- **Persistent Jobs**: When job persistence and monitoring are required.

# When Not to Use Hangfire

- **Simple Tasks**: For very simple b- ackground tasks, using a lightweight solution like IHostedService might be more appropriate.
- **Resource Constraints**: In environments with limited resources, the overhead of Hangfire might not be justified.

# Issues, Considerations, and Best Practices

## Issues

- **Database Dependency**: Hangfire's reliance on a database can be a point of failure if the database becomes unavailable.
- **Complexity**: The additional complexity may not be warranted for simple use cases.

## Considerations

- **Scalability**: Plan for horizontal scaling to handle increased job loads.
- **Error Handling**: Implement robust error handling and retry mechanisms.
- **Security**: Secure the Hangfire dashboard and job execution environment to prevent unauthorized access.

## Best Practices

- **Use Dependency Injection**: Leverage ASP.NET Core's DI system for injecting dependencies into job methods.
- **Monitor Resource Usage**: Regularly monitor the performance and resource usage of Hangfire to ensure it does not negatively impact your application.
- **Secure the Dashboard**: Protect the Hangfire dashboard with proper authentication and authorization.

# Conclusion

Hangfire provides a powerful and flexible solution for managing background jobs in ASP.NET Core applications. By integrating Hangfire with ASP.NET Core and worker services, you can leverage its robust features to handle complex background processing tasks. However, it's essential to weigh the advantages against the potential drawbacks and ensure it's the right fit for your specific needs.

# References

- [Hangfire Documentation](https://docs.hangfire.io/en/latest/)
