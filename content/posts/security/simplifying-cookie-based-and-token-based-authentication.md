+++
title = 'Simplifying Cookie Based and Token Based Authentication'
date = 2024-07-13T12:06:57+01:00
draft = false
tags = ["Security"]
+++

Authentication is a fundamental aspect of web security, ensuring that users are who they claim to be. Two prevalent methods for handling authentication are Cookie-Based and Token-Based authentication. In this blog, we’ll look into these methods, exploring their concepts, functionalities, advantages, and drawbacks. Additionally, we will provide detailed implementation guides for using Keycloak as an Identity Provider for Token-Based authentication in a .NET Core 8 API project and for Cookie-Based authentication in a React Next.js project using TypeScript.

# What is Authentication?

Authentication is the process of verifying the identity of a user or system. It ensures that the person or system accessing resources is indeed who they claim to be. Authentication is typically achieved through credentials such as usernames, passwords, tokens, or certificates.

# Cookie-Based Authentication

## What is Cookie-Based Authentication?

Cookie-Based Authentication uses HTTP cookies to manage user sessions. Upon successful login, the server creates a session and stores it on the server side. A cookie containing the session ID is sent to the client. For subsequent requests, the client sends the cookie to the server to verify the session.

## Key Concepts

**Sessions**: The server maintains the session state.
**Cookies**: Small pieces of data stored on the client side to maintain session state.
**Server-Side Storage**: Sessions are stored on the server, which can scale poorly with numerous users.

## Key Functions

**Session Creation**: After login, the server creates a session and sends a session ID in a cookie.
**Session Verification**: For each request, the server verifies the session ID from the cookie.

## Advantages

**Simplicity**: Easy to implement with built-in browser support.
**Automatic Handling**: Browsers automatically handle cookies.

## Disadvantages

**Scalability**: Server needs to manage and store sessions.
**Cross-Site Request Forgery (CSRF)**: Vulnerable to CSRF attacks.

## When to Use

**Small to Medium-Sized Applications**: Ideal for applications with a manageable number of users.

## When Not to Use

**Highly Scalable Applications**: Not suitable for applications with large user bases due to server-side session management overhead.

# Token-Based Authentication

## What is Token-Based Authentication?

Token-Based Authentication uses tokens, such as JSON Web Tokens (JWT), to manage user sessions. Upon successful login, the server issues a token to the client. The client includes this token in the header of subsequent requests.

## Key Concepts

- **Stateless**: Tokens are self-contained and do not require server-side storage.
- **Tokens**: Encoded data that can be verified and trusted.
- **Decentralized Storage**: Tokens are stored on the client side.

## Key Functions

Token Issuance: After login, the server issues a token to the client.
Token Verification: For each request, the server verifies the token.

## Advantages

- **Scalability**: Tokens do not require server-side session storage.
- **Cross-Domain Compatibility**: Tokens can be used across different domains.

## Disadvantages

- **Complexity**: More complex to implement than cookie-based authentication.
- **Token Revocation**: Difficult to revoke tokens.

## When to Use

**Microservices Architecture**- : Ideal for applications with distributed components.

- **APIs**: Suitable for API authentication due to stateless nature.

## When Not to Use

- **Simpler Applications**: Overkill for small applications with a limited user base.

# Implementing Token-Based Authentication with Keycloak in .NET Core 8 API

## Prerequisites

- .NET Core 8 SDK
- Keycloak server setup

## Step-by-Step Guide

### 1. Install Keycloak NuGet Packages:

```bash
dotnet add package Keycloak.Net
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### 2. Configure Keycloak in appsettings.json:

```json
{
  "Keycloak": {
    "Realm": "your-realm",
    "ClientId": "your-client-id",
    "ClientSecret": "your-client-secret",
    "Authority": "http://localhost:8080/auth/realms/your-realm"
  }
}
```

### 3. Configure Authentication in Program.cs:

```csharp
var builder = WebApplication.CreateBuilder(args);

var keycloakConfig = builder.Configuration.GetSection("Keycloak");
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(options =>
{
    options.Authority = keycloakConfig["Authority"];
    options.Audience = keycloakConfig["ClientId"];
    options.RequireHttpsMetadata = false;
});

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### 4. Protect API Endpoints:

```csharp
[Authorize]
[ApiController]
[Route("api/[controller]")]
public class SecureController : ControllerBase
{
    [HttpGet]
    public IActionResult GetSecureData()
    {
        return Ok("This is a secure endpoint");
    }
}
```

# Implementing Cookie-Based Authentication in React Next.js with TypeScript

## Prerequisites

- Node.js and npm
- Next.js project setup

## Step-by-Step Guide

### 1. Install Required Packages:

```bash
npm install next-cookies axios js-cookie
```

### 2. Setup Authentication Context:

```typescript
// context/AuthContext.tsx
import { createContext, useState, useEffect, ReactNode } from "react";
import Cookies from "js-cookie";
import axios from "axios";

interface AuthContextProps {
  user: string | null;
  login: (username: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextProps | undefined>(undefined);

export const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<string | null>(null);

  useEffect(() => {
    const savedUser = Cookies.get("user");
    if (savedUser) {
      setUser(savedUser);
    }
  }, []);

  const login = async (username: string, password: string) => {
    const response = await axios.post("/api/login", { username, password });
    Cookies.set("user", response.data.user);
    setUser(response.data.user);
  };

  const logout = () => {
    Cookies.remove("user");
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### 3. Create Login API Route:

```typescript
// pages/api/login.ts
import { NextApiRequest, NextApiResponse } from "next";

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const { username, password } = req.body;

  // Mock authentication
  if (username === "user" && password === "pass") {
    res.status(200).json({ user: "user" });
  } else {
    res.status(401).json({ message: "Invalid credentials" });
  }
}
```

### 4. Protect Client-Side Routes:

```typescript
// pages/protected.tsx
import { useContext, useEffect } from "react";
import { useRouter } from "next/router";
import { AuthContext } from "../context/AuthContext";

const ProtectedPage = () => {
  const authContext = useContext(AuthContext);
  const router = useRouter();

  useEffect(() => {
    if (!authContext?.user) {
      router.push("/login");
    }
  }, [authContext?.user]);

  if (!authContext?.user) {
    return <div>Loading...</div>;
  }

  return <div>This is a protected page</div>;
};

export default ProtectedPage;
```

# Issues, Considerations, and Best Practices

## Cookie-Based Authentication

### Issues

- **CSRF**: Cookies are automatically sent with requests, making CSRF attacks possible.
- **Storage Limitation**: Limited data storage capacity in cookies.

### Considerations

- **Use HttpOnly and Secure Flags**: Mitigates risks of XSS attacks.
- **CSRF Tokens**: Implement CSRF tokens to protect against CSRF attacks.

### Best Practices

- **Short Lifespan**: Keep session duration short.
- **Regular Rotation**: Rotate session IDs regularly.

## Token-Based Authentication

### Issues

- **Token Revocation**: Difficult to revoke tokens immediately.
- **Security**: Tokens can be intercepted if not properly secured.

### Considerations

- **Token Expiry**: Implement short-lived tokens with refresh tokens.
- **Secure Transmission**: Use HTTPS to secure token transmission.

### Best Practices

- **Audience and Issuer Validation**: Ensure tokens are valid for your audience and issuer.
- **Refresh Tokens**: Use refresh tokens to maintain long-term sessions securely.

# Conclusion

Understanding the differences between Cookie-Based and Token-Based authentication is crucial for building secure web applications. Each method has its unique advantages and disadvantages, making them suitable for different scenarios. By following the best practices and considerations outlined in this blog, you can implement robust authentication mechanisms tailored to your application's needs.

# References

- [OAuth 2.0 Framework](https://oauth.net/2/)
- [JWT RFC](https://oauth.net/2/jwt/)
