+++
title = 'Strategy Pattern'
date = 2022-04-25T11:26:09Z
draft = false
series = "Behavioral Design Patterns"
tags = ["Design Patterns", "Strategy"]
+++

When designing software systems, it's essential to create flexible and maintainable code. One way to achieve this is by using design patterns, which are well-established solutions to common software design problems. The Strategy Pattern is one such pattern that offers a structured approach to defining a family of algorithms and making them interchangeable.

In this blog post, we'll explore the Strategy Pattern in detail, its rationale, how it solves problems, and its application in various contexts, including microservices and modular monolithic architectures, using C# .NET Core Web API and repositories. We'll also discuss its advantages, disadvantages, best practices, and recommended use cases.

# What is the Strategy Pattern?

The Strategy Pattern is a behavioral design pattern that defines a family of algorithms, encapsulates each one, and makes them interchangeable. It allows a client to choose the appropriate algorithm to execute at runtime without altering the client's code. This promotes flexibility and ensures that changes in algorithm behavior do not affect the client code.

## Rationale

The Strategy Pattern aims to solve a common problem in software design: the need to define multiple algorithms for a particular task and dynamically select one at runtime. By encapsulating each algorithm within its own class, the Strategy Pattern enables the client to select the appropriate algorithm based on the context or user input. This separation of concerns makes the code more maintainable and less error-prone.

## Problems It Resolves

The Strategy Pattern addresses several issues in software design:

- **Reduced Code Duplication**: It helps eliminate code duplication by encapsulating each algorithm in a separate class.

- **Improved Code Maintenance**: Modifications or additions to algorithms don't affect the client code, making maintenance easier.

- **Dynamic Algorithm Selection**: It allows dynamic selection of algorithms at runtime based on changing requirements.

- **Testing and Debugging**: Individual algorithms are easier to test and debug in isolation.

# Using the Strategy Pattern in C# .NET Core

Let's illustrate the Strategy Pattern using C# .NET Core. In this example, we will create a simplified e-commerce system that calculates discounts for products. We'll define multiple discount calculation algorithms and allow the client to choose one at runtime.

## Step 1: Define the Strategy Interface

```csharp
public interface IDiscountStrategy
{
    decimal ApplyDiscount(decimal originalPrice);
}
```

## Step 2: Implement Concrete Strategies

```csharp
public class BlackFridayDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal originalPrice)
    {
        return originalPrice * 0.5m;
    }
}

public class ChristmasDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal originalPrice)
    {
        return originalPrice * 0.7m;
    }
}
// Additional discount strategies can be added as needed.

```

## Step 3: Context Class

```csharp
public class Product
{
    private IDiscountStrategy _discountStrategy;

    public Product(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal CalculateDiscountedPrice(decimal originalPrice)
    {
        return _discountStrategy.ApplyDiscount(originalPrice);
    }
}
```

## Step 4: Client Code

```csharp
var product = new Product(new BlackFridayDiscount());
var discountedPrice = product.CalculateDiscountedPrice(100);
```

In this example, the client can switch between different discount strategies by passing the desired strategy to the Product class. The Strategy Pattern allows us to add new discount strategies without modifying existing code.

# Strategy Pattern in Microservices and Modular Monolithic Architectures

The Strategy Pattern can be applied in both microservices and modular monolithic architectures to handle various tasks. In microservices, each microservice can encapsulate its logic using the Strategy Pattern, making it easy to replace or update individual components. In modular monolithic architectures, it enables better organization and maintenance of different modules.

# Advantages of the Strategy Pattern

- **Flexibility**: Easily switch between algorithms at runtime.
- **Extensibility**: Add new algorithms without changing existing code.
- **Testability**: Test individual strategies in isolation.
- **Maintenance**: Promotes clean, maintainable code.
- **Reusability**: Reuse strategies across different parts of the application.

# Disadvantages of the Strategy Pattern

- **Increased Number of Classes**: Implementing the pattern can lead to a larger number of classes.
- **Complexity**: Overusing the pattern can add unnecessary complexity.

# When to Use the Strategy Pattern

Use the Strategy Pattern when:

- You have multiple algorithms for a specific task.
- You need to choose an algorithm dynamically at runtime.
- You want to isolate and test algorithms independently.
- You anticipate changes or additions to the algorithms in the future.

# Use Cases/Scenarios

The Strategy Pattern is suitable for a wide range of scenarios:

- Payment Gateways: Different payment methods (credit card, PayPal, etc.) can be represented as strategies.
- Sorting Algorithms: Choose between quicksort, bubblesort, or mergesort at runtime.
- Validation Rules: Apply various validation rules to user input.
- AI Behavior: Implement different behaviors for AI characters in a game.

# Best Practices

- **Keep Strategies Simple**: Each strategy should be a self-contained, simple algorithm.
- **Clearly Define the Context**: Ensure the context class communicates the purpose of the pattern effectively.
- **Dependency Injection**: Use dependency injection to inject the desired strategy into the context.
- **Use Meaningful Names**: Choose descriptive names for strategies to enhance code readability.
- **Consider Default Strategy**: Provide a default strategy for situations where no specific strategy is selected.

# Conclusion

The Strategy Pattern is a powerful tool for designing flexible and maintainable software systems. By encapsulating algorithms in separate classes and allowing dynamic selection, it enhances code reusability, flexibility, and testability. When used appropriately, it can simplify code maintenance and promote clean design. Whether you're building microservices or working in a modular monolithic architecture, the Strategy Pattern can be a valuable addition to your design toolbox.

Remember that while design patterns like the Strategy Pattern can greatly improve your code, they should be applied judiciously and with a clear understanding of your project's requirements and constraints.#

# References:

- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)
- [Strategy Pattern on Refactoring Guru](https://refactoring.guru/design-patterns/strategy)
