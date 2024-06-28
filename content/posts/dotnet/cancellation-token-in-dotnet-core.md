+++
title = 'CancellationToken in .NET Core'
date = 2024-06-28T12:13:37+01:00
draft = false
tags = ["ASP.Net Core"]
+++

In .NET development, managing asynchronous operations efficiently is essential for building responsive and user-friendly applications. A powerful tool for this purpose is the CancellationToken. This blog will explore what a CancellationToken is, its rationale, the problems it resolves, and how it can be used with C# samples.

# What is a CancellationToken?

A CancellationToken in .NET is a struct that provides a way to signal that an operation should be canceled. It works in conjunction with a CancellationTokenSource, which controls the cancellation token.

## Why CancellationToken?

The primary rationale behind using CancellationToken is to allow cooperative cancellation of asynchronous tasks, which helps in managing resources efficiently and improving user experience by providing a way to cancel operations that are no longer needed.

## Problems It Resolves

- **Resource Management**: Prevents wasting resources on unnecessary tasks.
- **User Experience**: Enables users to cancel long-running operations, making applications more responsive.
- **System Responsiveness**: Ensures that the system does not become unresponsive due to long-running operations.

# Usage Patterns with Scenarios

1. The Problem With Cancelling All Functions
   Cancelling all functions without careful consideration can lead to inconsistent states, especially when functions involve side-effects such as writing to a database.

```csharp
public async Task FetchDataAsync(CancellationToken token)
{
    for (int i = 0; i < 10; i++)
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(1000); // Simulate work
    }
}
```

Using `ThrowIfCancellationRequested` ensures that the operation respects the cancellation request, preventing unnecessary work and resource consumption.

2. Avoid Cancelling Operations After Side-Effects
   Ensure that once a side-effect has occurred, subsequent operations are not canceled to avoid inconsistent states.

```csharp
public async Task ProcessOrderAsync(Order order, CancellationToken token)
{
    // Step 1: Validate order
    token.ThrowIfCancellationRequested();

    // Step 2: Charge payment (side-effect)
    await ChargePaymentAsync(order);

    // Step 3: Update inventory
    token.ThrowIfCancellationRequested(); // Ensure this is not called after side-effects

    await UpdateInventoryAsync(order);
}
```

Placing `ThrowIfCancellationRequested` before side-effects ensures the system remains consistent, avoiding partial operations that could lead to data corruption or other issues.

3. Optional Cancellation Token on Public API and Required Elsewhere
   Expose cancellation tokens as optional in public APIs but enforce their use internally to ensure operations can be canceled if needed.

```csharp
public async Task PublicApiMethodAsync(CancellationToken token = default)
{
    await InternalMethodAsync(token);
}

private async Task InternalMethodAsync(CancellationToken token)
{
    token.ThrowIfCancellationRequested();
    // Internal logic
}
```

This approach provides flexibility for API consumers while ensuring internal operations respect cancellation, maintaining responsiveness and resource efficiency.

4. Use CancellationToken.None After the Point of 'No Cancellation'
   Once an operation reaches a point where cancellation is no longer meaningful, use CancellationToken.None.

```csharp
public async Task CriticalSectionAsync(CancellationToken token)
{
    // Preliminary checks
    token.ThrowIfCancellationRequested();

    // Critical section starts here
    await PerformCriticalOperationAsync(CancellationToken.None);
}
```

Using `CancellationToken.None` ensures that the critical section completes without interruption, as cancellation is no longer relevant.

5. Check CancellationToken.CanBeCanceled
   Check if the token can be canceled before performing operations that involve cancellation logic to avoid unnecessary overhead.

```csharp
public async Task OptionalCancellationAsync(CancellationToken token)
{
    if (token.CanBeCanceled)
    {
        token.ThrowIfCancellationRequested();
    }

    // Perform work
    await Task.Delay(1000);
}
```

This check optimizes performance by avoiding unnecessary operations when cancellation is not supported, reducing overhead and improving efficiency.

6. Ignore CancellationToken if Work is Quick
   For short, non-blocking operations, it might be more efficient to ignore the CancellationToken.

```csharp
public void QuickOperation()
{
    // Quick, non-blocking operation
    Console.WriteLine("Quick operation completed.");
}
```

Skipping the token check for quick operations avoids unnecessary overhead, making the code simpler and more efficient.

# Advantages

- **Responsiveness**: Makes applications more responsive by allowing operations to be canceled.
- **Resource Management**: Frees up resources from unnecessary tasks, improving efficiency.
- **User Control**: Enhances user experience by allowing them to cancel tasks.

# Disadvantages

- **Complexity**: Introduces additional complexity in code management, requiring careful handling.
- **Consistency Issues**: Potential for inconsistent states if not handled properly, especially with side-effects.

# When to Use

- Handling long-running or potentially blocking operations.
- Improving user experience with cancelable tasks.
- Managing resources efficiently, especially in resource-constrained environments.

# When Not to Use

- Operations are extremely short-lived and non-blocking.
- Ensuring consistent state management is challenging.
- The complexity added by implementing cancellation outweighs the benefits.

# Issues, Considerations, and Best Practices

## Issues

- **State Management**: Ensure cancellation does not leave the system in an inconsistent state.
- **Performance**: Overhead of checking for cancellation in performance-critical sections, which could impact performance.

## Considerations

- **Token Propagation**: Ensure tokens are correctly propagated through async calls to maintain consistency.
- **Side-effects**: Carefully manage operations with side-effects to prevent partial execution that could lead to data corruption or other issues.

## Best Practices

- **Strategic Cancellation Points**: Use ThrowIfCancellationRequested at strategic points in your code to ensure efficient cancellation. -** Graceful Cancellation**: Ensure operations can be gracefully canceled without leaving side-effects, maintaining system integrity.
- **Optional Tokens**: Use optional tokens in public APIs for flexibility, while ensuring internal operations respect cancellation.

# Conclusion

CancellationToken is a powerful tool in .NET for managing asynchronous operations. By following these recommended patterns, you can create responsive, efficient, and user-friendly applications. Always consider the implications of cancellation on the system's state and manage tokens wisely.

For more detailed information, refer to [Microsoft documentation](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads).
