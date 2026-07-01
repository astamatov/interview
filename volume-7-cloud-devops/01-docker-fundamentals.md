# Docker Fundamentals

## Overview

Docker has become the standard way to package and deploy modern applications.

Before Docker, applications often worked perfectly on one machine but failed on another because of different operating systems, runtime versions or installed dependencies.

Docker solves this by packaging the application together with everything it needs to run.

The result is a consistent environment across development, testing and production.

---

# 1. What is Docker?

## Interview Answer

Docker is a containerization platform that packages an application together with its runtime, libraries and dependencies into a container.

The container behaves the same regardless of where it runs.

Example:

```
Developer Laptop

↓

Docker Image

↓

Docker Container

↓

QA

↓

Production

↓

Cloud
```

The same image runs everywhere.

---

# 2. What is a Container?

## Interview Answer

A container is a lightweight, isolated process that shares the host operating system kernel.

Unlike a virtual machine, a container does not include a full operating system.

```
Host OS

↓

Docker Engine

↓

Container A

Container B

Container C
```

Each container has its own filesystem, network and processes.

---

# 3. Container vs Virtual Machine

## Virtual Machine

```
Hardware

↓

Hypervisor

↓

Guest OS

↓

Application
```

Every VM includes a complete operating system.

Advantages:

- Strong isolation

Disadvantages:

- High memory usage
- Slow startup

---

## Container

```
Hardware

↓

Host OS

↓

Docker Engine

↓

Container
```

Containers share the host kernel.

Advantages:

- Starts in seconds (or less)
- Lower memory usage
- Higher density

---

# Interview Question

### Why are containers faster than virtual machines?

## Answer

Containers share the host operating system kernel.

Virtual machines boot an entire guest operating system.

This makes containers much lighter and faster to start.

---

# 4. Docker Image

## Interview Answer

A Docker Image is an immutable template used to create containers.

Think of it as a blueprint.

```
Docker Image

↓

Container 1

Container 2

Container 3
```

Multiple containers can be created from the same image.

---

# 5. Docker Container

A container is a running instance of an image.

```
Image

↓

Run

↓

Container
```

If the container stops, the image remains unchanged.

---

# 6. Dockerfile

## Interview Answer

A Dockerfile contains instructions for building an image.

Example:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

COPY . .

ENTRYPOINT ["dotnet", "MyApi.dll"]
```

Every instruction creates a new image layer.

---

# 7. Layers

Docker images are built in layers.

```
Base Image

↓

.NET Runtime

↓

Application Files

↓

Configuration
```

If only the application changes, Docker reuses the previous layers.

This makes builds significantly faster.

---

# 8. Multi-stage Builds

## Interview Answer

Multi-stage builds reduce image size.

Example:

```
SDK Image

↓

Build Application

↓

Copy Output

↓

Runtime Image
```

Only the compiled application is included in the final image.

The SDK is left behind.

---

## Example

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build

WORKDIR /src

COPY . .

RUN dotnet publish -c Release -o /publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

COPY --from=build /publish .

ENTRYPOINT ["dotnet", "MyApi.dll"]
```

This is the recommended approach for .NET applications.

---

# 9. Volumes

Containers are ephemeral.

If a container is deleted, its internal filesystem is also deleted.

Volumes provide persistent storage.

```
Container

↓

Volume

↓

Disk
```

Use volumes for:

- Database files
- Uploaded documents
- Logs (when appropriate)

---

# 10. Networking

Docker creates virtual networks.

Containers communicate using container names.

Example:

```
API

↓

SQL Server

↓

Redis
```

Instead of IP addresses, services often communicate using DNS names defined by Docker networking.

---

# 11. Environment Variables

Configuration should not be hardcoded.

Example:

```bash
ConnectionStrings__Default=...
```

ASP.NET Core automatically maps environment variables into configuration.

This allows different settings for development, staging and production.

---

# 12. Common Interview Questions

### Should SQL Server run inside Docker?

## Answer

It depends.

For local development, Docker is an excellent choice.

For production, many teams prefer managed services such as Azure SQL because Microsoft handles backups, patching and high availability.

---

### Can multiple containers share the same image?

Yes.

An image is read-only.

Each container has its own writable layer.

---

### Why are containers considered immutable?

The image should never change after it is built.

A new version of the application should result in a new image, not manual modifications to a running container.

---

### Should secrets be stored in Docker images?

No.

Use:

- Environment variables
- Azure Key Vault
- Kubernetes Secrets
- Docker Secrets

Never bake secrets into the image.

---

# Trade-offs

| Docker | Virtual Machine |
|---------|-----------------|
| Lightweight | Heavy |
| Fast startup | Slow startup |
| Shares host kernel | Separate guest OS |
| High density | Lower density |
| Excellent for microservices | Better isolation |

---

# Staff Engineer Insight

One of the most common mistakes is treating containers like virtual machines.

Example:

```
docker exec

↓

Install software

↓

Modify files
```

Those changes disappear when the container is replaced.

Containers should be disposable.

If you need a change:

1. Update the source code or Dockerfile.
2. Build a new image.
3. Deploy a new container.

Another common mistake is creating oversized images by using the full .NET SDK image in production.

Multi-stage builds keep images smaller, improve startup time and reduce the attack surface.

---

# Summary

- Docker packages applications with their dependencies.
- Containers share the host operating system kernel.
- Images are immutable templates.
- Containers are running instances of images.
- Dockerfiles define how images are built.
- Multi-stage builds produce smaller production images.
- Volumes provide persistent storage.
- Environment variables should be used for configuration.
- Treat containers as disposable, immutable units.
