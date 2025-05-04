+++
title = 'Implementing Range-Based Pagination With MongoDB C# Driver in ASP.NET Core 8'
date = 2025-03-08T12:27:06+01:00
draft = false
tags = ["Pagination", "MongoDB", "ASP.NET Core"]
+++

Pagination is essential for handling large datasets efficiently in web applications. While traditional offset-based pagination (using skip and limit) works well for small datasets, it can become inefficient for large collections due to performance degradation. Range-based pagination, also known as keyset pagination, is a more performant alternative that uses indexed fields (like timestamps or IDs) to retrieve results.

This blog post will explain how to implement Range-Based Pagination with the MongoDB C# driver in ASP.NET Core 8, covering:

- What Range-Based Pagination is and why it is needed.
- Key concepts, functions, and implementation details.
- A practical C# implementation in an ASP.NET Core 8 API.
- Advantages, disadvantages, and best practices.

# What is Range-Based Pagination?

Range-Based Pagination retrieves a limited number of records based on a range condition rather than skipping a certain number of records. Instead of using skip, we use a continuation token (e.g., a timestamp or an ID from the last retrieved document) to get the next batch.

## Why Range-Based Pagination

- **Performance Optimization**: Unlike offset-based pagination, range queries use indexes efficiently, making queries faster.
- **Scalability**: Works well with large datasets by avoiding costly skip operations.
- **Consistent Ordering**: Prevents inconsistencies when new records are inserted or removed.
- **Reduced Latency**: Provides faster response times for paginated queries.

## Key Concepts of Range-Based Pagination

- **Cursor/Continuation Token**: Instead of a numeric page index, a cursor (such as a timestamp or document ID) is used to determine the starting point for the next page.

**Indexed Sorting**: Sorting by an indexed field ensures efficient querying and consistency in pagination.

**Filtering and Ordering**: Results are filtered based on the cursor, ensuring only newer (or older) records are retrieved.

- **Page Size**: Specifies the number of records retrieved per request.

# Implementing Range-Based Pagination in ASP.NET Core 8

## Step 1: Install Dependencies

Ensure you have the required NuGet packages:

```bash
dotnet add package MongoDB.Driver
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
```

## Step 2: Configure MongoDB in ASP.NET Core

**appsettings.json**

```json
{
  "MongoDBSettings": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "SampleDB"
  }
}
```

**Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<MongoDBSettings>(
    builder.Configuration.GetSection("MongoDBSettings"));

builder.Services.AddSingleton<IMongoClient, MongoClient>(sp =>
{
    var settings = sp.GetRequiredService<IOptions<MongoDBSettings>>().Value;
    return new MongoClient(settings.ConnectionString);
});

builder.Services.AddControllers()
    .AddNewtonsoftJson(options =>
        options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore);

var app = builder.Build();
app.MapControllers();
app.Run();
```

## Step 3: Create MongoDB Repository

```csharp
public class MongoDBRepository<T>
{
    private readonly IMongoCollection<T> _collection;

    public MongoDBRepository(IMongoClient client, string dbName, string collectionName)
    {
        var database = client.GetDatabase(dbName);
        _collection = database.GetCollection<T>(collectionName);
    }

    public async Task<List<T>> GetPaginatedAsync(DateTime? lastSeen, int pageSize, SortDefinition<T> sort)
    {
        var filter = lastSeen == null ? Builders<T>.Filter.Empty : Builders<T>.Filter.Gt("CreatedAt", lastSeen);

        return await _collection
            .Find(filter)
            .Sort(sort)
            .Limit(pageSize)
            .ToListAsync();
    }
}
```

## Step 4: Create API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class SampleController : ControllerBase
{
    private readonly MongoDBRepository<MyDataModel> _repository;

    public SampleController(MongoDBRepository<MyDataModel> repository)
    {
        _repository = repository;
    }

    [HttpGet("paginated")]
    public async Task<IActionResult> GetPaginatedData([FromQuery] DateTime? lastSeen, [FromQuery] int pageSize = 10)
    {
        if (pageSize <= 0)
        {
            return BadRequest("Page size must be greater than 0.");
        }

        var sort = Builders<MyDataModel>.Sort.Ascending("CreatedAt");
        var data = await _repository.GetPaginatedAsync(lastSeen, pageSize, sort);

        return Ok(new
        {
            NextCursor = data.Any() ? data.Last().CreatedAt : null,
            Data = data
        });
    }
}
```

# Advantages and Disadvantage of Range-Based Pagination

| Advantage                                                                           | Disadvantage                                                                                        |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| ✅ High Performance: Uses indexed queries instead of costly skip operations.        | ❌ No Random Access: Cannot jump to arbitrary pages like offset-based pagination.                   |
| ✅ Consistency: Data order remains stable, even when new records are added.         | ❌ Complex Implementation: Requires careful handling of continuation tokens.                        |
| ✅ Scalability: Handles large datasets efficiently without performance degradation. | ❌ Strict Sorting Requirement: Data must always be sorted by the indexed field used for pagination. |

# When to Use Range-Based Pagination

- **Large Datasets**: Suitable for applications dealing with massive collections.
- **Real-Time Data Feeds**: Ensures consistent and performant updates in applications like social media feeds.
- **APIs with Infinite Scrolling**: Ideal for endless scrolling interfaces, as seen in mobile apps.

# When Not to Use Range-Based Pagination

- **Small Datasets**: Offset-based pagination may be simpler and sufficient.
- **Random Page Access Required**: If users need to jump to arbitrary pages, offset-based pagination is a better fit.

# Best Practices and Considerations

- **Use Indexed Fields**: Ensure the cursor field (e.g., CreatedAt) is indexed for optimal performance.
- **Consistent Sorting**: Always sort results to maintain consistent pagination.
- **Limit Page Size**: Set reasonable page size limits to avoid overwhelming the server.
- **Handle Edge Cases**: Ensure robust handling when reaching the last page.
- **Cache Results**: Consider caching frequently accessed data for further optimization.
