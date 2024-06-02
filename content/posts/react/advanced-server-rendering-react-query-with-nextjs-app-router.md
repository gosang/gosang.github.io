+++
title = 'Advanced Server Rendering: React Query With Next.js App Router'
date = 2024-06-02T11:50:29+01:00
draft = false
tags = ["React", "SSR", "React Query"]
+++

Server-side rendering (SSR) and static site generation (SSG) are essential techniques in modern web development to improve performance, SEO, and user experience. In this blog, we will look into how to effectively use React Query with the Next.js App Router for advanced server rendering. We will cover the concepts, setup, and practical implementations with React and TypeScript examples. Let's get started!

# Understanding Server Rendering

## Server-Side Rendering (SSR)

SSR generates HTML on the server for each request. This approach benefits applications by improving SEO, providing faster initial load times, and enhancing user experience.

### Benefits of SSR:

- **Improved SEO**: Search engines can better index pre-rendered content.
- **Faster Initial Load**: Users see a fully rendered page quicker, enhancing perceived performance.
- **Enhanced User Experience**: Less time is spent loading and rendering content on the client side.

## Static Site Generation (SSG)

SSG pre-renders pages at build time, generating static HTML files. These files are then served directly to users, providing fast load times and efficient caching.

### Benefits of SSG:

- **Performance**: Pre-rendered pages load quickly.
- **Scalability**: Static files can be served from a CDN.
- **Cost-Effective**: Reduced server load and maintenance costs.

# What is React Query?

React Query is a library for managing server-state in React applications. It simplifies data fetching, caching, synchronization, and more, making it easier to handle asynchronous operations.

## Key Features of React Query:

- **Data Caching**: Automatically caches fetched data to improve performance.
- **Synchronization**: Keeps data synchronized between server and client.
- **Query Invalidation**: Invalidates queries to refetch data when necessary.
- **Background Fetching**: Fetches data in the background, providing a seamless user experience.

# What is the Next.js App Router?

The Next.js App Router is an advanced routing mechanism providing better control over routing in Next.js applications. It enables nested routes, layouts, and flexible data fetching strategies.

# Combining SSR, React Query, and Next.js App Router

## Rationale

Combining SSR with React Query in Next.js provides a robust solution for handling server-state and routing. This approach addresses common challenges such as:

- **SEO Optimization**: Pre-rendered HTML content improves SEO.
- **Performance**: Faster page loads due to server-rendered content.
- **Data Management**: Simplified data fetching and synchronization using React Query.

## How They Work Together

- **Server Rendering**: Next.js pre-renders the page with initial data fetched using React Query.
- **Hydration**: The pre-rendered HTML is sent to the client, where React Query rehydrates with the fetched data.
- **Routing**: The Next.js App Router handles navigation and routing, allowing seamless transitions between pages.

# Setting Up Server Rendering with React Query and Next.js App Router

## Initial Setup

### Create a React Query Client

To manage server-state effectively with React Query, we need to create a QueryClient instance, which will handle our queries, mutations, and caching.

1. Install Dependencies:

```bash
npm install next react react-dom react-query
```

2. Create a QueryClient Instance:

```typescript
// lib/reactQueryClient.ts
import { QueryClient } from "react-query";

export const queryClient = new QueryClient();
```

In this file, we import `QueryClient` from `react-query` and export a new instance of `QueryClient`. This instance will be used to manage our queries and caching logic.

### Configure Next.js App

Next, we need to configure our Next.js app to use the `QueryClientProvider` to make the query client available throughout our application.

1. Configure Next.js App with QueryClientProvider:

```typescript
// pages/_app.tsx
import { AppProps } from "next/app";
import { QueryClientProvider } from "react-query";
import { queryClient } from "../lib/reactQueryClient";

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
    </QueryClientProvider>
  );
}

export default MyApp;
```

Here, we wrap our application with the `QueryClientProvider` and pass the `queryClient` instance to it. This setup ensures that React Query is properly configured and available across our entire app.

## Server-Side Data Fetching

Server-side data fetching involves pre-fetching data on the server before rendering the page. This data is then passed to the client, allowing for faster initial load times and better SEO.

### Fetch Data on the Server

1. Fetch Data on the Server:

```typescript
// pages/index.tsx
import { GetServerSideProps } from "next";
import { useQuery } from "react-query";
import { dehydrate, QueryClient } from "react-query/hydration";

const fetchUsers = async () => {
  const res = await fetch("https://jsonplaceholder.typicode.com/users");
  return res.json();
};

export const getServerSideProps: GetServerSideProps = async () => {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery("users", fetchUsers);

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
};

const HomePage = () => {
  const { data, error } = useQuery("users", fetchUsers);

  if (error) return <div>Error loading users</div>;

  return (
    <div>
      {data?.map((user: any) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
};

export default HomePage;
```

- **GetServerSideProps**: This function runs on the server side and allows us to fetch data before rendering the page.
- **QueryClient**: A new instance of `QueryClient` is created for each request.
- **PrefetchQuery**: We use `prefetchQuery` to fetch and cache the data on the server.
- **Dehydrate**: The `dehydrate` function serializes the query cache to be passed as props to the client.

## Hydration

Hydration is the process of synchronizing the server-rendered HTML with the client-side JavaScript. React Query uses hydration APIs to rehydrate the state on the client-side using the pre-fetched data from the server.

### Hydrate the State on the Client-Side

```typescript
// pages/_app.tsx
import { Hydrate } from "react-query/hydration";
import { AppProps } from "next/app";
import { QueryClientProvider } from "react-query";
import { queryClient } from "../lib/reactQueryClient";

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <Hydrate state={pageProps.dehydratedState}>
        <Component {...pageProps} />
      </Hydrate>
    </QueryClientProvider>
  );
}

export default MyApp;
```

- **Hydrate Component**: The Hydrate component takes the dehydratedState from pageProps and rehydrates the query cache on the client side. This ensures that the client has the same initial state as the server-rendered HTML.

### Improve Hydration with Hydration Boundaries

Using hydration boundaries, we can improve the performance and user experience by isolating parts of the UI that need to be hydrated separately. This approach ensures that critical parts of the UI are interactive sooner.

1. Create a Hydration Boundary Component:

```typescript
// components/HydrationBoundary.tsx
import { Hydrate } from "react-query/hydration";
import { ReactNode } from "react";

interface HydrationBoundaryProps {
  state: any;
  children: ReactNode;
}

const HydrationBoundary = ({ state, children }: HydrationBoundaryProps) => {
  return <Hydrate state={state}>{children}</Hydrate>;
};

export default HydrationBoundary;
```

2. Use Hydration Boundaries in Your App:

```typescript
// pages/_app.tsx
import { AppProps } from "next/app";
import { QueryClientProvider } from "react-query";
import { queryClient } from "../lib/reactQueryClient";
import HydrationBoundary from "../components/HydrationBoundary";

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <HydrationBoundary state={pageProps.dehydratedState}>
        <Component {...pageProps} />
      </HydrationBoundary>
    </QueryClientProvider>
  );
}

export default MyApp;
```

By isolating hydration with `HydrationBoundary`, we ensure that different parts of the application can be hydrated independently, improving overall performance and user experience.

# Advantages and Disadvantages

## Advantages

- **SEO Friendly**: Better indexing by search engines.
- **Faster Initial Load**: Improved user experience due to server-rendered content.
- **Simplified Data Management**: React Query handles data fetching and caching efficiently.

## Disadvantages

-** Increased Server Load**: SSR requires more resources on the server.

- **Complexity**: Combining SSR, React Query, and Next.js App Router can increase the complexity of the application.
- **Latency**: Server response time might be higher due to the initial data fetching process.

# When to Use and Not to Use SSR with React Query and Next.js App Router

## When to Use

SEO-Critical Pages: Content that must be indexed by search engines.
Fast Initial Load Required: Applications where the first contentful paint is crucial for user experience.
Dynamic Data: Pages that require real-time data fetching and rendering.

## When Not to Use

- **Highly Interactive UIs**: Applications with extensive client-side interactivity might not benefit from SSR.
- **Static Content**: Pages with static content that doesn’t change often can be pre-rendered using Static Site Generation (SSG).
- **Resource-Constrained Servers**: SSR increases server load, which might not be suitable for applications with limited server resources.

# Issues, Considerations, and Best Practices

## Issues

- **State Management**: Ensuring consistent state between server and client can be challenging.
- **Error Handling**: Properly handling errors during server-side data fetching.

## Considerations

- **Caching**: Implement effective caching strategies to reduce server load.
- **Performance Monitoring**: Monitor server performance to ensure SSR does not degrade user experience.

## Best Practices

- **Code Splitting**: Use code splitting to improve performance by loading only necessary code.
- **Prefetching**: Prefetch data on the server to reduce client-side load times.
- **Hydration Optimization**: Optimize hydration to ensure smooth transitions from server-rendered to client-rendered content.

# Conclusion

Server Rendering with React Query and Next.js App Router provides a powerful solution for creating high-performance, SEO-friendly web applications. By leveraging the strengths of each technology, developers can build applications that are both performant and user-friendly. However, it is crucial to understand the trade-offs and implement best practices to make the most out of this approach.

For further reading, check out the official documentation:

- [Next.js Documentation](https://nextjs.org/docs)
- [React Query Documentation](https://tanstack.com/query/latest/docs/framework/react/overview)
