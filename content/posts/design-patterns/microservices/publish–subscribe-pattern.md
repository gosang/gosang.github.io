+++
title = 'Publish–subscribe Pattern'
date = 2021-09-09T19:18:44Z
draft = false
series = "Design Patterns"
tags = ["Microservices", "Messaging"]
+++

The Publish–Subscribe pattern, a stalwart in the realm of messaging patterns, orchestrates communication between components in a decoupled manner. Here, publishers categorize messages into topics and broadcast them without knowing the identity of subscribers. Subscribers express interest in specific topics, receiving messages related to those topics. Let's delve into the intricacies of this pattern, exploring its rationale, problem-solving capabilities, and its implementation in the context of a .NET Core e-commerce system.

## The Rationale Behind Publish–Subscribe

The crux of the Publish–Subscribe pattern lies in fostering loose coupling among system components. By obviating direct dependencies, components can evolve independently, promoting modularity and scalability. The pattern excels in scenarios where flexibility and adaptability are paramount.

## Problems Resolved by Publish–Subscribe

- **Decoupling Components**: Mitigates direct dependencies, allowing components to evolve autonomously.
- **Scalability**: Facilitates horizontal scaling, accommodating a growing number of publishers and subscribers.
- **Flexibility**: Enables seamless addition or removal of components without disrupting the entire system.

# Implementation in .NET Core E-commerce System

## Setting the Stage: .NET Core Services and APIs

Our e-commerce system leverages .NET Core, with services and APIs orchestrated by the MediatR library for CQRS, AutoMapper for object mapping, EF Core InMemory database, and xUnit for robust unit testing.

#### 1. MediatR Request and Handler

Install the MediatR NuGet package:

```bash
dotnet add package MediatR
```

Create a MediatR request and handler for handling order placements:

```csharp
// OrderPlacedNotification.cs
public class OrderPlacedNotification : INotification
{
    public int OrderId { get; set; }
}

// OrderPlacedNotificationHandler.cs
public class OrderPlacedNotificationHandler : INotificationHandler<OrderPlacedNotification>
{
    public async Task Handle(OrderPlacedNotification notification, CancellationToken cancellationToken)
    {
        // Logic to handle the order placed notification
    }
}
```

#### 2. Utilizing Publish–Subscribe in E-commerce Services

In the Order Service, use MediatR to publish the OrderPlacedNotification upon order placement:

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
        // Logic to place an order

        // Publish the OrderPlacedNotification
        await _mediator.Publish(new OrderPlacedNotification { OrderId = order.Id });
    }
}
```

#### 3. Docker Compose for Local Deployment

Compose your services using Docker Compose for local deployment:

```yaml
version: "3.8"

services:
  order-service:
    image: order-service-image
    # Add other configurations for the order service

  notification-service:
    image: notification-service-image
    # Add other configurations for the notification service
```

Run your services locally using:

```bash
docker-compose up --build
```

# Advantages and Disadvantages

## Advantages

- **Loose Coupling**: Components operate independently, allowing for modular development.
- **Scalability**: Scales horizontally to accommodate a growing number of components.
- **Flexibility**: Simplifies addition or removal of components without system-wide impact.

## Disadvantages

- **Complexity**: Managing multiple topics can be intricate.
- **Debugging**: Indirect communication makes debugging challenging.

# Issues and Considerations

- **Topic Management**: Clearly define and document topics to avoid communication mismatches.
- **Error Handling**: Implement robust error handling to manage failures in publishing or subscribing.
- **Debugging Challenges**: Debugging can be challenging due to the indirect nature of communication.

# When to Use Publish–Subscribe

- **Scalability is a Concern**: When a system needs to scale horizontally with ease.
- **Components Need Independence**: In scenarios where independent evolution of components is crucial.
- **Event-Driven Architecture**: When events trigger actions across different parts of the system.

# When Not to Use Publish–Subscribe

- **Simple Systems**: For small, straightforward systems without a need for extensive scalability or flexibility.
- **Tight Component Coupling**: When components require direct and tightly-coupled communication.

# Best Practices

- **Clearly Defined Topics**: Document topics to ensure seamless communication.
- **Error Handling**: Implement robust error handling mechanisms.
- **Monitoring and Logging**: Integrate monitoring and logging for enhanced visibility.

In conclusion, the Publish–Subscribe pattern emerges as a potent tool in the microservices arsenal. Leveraging its capabilities in a .NET Core e-commerce system brings forth a resilient and scalable architecture, paving the way for flexible and independent component evolution.

# References:

- [MediatR Documentation](https://github.com/jbogard/MediatR)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Visual Studio Code Debugging with Docker](https://code.visualstudio.com/docs/containers/debug-common)
