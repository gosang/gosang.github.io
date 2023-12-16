+++
title = 'Caching Pattern'
date = 2023-07-28T08:43:48Z
draft = false
series = "Design Patterns"
tags = ["Caching"]
+++

Caching is a crucial aspect of modern software development, especially in the context of microservices or modular monolithic applications. It helps improve performance, reduce latency, and enhance the overall user experience. In this blog post, we will delve into five caching patterns: `Cache Aside`, `Write Through`, `Read Through`, `Write Back`, and `Write Around`, exploring their characteristics, use cases, and code snippets using .NET.

# 1. Cache Aside

## Overview:

Cache Aside, also known as Lazy Loading, is a popular caching pattern where the application code is responsible for managing the cache. When a request is made, the application checks the cache first. If the data is present, it's returned; otherwise, the data is fetched from the data source, and the cache is updated.

## Rationale:

This pattern provides a simple and flexible approach. It helps in reducing load on the database and promotes scalability by enabling developers to control what data is cached and when.

## Example implementation:

```csharp
// C# .NET Core Code
public async Task<Product> GetProductAsync(int productId)
{
    var cacheKey = $"Product:{productId}";
    var cachedProduct = await _cache.GetAsync<Product>(cacheKey);

    if (cachedProduct != null){
        return cachedProduct;
    }

    var databaseProduct = await _repository.GetProductAsync(productId);
    await _cache.SetAsync(cacheKey, databaseProduct);

    return databaseProduct;
}
```

# 2. Write Through

## Overview:

Write Through is a pattern where data is written to the cache and the underlying data source simultaneously. This ensures that the cache is always consistent with the database.

## Rationale:

It helps maintain data consistency between the cache and the data source. Although writes might be slower due to the dual write, reads benefit from the up-to-date cache.

## Example implementation:

```csharp
// C# .NET Core Code
public async Task UpdateProductAsync(Product product)
{
    var cacheKey = $"Product:{product.Id}";

    // Update cache and database simultaneously
    await _cache.SetAsync(cacheKey, product);
    await _repository.UpdateProductAsync(product);
}
```

# 3. Read Through

## Overview:

Read Through involves reading data from the cache. If the data is not found, it is loaded from the data source, and the cache is updated.

## Rationale:

This pattern is suitable for read-heavy workloads, as it minimizes the load on the data source by reading from the cache whenever possible.

## Example implementation:

```csharp
// C# .NET Core Code
public async Task<List<Product>> GetAllProductsAsync()
{
    var cacheKey = "AllProducts";
    var cachedProducts = await _cache.GetAsync<List<Product>>(cacheKey);

    if (cachedProduct != null){
        return cachedProduct;
    }

    var databaseProducts = await _repository.GetAllProductsAsync();
    await _cache.SetAsync(cacheKey, databaseProducts);

    return databaseProducts;
}
```

# 4. Write Back

## Overview:

Write Back, or Write Behind, involves writing data to the cache and deferring the actual write to the data source to a later time.

## Rationale:

This pattern optimizes write performance by allowing the application to continue without waiting for the data source write operation to complete.

## Example implementation:

```csharp
// C# .NET Core Code
public void AddProductToCart(Product product)
{
    var cacheKey = $"Cart:{_userId}";
    var currentCart = _cache.Get<List<Product>>(cacheKey) ?? new List<Product>();

    currentCart.Add(product);

    // Update cache immediately
    _cache.Set(cacheKey, currentCart);

    // Defer the actual database write
    Task.Run(() => _repository.AddProductToCartAsync(_userId, product));
}
```

# 5. Write Around

## Overview:

Write Around involves writing data directly to the data source without caching it. Subsequent reads fetch the data from the data source and populate the cache.

## Rationale:

This pattern is useful for large or infrequently accessed data that doesn't benefit significantly from caching.

## Example implementation:

```csharp
// C# .NET Core Code
public async Task DeleteProductAsync(int productId)
{
    // Remove from cache, actual delete in the database
    var cacheKey = $"Product:{productId}";
    await _cache.RemoveAsync(cacheKey);
    await _repository.DeleteProductAsync(productId);
}
```

# Advantages and Disadvantages

## Cache Aside:

**Advantages**: Developer control, flexibility.
**Disadvantages**: Cache staleness, code complexity.

## Write Through:

**Advantages**: Consistency, reduced staleness.
**Disadvantages**: Increased write latency.

## Read Through:

**Advantages**: Read optimization, reduced database load.
**Disadvantages**: Cache staleness, increased read latency.

## Write Back:

**Advantages**: Improved write performance.
**Disadvantages**: Potential data loss on system failure.

## Write Around:

**Advantages**: Minimized cache pollution, suitable for large data.
**Disadvantages**: Increased read latency for uncached data.

# Use Cases and Best Practices

## Cache Aside:

**Use Cases**: Frequently read but infrequently updated data.
**Best Practices**: Careful cache invalidation, monitor cache size.

## Write Through:

**Use Cases**: Systems requiring strong consistency.
**Best Practices**: Optimize write operations, handle failures gracefully.

## Read Through:

**Use Cases**: Read-heavy workloads.
**Best Practices**: Optimize cache refresh strategy, monitor cache hits.

## Write Back:

**Use Cases**: Write-intensive workloads with acceptable data loss.
**Best Practices**: Implement reliable background processes, handle failures gracefully.

## Write Around:

**Use Cases**: Large or infrequently accessed data.
**Best Practices**: Evaluate caching impact, monitor cache effectiveness.

# Conclusion

Caching patterns play a vital role in optimizing the performance of e-commerce systems. Choosing the right pattern depends on the specific requirements and characteristics of the application. By leveraging .NET, Redis, and React, developers can implement these patterns effectively, enhancing the overall user experience. Remember to consider the trade-offs and best practices to make informed decisions in your caching strategy.

# References:

- [Microsoft Documentation - Caching in .NET](https://docs.microsoft.com/en-us/aspnet/core/performance/caching)
- [Redis Documentation](https://redis.io/documentation)
