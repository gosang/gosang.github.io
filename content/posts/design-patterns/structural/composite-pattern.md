+++
title = 'Composite Pattern'
date = 2021-04-16T12:56:42Z
draft = false
series = "Structural Design Patterns"
tags = ["Design Patterns", "Composite"]
+++

The Composite Pattern is a structural design pattern that allows you to compose objects into tree structures to represent part-whole hierarchies. This pattern lets clients treat individual objects and compositions of objects uniformly. In simpler terms, it enables you to work with both individual objects and groups of objects in a unified manner.

# Rationale behind the Composite Pattern

The primary goal of the Composite Pattern is to treat individual objects and compositions of objects uniformly. This is achieved by defining a common interface for both leaf nodes (individual objects) and composite nodes (collections of objects). Clients can then operate on these elements without needing to distinguish between them.

# Problems Addressed by the Composite Pattern

- **Simplified Client Code**: Without the Composite Pattern, client code dealing with complex hierarchies becomes convoluted as it needs to handle individual objects and their compositions differently. The Composite Pattern simplifies this by offering a uniform interface.

- **Scalability**: As your system grows, managing complex object structures becomes challenging. The Composite Pattern provides a scalable solution by allowing you to treat both leaf and composite nodes uniformly.

# How the Composite Pattern Works

## Interface Definition

Let's start with an example in the context of an e-commerce system using C# .NET Core. Suppose we have a product hierarchy with both individual products and product categories:

```csharp
public interface IComponent
{
    void DisplayInfo();
}

public class Product : IComponent
{
    private readonly string _name;

    public Product(string name)
    {
        _name = name;
    }

    public void DisplayInfo()
    {
        Console.WriteLine($"Product: {_name}");
    }
}

public class ProductCategory : IComponent
{
    private readonly List<IComponent> _children = new List<IComponent>();
    private readonly string _name;

    public ProductCategory(string name)
    {
        _name = name;
    }

    public void Add(IComponent component)
    {
        _children.Add(component);
    }

    public void DisplayInfo()
    {
        Console.WriteLine($"Category: {_name}");

        foreach (var child in _children)
        {
            child.DisplayInfo();
        }
    }
}
```

## Using the Composite Pattern

Now, let's demonstrate how to use the Composite Pattern in the context of an e-commerce system:

```csharp
class Program
{
    static void Main()
    {
        var laptop = new Product("Laptop");
        var smartphone = new Product("Smartphone");

        var electronicsCategory = new ProductCategory("Electronics");
        electronicsCategory.Add(laptop);
        electronicsCategory.Add(smartphone);

        var clothing = new Product("Shirt");
        var shoes = new Product("Running Shoes");

        var apparelCategory = new ProductCategory("Apparel");
        apparelCategory.Add(clothing);
        apparelCategory.Add(shoes);

        var rootCategory = new ProductCategory("Root");
        rootCategory.Add(electronicsCategory);
        rootCategory.Add(apparelCategory);

        rootCategory.DisplayInfo();
    }
}
```

The output of this code will display the entire product hierarchy, treating both individual products and product categories uniformly.

# Advantages and Disadvantages

## Advantages of the Composite Pattern

- **Uniformity**: Treats individual objects and compositions uniformly, simplifying client code.
- **Scalability**: Easily scales to accommodate new types of objects in the hierarchy.
- **Flexibility**: Clients can work with both simple and complex structures seamlessly.

## Disadvantages of the Composite Pattern

- **Complexity**: Introducing the Composite Pattern may overcomplicate the code if the hierarchy is simple and unlikely to change.

- **Runtime Overhead**: There can be a slight performance overhead due to the recursive nature of operations on the composite structure.

# When to Use the Composite Pattern

Use the Composite Pattern when:

- You have a hierarchical structure of objects.
- You want clients to treat individual objects and compositions of objects uniformly.
- You need to represent part-whole hierarchies.

# Best Practices

- **Use Abstraction**: Define a common interface for both leaf and composite nodes to ensure a consistent API.

- **Recursive Traversal**: Implement a recursive traversal mechanism for handling complex hierarchies.

- **Careful Design**: Assess the complexity of your hierarchy before adopting the Composite Pattern to avoid unnecessary overhead.

# Conclusion

The Composite Pattern is a powerful tool for managing hierarchies in a unified manner. By providing a common interface for both individual objects and compositions, it simplifies client code, enhances scalability, and offers flexibility. However, its application should be considered judiciously, keeping in mind the structure and dynamics of your system.

# References:

- Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). Design Patterns: Elements of Reusable Object-Oriented Software. Addison-Wesley.
