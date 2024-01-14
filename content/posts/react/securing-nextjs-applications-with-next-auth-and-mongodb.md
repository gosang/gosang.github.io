+++
title = 'Securing Next.js 13 Applications With Next-Auth and MongoDB'
date = 2024-01-14T09:54:23Z
draft = false
series = "Next.js"
tags = ["React", "Next-Auth"]
+++

Authentication is a critical aspect of web applications, ensuring secure access to resources. In the React Next.js 13 ecosystem, Next-Auth emerges as a powerful solution for handling authentication seamlessly. In this blog, we'll explore Next-Auth Credentials, specifically focusing on its integration with MongoDB to enhance the security of our React Next.js 13 application.

# What is Next-Auth?

Next-Auth is an authentication library for Next.js applications that simplifies the implementation of authentication features. It supports various authentication providers, making it versatile for different use cases. Whether you need social logins, email/password authentication, or custom providers, Next-Auth provides a unified interface.

## Why Next-Auth and Problem Resolution

### Rationale

Next-Auth addresses the complexities associated with implementing authentication in Next.js applications. It streamlines the process, allowing developers to focus on building features rather than wrestling with authentication intricacies.

### Problems Resolved

- **Complexity**: Authentication often involves dealing with complex flows, token management, and security considerations. Next-Auth simplifies these aspects, offering a clean and understandable API.
- **Consistency**: Achieving consistency in authentication across different parts of an application can be challenging. Next-Auth provides a unified approach, making it easy to maintain consistency.
- **Scalability**: As applications grow, so do authentication requirements. Next-Auth supports scalability, adapting to the needs of both small and large projects.

# Getting Started with Next-Auth in Next.js 13

In the context of a React Next.js 13 application, let's dive into the implementation of Next-Auth Credentials using MongoDB for user registration, login, and logout. We'll utilize React 18, Next.js 13, Prisma for ORM and schema, and the Vitest testing framework for unit tests.

## Installation

To get started with Next-Auth in a Next.js 13 project, first install the required dependencies:

```bash
// Create a new next.js project if needed
npx create-next-app@latest
// follow the steps as needed

// Install Next-Auth
npm install next-auth

// Install Prisma for ORM
npm install prisma
npm i @prisma/client bcryptjs
npm i -D ts-node prisma @types/bcryptjs

// Install vitest for unit tests
npm install vitest
```

## Configuration

Configure related Next-Auth Credentials, API route, Session, Providers, Layout:

### Sample Next-Auth Credentials

```typescript
// src/lib/auth.ts
import type { NextAuthOptions } from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";

export const authOptions: NextAuthOptions = {
  session: {
    strategy: "jwt",
  },
  providers: [
    CredentialsProvider({
      name: "Sign in",
      credentials: {
        email: {
          label: "Email",
          type: "email",
          placeholder: "test@test.com",
        },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        const user = { id: "1", name: "test", email: "test@test.com" };
        return user;
      },
    }),
  ],
};
```

### Sample API Route for GET and POST

```typescript
// src/app/api/auth/[…nextauth]/route.ts

import { authOptions } from "@/lib/auth";
import NextAuth from "next-auth";

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

### Sample Session in an API Route

```typescript
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  const session = await getServerSession(authOptions);

  return NextResponse.json({
    authenticated: !!session,
    session,
  });
}
```

### Sample Session Provider

```typescript
// src/app/providers.tsx

"use client";

import { SessionProvider } from "next-auth/react";

type Props = {
  children?: React.ReactNode;
};

export const NextAuthProvider = ({ children }: Props) => {
  return <SessionProvider>{children}</SessionProvider>;
};
```

### Setup Prisma ORM

Setting up Prisma with MongoDB includes creating the datasource provider for MongoDB and scehma for the data model.

#### Create datasource provider

```bash
npx prisma init --datasource-provider mongoDB
```

#### Prisma schema

```typescript
// prisma/schema.prisma

// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongoDB"
  url      = env("DATABASE_URL")
}

model User {
  id       String @id @default(uuid())
  name     String
  email    String @unique
  password String
}

```

#### PrismaClient Instance

```typescript
// src/lib/prisma.ts

import { PrismaClient } from "@prisma/client";

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ["query"],
  });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

## Implementation in React Next.js 13 Application

### Registration

Implementing user registration involves creating a registration form and handling form submission:

#### Sample typescript code for registration

```typescript
// pages/register.tsx
import { useState } from "react";
import { useRouter } from "next/router";

const Register = () => {
  const router = useRouter();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleRegistration = async () => {
    // Perform registration logic using Next-Auth
    const result = await fetch("/api/auth/register", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password }),
    });

    if (result.ok) {
      // Redirect to login page upon successful registration
      router.push("/login");
    } else {
      // Handle registration error
      // ...
    }
  };

  return (
    <div>
      <h1>Register</h1>
      {/* Registration form */}
      <form>
        {/* Form fields for email and password */}
        <button onClick={handleRegistration}>Register</button>
      </form>
    </div>
  );
};

export default Register;
```

### Login

Creating a login page involves similar steps, handling form submission and redirecting upon successful login:

#### Sample typescript code for login

```typescript
// pages/login.tsx
import { useState } from "react";
import { useRouter } from "next/router";

const Login = () => {
  const router = useRouter();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleLogin = async () => {
    // Perform login logic using Next-Auth
    const result = await fetch("/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password }),
    });

    if (result.ok) {
      // Redirect to dashboard upon successful login
      router.push("/dashboard");
    } else {
      // Handle login error
      // ...
    }
  };

  return (
    <div>
      <h1>Login</h1>
      {/* Login form */}
      <form>
        {/* Form fields for email and password */}
        <button onClick={handleLogin}>Login</button>
      </form>
    </div>
  );
};

export default Login;
```

### Logout

Logging out can be implemented by calling the Next-Auth signOut function:

#### Sample typescript code for logout

```typescript
// pages/profile.tsx
import { useSession, signOut } from "next-auth/react";

const Profile = () => {
  const { data: session } = useSession();

  const handleLogout = () => {
    // Sign out using Next-Auth
    signOut();
  };

  return (
    <div>
      <h1>Welcome, {session?.user?.name || "Guest"}</h1>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
};

export default Profile;
```

In this example, we use the Credentials provider for custom email/password authentication. The Prisma adapter seamlessly integrates Next-Auth with the MongoDB database.

# Advantages and Disadvantages

## Advantages:

- **Simplicity**: Streamlined authentication setup with a unified API.
- **Versatility**: Supports various authentication providers.
- **Security**: Credentials provider allows for custom authentication logic, enhancing security.

## Disadvantages:

- **Learning Curve**: Depending on your authentication requirements, there might be a learning curve to understand and configure Next-Auth.

# Issues and Considerations

- **Compatibility**: Regularly check for updates to ensure compatibility with the latest versions of Next.js and other dependencies.
- **Customization**: Consider the effort required for customizing the default Next-Auth interface to match your application's design.
- **Session Storage**: Choose the appropriate session storage option based on your application's needs and security considerations.

# When to Use Next-Auth Credentials

- When you need a simple and unified authentication solution.
- When implementing custom email/password authentication.
- When MongoDB is your chosen database for user data storage.

# When Not to Use Next-Auth Credentials

- When your authentication requirements align better with other providers offered by Next-Auth.
- When the learning curve of Next-Auth outweighs the benefits for your specific use case.

# Best Practices

- **Secure Credentials**: Implement secure password storage mechanisms and encryption for enhanced security.
- **Customization**: Tailor the Next-Auth interface to align with your application's design and user experience.
- **Testing**: Thoroughly test authentication flows, including registration, login, and logout, using a testing framework like Vitest.
- **Regular Updates**: Stay informed about updates and security patches for Next-Auth and its dependencies.

# Conclusion

Next-Auth Credentials provide a robust solution for securing React Next.js 13 applications with MongoDB. By understanding its advantages, considerations, and best practices, developers can efficiently implement authentication and focus on building a secure and feature-rich application.

# References:

- [Next-Auth Documentation](https://next-auth.js.org/)
- [Next.js](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Vitest Documentation](https://vitest.dev/)
- [MongoDB Documentation](https://www.mongodb.com/docs/)
