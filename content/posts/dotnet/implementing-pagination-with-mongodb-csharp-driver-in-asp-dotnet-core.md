+++
title = 'Implementing Pagination with MongoDB C# Driver in ASP.NET Core 8'
date = 2025-03-02T12:27:06+01:00
draft = false
tags = ["Pagination", "MongoDB", "ASP.NET Core"]
+++

Pagination is a crucial aspect of application development, especially when dealing with large datasets in databases. It ensures that data is fetched in manageable chunks, improving performance and user experience. In this blog, we’ll explore how to implement pagination using the MongoDB C# driver in an ASP.NET Core 8 API project. We’ll cover the key concepts, functions, and best practices while discussing when and when not to use pagination.

# What is Pagination?

Pagination refers to the process of dividing a dataset into discrete pages. When querying large databases, fetching all records at once can be resource-intensive. Pagination helps limit the number of records returned, making the system more scalable, responsive, and user-friendly.

In MongoDB, pagination is commonly implemented using a combination of the `skip` and `limit` functions in conjunction with sorting mechanisms to ensure the data is retrieved in a consistent order.

## Why Pagination?

- **Performance**: Querying large datasets can cause performance bottlenecks. Pagination limits the number of records returned at a time, improving API response time.

- **Usability**: Returning large amounts of data in a single response can overwhelm the user interface. Pagination enhances user experience by showing data in smaller, manageable chunks.

- **Scalability**: As datasets grow, pagination helps scale the system efficiently by controlling resource utilization.

# Key Concepts of Pagination with MongoDB C\# Driver

- **`skip`**: The `skip` function is used to skip a certain number of records in a dataset. This is helpful when moving to subsequent pages.
- **`limit`**: The `limit` function is used to specify the maximum number of records to return in a query result. It defines the size of the page.
- **Sorting**: Pagination often requires a stable sort order to ensure consistency across pages. Without sorting, records may be returned in arbitrary order, which can lead to inconsistencies when paginating.
- **Filters**: You can apply filtering conditions to ensure that the paginated results meet specific criteria before being returned.
- **Page Number and Page Size**: These parameters dictate which page of the data is returned and how many records should be included on each page.

# Implementing Pagination in ASP.NET Core with MongoDB C\# Driver

Let’s walk through a step-by-step implementation of pagination in a sample ASP.NET Core 8 API project that uses MongoDB as the data store.

## Setting up MongoDB and ASP.NET Core 8

### Step 1: Install Dependencies

First, install the necessary NuGet packages:

```bash
dotnet add package MongoDB.Driver
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
```

### Step 2: Set Up MongoDB in ASP.NET Core

You’ll need to configure MongoDB in your `appsettings.json` and register the MongoDB client in the `Program.cs` file.

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

### Step 3: Create the MongoDB Repository

Create a repository to handle the MongoDB interaction logic, including paginated queries.

**MongoDBRepository.cs**:

```csharp
public class MongoDBRepository<T>
{
    private readonly IMongoCollection<T> _collection;

    public MongoDBRepository(IMongoClient client, string dbName, string collectionName)
    {
        var database = client.GetDatabase(dbName);
        _collection = database.GetCollection<T>(collectionName);
    }

    public async Task<List<T>> GetPaginatedAsync(int pageNumber, int pageSize, FilterDefinition<T>? filter = null, SortDefinition<T>? sort = null)
    {
        if (filter == null)
            filter = Builders<T>.Filter.Empty;

        if (sort == null)
            sort = Builders<T>.Sort.Ascending("_id"); // Ensure a stable sort order

        return await _collection
            .Find(filter)
            .Sort(sort)
            .Skip((pageNumber - 1) * pageSize)
            .Limit(pageSize)
            .ToListAsync();
    }

    public async Task<long> GetTotalCountAsync(FilterDefinition<T>? filter = null)
    {
        if (filter == null)
            filter = Builders<T>.Filter.Empty;

        return await _collection.CountDocumentsAsync(filter);
    }
}
```

### Step 4: Create an API Controller for Pagination

Now, let's create a controller that exposes an API endpoint for paginated data retrieval.

**SampleController.cs**:

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
    public async Task<IActionResult> GetPaginatedData([FromQuery] int pageNumber = 1, [FromQuery] int pageSize = 10)
    {
        if (pageNumber <= 0 || pageSize <= 0)
        {
            return BadRequest("Page number and page size must be greater than 0.");
        }

        var data = await _repository.GetPaginatedAsync(pageNumber, pageSize);
        var totalRecords = await _repository.GetTotalCountAsync();

        var response = new
        {
            TotalRecords = totalRecords,
            PageNumber = pageNumber,
            PageSize = pageSize,
            Data = data
        };

        return Ok(response);
    }
}
```

In this controller, we expose a GET endpoint /api/sample/paginated that accepts two query parameters: pageNumber and pageSize. The response includes the total number of records, the current page, the page size, and the actual data returned by MongoDB.

### Step 5: Run the Application

Now that the implementation is complete, we can run the application and query the paginated data.

**Example query**:

```bash
GET http://localhost:5000/api/sample/paginated?pageNumber=2&pageSize=5

```

This would return the second page of data with 5 records per page.

# Advantage and Disadvantage of Implementing Pagination with MongoDB C# Driver

## Advantage

| Advantage                                                                                                                             | Disadvantage                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ✅ Performance Optimization\*: Fetching a limited number of records at a time reduces the load on both the server and the client.     | ❌ Complexity: Pagination introduces additional complexity into your queries, requiring careful management of page numbers and sizes.                                                              |
| ✅ Memory Efficiency: By avoiding large result sets, you minimize memory consumption, improving the efficiency of your API.           | ❌ Inconsistent Data: When working with dynamic datasets (frequent inserts/updates), pages can become inconsistent as new records might shift previously retrieved pages.                          |
| ✅ Improved User Experience\*\*: End-users get a faster, more responsive experience, with data presented in small, manageable chunks. | ❌ Performance with Skip : MongoDB’s skip function can be inefficient for large offsets, especially as the number of skipped documents increases, since MongoDB must walk through skipped records. |

# When to Use Pagination

- **Large Datasets**: If your application deals with a large dataset where returning all records at once would be impractical.

- **User Interfaces**: For interfaces where users browse data in lists or tables and need the ability to navigate through pages.

- **APIs Serving Multiple Clients**: In situations where different clients may need different subsets of data (e.g., mobile apps, web apps).

# When Not to Use Pagination

- **Small Datasets**: If your dataset is small, pagination may introduce unnecessary complexity.

- **Real-Time Data**: When working with real-time systems where data is frequently updated, pagination may lead to inconsistent results.

# Issues, Considerations, and Best Practices

- **Use Indexing**: Indexing improves query performance.

- **Efficient Pagination**: Use skip and limit for small to moderate datasets. For larger datasets, consider alternatives like range queries using indexed fields (e.g., \_id or date ranges) to avoid the performance cost of skip.

- **Sort Consistency**: Always ensure your query is sorted by a unique field (e.g., \_id). Without sorting, pages can yield inconsistent results, especially if records are inserted or deleted during navigation.

- **Page Size Limits**: Set reasonable limits for the pageSize to avoid fetching too much data at once. For example, limit pageSize to 100 records per page.

- **Caching**: Consider caching frequent queries to reduce the load on your MongoDB instance, especially for high-traffic applications.

- **Handling Changes**: If your dataset changes frequently, consider implementing mechanisms to track consistency (e.g., using timestamps) or use snapshotting techniques to avoid inconsistent pagination.

# Conclusion

Implementing pagination in MongoDB using the C# driver within an ASP.NET Core application enhances efficiency and scalability. While the `skip` and `limit` approach works well for small to moderate datasets, alternative techniques like range-based pagination should be considered for larger datasets.

By following best practices, you can ensure a performant and reliable pagination system that enhances both user experience and backend performance.
