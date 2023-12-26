+++
title = 'Clean Architecture'
date = 2020-03-02T13:58:28Z
draft = false
series = "Design Architectures"
+++

In the ever-evolving world of software development, architects and developers are in pursuit of a design pattern that not only ensures the maintainability of their applications but also enables scalability and adaptability. Clean Architecture, a concept popularized by Uncle Bob (Robert C. Martin), has gained prominence for its ability to provide a structured approach to software design. In this blog post, we will explore Clean Architecture comprehensively, covering its key concepts, components, rationale, and real-world implementation in the context of an online shopping system. We will use .NET Core, C#, Web API, CQRS with MediatR, Repository Pattern with Entity Framework Core 6.0, React Next.js UI, Docker, and Docker Compose to exemplify Clean Architecture.

# Understanding Clean Architecture

At its core, Clean Architecture emphasizes a clear separation of concerns by organizing an application into distinct layers, each with a specific responsibility. The primary components of Clean Architecture are:

**Presentation Layer**

- The outermost layer is responsible for user interfaces, such as web applications, APIs, and CLI interfaces.
- It communicates with the Application Layer to handle user input and output.

**Application Layer**

- This layer contains application-specific business logic, including use cases and workflows.
- It relies on the Domain Layer for domain-specific rules and entities.

**Domain Layer**

- The heart of the application, it contains the core business logic, entities, and value objects.
- It is entirely independent of the Presentation and Infrastructure layers, making it highly testable and maintainable.

**Infrastructure Layer**

- Responsible for external concerns, such as databases, third-party libraries, and frameworks.
- Implements the interfaces defined in the Application and Domain layers.
- Decouples the core business logic from external dependencies.

# The Rationale Behind Clean Architecture

Clean Architecture addresses several critical issues in software development:

- **Maintainability**: By enforcing a strict separation of concerns, it becomes easier to make changes or extend the application without affecting the entire system. This separation reduces the risk of creating tightly coupled and hard-to-maintain code.

- **Testability**: With its clear division of responsibilities, Clean Architecture promotes unit testing by making it easier to test individual components in isolation. This results in more reliable and bug-free code.

- **Independence**: Clean Architecture ensures that the core business logic remains isolated from external dependencies. This independence enables the application to adapt to changes in technologies, frameworks, or databases.

- **Scalability**: Clean Architecture's modularity allows for straightforward scaling by adding or modifying components, making it well-suited for microservices and modular monolithic applications.

# Realizing Clean Architecture in an Online Shopping System

Let's explore Clean Architecture in action by considering an online shopping system. In both microservices and modular monolithic approaches, Clean Architecture can be effectively applied. We will delve into three essential use cases: Customer Management, Order Processing, and Shipping, along with email notifications.

### Project Structure

The organization of these use cases in a .NET Core project might look like this:

```markdown
- OnlineShoppingApplication
  - Presentation (Web API and React UI)
  - Application
    - Customer
    - Order
    - Shipping
  - Domain
    - Customer
    - Order
    - Shipping
  - Infrastructure
    - Repositories
    - EmailServices
- Docker Compose
```

### Customer Management

In Clean Architecture, the Customer Management use case can be represented as follows:

- **Entities**: Define a `Customer` entity with properties like name, email, and address.
- **Use Cases**: Implement business logic for customer creation, updates, and retrieval.
- **Interfaces**: Create interfaces for customer data access (repositories) and customer notification services (e.g., email).
- **Adapters**: Develop implementations for data access (using Entity Framework Core) and notification (SMTP email service).

#### Example Customer Domain Entity, Customer Application Service and Customer Repository:

```csharp

// Domain/Customer/Customer.cs

namespace OnlineShoppingApp.Domain.Customer
{
    public class Customer
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Email { get; set; }
        public string Address { get; set; }
    }
}

// Application/Customer/CustomerApplicationService.cs

using MediatR;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Application.Customer
{
    public class CustomerApplicationService
    {
        private readonly IMediator _mediator;

        public CustomerApplicationService(IMediator mediator)
        {
            _mediator = mediator;
        }

        public async Task<CustomerDto> CreateCustomer(CreateCustomerCommand command)
        {
            return await _mediator.Send(command);
        }

        // Other customer-related application methods...
    }
}

// Infrastructure/Repositories/CustomerRepository.cs

using AutoMapper;
using Microsoft.EntityFrameworkCore;
using OnlineShoppingApp.Domain.Customer;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Infrastructure.Repositories
{
    public class CustomerRepository : ICustomerRepository
    {
        private readonly AppDbContext _dbContext;
        private readonly IMapper _mapper;

        public CustomerRepository(AppDbContext dbContext, IMapper mapper)
        {
            _dbContext = dbContext;
            _mapper = mapper;
        }

        public async Task<Customer> GetByIdAsync(int id)
        {
            return await _dbContext.Customers.FirstOrDefaultAsync(c => c.Id == id);
        }

        public async Task<Customer> AddAsync(Customer customer)
        {
            _dbContext.Customers.Add(customer);
            await _dbContext.SaveChangesAsync();
            return customer;
        }

        // Other repository methods...
    }
}

// Presentation/Controllers/CustomerController.cs

using Microsoft.AspNetCore.Mvc;
using OnlineShoppingApp.Application.Customer;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Presentation.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CustomerController : ControllerBase
    {
        private readonly CustomerApplicationService _customerService;

        public CustomerController(CustomerApplicationService customerService)
        {
            _customerService = customerService;
        }

        [HttpPost]
        public async Task<ActionResult<CustomerDto>> CreateCustomer([FromBody] CreateCustomerCommand command)
        {
            var result = await _customerService.CreateCustomer(command);
            return Ok(result);
        }

        // Other customer-related API methods...
    }
}

```

### Order Processing

For the Order Processing use case:

- **Entities**: Represent an `Order` entity with attributes like order date, items, and customer reference.
- **Use Cases**: Implement logic for order placement, order fulfillment, and order history.
- **Interfaces**: Define interfaces for order data access and customer notification services.
- **Adapters**: Create implementations for data access (Entity Framework Core) and notification (email).

#### Example Order Domain Entity, Order Application Service and Order Repository:

```csharp

// Domain/Order/Order.cs

namespace OnlineShoppingApp.Domain.Order
{
    public class Order
    {
        public int Id { get; set; }
        public int CustomerId { get; set; }
        public string Items { get; set; }
        // Other order-related properties...
    }
}

// Application/Order/OrderApplicationService.cs

using MediatR;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Application.Order
{
    public class OrderApplicationService
    {
        private readonly IMediator _mediator;

        public OrderApplicationService(IMediator mediator)
        {
            _mediator = mediator;
        }

        public async Task<OrderDto> PlaceOrder(PlaceOrderCommand command)
        {
            return await _mediator.Send(command);
        }

        // Other order-related application methods...
    }
}

// Infrastructure/Repositories/OrderRepository.cs

using AutoMapper;
using Microsoft.EntityFrameworkCore;
using OnlineShoppingApp.Domain.Order;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Infrastructure.Repositories
{
    public class OrderRepository : IOrderRepository
    {
        private readonly AppDbContext _dbContext;
        private readonly IMapper _mapper;

        public OrderRepository(AppDbContext dbContext, IMapper mapper)
        {
            _dbContext = dbContext;
            _mapper = mapper;
        }

        public async Task<Order> GetByIdAsync(int id)
        {
            return await _dbContext.Orders.FirstOrDefaultAsync(o => o.Id == id);
        }

        public async Task<Order> AddAsync(Order order)
        {
            _dbContext.Orders.Add(order);
            await _dbContext.SaveChangesAsync();
            return order;
        }

        // Other repository methods...
    }
}

// Presentation/Controllers/OrderController.cs

using Microsoft.AspNetCore.Mvc;
using OnlineShoppingApp.Application.Order;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Presentation.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class OrderController : ControllerBase
    {
        private readonly OrderApplicationService _orderService;

        public OrderController(OrderApplicationService orderService)
        {
            _orderService = orderService;
        }

        [HttpPost]
        public async Task<ActionResult<OrderDto>> PlaceOrder([FromBody] PlaceOrderCommand command)
        {
            var result = await _orderService.PlaceOrder(command);
            return Ok(result);
        }

        // Other order-related API methods...
    }
}

// Presentation/Controllers/ShippingController.cs

using Microsoft.AspNetCore.Mvc;
using OnlineShoppingApp.Application.Shipping;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Presentation.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ShippingController : ControllerBase
    {
        private readonly ShippingApplicationService _shippingService;

        public ShippingController(ShippingApplicationService shippingService)
        {
            _shippingService = shippingService;
        }

        [HttpPost]
        public async Task<ActionResult<ShippingDto>> ShipOrder([FromBody] ShipOrderCommand command)
        {
            var result = await _shippingService.ShipOrder(command);
            return Ok(result);
        }

        // Other shipping-related API methods...
    }
}

// xUnit Tests
// Tests/Application.Tests/CustomerApplicationServiceTests.cs

using AutoFixture;
using FluentAssertions;
using Moq;
using OnlineShoppingApp.Application.Customer;
using OnlineShoppingApp.Domain.Customer;
using System.Threading.Tasks;
using Xunit;

public class CustomerApplicationServiceTests
{
    private readonly Fixture _fixture;
    private readonly Mock<IMediator> _mediatorMock;
    private readonly CustomerApplicationService _customerService;

    public CustomerApplicationServiceTests()
    {
        _fixture = new Fixture();
        _mediatorMock = new Mock<IMediator>();
        _customerService = new CustomerApplicationService(_mediatorMock.Object);
    }

    [Fact]
    public async Task CreateCustomer_ShouldReturnCustomerDto()
    {
        // Arrange
        var createCustomerCommand = _fixture.Create<CreateCustomerCommand>();
        var expectedCustomerDto = _fixture.Create<CustomerDto>();
        _mediatorMock.Setup(x => x.Send(createCustomerCommand, default)).ReturnsAsync(expectedCustomerDto);

        // Act
        var result = await _customerService.CreateCustomer(createCustomerCommand);

        // Assert
        result.Should().BeEquivalentTo(expectedCustomerDto);
    }

    // Additional tests for other customer-related methods...
}

```

### Shipping

In the context of microservices, the Shipping use case can be treated as a separate service, adhering to Clean Architecture principles. It would have its entities, use cases, interfaces, and adapters, similar to Customer and Order Management.

#### Example Shipping Domain Entity, Shipping Application Service and Shipping Repository:

```csharp

// Domain/Shipping/Shipping.cs

namespace OnlineShoppingApp.Domain.Shipping
{
    public class Shipping
    {
        public int Id { get; set; }
        public int OrderId { get; set; }
        public string Status { get; set; }
        // Other shipping-related properties...
    }
}

// Application/Shipping/ShippingApplicationService.cs

using MediatR;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Application.Shipping
{
    public class ShippingApplicationService
    {
        private readonly IMediator _mediator;

        public ShippingApplicationService(IMediator mediator)
        {
            _mediator = mediator;
        }

        public async Task<ShippingDto> ShipOrder(ShipOrderCommand command)
        {
            return await _mediator.Send(command);
        }

        // Other shipping-related application methods...
    }
}

// Infrastructure/Repositories/ShippingRepository.cs

using AutoMapper;
using Microsoft.EntityFrameworkCore;
using OnlineShoppingApp.Domain.Shipping;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Infrastructure.Repositories
{
    public class ShippingRepository : IShippingRepository
    {
        private readonly AppDbContext _dbContext;
        private readonly IMapper _mapper;

        public ShippingRepository(AppDbContext dbContext, IMapper mapper)
        {
            _dbContext = dbContext;
            _mapper = mapper;
        }

        public async Task<Shipping> GetByIdAsync(int id)
        {
            return await _dbContext.Shippings.FirstOrDefaultAsync(s => s.Id == id);
        }

        public async Task<Shipping> AddAsync(Shipping shipping)
        {
            _dbContext.Shippings.Add(shipping);
            await _dbContext.SaveChangesAsync();
            return shipping;
        }

        // Other repository methods...
    }
}

```

### Email Service

In the Infrastructure layer, we'll integrate an Email Service using AWS Simple Email Service (SES). This service allows you to send transactional and marketing emails.

#### Example Email Service using AWS SES:

```csharp
// install the AWS SDK for .NET
dotnet add package AWSSDK.SimpleEmail

// Infrastructure/EmailServices/AWSSesEmailService.cs

using Amazon.SimpleEmail;
using Amazon.SimpleEmail.Model;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Infrastructure.EmailServices
{
    public class AWSSesEmailService : IEmailService
    {
        private readonly IAmazonSimpleEmailService _sesClient;

        public AWSSesEmailService(IAmazonSimpleEmailService sesClient)
        {
            _sesClient = sesClient;
        }

        public async Task SendEmailAsync(string toAddress, string subject, string body)
        {
            var sendRequest = new SendEmailRequest
            {
                Source = "your-ses-verified-email@example.com",
                Destination = new Destination
                {
                    ToAddresses = new[] { toAddress }
                },
                Message = new Message
                {
                    Subject = new Content(subject),
                    Body = new Body
                    {
                        Html = new Content(body)
                    }
                }
            };

            await _sesClient.SendEmailAsync(sendRequest);
        }
    }
}

```

This class uses the AWS SDK to send emails through SES. Replace "your-ses-verified-email@example.com" with the email address you've verified in the AWS SES console.

Now, configure the AWS SES client in your `Startup.cs`:

```csharp
// Infrastructure/Extensions/AwsSesExtensions.cs

using Amazon.SimpleEmail;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace OnlineShoppingApp.Infrastructure.Extensions
{
    public static class AwsSesExtensions
    {
        public static void AddAwsSes(this IServiceCollection services, IConfiguration configuration)
        {
            services.AddDefaultAWSOptions(configuration.GetAWSOptions());
            services.AddAWSService<IAmazonSimpleEmailService>();
        }
    }
}
```

Ensure you call `services.AddAwsSes(configuration)`; in your `Startup.cs` `ConfigureServices` method.

Finally, update the `Infrastructure/EmailServices/EmailServiceModule.cs` to use the new AWS SES service:

```csharp
// Infrastructure/EmailServices/EmailServiceModule.cs

using OnlineShoppingApp.Application.Email;
using OnlineShoppingApp.Infrastructure.EmailServices;

namespace OnlineShoppingApp.Infrastructure.Modules
{
    public class EmailServiceModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            builder.RegisterType<AWSSesEmailService>().As<IEmailService>().InstancePerLifetimeScope();
        }
    }
}
```

Now, we have a functioning Email Service integrated with AWS SES in our Clean Architecture project.

# Leveraging C# and .NET Core

To implement Clean Architecture with C# and .NET Core, we can use popular software design patterns:

## CQRS with MediatR

Implement Command Query Responsibility Segregation (CQRS) for separating read and write operations. MediatR simplifies the implementation of the mediator pattern, allowing you to handle commands and queries efficiently.

#### Example MediatR Commands and Handlers for customer:

```csharp

// Application/Customer/CreateCustomerCommand.cs

using MediatR;

namespace OnlineShoppingApp.Application.Customer
{
    public class CreateCustomerCommand : IRequest<CustomerDto>
    {
        public string Name { get; set; }
        public string Email { get; set; }
        public string Address { get; set; }
    }
}

// Application/Customer/CreateCustomerCommandHandler.cs

using AutoMapper;
using MediatR;
using OnlineShoppingApp.Domain.Customer;
using OnlineShoppingApp.Infrastructure.Repositories;
using System.Threading;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Application.Customer
{
    public class CreateCustomerCommandHandler : IRequestHandler<CreateCustomerCommand, CustomerDto>
    {
        private readonly ICustomerRepository _customerRepository;
        private readonly IMapper _mapper;

        public CreateCustomerCommandHandler(ICustomerRepository customerRepository, IMapper mapper)
        {
            _customerRepository = customerRepository;
            _mapper = mapper;
        }

        public async Task<CustomerDto> Handle(CreateCustomerCommand request, CancellationToken cancellationToken)
        {
            var newCustomer = new Customer
            {
                Name = request.Name,
                Email = request.Email,
                Address = request.Address
            };

            await _customerRepository.AddAsync(newCustomer);

            return _mapper.Map<CustomerDto>(newCustomer);
        }
    }
}

```

Similar command and handler structures would be created for orders and shipping.

## MediatR PipelineBehavior

MediatR PipelineBehaviors allow you to add cross-cutting concerns to your MediatR requests and responses. In this case, let's use FluentValidation for request validation.

#### Example MediatR PipelineBehavior:

```csharp
//  install the necessary NuGet packages:
dotnet add package FluentValidation
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection

// Application/Behaviors/ValidationBehavior.cs

using FluentValidation;
using MediatR;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace OnlineShoppingApp.Application.Behaviors
{
    public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
        where TRequest : IRequest<TResponse>
    {
        private readonly IEnumerable<IValidator<TRequest>> _validators;

        public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
        {
            _validators = validators;
        }

        public async Task<TResponse> Handle(TRequest request, CancellationToken cancellationToken, RequestHandlerDelegate<TResponse> next)
        {
            var context = new ValidationContext(request);
            var failures = _validators
                .Select(v => v.Validate(context))
                .SelectMany(result => result.Errors)
                .Where(f => f != null)
                .ToList();

            if (failures.Count != 0)
            {
                throw new ValidationException(failures);
            }

            return await next();
        }
    }
}


// Register the behavior in the `Startup.cs`

// Presentation/Extensions/MediatrExtensions.cs
using FluentValidation;
using MediatR;
using Microsoft.Extensions.DependencyInjection;
using OnlineShoppingApp.Application.Behaviors;

namespace OnlineShoppingApp.Presentation.Extensions
{
    public static class MediatrExtensions
    {
        public static void AddMediatr(this IServiceCollection services)
        {
            services.AddMediatR(typeof(Startup));

            services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));

            services.AddValidatorsFromAssembly(typeof(Startup).Assembly);
        }
    }
}

```

Ensure you call `services.AddMediatr();` in your `Startup.cs` `ConfigureServices` method.

Now, every MediatR request will be automatically validated using FluentValidation.

## Repository Pattern with EF Core 6.0

The repository pattern abstracts data access and allows the application to interact with the database through interfaces. Entity Framework Core 6.0 is a robust choice for database interaction in .NET Core.

## Fluent Validation

Fluent validation can be implemented to define custom validations for our classes.

#### Example Fluent validation:

```csharp
// Application/Customer/CreateCustomerCommandValidator.cs

using FluentValidation;

namespace OnlineShoppingApp.Application.Customer
{
    public class CreateCustomerCommandValidator : AbstractValidator<CreateCustomerCommand>
    {
        public CreateCustomerCommandValidator()
        {
            RuleFor(x => x.Name).NotEmpty().MaximumLength(100);
            RuleFor(x => x.Email).NotEmpty().EmailAddress();
            RuleFor(x => x.Address).NotEmpty().MaximumLength(200);
        }
    }
}
```

Similar validators would be created for orders and shipping.

## AutoMapper

AutoMapper can be implemented for mapping objects.

#### Example AutoMapper configuration:

```csharp
// Application/Mappings/MappingProfile.cs

using AutoMapper;
using OnlineShoppingApp.Domain.Customer;
using OnlineShoppingApp.Domain.Order;
using OnlineShoppingApp.Domain.Shipping;

namespace OnlineShoppingApp.Application.Mappings
{
    public class MappingProfile : Profile
    {
        public MappingProfile()
        {
            CreateMap<Customer, CustomerDto>().ReverseMap();
            CreateMap<Order, OrderDto>().ReverseMap();
            CreateMap<Shipping, ShippingDto>().ReverseMap();
        }
    }
}
```

## Swagger

Swagger can be integrated for API documentation.

#### Example Swagger Integration:

```csharp
// Presentation/Extensions/SwaggerExtensions.cs

using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.OpenApi.Models;

namespace OnlineShoppingApp.Presentation.Extensions
{
    public static class SwaggerExtensions
    {
        public static void AddSwagger(this IServiceCollection services)
        {
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "OnlineShoppingApp API", Version = "v1" });
            });
        }

        public static void UseSwaggerAndUI(this IApplicationBuilder app)
        {
            app.UseSwagger();
            app.UseSwaggerUI(c =>
            {
                c.SwaggerEndpoint("/swagger/v1/swagger.json", "OnlineShoppingApp API V1");
            });
        }
    }
}
```

## Unit Test

Unit tests can be implemented using xUnit, AutoFixture, FluentAssertions.

#### Example xUnit Test:

```csharp
// Tests/Application.Tests/CustomerApplicationServiceTests.cs

using AutoFixture;
using FluentAssertions;
using Moq;
using OnlineShoppingApp.Application.Customer;
using OnlineShoppingApp.Domain.Customer;
using System.Threading.Tasks;
using Xunit;

public class CustomerApplicationServiceTests
{
    private readonly Fixture _fixture;
    private readonly Mock<IMediator> _mediatorMock;
    private readonly CustomerApplicationService _customerService;

    public CustomerApplicationServiceTests()
    {
        _fixture = new Fixture();
        _mediatorMock = new Mock<IMediator>();
        _customerService = new CustomerApplicationService(_mediatorMock.Object);
    }

    [Fact]
    public async Task CreateCustomer_ShouldReturnCustomerDto()
    {
        // Arrange
        var createCustomerCommand = _fixture.Create<CreateCustomerCommand>();
        var expectedCustomerDto = _fixture.Create<CustomerDto>();
        _mediatorMock.Setup(x => x.Send(createCustomerCommand, default)).ReturnsAsync(expectedCustomerDto);

        // Act
        var result = await _customerService.CreateCustomer(createCustomerCommand);

        // Assert
        result.Should().BeEquivalentTo(expectedCustomerDto);
    }

    // Additional tests for other customer-related methods...
}
```

Similar tests would be created for order and shipping.

## React UI

For the user interface, React with Next.js can be used to create a responsive and interactive web application that communicates with the Clean Architecture backend.

#### Sample Code for React Next.js Customer Order Acceptance

```tsx
// components/CustomerOrderForm.tsx

import React, { useState } from "react";
import axios from "axios";

const CustomerOrderForm: React.FC = () => {
  const [order, setOrder] = useState({
    customerName: "",
    items: "",
  });

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setOrder({ ...order, [name]: value });
  };

  const handleOrderSubmit = async () => {
    try {
      // Send the order data to the server using Axios
      await axios.post("/api/orders", order);

      // Clear the form after successful submission
      setOrder({
        customerName: "",
        items: "",
      });

      alert("Order submitted successfully!");
    } catch (error) {
      console.error("Error submitting order:", error);
      alert("Failed to submit the order. Please try again later.");
    }
  };

  return (
    <div>
      <h2>Customer Order Form</h2>
      <form>
        <div>
          <label htmlFor="customerName">Customer Name:</label>
          <input
            type="text"
            id="customerName"
            name="customerName"
            value={order.customerName}
            onChange={handleInputChange}
          />
        </div>
        <div>
          <label htmlFor="items">Items:</label>
          <input
            type="text"
            id="items"
            name="items"
            value={order.items}
            onChange={handleInputChange}
          />
        </div>
        <button type="button" onClick={handleOrderSubmit}>
          Submit Order
        </button>
      </form>
    </div>
  );
};

export default CustomerOrderForm;
```

In the above code, we've created a simple form for capturing customer name and order items. When the "Submit Order" button is clicked, the order data is sent to the server using Axios.

# Docker and Docker Compose

Docker and Docker Compose provide a powerful solution for containerizing your Clean Architecture application and managing dependencies. This enables easy deployment and scaling in both microservices and modular monolithic setups.

#### Example docker compose YAML:

```yaml
version: "3"

services:
  web:
    build:
    context: ./path/to/react-app # Path to the directory containing the Dockerfile
    ports:
      - "3000:3000" # Map container port 3000 to host port 3000
  webapi:
    image: online-shopping-app-webapi
    build:
      context: ./OnlineShoppingApp.Presentation
    ports:
      - "5000:80"
    depends_on:
      - db
    networks:
      - online-shopping-network

  db:
    image: mcr.microsoft.com/mssql/server
    environment:
      SA_PASSWORD: "YourPassword"
      ACCEPT_EULA: "Y"
    networks:
      - online-shopping-network

networks:
  online-shopping-network:
    driver: bridge
```

This `docker-compose.yaml` file defines three services: `web` for the React Application, `webapi` for the web API and `db` for the database. The `depends_on` option ensures that the `db` service starts before the webapi service.

Note: Make sure to replace "YourPassword" in the `SA_PASSWORD` environment variable with a strong password.

# Advantages and Disadvantages of Clean Architecture

## Advantages

- **Maintainability**: Clean Architecture simplifies maintenance by keeping core business logic separate from external concerns.
- **Testability**: The separation of concerns results in more straightforward unit testing, improving code quality.
- **Independence**: Clean Architecture ensures that the core business logic is decoupled from specific technologies, making the application adaptable.
- **Scalability**: Modularity allows for easy scaling by adding or modifying components.

## Disadvantages

- **Complexity**: Implementing Clean Architecture can be more complex than traditional approaches, especially for smaller applications.
- **Learning Curve**: Developers may need time to learn the architectural patterns and adapt to the separation of concerns.
- **Initial Development Time**: Creating the necessary infrastructure can extend the project's initial development time.

# When to Use Clean Architecture

Clean Architecture is a valuable choice when:

- Your project's long-term maintainability and adaptability are essential.
- You anticipate changes in your application's external dependencies, such as databases or user interfaces.
- You want to maximize code quality, testability, and separation of concerns.

# Best Practices

To effectively implement Clean Architecture:

- Begin by clearly defining the core business logic and use cases of your application.
- Always use interfaces and abstractions for external interactions, allowing for easy replacement of components.
- Utilize SOLID principles, especially the Single Responsibility Principle, to ensure each component has a well-defined purpose.
- Invest in unit tests to maintain code quality and validate the independence of components.
- Keep documentation and communication clear, as Clean Architecture can be conceptually challenging.

# Conclusion

Clean Architecture provides a solid foundation for designing maintainable, testable, and adaptable software systems. Whether you're building microservices or modular monolithic applications, Clean Architecture, combined with technologies like C#, .NET Core, React, and Docker, can empower your development team to create reliable and scalable solutions. By understanding the principles, advantages, and best practices of Clean Architecture, you can embark on a path toward building resilient software that stands the test of time.

For a more in-depth exploration of Clean Architecture, consider reading:

- Clean Architecture: A Craftsman's Guide to Software Structure and Design by Robert C. Martin

As you delve deeper into Clean Architecture, you'll discover new ways to refine and adapt this approach to your specific project requirements. Happy coding!
