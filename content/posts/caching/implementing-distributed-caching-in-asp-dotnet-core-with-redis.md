+++
title = 'Implementing Distributed Caching in ASP.NET Core with Redis'
date = 2025-03-29T12:27:06+01:00
draft = true
+++

Caching is a critical performance optimization technique in web applications. It reduces database load, improves response times, and enhances scalability. In ASP.NET Core, distributed caching is a powerful way to store and share cached data across multiple instances of an application. Redis, an in-memory data store, is a popular choice for distributed caching.

This blog explores distributed caching in ASP.NET Core with Redis, explaining its concepts, implementation, advantages, disadvantages, best practices, and real-world use cases.

# What is Distributed Caching?

Distributed caching is a technique that stores cached data across multiple servers or nodes, making it accessible to all instances of an application. Unlike in-memory caching (which is local to a single instance), distributed caching allows multiple application servers to share the same cache, ensuring data consistency and high availability.

## Key Benefits:

- **Scalability** – Works across multiple instances of an application.
- **Data Consistency** – Cached data remains consistent across all servers.
- **Better Performance** – Reduces database load and improves response times.
- **Fault Tolerance** – Data can be persisted and recovered after restarts.

## Why Use Redis for Distributed Caching in ASP.NET Core?

Redis (Remote Dictionary Server) is an open-source, high-performance, in-memory data store. It supports various data structures (strings, hashes, lists, sets, etc.) and provides features like persistence, replication, eviction policies, and high availability, making it an excellent choice for caching.

### Key Features of Redis for Caching:

- **Fast Read/Write** – Redis stores data in memory, enabling ultra-fast retrieval.
- **Persistence** – Data can be saved to disk for durability.
- **Expiration Policies** – Supports TTL (Time-To-Live) for cache expiration.
- **Support for Clustering** – Ensures scalability and high availability.

# Implementing Distributed Caching with Redis in ASP.NET Core

### Step 1: Install Redis and Required Packages

First, install Redis on your local machine or use a cloud-based service like Azure Redis Cache or AWS ElastiCache.

To integrate Redis into an ASP.NET Core application, install the required NuGet package:

```bash
Install-Package Microsoft.Extensions.Caching.StackExchangeRedis
```

### Step 2: Running Redis with Docker for Local Development

If you don’t want to install Redis directly on your system, you can use Docker to run a Redis container. Ensure that you have Docker installed, then execute the following command to pull and run a Redis instance:

```bash
docker run --name redis -d -p 6379:6379 redis
```

This command will:

- Download the latest Redis image (if not already downloaded).
- Run Redis in a detached mode.
- Map port 6379 from the container to the local machine.

To verify if Redis is running, use:

```bash
docker ps
```

or connect to Redis CLI inside the running container:

```bash
docker exec -it redis redis-cli
```

### Step 3: Configure Redis in ASP.NET Core

Modify appsettings.json to add the Redis connection string:

```json
{
  "Redis": {
    "ConnectionString": "localhost:6379"
  }
}
```

In `Program.cs`, configure Redis as the distributed cache provider:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add Redis distributed cache
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration["Redis:ConnectionString"];
});

var app = builder.Build();
app.Run();

```

### Step 4: Using Redis in a Controller

The following example demonstrates how to cache and retrieve data using Redis in an ASP.NET Core controller:
