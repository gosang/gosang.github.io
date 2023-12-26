+++
title = 'Saga Patterns'
date = 2021-10-30T19:18:44Z
draft = false
series = "Design Patterns"
tags = ["Microservices"]
+++

Microservices architecture has revolutionized the way we design and deploy applications, but it introduces challenges in managing transactions across distributed services. Saga Patterns emerge as a powerful solution to address these challenges and ensure data consistency in a microservices environment. In this blog post, we will explore the Saga Pattern, its variants (Choreographer and Orchestrator Patterns), anti-patterns, and provide practical insights using .NET Core for an e-commerce system.

# What is Saga Pattern?

The Saga Pattern is a design pattern for handling distributed transactions across multiple services in a microservices or modular monolithic architecture. It addresses the limitations of traditional ACID transactions, focusing on achieving eventual consistency rather than immediate consistency. In a Saga, a complex transaction is broken down into a series of smaller, more manageable steps.

# Rationale and Problems Resolved

The rationale behind Saga Patterns lies in the challenges posed by distributed systems:

- **Long-Running Transactions**: Traditional transactions can be impractical in distributed systems, especially when transactions span multiple services over an extended period.
- **Failure Handling**: Failures are inevitable in distributed environments. Saga Patterns introduce compensation actions to revert changes in case of failures, ensuring data consistency.
- **Independent Scaling**: Microservices often require independent scaling. Saga Patterns allow each service involved in a transaction to scale independently without compromising consistency.

# Transactions and ACID Properties

While traditional transactions adhere to the ACID properties (Atomicity, Consistency, Isolation, Durability), Saga Patterns embrace the BASE model (Basically Available, Soft state, Eventually consistent). This shift in mindset prioritizes system availability and eventual consistency over immediate consistency.

# Choreographer Pattern

## How It Works

In the Choreographer Pattern, services communicate through events. Each service publishes events when a state change occurs, and other services subscribe to these events to perform their actions independently. This approach ensures loose coupling among services.

# Orchestrator Pattern

## How It Works

The Orchestrator Pattern involves a central component, the orchestrator, that coordinates the execution of distributed transactions. The orchestrator communicates with individual services, instructing them on the steps to take. This approach provides a centralized view of the transaction's progress.

# Setting up Order-API Service with Choreographer and Orchestrator Patterns in C# .NET Core using MediatR

In this section, we will cover the sample setup of the Order-API service, including the implementation of the OrderService (Choreographer Pattern) and OrderOrchestrator (Orchestrator Pattern) in C# .NET Core using MediatR. Additionally, we'll include the necessary Dockerfiles and configurations for the MS SQL Server repository.

## Order-API Service

#### OrderService (Choreographer Pattern)

Let's start by creating the OrderService, which follows the Choreographer Pattern.

#### OrderService.cs

```csharp
// OrderService.cs
public class OrderService
{
    private readonly IMediator _mediator;

    public OrderService(IMediator mediator)
    {
        _mediator = mediator;
    }

    public async Task PlaceOrder(Order order)
    {
        // Business logic for placing an order

        // Publish an event
        await _mediator.Publish(new OrderPlacedEvent(order.Id));
    }
}
```

## OrderOrchestrator (Orchestrator Pattern)

Next, let's create the OrderOrchestrator, which follows the Orchestrator Pattern.

#### OrderOrchestrator.cs

```csharp
// OrderOrchestrator.cs
public class OrderOrchestrator
{
    private readonly IMediator _mediator;

    public OrderOrchestrator(IMediator mediator)
    {
        _mediator = mediator;
    }

    public async Task ProcessOrder(Order order)
    {
        // Orchestrate the order processing steps
        await _mediator.Send(new ValidateOrderCommand(order.Id));
        await _mediator.Send(new ChargePaymentCommand(order.Id));
        await _mediator.Send(new FulfillOrderCommand(order.Id));
    }
}
```

## Dockerfile for Order-API

Now, let's create the Dockerfile to build the Docker image for the Order-API service.

#### Sample Order-API Dockerfile

```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["OrderApi.csproj", "."]
RUN dotnet restore "OrderApi.csproj"
COPY . .
WORKDIR "/src"
RUN dotnet build "OrderApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "OrderApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "OrderApi.dll"]
```

### Dockerfile for MS SQL Server Repository

Now, let's create the Dockerfile for the MS SQL Server repository.

### Sample MS SQL Sever Dockerfile

```Dockerfile
FROM mcr.microsoft.com/mssql/server
ENV SA_PASSWORD=YourPassword
ENV ACCEPT_EULA=Y
ENV MSSQL_PID=Express
```

## MS SQL Server Repository in .NET Core API Project

### Repository for MS SQL Server

Now, let's create the repository for MS SQL Server.

#### OrderRepository.cs

```csharp
// OrderRepository.cs
public class OrderRepository : IOrderRepository
{
    private readonly OrderDbContext _dbContext;

    public OrderRepository(OrderDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task<Order> GetOrderByIdAsync(Guid orderId)
    {
        return await _dbContext.Orders.FindAsync(orderId);
    }

    // Additional repository methods for CRUD operations
}
```

### OrderDbContext

Create the OrderDbContext class to define the database context.

#### OrderDbContext.cs

```csharp
// OrderDbContext.cs
public class OrderDbContext : DbContext
{
    public OrderDbContext(DbContextOptions<OrderDbContext> options) : base(options)
    {
    }

    public DbSet<Order> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entity relationships and constraints
    }
}

```

This context class is used by Entity Framework Core to perform database operations.

## Dockerfile for MS SQL Server Repository

Create the Dockerfile to build the Docker image for the MS SQL Server repository.

#### Include MS SQL Server Repository in the Dockerfile

```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["OrderRepository.csproj", "."]
RUN dotnet restore "OrderRepository.csproj"
COPY . .
WORKDIR "/src"
RUN dotnet build "OrderRepository.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "OrderRepository.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "OrderRepository.dll"]
```

## Docker Compose YAML

Finally, let's update the Docker Compose YAML file to include both services.

### Order-API Service

Let's start by setting up the `order-api` service in Docker Compose. This service represents the microservice responsible for managing orders in our e-commerce system.

```yaml
services:
  order-api:
    image: order-api:latest
    build:
      context: .
      dockerfile: OrderApi/Dockerfile
    ports:
      - "5000:80"
    depends_on:
      - order-orchestrator
    environment:
      ASPNETCORE_ENVIRONMENT: Development
      ConnectionStrings__DefaultConnection: "Server=orderdb;Database=OrderDb;Integrated Security=True;"
```

- **image**: Specifies the Docker image for the `order-api` service. Ensure you have built this image using the specified Dockerfile.
- **build**: Defines the build context and Dockerfile for building the Docker image. Adjust the paths according to your project structure.
- **ports**: Maps the host machine's port 5000 to the container's port 80, allowing access to the API.
- **depends_on**: Ensures that the `order-api` service starts only after the `order-orchestrator` service is up and running.
- **environment**: Configures environment variables for the service. In this case, it sets the ASP.NET Core environment to Development and provides the connection string for the database.

### MS SQL Server Repository

Now, let's add the MS SQL Server repository (`orderdb`) to the Docker Compose configuration.

```yaml
services:
  orderdb:
    image: "mcr.microsoft.com/mssql/server"
    environment:
      SA_PASSWORD: "YourPassword"
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Express"
    ports:
      - "1433:1433"
```

- **image**: Specifies the MS SQL Server Docker image.
- **environment**: Configures environment variables for MS SQL Server. Set a secure password for the SA user and accept the end-user license agreement.
- **ports**: Maps the host machine's port 1433 to the container's port 1433, allowing communication with MS SQL Server.

### Docker Compose YAML

Finally, let's update the Docker Compose YAML file to include both services.

#### docker-compose.yml

```yaml
version: "3.8"

services:
  order-api:
    image: order-api:latest
    build:
      context: .
      dockerfile: OrderApi/Dockerfile
    ports:
      - "5000:80"
    depends_on:
      - order-orchestrator
    environment:
      ASPNETCORE_ENVIRONMENT: Development
      ConnectionStrings__DefaultConnection: "Server=orderdb;Database=OrderDb;Integrated Security=True;"

  order-orchestrator:
    image: order-orchestrator:latest
    build:
      context: .
      dockerfile: OrderOrchestrator/Dockerfile
    depends_on:
      - order-api
    environment:
      ASPNETCORE_ENVIRONMENT: Development
      ConnectionStrings__DefaultConnection: "Server=orderdb;Database=OrderDb;Integrated Security=True;"

  orderdb:
    image: order-repository:latest
    build:
      context: .
      dockerfile: OrderRepository/Dockerfile
    environment:
      SA_PASSWORD: "YourPassword"
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Express"
    ports:
      - "1433:1433"
```

Now, we have a comprehensive setup for the Order-API service with Choreographer and Orchestrator Patterns, along with the MS SQL Server repository, all orchestrated using Docker Compose.

## Debugging the Order-API

To enable debugging in the `order-api` service, update your Dockerfile (`OrderApi/Dockerfile`) with the following lines:

```Dockerfile
# Add this line to expose port 5000 for debugging
EXPOSE 5000
```

Now, when you run the `order-api` service, the debugger will be able to connect to it. Ensure your Visual Studio or VS Code debugger is configured to target this port for debugging.

## Running Docker Compose

Save the above Docker Compose configuration in a file named `docker-compose.yml`. To run the services, execute the following command in the terminal:

```bash
docker-compose up
```

This will start the `order-api` service, MS SQL Server (`orderdb`), and any other services specified in the `docker-compose.yml` file.

## Debugging the Order-API in Visual Studio

1. Open your `order-api` project in Visual Studio.
2. Set breakpoints in your code where you want to start debugging.
3. In the toolbar, select the appropriate configuration (e.g., Debug, x64).
4. Press F5 or click the "Start Debugging" button.
5. Visual Studio will attach to the running Docker container, and you can now debug your `order-api` service.

With this setup, you have a fully functional Docker Compose configuration for local testing and debugging of the `order-api` service and MS SQL Server repository in a .NET Core e-commerce system.

Adjust the configurations according to your project structure and requirements, and you are ready to run your e-commerce system locally for testing and debugging.

# Anti-Patterns in Saga-based Systems

## Synchronous Saga Series Anti-Pattern

The Synchronous Saga Series Anti-Pattern occurs when sagas are executed synchronously. This can lead to increased latency and reduced system responsiveness.

### Issues:

- **Increased Latency**: Each step must complete before the next one begins, leading to longer transaction times.
- **Reduced Scalability**: Synchronous execution can limit scalability, as services are blocked until the current step is finished.

## Saga Series Async Anti-Pattern

The Saga Series Async Anti-Pattern involves executing sagas asynchronously. However, it becomes problematic when subsequent steps are overly dependent on the completion of the preceding ones, potentially leading to inconsistencies.

### Issues:

- **Data Inconsistency**: Asynchronous execution may lead to intermediate states that are inconsistent, impacting the eventual consistency of the system.
- **Complex Error Handling**: Handling errors and compensating actions becomes more complex when steps can complete out of order.

# Advantages and Disadvantages of Saga Patterns

## Advantages

- **Scalability**: Independent scaling of services.
- **Flexibility**: Choreography offers flexibility, while orchestration provides centralized control.

## Disadvantages

- **Complexity**: Additional complexity, especially in long-running transactions.
  Consistency Challenges: Achieving consistency in a distributed environment can be challenging.

# Issues and Considerations for Saga Patterns

- **Long-Running Transactions**: Be mindful of the duration of sagas to avoid performance issues and potential data inconsistencies.
- **Compensation Actions**: Design robust compensation actions for each step to revert changes in case of failures and ensure data consistency.
- **Eventual Consistency**: Embrace the eventual consistency mindset and design with the understanding that consistency may not be immediate.

# When to Use Saga Patterns

Use Saga Patterns when:

- **Long-Running Transactions**: Transactions span multiple services over an extended period.
- **Loose Coupling**: Minimize dependencies between services for better modularity.
- **Event-Driven Architecture**: The system benefits from an event-driven architecture where services react to state changes.

# When Not to Use Saga Patterns

Avoid Saga Patterns when:

- **Immediate Consistency is Critical**: If your application demands immediate consistency at all times, traditional ACID transactions might be more suitable.

# Best Practices

- **Event Naming Conventions**: Maintain consistent and clear event naming conventions.
- **Idempotency**: Ensure each step in the saga is idempotent to handle retries and failures gracefully.
- **Compensation Actions**: Implement compensation actions for each step to rollback changes in case of failures.
- **Monitoring and Logging**: Establish robust monitoring and logging for effective troubleshooting.

# Conclusion

Saga Patterns provide a powerful approach to managing distributed transactions in microservices architecture. Whether you choose the Choreographer or Orchestrator Pattern depends on your specific requirements. By understanding the advantages, disadvantages, and best practices, you can effectively leverage Saga Patterns to build scalable and consistent systems.

# References:

- [Microsoft Docs - Sagas](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- [NServiceBus - Sagas](https://docs.particular.net/nservicebus/sagas/)
- Chris Richardson - "Microservices Patterns"
