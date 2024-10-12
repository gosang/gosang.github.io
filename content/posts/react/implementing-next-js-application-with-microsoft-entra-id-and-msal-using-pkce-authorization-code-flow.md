+++
title = 'Implementing Next.js 14 Application with Microsoft Entra ID and MSAL Using PKCE Authorization Code Flow'
date = 2024-10-12T11:17:42+01:00
draft = true
tags = ["React", "OAuth"]
+++

In this blog post, we will walk through how to implement a Next.js 14 application that integrates with Microsoft Entra ID (formerly Azure Active Directory) and Microsoft Authentication Library (MSAL) using the Authorization Code Flow with PKCE (Proof Key for Code Exchange). This implementation will use TypeScript, React Hook Form, and two Azure AD groups to differentiate user roles: one for Admin permissions and another for Read-Only permissions.

We’ll also cover key issues, considerations, and best practices to follow when implementing Microsoft Entra ID and MSAL in a Next.js application.

# 1. Introduction to MSAL and Microsoft Entra ID

Microsoft Entra ID is Microsoft's cloud-based identity and access management service. It offers secure sign-in options, role-based access control (RBAC), and integrates seamlessly with Microsoft applications and APIs.

The Microsoft Authentication Library (MSAL) allows you to authenticate users and acquire tokens for API access. MSAL supports multiple authentication flows, but in this example, we'll focus on the Authorization Code Flow with PKCE, which enhances security, especially in single-page and server-side applications like Next.js.

# 2. Why Use Authorization Code Flow with PKCE?

Authorization Code Flow with PKCE (Proof Key for Code Exchange) is recommended for public clients like single-page applications (SPAs) that can't keep a client secret securely. The flow provides:

- Secure authentication by using an exchangeable code instead of passing tokens directly.
- Prevention of code interception attacks, as PKCE adds a layer of protection by requiring a code verifier to redeem the authorization code.

# 3. Setting Up the Azure AD App Registration

## 1. Register an Application in Azure AD:

- Go to the Azure portal > Azure Active Directory > App registrations > New registration.
- Set the redirect URI to `http://localhost:3000/api/auth/callback/microsoft`.
- Configure supported account types, typically "Accounts in this organizational directory only" for internal applications.

## 2. Configure API Permissions:

- Add Microsoft Graph API permissions for user profile and role data. For example, `User.Read` and `Directory.Read.All`.

## 3. Expose an API (Optional):

- If your application needs to be consumed by other clients, expose an API and define the roles (Admin, Read-Only).

## 4. Create AD Groups for Authorization:

- In Azure AD > Groups, create two groups:
  - Admin Group (e.g., AdminGroup).
  - Read-Only Group (e.g., ReadOnlyGroup).

# 4. Creating the Next.js 14 Application

Create a new Next.js project with TypeScript:

```bash
npx create-next-app@14 my-msal-app --typescript
cd my-msal-app
```

Install the necessary packages:

```bash
npm install @azure/msal-browser @azure/msal-react react-hook-form
```

# 5. Implementing MSAL in Next.js with TypeScript

We'll use @azure/msal-react to configure and authenticate users with Microsoft Entra ID.

## 1. MSAL Configuration

Create an MSAL configuration file (`msalConfig.ts`):

```typescript
import { PublicClientApplication, Configuration } from "@azure/msal-browser";

const msalConfig: Configuration = {
  auth: {
    clientId: process.env.NEXT_PUBLIC_CLIENT_ID!,
    authority: `https://login.microsoftonline.com/${process.env.NEXT_PUBLIC_TENANT_ID}`,
    redirectUri: "http://localhost:3000/api/auth/callback/microsoft",
  },
  cache: {
    cacheLocation: "sessionStorage",
    storeAuthStateInCookie: false,
  },
};

export const msalInstance = new PublicClientApplication(msalConfig);
```

## 2. Wrapping Application with MSAL Provider

In `_app.tsx`, wrap your app with the `MsalProvider`:

```typescript
import { MsalProvider } from "@azure/msal-react";
import { msalInstance } from "../msalConfig";
import type { AppProps } from "next/app";

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <MsalProvider instance={msalInstance}>
      <Component {...pageProps} />
    </MsalProvider>
  );
}

export default MyApp;
```

## 3. Creating a Login Component with React Hook Form

Use React Hook Form to handle the login input:

```typescript
import { useMsal } from "@azure/msal-react";
import { InteractionType } from "@azure/msal-browser";
import { useForm } from "react-hook-form";

const LoginForm = () => {
  const { instance } = useMsal();
  const { handleSubmit } = useForm();

  const onLogin = () => {
    instance.loginRedirect({
      scopes: ["User.Read"],
    });
  };

  return (
    <form onSubmit={handleSubmit(onLogin)}>
      <button type="submit">Login with Microsoft</button>
    </form>
  );
};

export default LoginForm;
```

# 6. Handling AD Groups and Role-based Authorization

Once authenticated, use Microsoft Graph API to retrieve group memberships and enforce role-based access control (RBAC). You can differentiate user roles by the group they belong to in Azure AD.

## 1. Fetching Group Data Using MS Graph

Add the following in your API route or React component:

```typescript
import { graphClient } from "../lib/graphClient";
import { useMsal } from "@azure/msal-react";

const checkUserGroup = async () => {
  const { accounts } = useMsal();
  const accessToken = await instance.acquireTokenSilent({
    account: accounts[0],
    scopes: ["Directory.Read.All"],
  });

  const userGroups = await graphClient
    .api(`/me/memberOf`)
    .header("Authorization", `Bearer ${accessToken}`)
    .get();

  const isAdmin = userGroups.value.some(
    (group) => group.displayName === "AdminGroup"
  );
  const isReadOnly = userGroups.value.some(
    (group) => group.displayName === "ReadOnlyGroup"
  );

  return { isAdmin, isReadOnly };
};
```

This function fetches the user’s group membership from Azure AD and checks if they belong to the Admin or Read-Only group. Use this to control access within your application.

# 7. Advantages and Disadvantages

| Advantage                                                                                                                                                | Disadvantage                                                                                                                               |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Enhanced Security**: Using the Authorization Code Flow with PKCE ensures better protection against token interception attacks.                         | **Complex Configuration**: Setting up an Azure AD app registration and configuring permissions can be challenging.                         |
| **Seamless Integration**: MSAL simplifies authentication with Azure AD, making it easy to work with Microsoft Graph API and manage users.                | **Extra Network Calls**: Additional network calls to the Graph API can increase latency, especially when verifying user roles.             |
| **Role-Based Access Control (RBAC)**: By utilizing AD groups, you can easily implement role-based security to control who can access specific resources. | **Token Caching Issues**: Improperly managing token caches (e.g., storing tokens insecurely) can expose the application to security risks. |

# 8. Key Issues, Considerations, and Best Practices

- Token Security

  - Store tokens securely (e.g., in session storage or a secure cookie).
    Never expose sensi

- Authorization Strategy

  - Use Microsoft Graph to fetch user roles and enforce RBAC based on group membership.
  - Always verify user roles server-side for critical operations.

- Error Handling

  - Handle token expiration and authentication errors gracefully by using MSAL’s error management hooks (useMsalAuthentication).

- PKCE Best Practices
  - Always use PKCE to protect your Authorization Code flow in public clients like SPAs.
  - Rotate and monitor client secrets if used in backend services.

# 9. Conclusion

Integrating a Next.js 14 application with Microsoft Entra ID and MSAL using the Authorization Code Flow with PKCE ensures a secure, scalable, and user-friendly authentication process. By leveraging Azure AD groups, role-based access can be implemented effectively. While there are challenges, following best practices around security and role management can help overcome them.

# References

- [MSAL.js](https://learn.microsoft.com/en-us/entra/identity-platform/msal-overview)
- [Azure Active Directory](https://learn.microsoft.com/en-us/entra/identity/)
