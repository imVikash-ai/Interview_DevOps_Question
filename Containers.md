## What is a container?

- A container is essentially a **process with extra isolation and resource management**.
- It gets its own virtualized view of an operating system while still using the host's resources.
- Containers rely on Linux kernel features:
  - **Namespaces** provide isolation.
  - **cgroups** limit and monitor resources such as CPU, memory, and network bandwidth.
- These internals are complex, and most developers don't want to manage them by hand. That is where **container runtimes** like containerd help.

---

## What is containerd?

- containerd is an **open source, high-level container runtime**: a tool built specifically to run containers.
- It sits on top of kernel features and adds an **abstraction layer** that handles namespaces, cgroups, union file systems, networking, and more, so developers don't have to deal with that complexity directly.
- It manages the core container lifecycle: **creating, starting, and stopping** containers.
- By default it uses **OCI-compliant runtimes** (Open Container Initiative) such as **runc**, a lower-level runtime, which keeps things standardized and interoperable.
- It works for both small deployments and large enterprise environments, including **Kubernetes**.


## How containerd relates to Docker

- containerd talks directly to the operating system to carry out container operations.
- The **Docker Engine sits on top of containerd** and adds extra functionality and a better developer experience.

```
Docker CLI
    │  (REST API)
    ▼
Docker daemon (dockerd)
    │
    ▼
containerd            ← high-level runtime
    │  (via shim)
    ▼
runc                  ← low-level OCI runtime
    │
    ▼
Linux kernel (namespaces, cgroups)
```

---

## What happens when you run `docker run`

1. The **Docker CLI** sends the `run` command and its arguments to the Docker daemon (**dockerd**) through a REST API call.
2. **dockerd** parses and validates the request and checks whether the image exists locally. If not, it pulls the image from the registry.
3. Once the image is ready, dockerd **hands control to containerd** to create the container from the image.
4. **containerd** sets up the container environment: file system, network interfaces, and isolation features.
5. containerd **delegates actually running the container to runc**, using a **shim** process, which creates and starts the container.
6. After startup, containerd **monitors the container's status** and manages its lifecycle.

---

## containerd vs. Docker at a glance

| | **containerd** | **Docker** |
|---|---|---|
| **What it is** | High-level container runtime | Complete platform and toolchain for containers |
| **Focus** | Core job of running containers and managing their lifecycle | Full developer experience: build, run, test, verify, share |
| **Relationship** | Used *by* Docker as its runtime layer | Built *on top of* containerd |
| **Best for** | Developers needing lower-level access to container internals and advanced features; also used by platforms such as Kubernetes | Developers wanting a cohesive, easy-to-use workflow end to end |
| **Governance** | CNCF graduated project | Docker's products plus open source components |

---

## Docker and containerd: better together

Docker helped create containerd, donated it to the CNCF, and continues to maintain and evolve it. containerd focuses on the essentials of running containers, while Docker builds on it to provide tooling across the whole software lifecycle:

| Stage | Tools mentioned in the post | Purpose |
|---|---|---|
| **Build + Run** | Docker Desktop, Docker CLI, Docker Compose | Define, build, and run single or multi-container environments; integrates with IDEs and CI/CD |
| **Test** | Testcontainers | Reproducible environments, isolated dependencies, parallel testing, simpler CI/CD |
| **Verify** | Docker Scout | Analyze images, generate an SBOM, find and fix vulnerabilities (shift left) |
| **Share** | Docker Registry / Docker Hub | Securely push images to a shared repository |
