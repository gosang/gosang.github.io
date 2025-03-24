+++
title = 'Callback Promise Async Await in Javascript'
date = 2025-01-20T20:34:35Z
draft = false
tags = ["Callback", "Promise", "Async/Await", "JavaScript"]
+++

JavaScript is a single-threaded language, which means it executes one operation at a time. But real-world applications often need to handle multiple tasks, like fetching data, reading files, or waiting for user input. To manage these asynchronous operations, JavaScript provides Callbacks, Promises, and Async/Await. This blog explores these tools in depth, covering their rationale, usage, advantages, and challenges.

# 1. Callback

A `Callback` is a function passed as an argument to another function and executed after the completion of that function. It allows for non-blocking asynchronous programming.

## Key Concepts

- A callback is invoked after a task is completed.
- Can be synchronous or asynchronous, depending on its execution.
- Heavily used in libraries like jQuery and Node.js.
- **Non-blocking**: Callbacks prevent the main thread from waiting for an operation to complete.
- **Flexibility**: Enables custom handling of asynchronous results.

Example

```javascript
function fetchData(callback) {
  setTimeout(() => {
    const data = { name: "John", age: 30 };
    callback(null, data);
  }, 2000);
}

fetchData((error, result) => {
  if (error) {
    console.error("Error fetching data:", error);
  } else {
    console.log("Fetched data:", result);
  }
});
```

## Advantages

- Simple to implement for basic use cases.
- Works well for small, isolated tasks.

## Disadvantages

- **Callback Hell**: Nested callbacks can make code hard to read and maintain.
- **Error Handling**: Managing errors across callbacks can become messy.

# 2. Promise

A Promise represents a value that may be available now, in the future, or never. It is an object representing the eventual completion (or failure) of an asynchronous operation. Introduced in ES6, Promises provide a more structured way of handling asynchronous operations, avoiding the "callback hell" problem.

## Key Concepts

- It has three
  - **states**:
    - **Pending**: Initial state.
    - **Fulfilled**: Operation completed successfully.
    - **Rejected**: Operation failed.
  - **Methods**:
    - .then(onFulfilled, onRejected) for handling success and failure.
    - .catch(onRejected) for handling errors.
    - .finally(onFinally) for cleanup actions.
- **Chaining**: Promises allow chaining `.then()` for sequential operations.
- **Cleaner Syntax**: Reduces nested callbacks.

Example

```javascript
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      if (success) {
        resolve({ name: "John", age: 30 });
      } else {
        reject("Failed to fetch data");
      }
    }, 2000);
  });
}

fetchData()
  .then((result) => {
    console.log("Fetched data:", result);
  })
  .catch((error) => {
    console.error("Error fetching data:", error);
  });
```

## Advantages

- Avoids callback hell.
- Simplifies complex asynchronous workflows.
- Provides robust error handling with `.catch()`.
- Composability: Use methods like `Promise.all()` and `Promise.race()` for managing multiple promises.

## Disadvantages

- Can become verbose for multiple sequential operations.
- Debugging can be tricky in some cases.

# 3. Async/Await

Async/Await, introduced in ES2017 (ES8), is syntactic sugar built on Promises, making asynchronous code look synchronous. The `async` keyword declares an asynchronous function, and the `await` keyword pauses execution until a Promise is resolved or rejected.

## Key Concepts

- **Readability**: Simplifies asynchronous code, making it more readable.
- **Error Handling**: Uses `try...catch` for errors, akin to synchronous code.

Example

```javascript
async function fetchData() {
  try {
    const response = await new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({ name: "John", age: 30 });
      }, 2000);
    });
    console.log("Fetched data:", response);
  } catch (error) {
    console.error("Error fetching data:", error);
  }
}

fetchData();
```

## Advantages

- Cleaner and more readable syntax.
- Easier error handling with try...catch.
- Debugging is simpler with better stack traces.

## Disadvantages

- Still relies on Promises under the hood and hence it must be used with Promises.
- Not suitable for every scenario, like parallel execution and can lead to blocking code if not used carefully.

# Comparing Callback, Promise, and Async/Await

| Feature        | Callback            | Promise                    | Async/Await                   |
| -------------- | ------------------- | -------------------------- | ----------------------------- |
| Readability    | Low (callback hell) | Moderate                   | High (linear flow)            |
| Error Handling | Complex             | Simplified with `.catch()` | Simplified with `try...catch` |
| Debugging      | Challenging         | Moderate                   | Easier (better stack traces)  |
| Composition    | Difficult           | Easy (`Promise.all`)       | Easy (using multiple `await`) |
| Overhead       | Minimal             | Slight                     | Moderate                      |
| Usage          | Basic scenarios     | Intermediate workflows     | Modern applications           |

# Issues, Considerations, and Best Practices

## Callback

- **Issues**: Prone to callback hell, difficult error management.
- **Best Practices**:
  - Avoid deeply nested callbacks by splitting logic into smaller functions or using Promises.
  - Use named functions instead of inline callbacks for better readability.

## Promise

- **Issues**: Chaining multiple `.then()` calls can still become hard to read.
- **Best Practices**:
  - Use `Promise.all()` for concurrent operations and `.catch()` for centralized error handling.
  - Use `.catch()` for error handling and avoid redundant `.then()`.

## Async/Await

- **Issues**:
  - Modern applications: Requires Promises, not inherently suitable for parallel operations.
  - Mixing `await` with blocking operations can degrade performance.
- **Best Practices**:
  - Use await only where necessary.
  - Combine with `Promise.all` for parallel execution, instead of multiple await calls.

## When deciding which to use:

- Use Promises or Async/Await instead of callbacks for better maintainability.
- Opt for Callbacks for very simple asynchronous operations.
- Use Promises for complex workflows or when transitioning legacy code.
- Leverage Async/Await for clean, readable, and modern JavaScript applications.
- Always handle errors to avoid unhandled rejections or exceptions.
- Use tools like linters or debuggers to maintain clean and efficient code.

# Conclusion

Each method, Callback, Promise, and Async/Await has its place in JavaScript. While Callbacks are simple and foundational, Promises bring clarity, and Async/Await offers the best readability and error handling. For modern applications, Async/Await is often the preferred choice due to its simplicity and maintainability.

# References

- [MDN Web Docs: Callbacks](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)
- [MDN Web Docs: Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN Web Docs: Async/Await](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Promises)
