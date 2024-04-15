+++
title = 'Liskov Substitution Principle (LSP)'
date = 2023-04-01T13:13:36Z
draft = false
series = "SOLID Design Principles"
tags = ["SOLID", "LSP"]
+++

In the realm of object-oriented design, the Liskov Substitution Principle (LSP) stands as a guiding light, emphasizing the importance of substitutability among objects of a base class and its derived classes. Coined by Barbara Liskov, this principle ensures that derived classes can be used interchangeably with their base classes without affecting the correctness of the program. In this blog post, we will look into the Liskov Substitution Principle, its rationale, practical examples in C# .NET Core within the context of e-commerce, and discuss its relevance in both microservices and modular monolithic architectures.

# What is the Liskov Substitution Principle?

Liskov Substitution Principle (LSP): Subtypes must be substitutable for their base types without altering the correctness of the program.

# Rationale

The Liskov Substitution Principle builds on the idea of polymorphism, ensuring that objects of a derived class can be used wherever objects of the base class are expected. This principle aims to maintain the integrity of the program's logic when substituting objects, promoting code reliability and flexibility.

# LSP in Action: C# .NET Core Examples

Let's explore the Liskov Substitution Principle in the context of an e-commerce system with different payment service providers.

```csharp
// Base class
public class PaymentProvider
{
    public virtual void ProcessPayment(decimal amount) { /* common logic */ }
}

// Derived class 1
public class CreditCardPaymentProvider : PaymentProvider
{
    public override void ProcessPayment(decimal amount)
    {
        // credit card specific logic
        base.ProcessPayment(amount);
    }
}

// Derived class 2
public class PayPalPaymentProvider : PaymentProvider
{
    public override void ProcessPayment(decimal amount)
    {
        // PayPal specific logic
        base.ProcessPayment(amount);
    }
}
```

In this example, both `CreditCardPaymentProvider` and `PayPalPaymentProvider` are substitutable for the base class `PaymentProvider` without altering the correctness of the program.

# Advantages and Disadvantages

## Advantages

- **Flexibility**: Enables the use of different implementations interchangeably.
- **Scalability**: New implementations can be added without modifying existing code.
- **Maintainability**: Changes to derived classes do not impact the correctness of the program.

## Disadvantages

**Complex Hierarchies**: Overly complex class hierarchies can make the system harder to understand.

# When to Use LSP

Use LSP when:

- You want to ensure substitutability among objects.
- You aim for a flexible and scalable codebase.
- You are working on a project with evolving requirements.

# Best Practices

- **Understand Contracts**: Clearly define contracts (interfaces or base classes) that derived classes must adhere to.
- **Documentation**: Document the expected behavior of base and derived classes for future reference.
- **Unit Testing**: Implement unit tests to verify that substitutability is maintained.
- **Review Class Hierarchies**: Regularly review and simplify class hierarchies to avoid unnecessary complexity.
- **Educate the Team**: Ensure the development team understands the principles and benefits of LSP.

# Conclusion

The Liskov Substitution Principle empowers developers to create adaptable and extensible systems. Whether in microservices or modular monoliths, adhering to LSP ensures that new implementations seamlessly integrate with existing code. By embracing LSP in C# .NET Core, developers can build robust and maintainable solutions that gracefully accommodate changes and enhancements.

# References:

- Liskov, B. (1987). Data Abstraction and Hierarchy. SIGPLAN Notices, 23(5), 17-34.
- Martin, R. C. (2003). Agile Software Development: Principles, Patterns, and Practices. Prentice Hall.
