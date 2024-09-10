+++
title = 'Implementing Minimal API With ASP.NET Core Using Request Endpoint Response'
date = 2024-09-06T17:34:32+01:00
draft = false
tags = ["Minimal API", "PEPR", "ASP.Net Core"]
+++

With the release of ASP.NET Core 6, the framework introduced Minimal APIs, providing a streamlined approach to building APIs. This model is particularly well-suited for small-scale, microservices-based applications or simple APIs that don't require the overhead of a full MVC structure. In this blog, we will explore the Minimal API approach using the REPR (Request-Endpoint-Response) pattern, which simplifies API development by breaking down the process into three primary steps: handling requests, defining endpoints, and crafting responses.

# What is ASP.NET Core 8 Minimal API with REPR?

ASP.NET Core 8 Minimal API offers a lightweight way to create HTTP APIs with minimal dependencies, setup, and boilerplate code. The REPR pattern in this context stands for Request-Endpoint-Response:

- **Request**: The incoming HTTP request that the API receives.
- **Endpoint**: The specific route or URL path that the API listens to. It typically includes the business logic to process the request.
- **Response**: The HTTP response returned to the client, often containing data or status information.

# Why Minimal APIs and REPR?

The main goal of Minimal APIs is to simplify the API development process by removing unnecessary abstractions and focusing on what’s truly essential. This model aligns with the principles of microservices, where simplicity and lightweight design are key. The REPR pattern fits perfectly into this model by further streamlining the API development process.

# Key Concepts

- **Simplicity**: Minimal APIs reduce the boilerplate code needed, making it easier and faster to create APIs.
- **Flexibility**: Developers can define endpoints without the need for complex routing configurations.
- **Performance**: With less overhead, Minimal APIs can perform better in certain scenarios, particularly when used in microservices or serverless environments.

# Minimal API vs. Controller-based API

| Feature          | Minimal API                                        | Controller-based API                                |
| ---------------- | -------------------------------------------------- | --------------------------------------------------- |
| Setup Complexity | Simple, with less code and configuration           | Requires more setup with controllers and routing    |
| Flexibility      | High, ideal for small APIs and microservices       | Structured, suitable for large applications         |
| Routing          | Defined inline with endpoints                      | Defined via attribute routing or convention-based   |
| Testability      | Requires a different approach for unit testing     | Easier to unit test with dedicated controllers      |
| Maintainability  | May become harder to maintain as the project grows | Easier to maintain with well-structured controllers |

# Implementing Minimal API with REPR in ASP.NET Core 8

## Project Setup

First, let's create a new ASP.NET Core 8 project:

```bash
dotnet new web -n MinimalApiWithREPR
cd MinimalApiWithREPR
```

## Creating a Minimal API

Now, let’s implement a simple CRUD API for managing Product entities using the REPR pattern.

### Define the Model

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

### In-Memory Data Store

```csharp
public static class ProductStore
{
    public static List<Product> Products { get; set; } = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Price = 1000M },
        new Product { Id = 2, Name = "Phone", Price = 500M }
    };
}
```

### Implementing CRUD Endpoints

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", () => Results.Ok(ProductStore.Products)) // READ (Request)
    .WithName("GetProducts") // Endpoint
    .WithTags("Products"); // Response

app.MapGet("/products/{id}", (int id) =>
{
    var product = ProductStore.Products.FirstOrDefault(p => p.Id == id);
    return product != null ? Results.Ok(product) : Results.NotFound();
})
.WithName("GetProductById")
.WithTags("Products");

app.MapPost("/products", (Product product) =>
{
    product.Id = ProductStore.Products.Max(p => p.Id) + 1;
    ProductStore.Products.Add(product);
    return Results.Created($"/products/{product.Id}", product);
})
.WithName("CreateProduct")
.WithTags("Products");

app.MapPut("/products/{id}", (int id, Product updatedProduct) =>
{
    var product = ProductStore.Products.FirstOrDefault(p => p.Id == id);
    if (product is null)
    {
        return Results.NotFound();
    }

    product.Name = updatedProduct.Name;
    product.Price = updatedProduct.Price;

    return Results.NoContent();
})
.WithName("UpdateProduct")
.WithTags("Products");

app.MapDelete("/products/{id}", (int id) =>
{
    var product = ProductStore.Products.FirstOrDefault(p => p.Id == id);
    if (product is null)
    {
        return Results.NotFound();
    }

    ProductStore.Products.Remove(product);
    return Results.NoContent();
})
.WithName("DeleteProduct")
.WithTags("Products");

app.Run();
```

In this example:

- **Request**: Each endpoint receives an HTTP request. For instance, app.MapGet("/products") handles a GET request to retrieve all products.
- **Endpoint**: The logic for handling each request is defined within the endpoint itself. This keeps the API definition clear and concise.
- **Response**: The API returns a response using methods like Results.Ok(), Results.Created(), Results.NoContent(), etc. These methods standardize the HTTP response generation.

# Advantages and Disadvantages of Minimal API with REPR

| Advantage                                                                                                                             | Disadvantage                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Reduced Boilerplate**: The Minimal API approach requires less code, making it quicker to set up.                                    | **Scalability Issues**: As the project grows, the lack of structure can lead to maintenance challenges. |
| **High Performance**: With fewer layers of abstraction, Minimal APIs can be more performant, especially in high-throughput scenarios. | **Testability**: Unit testing may require more effort compared to the Controller-based approach.        |
| **Easy Prototyping**: Ideal for building prototypes or small services quickly.                                                        | **Limited Features**: Lacks the rich features available in MVC like model binding, filters, etc.        |

# When to Use Minimal API with REPR

| When to use                                                                      | When to avoid                                                                                                                               |
| -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Microservices**: When developing microservices or small, independent services. | **Large Applications**: When building large, complex applications that require sophisticated routing, validation, and dependency injection. |
| **Prototyping**: Quick prototyping and experimentation.                          | **Complex Business Logic**: When the business logic is complex and requires better separation of concerns.                                  |
| **Serverless Functions**: Lightweight functions that need minimal setup.         |                                                                                                                                             |

# Issues, Considerations, and Best Practices

- **Code Organization**: As the number of endpoints grows, consider organizing them in separate classes or files to maintain readability.
- **Security**: Implement appropriate security measures like authentication and authorization.
- **Versioning**: Plan for API versioning if you anticipate future changes.
- **Testing**: Ensure you have a testing strategy in place, as testing minimal APIs may require different approaches compared to controller-based APIs.
- **Error Handling**: Implement consistent error handling across all endpoints to ensure reliable API behavior.

# Conclusion

ASP.NET Core 8's Minimal API with the REPR pattern is a powerful tool for building lightweight, performant APIs. By following the principles of Request-Endpoint-Response, developers can create clean, efficient APIs suited to a variety of use cases, from microservices to serverless functions. However, careful consideration should be given to the scalability and maintainability of such APIs as the application grows.

A sample Minimal API can be found at [CQRS.MinimalAPI.Sample](https://github.com/gosang/CQRS.MinimalAPI.Sample).

# References

- [Microsoft Docs on Minimal APIs](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [ASP.NET Core 8 Documentation](https://learn.microsoft.com/en-us/aspnet/core/introduction-to-aspnet-core?view=aspnetcore-8.0)
