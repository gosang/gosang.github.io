+++
title = 'Implementing Dapper With ASP.Core 6'
date = 2024-01-21T16:58:14Z
draft = false
series = "Data Store"
tags = ["Dapper", "ASP.NET Core"]
+++

# Introduction

In the realm of data access in ASP.NET Core 6, developers often face choices between various Object-Relational Mapping (ORM) tools. One such tool that has gained popularity is Dapper. In this blog post, we will look into what Dapper is, its rationale, and how it can be effectively implemented in an ASP.NET Core 6 E-Commerce system.

# Understanding Dapper

## What is Dapper?

Dapper is a simple, lightweight, and high-performance micro ORM developed by the creators of Stack Overflow. Unlike Entity Framework (EF) Core 6, Dapper focuses on providing a minimalistic set of features to map SQL queries to .NET objects without introducing much abstraction.

## Why Dapper

Dapper was created to address the limitations and performance bottlenecks observed in full-fledged ORMs like EF Core 6. Its primary goal is to provide a fast, efficient, and easy-to-use data access layer for applications that require direct control over SQL queries and performance.

## Problems Dapper Resolves

- **Performance Optimization**: Dapper excels in scenarios where raw SQL performance is critical, outperforming ORMs like EF Core 6 due to its lightweight nature.
- **Control over Queries**: Developers often need to write complex SQL queries, and Dapper allows them to execute these queries directly without significant abstraction.
- **Reduced Abstraction Overhead**: Dapper minimizes the abstraction layer between the application and the database, leading to faster execution times and reduced memory footprint.

## Dapper vs. EF Core 6

While both Dapper and EF Core 6 serve as ORMs, they have different use cases:

1. EF Core 6 is feature-rich and abstracts away much of the database interaction, making it easier to use but potentially slower in certain scenarios.
2. Dapper is more hands-on, providing direct control over SQL queries and delivering superior performance for read-heavy operations.

# Implementing Dapper in ASP.NET Core 6

## E-Commerce System Use Case

In an E-Commerce system built on ASP.NET Core 6, efficient data access is crucial for managing product information, customer data, and order processing. Microservices or modular monolithic components are employed to ensure scalability and maintainability.

## Web API Project

The Web API Project serves as the entry point for interacting with the E-Commerce system. It exposes a set of RESTful endpoints to handle requests related to products, orders, and customer information. This project is responsible for processing incoming HTTP requests, invoking the appropriate business logic, and returning responses.

```csharp
// Product API
[Route("api/[controller]")]
[ApiController]
public class ProductController : ControllerBase
{
    private readonly IUnitOfWork unitOfWork;
    public ProductController(IUnitOfWork unitOfWork)
    {
        this.unitOfWork = unitOfWork;
    }
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var data = await unitOfWork.Products.GetAllAsync();
        return Ok (data);
    }
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var data = await unitOfWork.Products.GetByIdAsync(id);
        if (data == null) return Ok();
        return Ok(data);
    }
    [HttpPost]
    public async Task<IActionResult> Add(ProductRequest product)
    {
        var data = await unitOfWork.Products.AddAsync(product);
        return Ok(data);
    }
    [HttpDelete]
    public async Task<IActionResult> Delete(int id)
    {
        var data = await unitOfWork.Products.DeleteAsync(id);
        return Ok(data);
    }
    [HttpPut]
    public async Task<IActionResult> Update(ProductRequest product)
    {
        var data = await unitOfWork.Products.UpdateAsync(product);
        return Ok(data);
    }
}

```

## Repository Project Setup

To set up the Repository Project, the following packages need to be installed:

```powershell
Install-Package Dapper
Install-Package Microsoft.Extensions.Configuration
Install-Package Microsoft.Extensions.DependencyInjection.Abstractions
Install-Package System.Data.SqlClient
```

These packages provide essential dependencies for Dapper, configuration management, dependency injection, and SQL Server connectivity.

## IGenericRepository Interface

Create an interface IGenericRepository to define generic CRUD operations that the repository and unit of work will use:

```csharp

public interface IGenericRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IReadOnlyList<T>> GetAllAsync();
    Task<int> AddAsync(T entity);
    Task<int> UpdateAsync(T entity);
    Task<int> DeleteAsync(int id);
}
```

## Dapper Repository Project using Unit of Work Pattern and SQL Server Database

### Repository and Unit of Work

Implement a generic repository class that utilizes Dapper for data access. The repository, along with the Unit of Work pattern, ensures transactional integrity:

```csharp
// product entity
public class Product
{
    public int Id { get; set; }
    public string Name{ get; set; }
    public string Description { get; set; }
    public string Barcode { get; set; }
    public decimal Rate { get; set; }
    public DateTime AddedOn { get; set; }
    public DateTime ModifiedOn { get; set; }

// IGenericRepository Interface with T as the Product
public interface IProductRepository : IGenericRepository<Product>
{
}

// IUnitOfWork for IProductRepository
public interface IUnitOfWork
{
    IProductRepository  Products { get; }
}

// ProductRepository
public class ProductRepository : IProductRepository
{
    private readonly IConfiguration configuration;
    public ProductRepository(IConfiguration configuration)
    {
        this.configuration = configuration;
    }
    public async Task<int> AddAsync(Product entity)
    {
        entity.AddedOn = DateTime.Now;
        var sql = "Insert into Products (Name,Description,Barcode,Rate,AddedOn) VALUES (@Name,@Description,@Barcode,@Rate,@AddedOn)";
        using (var connection = new SqlConnection(configuration.GetConnectionString("DefaultConnection")))
        {
            connection.Open();
            var result = await connection.ExecuteAsync(sql, entity);
            return result;
        }
    }
    public async Task<int> DeleteAsync(int id)
    {
        var sql = "DELETE FROM Products WHERE Id = @Id";
        using (var connection = new SqlConnection(configuration.GetConnectionString("DefaultConnection")))
        {
            connection.Open();
            var result = await connection.ExecuteAsync(sql, new { Id = id });
            return result;
        }
    }
    public async Task<IReadOnlyList<Product>> GetAllAsync()
    {
        var sql = "SELECT * FROM Products";
        using (var connection = new SqlConnection(configuration.GetConnectionString("DefaultConnection")))
        {
            connection.Open();
            var result = await connection.QueryAsync<Product>(sql);
            return result.ToList();
        }
    }
    public async Task<Product> GetByIdAsync(int id)
    {
        var sql = "SELECT * FROM Products WHERE Id = @Id";
        using (var connection = new SqlConnection(configuration.GetConnectionString("DefaultConnection")))
        {
            connection.Open();
            var result = await connection.QuerySingleOrDefaultAsync<Product>(sql, new { Id = id });
            return result;
        }
    }
    public async Task<int> UpdateAsync(Product entity)
    {
        entity.ModifiedOn = DateTime.Now;
        var sql = "UPDATE Products SET Name = @Name, Description = @Description, Barcode = @Barcode, Rate = @Rate, ModifiedOn = @ModifiedOn  WHERE Id = @Id";
        using (var connection = new SqlConnection(configuration.GetConnectionString("DefaultConnection")))
        {
            connection.Open();
            var result = await connection.ExecuteAsync(sql, entity);
            return result;
        }
    }
}

// UnitOfWork inherit from the interface IUnitOfWork
public class UnitOfWork : IUnitOfWork
{
    public UnitOfWork(IProductRepository productRepository)
    {
        Products = productRepository;
    }
    public IProductRepository Products { get; }
}

```

### ConnectionStrings Configuration

Ensure that the ConnectionStrings are configured in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "your_connection_string_here"
  }
}
```

### Service Registration

Register the repository and unit of work in the ASP.NET Core Dependency Injection container:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Other service registrations...

    services.AddTransient<IDbConnection>((sp) =>
    {
        var config = sp.GetRequiredService<IConfiguration>();
        return new SqlConnection(config.GetConnectionString("DefaultConnection"));
    });

   services.AddTransient<IProductRepository, ProductRepository>();
        services.AddTransient<IUnitOfWork, UnitOfWork>();
}

```

### AutoMapper Configuration

Use AutoMapper to simplify the mapping between API requests and domain models:

```csharp
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<ProductRequest, Product>();
        CreateMap<Product, ProductResponse>();
        // Add other mappings...
    }
}
```

### xUnit for Unit Tests using dotnet testcontainers

For unit testing with xUnit and Testcontainers, set up a DatabaseFixture class to handle the Docker container setup:

```csharp
public class DatabaseFixture : IDisposable
{
    public IDbConnection Connection { get; private set; }

    public DatabaseFixture()
    {
        // Set up a Docker container with SQL Server using Testcontainers
        // Initialize Connection...
    }

    public void Dispose()
    {
        // Cleanup resources, e.g., stop and remove the Docker container
        // ...
    }
}

public class ProductRepositoryTests : IClassFixture<DatabaseFixture>
{
    private readonly IDbConnection _dbConnection;

    public ProductRepositoryTests(DatabaseFixture databaseFixture)
    {
        _dbConnection = databaseFixture.Connection;
    }

    [Fact]
    public void GetProducts_ShouldReturnProducts()
    {
        using (var unitOfWork = new UnitOfWork(_dbConnection))
        {
            var productRepository = unitOfWork.ProductRepository;
            var products = productRepository.GetAll();

            Assert.NotNull(products);
            // Add specific assertions based on test requirements
        }
    }
}
```

In this setup, the DatabaseFixture class ensures the creation and disposal of a temporary SQL Server Docker container during unit tests.

By incorporating these elements into the ASP.NET Core 6 E-Commerce system, developers can achieve a well-structured, performant, and testable data access layer using Dapper. The combination of Dapper, the Unit of Work Pattern, AutoMapper, and xUnit with Testcontainers provides a robust foundation for building scalable and reliable applications.

**NOTE**: The code snippets and recommendations provided are for educational purposes and may need to be adapted to specific project requirements.

# Advantages and Disadvantages

## Advantages

- **Performance**: Dapper's lightweight nature translates to faster execution times, making it ideal for read-heavy operations.
- **Control**: Developers have direct control over SQL queries, enabling optimization for specific use cases.

## Disadvantages

- **Abstraction**: Lack of features compared to EF Core 6 may require developers to write more boilerplate code for complex scenarios.
- **Learning Curve**: Developers familiar with full-fledged ORMs might face a learning curve when transitioning to Dapper.

# Issues and Considerations

- **Connection Management**: Ensure proper connection management, especially when dealing with distributed systems or microservices. Consider implementing connection pooling to enhance performance.

- **SQL Injection**: Because Dapper allows for direct SQL execution, developers must be cautious about SQL injection vulnerabilities. Always use parameterized queries to mitigate this risk.

# When to Use Dapper

- **Read-Heavy Operations**: Dapper excels in scenarios where efficient retrieval of data is crucial.
- **Raw SQL Control**: When direct control over SQL queries is necessary, Dapper provides a straightforward approach.

# When Not to Use Dapper

- **Rapid Prototyping**: For quick application prototyping where speed of development is a priority, a higher-level ORM like EF Core 6 may be more suitable.
- **Complex Data Relationships**: In scenarios where complex data relationships and intricate mappings are prevalent, a more feature-rich ORM may be preferable.

# Best Practices

- **Parameterized Queries**: Always use parameterized queries to prevent SQL injection.
- **Use Repositories and Unit of Work**: Implement the repository and unit of work patterns for better organization and maintainability.
- **Performance Testing**: Conduct thorough performance testing to ensure Dapper meets the specific performance requirements of your application.

# Conclusion

In the realm of ASP.NET Core 6 development, Dapper offers a compelling alternative to traditional ORMs like EF Core 6. By providing a lightweight, high-performance solution, Dapper empowers developers to optimize data access for read-heavy operations. However, it is crucial to consider the specific needs of your application, weighing the advantages and disadvantages before choosing Dapper as your data access tool. As always, thorough testing and adherence to best practices will contribute to the success of your implementation.

# References:

- [Dapper GitHub Repository](https://github.com/DapperLib/Dapper)
- [Entity Framework Core Documentation](https://learn.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/)
- [AutoMapper Documentation](https://docs.automapper.org/en/stable/)
- [Testcontainers GitHub Repository]()
- [xUnit Documentation](https://xunit.net/)
