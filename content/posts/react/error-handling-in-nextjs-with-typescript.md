+++
title = 'Error Handling in Next.js With TypeScript'
date = 2025-01-20T28:34:35Z
draft = false
tags = ["Error handling", "Next.js"]
+++

Error handling is a critical aspect of building robust and maintainable web applications. In the context of Next.js, a powerful React framework, and TypeScript, a typed superset of JavaScript, error handling is even more crucial. Combining Next.js with TypeScript allows developers to catch errors at compile time, but runtime errors and exceptions still need careful handling. This blog explores the best practices for error handling in Next.js with TypeScript, providing actionable insights and detailed code samples.

# What is Error Handling in Next.js with TypeScript?

Error handling in Next.js involves identifying, managing, and resolving errors that occur during the application's lifecycle. With TypeScript, the process becomes safer because static type-checking ensures that many potential errors are caught during development. However, runtime errors, such as API failures or unexpected user inputs, still require structured handling to ensure a seamless user experience.

## Why It Matters:

- **User Experience**: A poorly handled error can lead to a broken interface or an incomplete user journey.

- **Maintainability**: Structured error handling ensures the codebase remains clean and manageable.

- **Scalability**: Proper practices reduce the risk of unforeseen issues as the application grows.

# Key Concepts of Error Handling in Next.js

- **Compile-Time vs. Runtime Errors**:

  - Compile-time errors are caught by TypeScript during development.
  - Runtime errors occur during application execution and must be handled programmatically.

- Types of Errors in Next.js:

  - **Server-Side Errors**: Occur during data fetching or server-side rendering (SSR).

  - **Client-Side Errors**: Occur in the browser during user interactions.

  - **API Errors**: Errors returned by third-party or internal APIs.

- **Error Boundaries**: React's mechanism for catching errors during rendering on the client side.

- **Next.js Error Pages**: Built-in support for custom error pages (e.g., `404.js` and `500.js`).

# Best Practices for Error Handling in Next.js with TypeScript

## 1. Static Typing with TypeScript

Static typing is the foundation of error prevention. By using TypeScript, you can:

- Define types for your data and functions.

- Reduce runtime errors due to type mismatches.

Example:

```typescript
type User = {
  id: string;
  name: string;
};

async function fetchUser(userId: string): Promise<User> {
  const response = await fetch(`/api/user/${userId}`);

  if (!response.ok) {
    throw new Error("Failed to fetch user");
  }

  return response.json();
}
```

In this example:

- The User type defines the expected structure of user data.

- TypeScript ensures that the function adheres to this structure.

## 2. Centralized Error Handling

Consolidate error-handling logic to maintain consistency and reduce redundancy.

Example:

```typescript
// utils/errorHandler.ts
export class AppError extends Error {
  constructor(public message: string, public statusCode: number) {
    super(message);
    this.name = "AppError";
  }
}

export function handleError(err: unknown): void {
  if (err instanceof AppError) {
    console.error(`Error: ${err.message} (Status: ${err.statusCode})`);
  } else {
    console.error("An unknown error occurred", err);
  }
}
```

Usage:

```typescript
try {
  const user = await fetchUser("123");
} catch (err) {
  handleError(err);
}
```

In this example code:

- Custom `AppError` class standardizes error objects.

- Centralized `handleError` function ensures consistency in error logging.

## 3. Error Boundaries for React Components

React error boundaries catch rendering errors but only for child components.

Example:

```typescript
import React from "react";

class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(): { hasError: boolean } {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo): void {
    console.error("Error caught by ErrorBoundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

Usage:

```typescript
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

## 4. Custom Error Pages

Next.js supports custom `404.js` and `500.js` pages.

Example:

```typescript
// pages/404.tsx
export default function Custom404() {
  return <h1>404 - Page Not Found</h1>;
}

// pages/500.tsx
export default function Custom500() {
  return <h1>500 - Server-Side Error</h1>;
}
```

These pages provide a user-friendly experience for errors.

## 5. Error Handling in Data Fetching

For server-side or static data fetching, use structured error handling.

Example:

```typescript
// pages/api/user/[id].ts
import { NextApiRequest, NextApiResponse } from "next";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const { id } = req.query;
    if (!id) throw new AppError("User ID is required", 400);

    const user = await getUserFromDatabase(id as string);
    if (!user) throw new AppError("User not found", 404);

    res.status(200).json(user);
  } catch (err) {
    handleError(err);
    res
      .status(err instanceof AppError ? err.statusCode : 500)
      .json({ message: err.message });
  }
}
```

Validate inputs and handle errors in the API route.

## 6. Use Next.js Middleware for Validation

Middleware can handle errors like unauthorized access globally.

Example:

```typescript
// middleware.ts
import { NextResponse } from "next/server";

export function middleware(req: Request) {
  if (!req.headers.get("Authorization")) {
    return new NextResponse("Unauthorized", { status: 401 });
  }
}
```

In this example, the Middleware ensures error handling is centralized for authentication.

## 7. TypeScript-Specific Enhancements

- Define clear types and interfaces to improve error readability.
- Use enums for handling predictable error types.

Example:

```typescript
enum ErrorTypes {
  NOT_FOUND = "NOT_FOUND",
  SERVER_ERROR = "SERVER_ERROR",
}

interface CustomError extends Error {
  type: ErrorTypes;
}
```

# Advantages and Disadvantages of Error Handling in Next.js with TypeScript

## Advantages:

- **Type Safety**: Prevents many common bugs during development.

- **Scalability**: Centralized and structured handling makes it easier to manage large applications.

- **Improved Debugging**: Detailed error messages help developers quickly identify issues.

## Disadvantages:

- **Complexity**: Setting up a robust error-handling system requires effort.

- **Performance Overhead**: Excessive error handling can impact runtime performance if not optimized.

# Issues and Considerations

- **User Experience**: Ensure errors are user-friendly and do not expose sensitive information.

- **Global Error Logging**: Use tools like Sentry for centralized error reporting.

- **Testing**: Validate error handling through unit tests and integration tests.

By adopting these best practices, you can build a robust error-handling system in your Next.js application with TypeScript, ensuring better maintainability, scalability, and user experience.

# References

- [Next.js Documentation](https://nextjs.org/docs)

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

- [React Error Boundaries](https://legacy.reactjs.org/docs/error-boundaries.html)
