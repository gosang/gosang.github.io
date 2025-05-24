+++
title = 'Implementing Authentication in Next.js Using Better Auth'
date = 2025-05-17T11:24:22+01:00
draft = true
+++

Authentication is a foundational part of any modern web application. In this blog post, we'll explore how to implement robust, scalable authentication in Next.js using a modern and efficient library called Better Auth. We'll discuss what Better Auth is, its rationale, key concepts, and provide a complete guide on integrating it into a Next.js app. We'll also cover the pros and cons, use cases, potential issues, and best practices.

# What is Better Auth?

Better Auth is an open-source authentication library built to simplify and secure authentication for modern JavaScript frameworks like Next.js, React, and Node.js. Inspired by the limitations and complexity of libraries like next-auth, Better Auth emphasizes:

- Lightweight and unopinionated structure
- Simple, secure token management
- First-class support for modern frameworks
- Modular adapters for databases, OAuth, and custom strategies

## Why Better Auth?

Traditional auth solutions can become complex due to their tightly coupled components and default configurations. Better Auth provides:

- Customization without boilerplate
- Secure defaults with extensibility
- Better DX (developer experience)
- Clear separation between frontend and backend logic

## Core Concepts of Better Auth

Better Auth follows a clean, modern approach:
| Concept | Description |
| -------------- | ---------------------------------------------------------------- |
| Session | JWT or cookie-based sessions for stateless authentication |
| Adapter | Interface for connecting to databases (e.g. PostgreSQL, MongoDB) |
| Provider | OAuth providers or custom login flows |
| Middleware | Protects API routes and pages |
| Token Handling | JWT with automatic refresh and rotation |

# Setting Up Authentication in Next.js with Better Auth

Let’s walk through a practical implementation.

#### 🧱 Step 1: Install Dependencies

```bash
npm install better-auth
better-auth-adapter-prisma @prisma/client
```

#### 🧰 Step 2: Configure Prisma

Better Auth works seamlessly with Prisma.

```prisma
// schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  password  String
  createdAt DateTime @default(now())
}
```

Then generate the client:

```bash
npx prisma generate
```

#### ⚙️ Step 3: API Route for Auth

Create an API route to handle login, register, and logout.

```ts
// pages/api/auth/[...auth].ts
import { BetterAuth } from "better-auth";
import { PrismaAdapter } from "better-auth-adapter-prisma";
import prisma from "../../../lib/prisma";
export default BetterAuth({
  adapter: PrismaAdapter(prisma),
  session: {
    strategy: "jwt",
    maxAge: 60 * 60 * 24 * 7, // 7 days
  },
  providers: [
    {
      type: "credentials",
      authorize: async (credentials) => {
        const user = await prisma.user.findUnique({
          where: { email: credentials.email },
        });
        if (!user || user.password !== credentials.password) {
          throw new Error("Invalid login");
        }
        return user;
      },
    },
  ],
});
```

#### 🧑‍💻 Step 4: Frontend Login Form
