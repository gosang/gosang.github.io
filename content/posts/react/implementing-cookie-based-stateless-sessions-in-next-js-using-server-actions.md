+++
title = 'Implementing Cookie Based Stateless Sessions in Next Js Using Server Actions'
date = 2024-08-18T12:44:19+01:00
draft = false
tags = ["Next.js", "SSR", "Cookie Sessions"]
+++

# Introduction

In modern web development, managing user sessions efficiently and securely is crucial. One approach is to use cookie-based stateless sessions. This method leverages cookies to store session data client-side, reducing server-side complexity and improving scalability. In this blog post, we'll explore how to implement cookie-based stateless sessions in a Next.js application using Server Actions, focusing on managing user settings like theme preferences.

# What is Cookie-Based Stateless Session?

A cookie-based stateless session involves storing session data in cookies, allowing the server to remain stateless. Each request from the client includes the session data, which the server can validate and use without needing to maintain session state between requests. This approach is particularly beneficial for scaling applications, as it eliminates the need for server-side session storage.

# Why Cookie-Based Stateless Session?

Using cookie-based stateless sessions in Next.js with Server Actions provides several benefits:

- **Scalability**: No need for centralized session storage, reducing server load.
- **Performance**: Faster request processing since session data is sent with each request.
- **Simplicity**: Simplifies backend logic by offloading session management to the client.

# Key Concepts

## Cookies

Cookies are small pieces of data stored on the client's browser. They can be used to persist user sessions, preferences, and other stateful information.

## Stateless Sessions

In a stateless session, the server does not maintain any session information. Each request is self-contained and carries all the necessary data for the server to process it.

## Server Actions

Server Actions in Next.js allow you to perform server-side logic as part of your application's data fetching and handling flow. They are useful for handling operations that require server-side validation or data manipulation.

# Key Functions

- **Setting Cookies**: Use set-cookie headers to store session data.
- **Reading Cookies**: Parse cookies from incoming requests to retrieve session data.
- **Server Actions**: Handle server-side logic and data processing.

# A Sample Implementation

Let's create a sample Next.js project to demonstrate a cookie-based stateless session that stores and retrieves user settings like theme preferences.

## Step 1: Setup Next.js Project

Create a new Next.js project:

```bash
npx create-next-app@latest my-nextjs-app
cd my-nextjs-app
```

Install necessary dependencies:

```bash
npm install cookie
npm install typescript @types/node @types/react --save-dev
```

## Step 2: Create Utility Functions for Cookies

Create a utility file lib/cookies.ts to handle cookie operations.

```typescript
// lib/cookies.ts

import { NextApiResponse } from "next";
import { parse, serialize } from "cookie";

const COOKIE_NAME = "user-settings";

export const setCookie = (
  res: NextApiResponse,
  value: string,
  options: any = {}
) => {
  const stringValue =
    typeof value === "object" ? JSON.stringify(value) : String(value);
  const serializedCookie = serialize(COOKIE_NAME, stringValue, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    maxAge: 60 * 60 * 24 * 30, // 30 days
    path: "/",
    ...options,
  });
  res.setHeader("Set-Cookie", serializedCookie);
};

export const getCookie = (req: any) => {
  const cookies = parse(req.headers.cookie || "");
  return cookies[COOKIE_NAME] ? JSON.parse(cookies[COOKIE_NAME]) : {};
};
```

## Step 3: Implement Server Action to Set and Get Theme

Create a new API route pages/api/settings.ts to handle user settings.

```typescript
// pages/api/settings.ts

import { NextApiRequest, NextApiResponse } from "next";
import { getCookie, setCookie } from "../../lib/cookies";

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === "POST") {
    const { theme } = req.body;
    setCookie(res, { theme });
    res.status(200).json({ message: "Settings saved" });
  } else if (req.method === "GET") {
    const settings = getCookie(req);
    res.status(200).json(settings);
  } else {
    res.setHeader("Allow", ["GET", "POST"]);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

## Step 4: Create a Theme Context

Create a context to manage theme settings in context/ThemeContext.tsx.

```typescript
// context/ThemeContext.tsx

import { createContext, useContext, useEffect, useState } from "react";

const ThemeContext = createContext({
  theme: "light",
  setTheme: (theme: string) => {},
});

export const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  const [theme, setTheme] = useState("light");

  useEffect(() => {
    const fetchSettings = async () => {
      const response = await fetch("/api/settings");
      const data = await response.json();
      if (data.theme) {
        setTheme(data.theme);
      }
    };
    fetchSettings();
  }, []);

  const saveTheme = async (newTheme: string) => {
    setTheme(newTheme);
    await fetch("/api/settings", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ theme: newTheme }),
    });
  };

  return (
    <ThemeContext.Provider value={{ theme, setTheme: saveTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => useContext(ThemeContext);
```

## Step 5: Integrate Theme Context in Application

Wrap your application with the ThemeProvider in pages/\_app.tsx.

```typescript
// pages/_app.tsx

import type { AppProps } from "next/app";
import { ThemeProvider } from "../context/ThemeContext";

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ThemeProvider>
      <Component {...pageProps} />
    </ThemeProvider>
  );
}

export default MyApp;
```

## Step 6: Create Theme Toggle Component

Create a component to toggle between light and dark themes.

```typescript
// components/ThemeToggle.tsx

import { useTheme } from "../context/ThemeContext";

const ThemeToggle = () => {
  const { theme, setTheme } = useTheme();

  const toggleTheme = () => {
    setTheme(theme === "light" ? "dark" : "light");
  };

  return (
    <button onClick={toggleTheme}>
      Toggle to {theme === "light" ? "dark" : "light"} theme
    </button>
  );
};

export default ThemeToggle;
```

## Step 7: Use Theme Toggle in a Page

Use the ThemeToggle component in your index.tsx page.

```typescript
// pages/index.tsx

import ThemeToggle from "../components/ThemeToggle";

export default function Home() {
  return (
    <div>
      <h1>Welcome to Next.js with Cookie-Based Stateless Session</h1>
      <ThemeToggle />
    </div>
  );
}
```

# Advantages and Disadvantages

| Advantages                                                               | Disadvantages                                                                         |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| **Scalability**: No need for server-side session storage.                | **Security**: Requires careful handling to prevent vulnerabilities like XSS and CSRF. |
| **Performance**: Reduces server processing time.                         | **Storage Limitations**: Limited by cookie size constraints (typically 4KB).          |
| **Simplicity**: Easier to manage without centralized session management. |                                                                                       |

# When to Use

| When to use                                                                                     | When not to use                                                                                                   |
| ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Highly Scalable Applications**: Where maintaining server-side session storage is impractical. | **Sensitive Data**: Avoid storing sensitive information in cookies.                                               |
| **Simple Session Data**: Suitable for lightweight session data like user preferences.           | **Complex Session Data**: If the session data is large or complex, server-side storage might be more appropriate. |

# Issues, Considerations, and Best Practices

## Security

- **HTTPOnly and Secure Flags**: Use these flags to prevent client-side scripts from accessing the cookies.
- **Encryption**: Consider encrypting cookie data to enhance security.

## Performance

- **Cookie Size**: Keep the cookie size small to avoid performance issues.
- **Cache-Control**: Ensure proper cache control to prevent stale data.

## Best Practices

- **Validation**: Always validate and sanitize data stored in cookies.
- **Expiry**: Set appropriate expiry times for cookies to manage session duration.

# Conclusion

Implementing cookie-based stateless sessions in Next.js using Server Actions can significantly enhance the scalability and performance of your application. By following the outlined steps and best practices, you can efficiently manage user settings and other lightweight session data securely. This approach is especially beneficial for applications requiring high scalability and simple session management.
