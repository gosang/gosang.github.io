+++
title = 'Open-Closed Principle (OCP)'
date = 2023-02-11T13:13:36Z
draft = false
series = "SOLID Design Principles"
tags = ["SOLID", "OCP"]
+++

In the world of software design, the Open-Closed Principle (OCP) stands tall as a beacon of extensibility and adaptability. Introduced by Bertrand Meyer, this principle advocates that a class should be open for extension but closed for modification. In this blog post, we will explore the Open-Closed Principle, understand its rationale, and see how it can be applied in the context of e-commerce, particularly in order management, shopping cart, and pricing modules. We will provide practical examples in C# .NET Core, discuss advantages and disadvantages, and offer best practices for effective implementation.

# What is the Open-Closed Principle?

Open-Closed Principle (OCP): Software entities (classes, modules, functions) should be open for extension but closed for modification.

# Rationale

The OCP addresses the challenges associated with modifying existing code. By adhering to this principle, we aim to extend the behavior of a system without altering its source code. This contributes to a more maintainable and less error-prone codebase, especially when dealing with evolving requirements.

# OCP in Action: C# .NET Core Examples

Let's delve into the Open-Closed Principle with examples in the context of e-commerce.

#### Order Management

```csharp
// Original class
public class Order
{
    public decimal CalculateTotal() { /* original logic to calculate total */ }
}

// Extension through inheritance
public class DiscountedOrder : Order
{
    public decimal ApplyDiscount() { /* logic to apply discount */ }
}
```

In the order management module, the `Order` class is closed for modification but open for extension. The `DiscountedOrder` class extends the functionality by applying discounts without modifying the original Order class. This adheres to the Open-Closed Principle.

#### Shopping Cart

```csharp
// Original class
public class ShoppingCart
{
    public decimal CalculateTotal() { /* original logic to calculate total */ }
}

// Extension through composition
public class DiscountedShoppingCart : ShoppingCart
{
    private readonly IDiscountStrategy _discountStrategy;

    public DiscountedShoppingCart(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal CalculateTotalWithDiscount()
    {
        // logic to apply discount using the strategy
        return _discountStrategy.ApplyDiscount(base.CalculateTotal());
    }
}
```

In the shopping cart module, the `ShoppingCart` class remains closed for modification, and the `DiscountedShoppingCart` class extends functionality through composition. It applies the discount strategy without altering the original `ShoppingCart` class, aligning with OCP.

#### Pricing Module

```csharp
// Original class
public class PricingCalculator
{
    public decimal CalculatePrice() { /* original logic to calculate price */ }
}

// Extension through composition
public class DiscountedPricingCalculator : PricingCalculator
{
    private readonly IDiscountStrategy _discountStrategy;

    public DiscountedPricingCalculator(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal CalculateDiscountedPrice()
    {
        // logic to apply discount using the strategy
        return _discountStrategy.ApplyDiscount(base.CalculatePrice());
    }
}
```

In the pricing module, the `PricingCalculator` class remains closed for modification, and the `DiscountedPricingCalculator` class extends functionality through composition. It applies the discount strategy without altering the original `PricingCalculator` class, following the Open-Closed Principle.

## Implementing OCP Outcome

Implementing the Open-Closed Principle in the context of e-commerce modules enhances the system's adaptability and maintainability. By extending functionalities without modifying existing code, developers can build scalable and resilient systems that readily embrace changes and additions.

# Advantages and Disadvantages

## Advantages

- **Extensibility**: New functionality can be added without modifying existing code.
- **Maintainability**: Changes are localized to the new code, reducing the risk of unintended side effects.
- **Reusability**: Existing classes can be reused as they are closed for modification.

## Disadvantages

- **Initial Complexity**: Implementing OCP might introduce initial complexity, especially when using design patterns like Strategy or Composition.

# When to Use OCP

Use OCP when:

- You anticipate changes or extensions in the functionality of a system.
- You want to minimize the impact of modifications on existing code.
- You are working on a project with evolving requirements.

# Best Practices

- **Identify Stable and Unstable Areas**: Identify parts of the codebase likely to change and design them to be open for extension.
- **Use Design Patterns**: Explore design patterns like Strategy, Composition, or Dependency Injection to implement OCP.
- **Regular Refactoring**: Periodically review and refactor the codebase to maintain adherence to OCP.
- **Document Extension Points**: Clearly document extension points to guide future developers.
- **Educate the Team**: Ensure the development team understands the principles and benefits of OCP.

# Conclusion

The Open-Closed Principle provides a blueprint for creating adaptable and extensible software. Whether in microservices or modular monoliths, adhering to OCP contributes to a codebase that is resilient to change and readily accommodates new functionality. By leveraging OCP in C# .NET Core, developers can build systems that stand the test of time and evolving requirements.

# References:

- Meyer, B. (1988). Object-Oriented Software Construction. Prentice Hall.
- Martin, R. C. (2003). Agile Software Development: Principles, Patterns, and Practices. Prentice Hall.
