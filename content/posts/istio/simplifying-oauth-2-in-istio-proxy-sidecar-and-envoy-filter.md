+++
title = 'Simplifying Oauth 2 in Istio Proxy Sidecar and Envoy Filter'
date = 2024-07-07T12:02:22+01:00
draft = false
tags = ["Istio", "OAuth 2.0"]
+++

In the context of microservices and cloud-native applications, securing APIs and ensuring proper authentication are critical. OAuth 2.0 is a widely adopted authorization framework designed to provide secure access to resources. When working with Istio, a popular service mesh for managing microservices, integrating OAuth 2.0 can be achieved through various mechanisms such as OAuth 2.0 Proxy, OAuth 2.0 Sidecar Proxy, and OAuth 2.0 Envoy Filter. This blog aims to explain these concepts, their rationale, main functions, sample implementation with Keycloak as an Identity Provider, and provide guidance on their use cases.

# What is Istio?

Istio is an open-source service mesh that provides a uniform way to secure, connect, and observe microservices. It works by deploying a network of proxies (Envoy) alongside microservices, handling the inter-service communication. Istio offers capabilities such as traffic management, security, and observability without requiring changes to the application code.

# What is OAuth 2.0?

OAuth 2.0 is an authorization framework that allows applications to obtain limited access to user accounts on an HTTP service. It works by delegating user authentication to the service that hosts the user account and authorizing third-party applications to access the user account. The primary components include:

- **Resource Owner**: The user who authorizes an application to access their account.
- **Client**: The application requesting access to the user’s account.
- **Authorization Server**: The server issuing access tokens to the client after successfully authenticating the resource owner.
- **Resource Server**: The server hosting the protected resources, accepting and responding to protected resource requests using access tokens.

# OAuth 2.0 Proxy, Sidecar Proxy, and Envoy Filter in Istio

## OAuth 2.0 Proxy

An OAuth 2.0 Proxy acts as an intermediary between the client and the backend services. It handles the OAuth 2.0 flow, authenticating users and injecting tokens into requests.

### Rationale and Key Functions:

- Centralized authentication handling.
- Simplifies integrating OAuth 2.0 for applications.
- Manages token refresh and validation.

## OAuth 2.0 Sidecar Proxy

The Sidecar Proxy is a pattern where an OAuth 2.0 proxy is deployed alongside each application instance. This sidecar container intercepts traffic to and from the application, managing OAuth 2.0 authentication.

### Rationale and Key Functions:

- Provides isolation between application code and authentication logic.
- Allows for per-service authentication policies.
- Simplifies scaling and maintenance.

## OAuth 2.0 Envoy Filter

An OAuth 2.0 Envoy Filter is a custom filter applied in the Envoy proxies managed by Istio. This filter enforces OAuth 2.0 authentication on incoming requests at the ingress or within the service mesh.

### Rationale and Key Functions:

- Deeply integrated with Istio’s traffic management.
- Centralized policy enforcement at the ingress gateway.
- Reduces operational overhead by leveraging Istio’s control plane.

# Implementing OAuth 2.0 in Istio with Keycloak

## Prerequisites

- Istio installed and configured.
- Keycloak server set up as the Identity Provider.
- Two sample .NET Core 8 APIs.
- OAuth 2.0 Proxy Implementation

## Deploy OAuth 2.0 Proxy

Create a `deployment-oauth2-proxy.yaml` file with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oauth2-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oauth2-proxy
  template:
    metadata:
      labels:
        app: oauth2-proxy
    spec:
      containers:
        - name: oauth2-proxy
          image: quay.io/oauth2-proxy/oauth2-proxy:v7.1.3
          args:
            - --provider=keycloak
            - --keycloak-issuer-url=https://keycloak.example.com/auth/realms/myrealm
            - --client-id=myclient
            - --client-secret=mysecret
            - --upstream=http://myapi:8080
            - --cookie-secret=asecrethash
          ports:
            - containerPort: 4180
```

This YAML file defines a Kubernetes deployment for the OAuth2 Proxy. It configures the proxy to use Keycloak as the OAuth2 provider and specifies the upstream service (your backend API) and the necessary secrets.

### Apply the deployment:

```sh
kubectl apply -f deployment-oauth2-proxy.yaml
```

## Configure Istio Ingress Gateway

Create an `ingress-gateway.yaml` file with the following content:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapi
spec:
  gateways:
    - mygateway
  hosts:
    - "*"
  http:
    - route:
        - destination:
            host: oauth2-proxy
            port:
              number: 4180
```

This configuration creates an Istio Gateway and a VirtualService. The Gateway defines the entry point for external traffic, while the VirtualService routes incoming traffic to the OAuth2 Proxy.

### Apply the configuration:

```sh
kubectl apply -f ingress-gateway.yaml
```

- **Gateway**: Defines the external access point to the mesh. Here, it's configured to accept HTTP traffic on port 80 for all hosts (\*).
- **VirtualService**: Routes incoming requests to the OAuth2 Proxy service, which handles authentication before forwarding requests to the backend API.

## OAuth 2.0 Sidecar Proxy Implementation

### Create a `deployment-sidecar-proxy.yaml` file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapi
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapi
  template:
    metadata:
      labels:
        app: myapi
    spec:
      containers:
        - name: myapi
          image: myapi:latest
          ports:
            - containerPort: 8080
        - name: oauth2-proxy
          image: quay.io/oauth2-proxy/oauth2-proxy:v7.1.3
          args:
            - --provider=keycloak
            - --keycloak-issuer-url=https://keycloak.example.com/auth/realms/myrealm
            - --client-id=myclient
            - --client-secret=mysecret
            - --upstream=http://localhost:8080
            - --cookie-secret=asecrethash
          ports:
            - containerPort: 4180
```

This YAML defines a deployment for your application (myapi) with an OAuth2 Proxy sidecar container. The sidecar proxy is configured to handle authentication for requests to the application.

### Apply the deployment:

```sh
kubectl apply -f deployment-sidecar-proxy.yaml
```

## Service and VirtualService Configuration

Create a `service-virtualservice.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapi
spec:
  selector:
    app: myapi
  ports:
    - protocol: TCP
      port: 80
      targetPort: 4180
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapi
spec:
  hosts:
    - "*"
  gateways:
    - mygateway
  http:
    - match:
        - uri:
            prefix: /api
      route:
        - destination:
            host: myapi
            port:
              number: 80
```

This configuration defines a Kubernetes Service for myapi and a VirtualService that routes traffic through the OAuth2 Proxy sidecar.

### Apply the configuration:

```sh
kubectl apply -f service-virtualservice.yaml
```

- **Service**: Exposes the application, forwarding traffic to the OAuth2 Proxy sidecar on port 4180.
- **VirtualService**: Routes requests matching the /api prefix to the myapi service, which handles authentication via the sidecar proxy.

## OAuth 2.0 Envoy Filter Implementation

Create an `envoy-filter.yaml` file

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: oauth2-filter
spec:
  workloadSelector:
    labels:
      app: myapi
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_INBOUND
        listener:
          filterChain:
            filter:
              name: envoy.filters.network.http_connection_manager
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.http.jwt_authn
          config:
            providers:
              keycloak:
                issuer: https://keycloak.example.com/auth/realms/myrealm
                remote_jwks:
                  http_uri:
                    uri: https://keycloak.example.com/auth/realms/myrealm/protocol/openid-connect/certs
                    cluster: jwt_cluster
                  cache_duration:
                    seconds: 600
            rules:
              - match:
                  prefix: "/"
                requires:
                  provider_name: "keycloak"
```

This YAML defines an EnvoyFilter that inserts a JWT authentication filter into the Envoy proxy's filter chain. The filter enforces authentication using tokens issued by Keycloak.

### Apply the configuration:

```sh
kubectl apply -f envoy-filter.yaml
```

## ServiceEntry for Keycloak

Create a `service-entry.yaml` file:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: keycloak
spec:
  hosts:
    - "keycloak.example.com"
  ports:
    - number: 443
      name: https
      protocol: HTTPS
  resolution: DNS
```

This YAML defines a ServiceEntry that allows Istio to route traffic to the Keycloak server.

### Apply the configuration:

```sh
kubectl apply -f service-entry.yaml
```

- **EnvoyFilter**: Adds a custom filter to the Envoy proxy to enforce OAuth2 authentication using JWTs from Keycloak.
- **ServiceEntry**: Configures Istio to recognize the Keycloak server and route traffic to it.

# Advantages and Disadvantages

| Approach                | Advantages                                                      | Disadvantages                      |
| ----------------------- | --------------------------------------------------------------- | ---------------------------------- |
| OAuth 2.0 Proxy         | Centralizes authentication handling.                            | Single point of failure.           |
|                         | Simplifies application code by offloading authentication tasks. | May introduce latency.             |
| OAuth 2.0 Sidecar Proxy | Isolates authentication logic from application code.            | Increased resource consumption.    |
|                         | Flexible and scalable.                                          | Complex deployment and management. |
| OAuth 2.0 Envoy Filter  | Deep integration with Istio.                                    | Steeper learning curve.            |
|                         | Centralized policy enforcement.                                 | Debugging can be challenging.      |
|                         | Minimal changes to application code.                            |                                    |

# When to Use Each Approach

| Approach                | When to use                                                                                                                 | When not to use                                                                 |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| OAuth 2.0 Proxy         | Use when you want a simple and centralized way to manage authentication without changing application code.                  | Avoid if you have high availability requirements or need low-latency solutions. |
| OAuth 2.0 Sidecar Proxy | Use for microservices requiring isolated authentication logic and where scalability is a concern.                           | Avoid if resource overhead is a significant issue.                              |
| OAuth 2.0 Envoy Filter  | Use for complex environments where deep integration with Istio is beneficial, and centralized policy enforcement is needed. | Avoid if team lacks expertise in Istio or when rapid debugging is required.     |

# Issues, Considerations, and Best Practices

- **Security**
  - Ensure secure storage of client secrets.
  - Use HTTPS for all communications.
  - Regularly update and patch OAuth 2.0 proxies and filters.
- **Token Management**
  - Ensure proper handling of token expiration and refresh.
- **Performance**
  - Monitor the performance impact of additional proxies.
  - Optimize configurations to reduce latency.
- **Scalability**
  - Plan for horizontal scaling, especially for sidecar proxies.
  - Use appropriate resource limits for sidecar containers.
- **Maintenance**
  - Automate deployment and updates using CI/CD pipelines.
  - Maintain version control for configuration files.
- **Logging and Monitoring**
  - Implement robust logging and monitoring for all OAuth 2.0 components.
  - Implement comprehensive monitoring for proxies and filters.
  - Use centralized logging solutions for easier troubleshooting.

# Conclusion

Implementing OAuth 2.0 in Istio provides robust authentication mechanisms for securing APIs. The choice between OAuth 2.0 Proxy, OAuth 2.0 Sidecar Proxy, and OAuth 2.0 Envoy Filter depends on your specific requirements, such as centralization, isolation, or deep integration with Istio. Each approach has its own advantages and disadvantages:

- **OAuth 2.0 Proxy**: Best for simple, centralized authentication but introduces additional proxy layers.
- **OAuth 2.0 Sidecar Proxy**: Offers isolation and per-service policies but increases resource consumption.
- **OAuth 2.0 Envoy Filter**: Provides centralized policy enforcement with reduced overhead but requires more complex configuration.

By understanding these approaches and their trade-offs, you can effectively secure your APIs in Istio with OAuth 2.0, providing a robust and scalable authentication solution.

# References

- [Istio Documentation](https://istio.io/latest/docs/)
- [OAuth2 Proxy GitHub](https://github.com/oauth2-proxy/oauth2-proxy)
- [Keycloak Documentation](https://www.keycloak.org/documentation)
