+++
title = 'Implementing Mongodb Atlas With ASP.NET Core Using CQRS'
date = 2022-06-27T18:01:02Z
draft = false
series = "Data Store"
tags = ["ASP.Net Core", "MongoDB", "CQRS"]
+++

# Introduction

In modern application development, choosing the right database solution is crucial for building scalable and efficient systems. MongoDB Atlas, combined with ASP.NET Core and the CQRS (Command Query Responsibility Segregation) pattern, offers a powerful and flexible solution for developing microservices-based applications. This technical blog aims to guide you through the process of implementing MongoDB Atlas with ASP.NET Core using the CQRS pattern, covering key aspects such as MediatR, Docker, and best practices.

# What is MongoDB Atlas?

MongoDB Atlas is a fully managed cloud database service for MongoDB, a NoSQL database. It provides a scalable, secure, and globally distributed environment for storing and retrieving data.

## Resolving Common Problems

- **Scalability**: MongoDB Atlas excels at horizontal scalability, allowing applications to handle growing data and user loads by distributing data across multiple servers.
- **Automated Operations**: MongoDB Atlas automates routine database management tasks such as backups, updates, and scaling, reducing operational overhead.
- **Global Distribution**: With support for multi-region deployments, MongoDB Atlas ensures low-latency access for users around the world, enhancing the user experience.

## MongoDB Atlas vs. SQL Server Database

- **Data Model**: MongoDB Atlas uses a flexible and dynamic schema with BSON (Binary JSON), allowing for easy adaptation to changing data requirements. In contrast, SQL Server relies on a rigid schema.
- **Scaling**: MongoDB Atlas supports horizontal scaling by distributing data across multiple nodes, while SQL Server typically scales vertically by upgrading hardware.
- **Query Language**: MongoDB Atlas uses a rich query language that supports JSON-like documents, while SQL Server uses SQL for querying relational data.

# Implementing MongoDB Atlas with ASP.NET Core and CQRS

Let's delve into a practical example of building an e-commerce system using MongoDB Atlas and ASP.NET Core using CQRS. We'll leverage Docker for containerization, MediatR for implementing the CQRS pattern, FluentValidation for validating input request, AutoMapper for object mapping, and xUnit for unit tests.

## Key Technologies and Libraries

- **ASP.NET Core 6**: The latest version of the cross-platform, high-performance framework for building modern, cloud-based, and internet-connected applications.
- **MongoDB.Driver**: The official .NET driver for MongoDB, enabling seamless integration with MongoDB Atlas.
- **MediatR**: A library facilitating the implementation of the CQRS pattern, promoting command and query separation.
- **FluentValidation**: Used for API request validation and integrated into the CQRS validation pipeline with MediatR.
- **AutoMapper**: Streamlines the object-to-object mapping of entities to models, enhancing code readability and maintainability.
- **Docker and Docker Compose**: Facilitates containerization and orchestration of microservices, ensuring consistency across development, testing, and production environments.
- **xUnit**: The testing framework for unit tests, ensuring the reliability of the application.

## Sample Project Structure

```markdown
- ECommerce.API (ASP.NET Core API)
  - Controllers
  - MediatR
    - Commands
      - CreateOrderCommand.cs
    - Queries
      - GetProductQuery.cs
    - PipelineBehaviors
      - ValidationBehavior.cs
- ECommerce.Services.Order (Microservice for Order Management)
  - Handlers
    - CreateOrderCommandHandler.cs
  - MediatR
    - Commands
      - CreateOrderCommand.cs
    - Queries
      - GetOrderQuery.cs
  - Validators
    - CreateOrderCommandValidator.cs
- ECommerce.Services.Product (Microservice for Product Management)
  - Handlers
    - GetProductQueryHandler.cs
  - MediatR
    - Commands
      - CreateProductCommand.cs
    - Queries
      - GetProductQuery.cs
  - Validators
    - CreateProductCommandValidator.cs
- ECommerce.Common (Shared Utilities)
  - Commands
    - CreateOrderCommand.cs
    - CreateProductCommand.cs
  - Queries
    - GetProductQuery.cs
  - Validators
    - CreateOrderCommandValidator.cs
    - CreateProductCommandValidator.cs
- ECommerce.Tests (Unit Tests)
  - Handlers
    - CreateOrderCommandHandlerTests.cs
    - GetProductQueryHandlerTests.cs
  - Validators
    - CreateOrderCommandValidatorTests.cs
    - CreateProductCommandValidatorTests.cs
```

## Setting up the .NET Core API project

### Step 1: Prerequisites

Make sure you have the following installed on your machine:

- .NET SDK
- Docker Desktop
- Visual Studio Code (Optional, but recommended for a lightweight IDE)
- MongoDb Atlas

### Step 2: Project Initialization

1. Create a new .NET Core API project using the following command:

```bash
dotnet new webapi -n ECommerce.API
```

2. Navigate to the project directory:

```bash
cd ECommerce.API
```

3. Add necessary NuGet packages to the project:

```bash
dotnet add package MediatR
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
dotnet add package MongoDB.Driver
```

### Step 3: Docker Configuration

1. Create a Dockerfile in the project directory (`ECommerce.API/Dockerfile`) with the following content:

```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["ECommerce.API/ECommerce.API.csproj", "ECommerce.API/"]
RUN dotnet restore "ECommerce.API/ECommerce.API.csproj"
COPY . .
WORKDIR "/src/ECommerce.API"
RUN dotnet build "ECommerce.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "ECommerce.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "ECommerce.API.dll"]
```

2. Create a docker-compose.yml file in the project directory with the following content:

```yaml
version: "3.4"

services:
  ecommerce.api:
    image: ecommerce-api
    build:
      context: .
      dockerfile: ECommerce.API/Dockerfile
    ports:
      - "5000:80"
```

### Step 4: Debugging Configuration

1. Open the ECommerce.API.csproj file in the ECommerce.API project and add the following property group to enable debugging:

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

### Step 5: Running the Application

1. Build the Docker image:

```bash
docker-compose build
```

2. Run the Docker container:

```bash
docker-compose up
```

3. Open a web browser and navigate to http://localhost:5000/swagger to access the Swagger UI for the API.

These are the initial steps for setting up our .net core project. Additional setups or configurations should be done as required.

### MongoDB Atlas Setup

1. Create MongoDB Atlas Account

   - Visit the [MongoDB Atlas website](https://www.mongodb.com/cloud/atlas/register).
   - Sign up for a new account or log in if you already have one.

2. Create a Cluster

   - Once logged in, click on "Build a Cluster."
   - Choose your preferred cloud provider, region, and other settings.
   - Click "Create Cluster" to initiate the cluster creation.

3. Configure a Database User

   - In the MongoDB Atlas dashboard, navigate to "Database Access" under the "Security" section.
   - Add a new database user with the necessary privileges.

4. Configure Network Access

   - In the MongoDB Atlas dashboard, navigate to "Network Access" under the "Security" section.
   - Add your IP address to the IP Whitelist to allow connections.

5. Get Connection String
   - In the MongoDB Atlas dashboard, click on "Connect" for your cluster.
   - Choose "Connect your application" and copy the connection string.

### Configure MongoDB

#### Configure MongoDB Connection

In our ASP.NET Core application, we need to configure the connection to our MongoDB Atlas cluster. This typically involves setting the connection string, which includes the credentials and cluster information.

```csharp
// Add this to your appsettings.json or appsettings.Development.json
{
  "MongoDBSettings": {
    "ConnectionString": "your_mongodb_connection_string",
    "DatabaseName": "your_database_name"
  }
}
```

#### Register MongoDB Services

In the `Startup.cs` file, configure and register the MongoDB services using the provided connection string and database name.

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using MongoDB.Driver;

public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        // Other service configurations...

        // Register MongoDB services
        services.AddSingleton<IMongoClient>(sp =>
        {
            var settings = Configuration.GetSection("MongoDBSettings");
            var connectionString = settings.GetValue<string>("ConnectionString");
            return new MongoClient(connectionString);
        });

        services.AddScoped<IMongoDatabase>(sp =>
        {
            var client = sp.GetRequiredService<IMongoClient>();
            var settings = Configuration.GetSection("MongoDBSettings");
            var databaseName = settings.GetValue<string>("DatabaseName");
            return client.GetDatabase(databaseName);
        });

        // Other service configurations...
    }

    // Other methods...
}
```

#### Inject IMongoDatabase in Services

Now that we've registered the MongoDB services, we can inject IMongoDatabase into our services or controllers where needed.

```csharp
public class SampleService
{
    private readonly IMongoDatabase _database;

    public YourService(IMongoDatabase database)
    {
        _database = database;
    }

    public void SampleMethod()
    {
        // Use _database to interact with MongoDB
        var collection = _database.GetCollection<theEntityDocument>("the_collection_name");
        // Perform MongoDB operations...
    }
}
```

Remember to handle exceptions, implement error handling, and follow best practices for secure database interactions in production application.

## E-commerce Microservices Implementation with CQRS in .Net Core

In this project structure, we implement an E-commerce system utilizing microservices architecture, CQRS pattern, FluentValidation for request validation, and MediatR for command and query handling. Below is a detailed explanation of each component, along with corresponding C# code snippets for clarity.

### ECommerce.API (ASP.NET Core API)

- **Controllers**: Houses ASP.NET Core controllers responsible for handling incoming HTTP requests.
- **MediatR**
  - **Commands**: Contains command classes for actions like creating orders (`CreateOrderCommand`).
  - **Queries**: Holds query classes for retrieving data, such as product details (`GetProductQuery`).
  - **PipelineBehaviors**: Includes the `ValidationBehavior` class responsible for request validation.

#### Code Snippet: ECommerce.API

```csharp
// ECommerce.API/Controllers/OrderController.cs

[ApiController]
[Route("api/orders")]
public class OrderController : ControllerBase
{
    private readonly IMediator _mediator;

    public OrderController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderCommand command)
    {
        var orderId = await _mediator.Send(command);
        return Ok(new { OrderId = orderId });
    }
}

```

### ECommerce.Services.Order (Microservice for Order Management)

- **Handlers**: Contains classes responsible for handling commands and queries related to order management.
- **MediatR**
  - **Commands**: Includes command classes specific to order management, like `CreateOrderCommand`.
  - **Queries**: Holds query classes, such as `GetOrderQuery`.
  - **Validators**: Houses validators for commands, like `CreateOrderCommandValidator`.

#### Code Snippet: ECommerce.Services.Order

```csharp
// ECommerce.Services.Order/Handlers/CreateOrderCommandHandler.cs

public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    private readonly IMongoDatabase _database;

    public CreateOrderCommandHandler(IMongoDatabase database)
    {
        _database = database;
    }

    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        // Implement logic to create an order in the MongoDB database
        // ...

        return Guid.NewGuid(); // Return the newly created order ID
    }
}
```

### ECommerce.Services.Product (Microservice for Product Management)

- **Handlers**: Contains classes for handling commands and queries related to product management.
- **MediatR**
  - **Commands**: Includes command classes specific to product management, such as `CreateProductCommand`.
  - **Queries**: Holds query classes, like `GetProductQuery`.
  - **Validators**: Houses validators for commands, for instance, `CreateProductCommandValidator`.

#### Code Snippet: ECommerce.Services.Product

```csharp

// ECommerce.Services.Product/Handlers/GetProductQueryHandler.cs

public class GetProductQueryHandler : IRequestHandler<GetProductQuery, ProductDto>
{
    private readonly IMongoDatabase _database;

    public GetProductQueryHandler(IMongoDatabase database)
    {
        _database = database;
    }

    public async Task<ProductDto> Handle(GetProductQuery request, CancellationToken cancellationToken)
    {
        // Implement logic to retrieve product details from MongoDB
        // ...

        return new ProductDto(); // Return DTO with product details
    }
}

```

### ECommerce.Common (Shared Utilities)

- **Commands**: Contains command classes shared among microservices, like `CreateOrderCommand` and `CreateProductCommand`.
- **Queries**: Holds query classes shared among microservices, for example, `GetProductQuery`.
- **Validators**: Houses validators shared among microservices, such as `CreateOrderCommandValidator` and `CreateProductCommandValidator`.

#### Code Snippet: ECommerce.Common

```csharp
// ECommerce.Common/Commands/CreateOrderCommand.cs

public class CreateOrderCommand : IRequest<Guid>
{
    public string ProductId { get; set; }
    public int Quantity { get; set; }
    // Other properties...
}
```

FluentValidation for Commands

### Code Snippets for Validation and PipelineBehavior

```csharp
// ECommerce.Services.Order/Validators/CreateOrderCommandValidator.cs

public class CreateOrderCommandValidator : AbstractValidator<CreateOrderCommand>
{
    public CreateOrderCommandValidator()
    {
        RuleFor(command => command.ProductId).NotEmpty().WithMessage("Product ID cannot be empty.");
        RuleFor(command => command.Quantity).GreaterThan(0).WithMessage("Quantity must be greater than zero.");
        // Additional validation rules...
    }
}
```

FluentValidation for Queries (Optional)

```csharp
// ECommerce.Common/Queries/GetProductQuery.cs

public class GetProductQuery : IRequest<ProductDto>
{
    public Guid ProductId { get; }

    public GetProductQuery(Guid productId)
    {
        ProductId = productId;
    }
}
```

```csharp
// ECommerce.Services.Product/Validators/GetProductQueryValidator.cs

public class GetProductQueryValidator : AbstractValidator<GetProductQuery>
{
    public GetProductQueryValidator()
    {
        RuleFor(query => query.ProductId).NotEmpty().WithMessage("Product ID cannot be empty.");
        // Additional validation rules...
    }
}
```

MediatR PipelineBehavior for Validation

```csharp
// ECommerce.API/MediatR/PipelineBehaviors/ValidationBehavior.cs

public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(TRequest request, CancellationToken cancellationToken, RequestHandlerDelegate<TResponse> next)
    {
        var context = new ValidationContext<TRequest>(request);
        var validationFailures = _validators
            .Select(validator => validator.Validate(context))
            .SelectMany(result => result.Errors)
            .Where(failure => failure != null)
            .ToList();

        if (validationFailures.Any())
        {
            throw new ValidationException(validationFailures);
        }

        return await next();
    }
}
```

Registering FluentValidation and PipelineBehavior in Startup.cs

```csharp
// ECommerce.API/Startup.cs

public void ConfigureServices(IServiceCollection services)
{
    // Other configurations...

    // Register FluentValidation for commands and queries
    AssemblyScanner
        .FindValidatorsInAssembly(Assembly.GetExecutingAssembly())
        .ForEach(result => services.AddScoped(result.InterfaceType, result.ValidatorType));

    // Register MediatR PipelineBehavior for validation
    services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));

    // Other services...
}
```

### ECommerce.Tests (Unit Tests)

- **Handlers**: Contains unit tests for various handlers to validate their behavior.
- **Validators**: Includes unit tests for validators to ensure proper validation.

#### Code Snippet: ECommerce.Tests

```csharp
// ECommerce.Tests/Handlers/CreateOrderCommandHandlerTests.cs

public class CreateOrderCommandHandlerTests
{
    // Implement unit tests for the CreateOrderCommandHandler
    // ...
}
```

```csharp
// ECommerce.Tests/Validators/CreateOrderCommandValidatorTests.cs

public class CreateOrderCommandValidatorTests
{
    // Implement unit tests for the CreateOrderCommandValidator
    // ...
}
```

# Advantages and Disadvantages of MongoDB Atlas with ASP.NET Core and CQRS

## Advantages

- **Scalability**: MongoDB Atlas provides horizontal scalability, allowing applications to handle increased loads effortlessly.
- **Flexibility**: The dynamic schema of MongoDB Atlas accommodates evolving data models, promoting flexibility in application development.
- **Automation**: Automated operations in MongoDB Atlas reduce administrative overhead, enabling developers to focus on building features.

## Disadvantages

- **Learning Curve**: Developers accustomed to relational databases may face a learning curve when transitioning to MongoDB's NoSQL model.
- **Data Integrity Trade-offs**: MongoDB prioritizes flexibility over strict data integrity constraints, requiring careful consideration in application design.

# Issues and Considerations

- **Connection Handling**: Proper connection pooling and handling must be implemented to manage connections effectively and prevent resource exhaustion.
- **Security Configuration**: MongoDB Atlas requires robust security configurations, including authentication, encryption, and access control, to ensure data safety.
- **Schema Design**: Designing an efficient schema that aligns with application requirements is crucial to avoid performance issues.

# Best Practices

- **Indexing**: Utilize appropriate indexes to optimize query performance and enhance database efficiency.
- **Connection Pooling**: Implement and configure connection pooling to efficiently manage database connections.
- **Data Modeling**: Carefully design the data model to match the application's query patterns and requirements.
- **Logging and Monitoring**: Implement comprehensive logging and monitoring to identify and address issues promptly.

# Conclusion

In this blog, we explored the integration of MongoDB Atlas with ASP.NET Core using the CQRS pattern, showcasing its benefits, considerations, and best practices. MongoDB Atlas empowers developers to build scalable and flexible applications, and when combined with the CQRS pattern, it contributes to a modular and maintainable codebase. By following best practices and considering potential challenges, developers can harness the full potential of MongoDB Atlas in their ASP.NET Core applications.

# References

- [MongoDB Atlas Documentation](https://www.mongodb.com/docs/atlas/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/introduction-to-aspnet-core)
- [FluentValidation Documentation](https://docs.fluentvalidation.net/en/latest/)
- [AutoMapper Documentation](https://docs.automapper.org/en/stable/)
- [Docker Documentation](https://docs.docker.com/guides/get-started/)
- [xUnit Documentation](https://xunit.net/)
