+++
title = 'Implementing PostgreSQL with ASP.NET Core'
date = 2024-10-13T18:01:09+01:00
draft = true
+++

PostgreSQL is a robust, open-source relational database management system (RDBMS) that has gained significant traction in modern web applications. It is known for its powerful features, high compliance with SQL standards, and flexibility in handling complex queries and data types. With ASP.NET Core 8, developers can integrate PostgreSQL seamlessly into their web applications, leveraging its performance and reliability to build scalable, high-performing systems.

This blog covers the integration of PostgreSQL with ASP.NET Core 8, including an overview of PostgreSQL, a comparison with other databases like SQL Server and MongoDB, and the step-by-step implementation of a CRUD API project. We will also look into best practices and recommendations for building scalable applications using PostgreSQL and ASP.NET Core 8.

# What is PostgreSQL?

PostgreSQL is an open-source object-relational database system (ORDBMS) with a strong emphasis on standards compliance and extensibility. Initially released in 1996, it has since become one of the most reliable and versatile database systems. It supports both SQL (relational) and JSON (non-relational) queries, making it suitable for handling structured and unstructured data.

# Why PostgreSQL?

- **Standards Compliance**: PostgreSQL adheres to the SQL standard (SQL:2011) and provides extensions that allow for advanced operations.
- **Extensibility**: It supports custom data types, functions, and operators, allowing developers to build more complex and domain-specific logic directly in the database.
- **Reliability**: PostgreSQL is known for ACID compliance (Atomicity, Consistency, Isolation, Durability), ensuring data integrity and reliable transactions.
- **Performance**: With features like advanced indexing techniques (e.g., GiST, GIN), parallel query processing, and robust optimization strategies, PostgreSQL is a performant choice for many applications.

# Problems PostgreSQL Solves

- **Complex data relationships**: PostgreSQL is ideal for applications with intricate data relationships, such as e-commerce, financial systems, and social networks.
- **High availability and replication**: It supports various replication methods, including streaming replication and logical replication, for fault tolerance and load balancing.
- **Full-text search**: PostgreSQL offers built-in support for full-text search, eliminating the need for external systems for indexing and searching text data.

# Comparing PostgreSQL, SQL Server, and MongoDB

| Feature            | PostgreSQL                                       | SQL Server              | MongoDB                         |
| ------------------ | ------------------------------------------------ | ----------------------- | ------------------------------- |
| Type               | Relational, Object-Relational                    | Relational              | NoSQL (Document-Oriented)       |
| Transactions       | ACID-compliant                                   | ACID-compliant          | Not natively ACID (uses BASE)   |
| Schema             | Flexible with support for NoSQL                  | Schema-bound            | Schema-less                     |
| JSON Support       | Native JSONB storage & querying                  | Limited JSON support    | Native JSON support             |
| Horizontal Scaling | Limited (but improving with sharding)            | Limited                 | Designed for horizontal scaling |
| Use Case           | Complex data relationships, hybrid (SQL & NoSQL) | Enterprise applications | Big data, real-time analytics   |

# Implementing PostgreSQL with ASP.NET Core 8

Now that we understand PostgreSQL and its advantages, let's integrate it into an ASP.NET Core 8 API project. We will implement a CRUD (Create, Read, Update, Delete) API using the Repository Pattern, CQRS (Command Query Responsibility Segregation) pattern, MediatR for handling requests, and AutoMapper for mapping between data models and DTOs.

## Setting Up the Environment

- **Install PostgreSQL**: Make sure you have PostgreSQL installed on your machine or use a cloud provider like Amazon RDS or Azure PostgreSQL.

- **Create a New ASP.NET Core Project**:

```bash
dotnet new webapi -n PostgresDemoAPI
```

- **Install NuGet Packages**: Install the required packages using the following commands:

```bash
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package MediatR
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

- **Configuring PostgreSQL in ASP.NET Core**
  In the `appsettings.json`, configure the PostgreSQL connection string:

```json
{
  "ConnectionStrings": {
    "PostgresConnection": "Host=localhost;Database=PostgresDemoDB;Username=postgres;Password=yourpassword"
  }
}
```

In `Program.cs` or `Startup.cs`, add the PostgreSQL context configuration:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(Configuration.GetConnectionString("PostgresConnection")));

        services.AddMediatR(typeof(Startup));  // Register MediatR
        services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());  // Register AutoMapper
        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseEndpoints(endpoints => { endpoints.MapControllers(); });
    }
}
```

## Creating the Models and DbContext

Create an entity model Product and configure the `AppDbContext`:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products { get; set; }
}
```

## Implementing the Repository Pattern

Create an interface for the repository:

```csharp
public interface IProductRepository
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product> GetByIdAsync(int id);
    Task<Product> AddAsync(Product product);
    Task<Product> UpdateAsync(Product product);
    Task DeleteAsync(int id);
}
```

### Implement the repository:

```csharp
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Product>> GetAllAsync()
    {
        return await _context.Products.ToListAsync();
    }

    public async Task<Product> GetByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }

    public async Task<Product> AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }

    public async Task<Product> UpdateAsync(Product product)
    {
        _context.Products.Update(product);
        await _context.SaveChangesAsync();
        return product;
    }

    public async Task DeleteAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product != null)
        {
            _context.Products.Remove(product);
            await _context.SaveChangesAsync();
        }
    }
}
```
