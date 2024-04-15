+++
title = 'Configuring Resources With YAML Manifests in Kubernetes'
date = 2024-02-04T16:42:31Z
draft = false
series = "Kubernetes"
tags = ["YAML", "Resources"]
+++

# Introduction

Configuring resources in Kubernetes is a fundamental aspect of managing containerized applications. The use of YAML manifests allows developers and operators to define the desired state of their applications, making it easy to manage, deploy, and scale applications in a consistent manner. In this blog post, we will explore the concept of configuring resources with YAML manifests, its implementation, and its relevance in the context of microservices or modular monolithic architectures.

# What is Configuring Resources with YAML Manifests?

Configuring resources with YAML manifests involves defining the desired state of Kubernetes objects, such as pods, deployments, services, and ingresses, in a YAML file. These declarative configurations serve as blueprints for creating and managing resources within a Kubernetes cluster.

## Implementation

YAML (YAML Ain't Markup Language) is a human-readable data serialization format. It allows users to express configurations concisely and in a format that is easy to understand. Let's look into how YAML manifests are used to configure different Kubernetes resources.

# Configuring Basic Kubernetes Components with YAML Manifests

## 1. Nodes

Nodes represent individual machines in the cluster where containers run. A basic YAML manifest for a node may include metadata and specifications, as follows:

```yaml
apiVersion: v1
kind: Node
metadata:
  name: node-1
  labels:
    environment: production
spec:
  capacity:
    cpu: "2"
    memory: 4Gi
  allocatable:
    cpu: "1"
    memory: 2Gi
```

## 2. Pods

Pods are the smallest deployable units in Kubernetes, hosting one or more containers. Here's a simple pod configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

## 3. Deployments

Deployments manage the lifecycle of pods and enable declarative updates. The following YAML defines a basic deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

## 4. Services

Services expose pods to the network within or outside the cluster. Here's a YAML manifest for a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

## 5. Ingresses

Ingresses provide HTTP and HTTPS routing to services. A basic YAML manifest for an ingress might look like this:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
    - host: nginx.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

# Issues and Considerations for Configuring Resources with YAML Manifests

- **Complexity**: As the number of resources and configurations increases, YAML files can become complex and challenging to manage.
- **Versioning**: Changes in the Kubernetes API may require adjustments to YAML files, impacting version compatibility.

# When to Configure Resources with YAML Manifests: Use Cases/Scenarios

- **Infrastructure as Code (IaC)**: Configuring resources with YAML aligns with IaC principles, enabling version control and reproducibility.
- **Continuous Deployment**: YAML manifests facilitate automated deployments, making them suitable for CI/CD pipelines.

# Best Practices

- **Modularity**: Break down configurations into modular components for easier maintenance.
- **Versioning**: Use version control systems for managing changes to YAML files.
- **Documentation**: Clearly document configurations to enhance readability and collaboration.
- **Validation**: Leverage tools like kubectl apply and linting tools to validate YAML syntax and structures.

# Conclusion

Configuring resources with YAML manifests is a cornerstone of Kubernetes application management. By understanding how to express desired states through YAML, developers and operators can efficiently deploy and manage applications at scale. Embracing best practices, considering use cases, and staying mindful of potential issues contribute to a smoother and more robust Kubernetes deployment process.

# References

- [Kubernetes YAML Specification](https://kubernetes.io/docs/concepts/overview/working-with-objects/)
- [YAML Ain't Markup Language (YAML)](https://yaml.org/)
- [Kubernetes Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
