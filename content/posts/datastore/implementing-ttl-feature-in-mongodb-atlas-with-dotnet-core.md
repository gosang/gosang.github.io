+++
title = 'Implementing TTL (Time To Live) Feature in Mongodb Atlas With Dotnet Core'
date = 2024-01-13T12:42:54Z
draft = false
series = "Data Store"
tags = ["TTL", "ASP.Net Core", "MongoDB"]
+++

# Introduction

MongoDB, a popular NoSQL database, offers a unique feature called Time to Live (TTL), which allows documents to automatically expire after a specified period. In this blog post, we will explore the concept of TTL, its rationale, and how to implement it in an ASP.NET Core application using the MongoDB driver. We will use an e-commerce system as a case study to demonstrate the practical application of TTL in the context of microservices.

# Understanding Time to Live (TTL)

## What is TTL?

Time to Live (TTL) is a mechanism that enables documents in a MongoDB collection to have a limited lifespan. When a TTL index is applied to a field in a document, MongoDB automatically removes documents where the TTL field value is older than the specified duration.

## Why TTL and Problem Resolution

TTL is particularly useful for scenarios where data has a finite relevance or validity period. In an e-commerce system, for instance, order or shopping cart documents may become irrelevant after a certain time, reducing database clutter and improving performance.

# Ways to Implement TTL in MongoDB

## 1. Expire Documents

The simplest way to use TTL is to let MongoDB expire documents after a certain number of seconds. This method is suitable when documents have a fixed lifespan, such as session data or temporary records.

```csharp
var keys = Builders<BsonDocument>.IndexKeys.Ascending("expireAt");
var options = new CreateIndexOptions { ExpireAfter = new TimeSpan(0, 0, 0, 3600) };
collection.Indexes.CreateOne(new BsonDocumentIndex(keys, options));
```

## 2. Expire Documents with Filter Conditions

For more granular control, TTL can be applied based on specific filter conditions. This is beneficial when different documents in the same collection have varying lifespans.

```csharp
var filter = Builders<BsonDocument>.Filter.Eq("status", "expired");
var keys = Builders<BsonDocument>.IndexKeys.Ascending("expireAt");
var options = new CreateIndexOptions { ExpireAfter = new TimeSpan(0, 0, 0, 0) };
collection.Indexes.CreateOne(new BsonDocumentIndex(keys, options));
```

## 3. Expire Documents at a Specific Clock Time

In some scenarios, it might be necessary to expire documents at a specific clock time rather than a duration. This can be achieved using the expireAfterSeconds option with a negative value.

```csharp
var keys = Builders<BsonDocument>.IndexKeys.Ascending("expireAt");
var options = new CreateIndexOptions { ExpireAfter = new TimeSpan(0, 0, 0, -1) };
collection.Indexes.CreateOne(new BsonDocumentIndex(keys, options));
```

# Implementation in ASP.NET Core E-commerce System

In our ASP.NET Core e-commerce system, we'll illustrate the implementation of the TTL feature using MongoDB Atlas as the database. Assume an example use case where order documents should expire one hour after their creation.

## Example Use Case

In an e-commerce system, consider the need to automatically remove orders that have not been completed within a specific timeframe. The TTL feature will be applied to the order documents, ensuring they expire and are removed from the database after one hour.

Repository C# Code

```csharp
// Order Model representing MongoDB document
public class Order
{
    public ObjectId Id { get; set; }
    public string OrderNumber { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime ExpireAt { get; set; }
    // Other order properties...
}

// OrderRepository responsible for interacting with MongoDB
public class OrderRepository
{
    private readonly IMongoCollection<Order> _collection;
    private readonly TimeSpan _orderTtl;

    public OrderRepository(IMongoClient client, IConfiguration configuration, IOptions<OrderRepositorySettings> settings)
    {
        var databaseName = configuration["MongoDB:DatabaseName"];
        var collectionName = "orders";

        var database = client.GetDatabase(databaseName);
        _collection = database.GetCollection<Order>(collectionName);

        // Retrieve TTL from app settings
        _orderTtl = settings.Value.OrderTtl;

        // TTL index creation
        var keys = Builders<Order>.IndexKeys.Ascending("expireAt");
        var options = new CreateIndexOptions { ExpireAfter = _orderTtl };
        _collection.Indexes.CreateOne(new BsonDocumentIndex(keys, options));
    }

    // Method to create a new order
    public void CreateOrder(Order order)
    {
        // Set expiration time
        order.ExpireAt = order.CreatedAt.Add(_orderTtl);

        // Insert order into MongoDB
        _collection.InsertOne(order);
    }

    // Other repository methods...
}

// OrderRepositorySettings class to read TTL settings from app settings
public class OrderRepositorySettings
{
    public TimeSpan OrderTtl { get; set; }
}
```

In this code snippet, the `OrderRepository` class includes the creation of the TTL index, setting the expiration time for orders, and inserting orders into the MongoDB collection. The `OrderRepositorySettings` class is used to read the TTL value from the app settings.

## Unit Tests using xUnit and Testcontainers

To ensure the correctness of the repository code, let's create unit tests using xUnit and Testcontainers. These tests will utilize a MongoDB container for testing purposes.

```csharp
public class OrderRepositoryTests : IClassFixture<MongoDbFixture>
{
    private readonly IMongoClient _client;
    private readonly OrderRepository _repository;

    public OrderRepositoryTests(MongoDbFixture fixture)
    {
        _client = fixture.Client;
        var configuration = new ConfigurationBuilder().Build(); // Add any necessary configurations for testing
        var options = Options.Create(new OrderRepositorySettings { OrderTtl = TimeSpan.FromHours(1) });

        _repository = new OrderRepository(_client, configuration, options);
    }

    [Fact]
    public void CreateOrder_ShouldInsertOrderWithExpiration()
    {
        // Arrange
        var order = new Order { OrderNumber = "123", CreatedAt = DateTime.Now };

        // Act
        _repository.CreateOrder(order);

        // Assert
        var retrievedOrder = _client.GetDatabase("test").GetCollection<Order>("orders").Find(o => o.OrderNumber == "123").FirstOrDefault();
        Assert.NotNull(retrievedOrder);
        Assert.Equal(order.ExpireAt, retrievedOrder.ExpireAt);
    }

    // Other test methods...
}
```

In the `OrderRepositoryTests` class, we use the `MongoDbFixture` to set up a MongoDB container for testing. The `CreateOrder_ShouldInsertOrderWithExpiration` test method validates that the order is inserted into the database with the correct expiration time.

This approach ensures that the repository code, including the TTL feature, works as expected in a controlled testing environment.

## Verifying the TTL Implementation in MongoDB Atlas Database

After implementing the TTL feature in your ASP.NET Core e-commerce system using MongoDB Atlas, it's crucial to verify that the TTL functionality works as expected. Here's a succinct guide on how to confirm the correct functioning of the example use case.

### Prerequisites

1. **MongoDB Atlas Account**: Ensure you have access to a MongoDB Atlas account with a configured cluster.
2. **Connection String**: Obtain the connection string for your MongoDB Atlas cluster.

### Steps to Verify TTL Implementation

1. **Connect to MongoDB Atlas Cluster**:

   - Use the MongoDB shell or a tool like MongoDB Compass to connect to your MongoDB Atlas cluster using the provided connection string.

2. **Select Database and Collection**:
   - Choose the database and collection where your orders are stored. In the case of the e-commerce system, this would be the "orders" collection in the selected database.

```sh
use YourDatabaseName
```

3. **Check TTL Index**:

   - Confirm that the TTL index has been created on the "expireAt" field.

```sh
db.orders.getIndexes()
```

- Ensure that an index similar to the following is present:

```json
{
  "v": 2,
  "key": {
    "expireAt": 1
  },
  "name": "expireAt_1",
  "expireAfterSeconds": 3600,
  "background": true
}
```

- The `expireAfterSeconds` value should match the TTL duration set in your application.

4. **Insert Test Order**:

   - Insert a test order into the "orders" collection using the MongoDB shell or your preferred tool.

```sh
db.orders.insertOne({
    "orderNumber": "test123",
    "createdAt": ISODate("2024-01-15T12:00:00Z"),
    "expireAt": ISODate("2024-01-15T13:00:00Z")
})
```

- Adjust the timestamps based on your current time and the TTL duration.

5. **Check Order Expiration**:

   - Monitor the "orders" collection to verify that the test order is automatically removed after the TTL duration.

   ```sh
   db.orders.find()
   ```

   - Confirm that the test order is no longer present in the collection after the specified TTL duration has elapsed.

### Verification Summary

By following these steps, you can ensure that the TTL feature in MongoDB Atlas, as implemented in your ASP.NET Core e-commerce system, is working correctly. The TTL index, insertion of orders with expiration times, and automatic removal of expired orders should align with your expectations.

This verification process is crucial for maintaining data integrity and confirms that the TTL functionality effectively manages the lifecycle of documents in your MongoDB Atlas database.

# Advantages and Disadvantages

## Advantages

- **Automated Data Cleanup**: TTL automates the process of removing stale data, reducing the need for manual cleanup tasks.
- **Improved Performance**: Eliminating expired data enhances database performance and reduces query times.

## Disadvantages

- **Overhead**: Maintaining TTL indexes incurs additional overhead on the database, impacting write performance.
- **Not Suitable for All Data**: TTL is best suited for data with a clear expiration, and its implementation may not be optimal for all use cases.

# Issues and Considerations

- **Index Size**: The TTL index can grow significantly, affecting the overall database size.
- **Impact on Write Performance**: Frequent updates to documents with TTL indexes may result in increased write latencies.

# When to Use and When Not to Use TTL

Use TTL When:

- **Data has a Finite Lifespan**: When documents in your collection have a clear expiration.
- **Automated Cleanup is Preferred**: For scenarios where manual data cleanup is impractical or inefficient.

Avoid TTL When:

- **Data Retention is Critical**: If retaining historical data is essential, TTL may not be suitable.
- **Performance Impact is a Concern**: In write-heavy applications where the overhead of maintaining TTL indexes could impact performance.

# Best Practices

- **Monitor Index Size**: Regularly monitor the size of TTL indexes to ensure they do not become a performance bottleneck.
- **Choose Appropriate Expiration Strategy**: Select the TTL expiration strategy based on the nature of the data in your collection.
- **Consider Data Archiving**: For scenarios requiring historical data, implement a data archiving strategy alongside or instead of TTL.

# Conclusion

Implementing Time to Live (TTL) in MongoDB with ASP.NET Core can significantly enhance the efficiency of your applications, particularly in scenarios where data has a limited relevance period. However, careful consideration of the nature of the data, potential performance impacts, and adherence to best practices is crucial for successful implementation. By following the outlined guidelines and utilizing the provided code snippets, you can leverage MongoDB's TTL feature effectively in your ASP.NET Core applications.

# References:

[MongoDB Documentation - TTL Indexes](https://www.mongodb.com/docs/manual/core/index-ttl/#ttl-indexes)
[MongoDB .NET Driver Documentation](https://www.mongodb.com/docs/drivers/csharp/current/#mongodb-c--driver)
[MongoDB Compass](https://www.mongodb.com/products/tools/compass)
[AutoMapper Documentation](https://www.mongodb.com/docs/drivers/csharp/current/#mongodb-c--driver)
[Testcontainers Documentation](https://dotnet.testcontainers.org/)
[xUnit Documentation](https://xunit.net/)
