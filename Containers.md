# **Containers & Kubernetes - DevOps Interview Questions**

---

### **1. What is a Container and how does it fundamentally differ from a Virtual Machine?**
**Answer:** A container is a software package that bundles an application's code with its libraries and dependencies, so the same code runs consistently on any machine. It virtualizes the operating system, and a container engine (such as Docker) manages it.

A Virtual Machine is a digital copy of a physical machine. It runs its own full guest OS on top of a host, with a hypervisor sharing the hardware between VMs.

Key differences:

- **Virtualization:**: Containers virtualize the OS. VMs virtualize the physical hardware.
- **Size:**: Containers are lightweight, measured in MBs, because they package only what one application needs. VMs are heavy, measured in GBs, because each includes a full OS.
- **Speed and scaling:**: Containers are quick to build, change and scale, and they suit microservices. VMs are slower to rebuild and costlier to scale.
- **Control:**: VMs give more control over the whole environment. Containers are static and portable, so they move easily between on-premises and cloud.

### **2. Explain the 7 Linux Namespaces that provide container isolation.**
**Answer:**
1. **PID (Process ID):** Isolates the process tree (container sees its main process as PID 1).
2. **NET (Networking):** Isolates network devices, IP routing tables, port numbers, and firewall rules.
3. **MNT (Mount):** Isolates filesystem mount points (container sees its own root filesystem).
4. **IPC (Inter-Process Communication):** Isolates POSIX message queues and shared memory segments.
5. **UTS (UNIX Timesharing System):** Isolates hostname and domain name.
6. **USER (User IDs):** Maps UID/GID inside the container to different UIDs on the host (e.g., container root UID 0 maps to unprivileged host UID 10001).
7. **CGROUP (Control Group):** Isolates cgroup root hierarchy visibility.

### **3. What is the Open Container Initiative (OCI)?**
**Answer:** The OCI is an open governance structure, run under the Linux Foundation, that creates open industry standards for container formats and runtimes.

The three specifications:

- **Runtime Specification (runtime-spec):** describes how to run a "filesystem bundle" that has been unpacked on disk.
- **Image Specification (image-spec):** defines the format of a container image.
- **Distribution Specification (distribution-spec):** defines how images are distributed through registries.

### **4. What is `containerd` and what is `containerd-shim`?**
**Answer:** `containerd` is an industry-standard core container runtime managing image transfer, storage, and execution. The `containerd-shim` is a lightweight daemon that sits between containerd and `runc`. It stays alive to maintain open standard I/O (stdin/stdout/stderr) streams and exit codes without keeping the main containerd daemon attached, enabling daemon restarts with zero container downtime.

### **5. What is an Overlay Filesystem (Overlay2)?**
**Answer:** A union mount filesystem that combines multiple directory layers into a single unified view:
- **LowerDir (Read-Only):** The immutable image layers.
- **UpperDir (Read-Write):** The ephemeral container layer where modifications are made.
- **WorkDir:** Internal scratch space used for atomic copy-up operations.
- **MergedDir:** The unified filesystem presented to the container.

### **6. What is Copy-on-Write (CoW) in Docker?**
**Answer:** Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If a file or directory exists in a lower layer within the image, and another layer (including the writable layer) needs read access to it, it just uses the existing file. The first time another layer needs to modify the file (when building the image or running the container), the file is copied into that layer and modified. This minimizes I/O and the size of each of the subsequent layers. These advantages are explained in more depth below.

### **7. What is a Multi-Stage Dockerfile and why is it mandatory for production?**
**Answer:** A Multi-Stage Dockerfile is a optimization pattern that uses multiple `FROM` instructions within a single Dockerfile. Each `FROM` statement initiates a fresh, isolated build stage. This allows you to use a heavy base image with all your compilers, package managers, and development dependencies in an early stage (the "builder" phase), and then selectively copy only the final compiled artifacts into a lightweight, minimal runtime image (the "production" phase), completely leaving behind the build-time bloat.

### **8. What are Distroless Images (Google)?**
**Answer:** Container base images containing *only* the application binary and its direct runtime dependencies. They contain **zero package managers, zero shells (`/bin/sh`, `/bin/bash`), and zero OS utilities**, drastically reducing attack surfaces and CVE counts.
Traditional container images include a full OS, often more than what is needed to run an application. In contrast, minimal or distroless images include only:

- The application binary
- Its runtime dependencies (e.g., libc, Java, Python)
- Any explicitly required configuration or metadata

**They typically exclude:**
- OS tools (e.g., `ls`, `ps`, `cat`)
- Shells (e.g., `sh`, `bash`)
- Package managers (e.g., `apt`, `apk`)
- Debugging utilities (e.g., `curl`, `wget`, `strace`)

### **9. What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?**
**Answer:**
- **`ENTRYPOINT`:** `ENTRYPOINT` sets a fixed executable that always runs and cannot be easily overridden. Any arguments passed via `docker run` are appended to it as parameters.
- **`CMD`:** `CMD` sets a default command or parameters that can be completely replaced if you pass any arguments at the end of the `docker run` command.

### **10. Explain `CMD ["node", "server.js"]` (Exec Form) vs `CMD node server.js` (Shell Form).**
**Answer:**
- **Exec Form (JSON Array):** Exec form, `CMD ["node", "server.js"]`, runs Node.js directly without an intermediate shell, giving more predictable argument handling and more direct signal delivery.
- **Shell Form:** Shell form, `CMD node server.js`, runs through a shell, which supports shell features such as variable expansion and command chaining but can complicate signal handling.
- For most production application containers, I prefer exec form so the application can receive termination signals directly and shut down gracefully.

### **11. What is Rootless Docker / Rootless Podman?**
**Answer:** **Rootless Docker** and **Rootless Podman** are containerization technologies that allow you to install, run, and manage containers without requiring root or administrator privileges on the host system. This represents a significant shift from traditional container setups, where the runtime engine must run with elevated `sudo` or `root permissions`.

### **12. Why is running containers with `--privileged` dangerous?**
**Answer:** Running containers with the `--privileged` flag is dangerous because it effectively breaks container isolation and grants processes inside the container full root-level access to the underlying host system.

**Key Security Risks:**
- **Full Linux Capabilities:** It grants the container all default Linux capabilities, including `SYS_ADMIN`, allowing deep kernel-level manipulations and administrative control.
- **Disabling Security Profiles:** It disables or bypasses important security mechanisms like AppArmor, SELinux confinement, and custom seccomp system call filters.
- **Direct Container Escape:** A compromised process can mount the host's root filesystem (such as `/dev/sda1` into `/host`), read sensitive files like `/etc/shadow`, overwrite SSH keys (`authorized_keys`), or spawn a `chroot` shell directly into the host operating system.
- **Host Hardware & Device Access:** It exposes all host hardware devices under `/dev` and lifts device cgroup limitations.

### **13. What is a Zombie Process (Defunct Process) in containers?**
**Answer:** 
- A **zombie process** (also known as a **defunct process**) is a child process that has completed its execution but still has an entry in the operating system's process table.
- It is called a "zombie" because it is technically dead—it consumes no CPU or memory—but it remains in the system as a placeholder because its parent process has not yet executed the `wait()` or `waitpid()` system call to read its exit status.
- While zombie processes occur on any Linux system, they are uniquely problematic inside **containers** (like Docker or Kubernetes) due to the architecture of containerized environments.

### **14. What is Docker BuildKit?**
**Answer:** **BuildKit** is the builder backend used by Docker. BuildKit provides improved functionality and improves your builds' performance over the legacy builder used in earlier versions of Docker. It also introduces support for handling more complex scenarios:
- Detect and skip executing unused build stages
- Parallelize building independent build stages
- Incrementally transfer only the changed files in your build context between builds
- Detect and skip transferring unused files in your build context
- Use Dockerfile frontend implementations with many additional features
- Avoid side effects with rest of the API (intermediate images and containers)
- Prioritize your build cache for automatic pruning

### **15. What is Multi-Architecture Building (`docker buildx`)?**
**Answer:** **Multi-architecture building** with `docker buildx` is an advanced Docker feature that allows you to create a single container image that can run on multiple processor architectures (such as x86_64, ARM64, and ARMv7) without modifying your source code.

### **16. What is an OCI Image Manifest and Manifest List?**
**Answer:** 

**OCI Image Manifest:**
- **Definition:** A single metadata file (`application/vnd.oci.image.manifest.v1+json`) that describes one specific container image.
- **Content:**
  - A reference to a config blob (which contains environment variables, entrypoints, and build history).
  - An ordered list of filesystem layers (stored as compressed tarball blobs) that build the container's root filesystem.
- **Key Goal:** Enables content-addressable storage by hashing the image components with SHA-256 to generate a unique digital fingerprint (digest).


**OCI Image Manifest List (Image Index)** 
- **Definition:** A higher-level index file (`application/vnd.oci.image.index.v1+json` or similar) that points to a collection of individual image manifests.
- **Content:** A list of references to different image manifests, each tagged with specific platform criteria like os (e.g., Linux, Windows) and `architecture` (e.g., `amd64`, `arm64`).
- **Key Goal:** Enables multi-architecture support. When a user runs `docker pull` or `podman pull` on a multi-arch tag, the container runtime inspects the manifest list, matches the host machine's OS and CPU architecture, and downloads the correct underlying image manifest and layers.

### **17. Compare Docker Bridge vs Host vs None Network Modes.**
**Answer:**
- **Bridge (Default):** Creates a private virtual Ethernet bridge (`docker0`) on the host; containers get private IPs and communicate via NAT/port forwarding.
- **Host:** Container shares the host network namespace directly with zero network isolation (no port mapping needed, maximum throughput).
- **None:** Container has no network interface except loopback (`lo`), completely air-gapped.

### **18. What is Docker Swarm vs Kubernetes?**
Both **Docker Swarm** and **Kubernetes (K8s)** are **open-source container orchestration tools designed to manage, scale, and deploy clustered application containers**. However, they solve this problem with entirely different approaches to complexity, control, and scale.

**The core distinction:** Docker Swarm focuses on simplicity and ease of use, making it suitable for smaller setups, while Kubernetes focuses on extensibility and control, making it a popular choice for large, complex enterprise environments.

### At-a-Glance Comparison

| Feature | Docker Swarm | Kubernetes (K8s) |
|---|---|---|
| **Setup & Installation** | **Very simple:** Built into Docker Engine. Start with `docker swarm init`. | **More complex:** Requires tooling such as `kubeadm` or a managed Kubernetes service. |
| **Learning Curve** | **Low:** Uses the familiar Docker CLI and Compose-style workflows. | **Steeper:** Introduces abstractions such as Pods, Deployments, Services, and CRDs. |
| **Scaling** | Supports manual scaling of services and replicas. | Supports manual scaling and automated scaling through features such as the Horizontal Pod Autoscaler (HPA). |
| **Load Balancing** | Built-in routing mesh and service load balancing. | Service-based load balancing, with Ingress controllers or Gateway implementations for external HTTP routing. |
| **Self-Healing** | Reconciles services and replaces failed task replicas. | Supports workload reconciliation, health probes, and automated container restarts or replacement. |
| **Ecosystem & Community** | Smaller ecosystem, closely integrated with Docker tooling. | Extensive ecosystem with CNCF projects and many integrations. |

### Understanding Docker Swarm

**Docker Swarm** is Docker's native clustering tool. Because it is built directly into the Docker engine, turning a machine into a cluster manager takes a single command. It uses standard Docker Compose files `(yaml)` to deploy applications across multiple nodes.

- **Best for:** Small teams (typically under 10 engineers), simple stateless architectures, development/testing environments, and projects where minimizing configuration overhead is a priority.
- **The Catch:** It lacks advanced native scheduling, has minimal built-in security isolation (RBAC), and monitoring requires piecing together third-party tools.

### Understanding Kubernetes

**Kubernetes (K8s)**Originally built by Google, Kubernetes is an advanced, production-grade platform designed to control every aspect of microservice architecture. It operates as a highly customisable, self-healing framework capable of running thousands of containers across multi-cloud environments

- **Best for:** Large-scale distributed applications, multi-tenant environments, and organizations requiring high automation, custom routing rules, or rigid compliance.
- **The Catch:** It comes with high operational overhead. Managing bare Kubernetes requires dedicated platform engineering resources, though cloud-managed services (like AWS EKS or Google GKE) help offset the infrastructure headache.

### **19. What is a Container Healthcheck (`HEALTHCHECK`)?**
**Answer:** A **Container Healthcheck (HEALTHCHECK)** is a configuration instruction used in containerization (most notably in **Docker**) to tell the container runtime how to test whether the container is still working correctly.

### **20. What is Docker Volume vs Bind Mount vs tmpfs?**
**Answer:**
**Direct Comparison**

| **Feature**              | **Docker Volume**                                                            | **Bind Mount**                                                               | **tmpfs Mount**                                                          |
| ------------------------ | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Storage Location**     | **Host filesystem** (Docker-managed area, e.g., `/var/lib/docker/volumes/`)  | **Anywhere on the host** filesystem                                          | **Host memory (RAM)** only; never written to disk                        |
| **Lifecycle Management** | **Managed by Docker**. Persists after container deletion.                    | **Managed by the user**. You manage paths, permissions, and cleanup.         | **Tied to the container**. Automatically wiped when the container stops. |
| **Portability**          | **High**. Independent of the host OS file structure.                         | **Low**. Relies on specific host paths, which can break across OS platforms. | **Medium** (Exclusive to Linux hosts).                                   |
| **Performance**          | **Excellent** (native speed on Linux; highly optimized).                     | Can suffer from **I/O overhead** on macOS/Windows due to VM layers.          | **Fastest** read/write speeds because it operates in RAM.                |
| **Best Used For**        | Databases, production data persistence, and sharing data between containers. | Local development (live code reloads) and sharing host configurations.       | Sensitive credentials, API keys, or high-speed cache/session state.      |


### **21. What is Docker Compose?**
**Answer:** **Docker Compose** is **a declarative tool developed by Docker used for defining and running multi-container applications using a single configuration file**. Instead of managing several different containers using long, error-prone `docker run` commands in the terminal, you describe your entire application stack—services, databases, networks, storage, and variables—in a human-readable **YAML file** (`compose.yaml`).

With this blueprint in place, a developer can build, start, and stop the entire multi-container environment using a **single command**.

**Dockerfile vs. Docker Compose**

It is common to confuse Dockerfiles with Docker Compose files, but they serve entirely distinct roles in a containerized pipeline: 

| **Feature**      | **Dockerfile**                                   | **Docker Compose (`compose.yaml`)**                         |
| ---------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| **Primary Goal** | **Builds** a single container image.             | **Runs and orchestrates** one or more containers.           |
| **Scope**        | One individual component (e.g., just the API).   | The entire application stack (API + DB + Cache).            |
| **Syntax**       | Imperative steps (`FROM`, `RUN`, `COPY`, `CMD`). | Declarative configuration (`services`, `ports`, `volumes`). |


**How the Workflow Operates**

1. **Write the Configuration:** You create a `compose.yaml` file defining your services. You can either pull pre-built images from Docker Hub or tell Compose to build them locally from a `Dockerfile`. 
2. **Launch the Stack:** Run `docker compose up -d`. Compose intelligently analyzes the file, pulls/builds the images, spins up the networks, maps host ports, and launches the containers in the correct order using dependency rules (like `depends_on`).
3. **Iterate Effeciently:** If you change a configuration or update a codebase, running `docker compose up` again will only recreate the specific containers that were modified rather than rebuilding the entire stack from scratch.
4. **Clean Up:** Run `docker compose down` to stop the containers and cleanly wipe the temporary networks, returning your host machine to its original state. 

### **22. What is Kaniko?**
**Answer:** An open-source tool developed by Google that builds container images from a Dockerfile inside a Kubernetes pod or container without requiring a Docker daemon or privileged root access.

### **23. How do you pass secrets to Docker builds without exposing them in image history?**
**Answer:** Use BuildKit secret mounts: `RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret`. Secrets are mounted into memory during build time and are never stored in any image layer.

### **24. What is Image Layer Caching in Docker?**
**Answer:** **Docker Image Layer Caching** is a mechanism that speeds up Docker image build times by reusing unchanged image layers from previous builds instead of recreating them from scratch.
When you run a build, Docker checks its cache for each instruction in your `Dockerfile`. If it finds a match, it achieves a cache hit and instantly skips that step. If it doesn't, it triggers a cache miss and rebuilds that layer and all subsequent layers.

### **25. What is `.dockerignore` and why is it critical?**
**Answer:** A `.dockerignore` file is a plain-text configuration file placed in the root of a project's build context. It tells the Docker CLI which files and directories to exclude when packing up the project and sending it to the Docker daemon. It works similarly to a `.gitignore` file, though with distinct path-matching rules.

#### Why It Is Critical:

- **Speeds Up Build Times:** When you run `docker build`, the Docker CLI packs every file in your directory into a tar archive and sends it to the daemon as the build context. Excluding heavy, non-essential directories—like `node_modules` or local data dumps—prevents hundreds of megabytes of unnecessary data from transferring on every single build.
- **Prevents Permanent Security Leaks:** Files copied into a Docker image leave permanent traces in the image's history layers. If you accidentally copy a `.env` file, database password, or private SSH key into a build—even if you delete it in a later `Dockerfile` step—anyone who downloads or inspects the image can recover those credentials.
- **Avoids Unnecessary Rebuilds:** Noisy files that change constantly (such as `.git` history folders or local log files) can invalidate Docker's layer cache. Excluding them ensures Docker only rebuilds your image when actual application code changes.
- **Reduces Image Size:** Keeping out test suites, markdown documentation, and local build artifacts keeps your final container footprint lean, which accelerates container pull times and deployments across servers.

### **26. What is Container Network Interface (CNI)?**
**Answer:** A CNCF specification and library providing a standardized interface for configuring network interfaces, IP addresses, and routing in Linux containers and Kubernetes pods (e.g., Calico, Cilium, AWS VPC CNI).

### **27. What is Docker Content Trust (DCT)?**
**Answer:** **Docker Content Trust (DCT)** is a security feature that enables digital signature verification of container data sent to and received from remote Docker registries. It allows image publishers to cryptographically sign their image tags, enabling consumers to verify the **integrity** (ensuring the image wasn't altered in transit) and the **publisher** (authenticity of the source) of the data before it is run or built upon.

### **28. What is an OCI Artifact?**
**Answer:** An **Open Container Initiative (OCI) Artifact** is any non-container-image file or package-such as a Helm chart, WebAssembly (Wasm) module, cryptographic signature, Software Bill of Materials (SBOM), or AI model-stored and distributed inside standard OCI-compliant container registries.

#### Key Concepts

- **Beyond Containers:** While OCI registries were originally built strictly for Docker/container images, the OCI specification allows them to act as general-purpose, content-addressable storage for any software asset.
- **How They Work:** They extend the standard OCI image manifest and layer structure, using custom media types or tools like **ORAS** (OCI Registry As Storage) to push and pull arbitrary files without needing a container runtime to execute them.


### **29. What is Docker Scout / Trivy?**
**Answer:** **Docker Scout** and **Trivy** are both **security tools used to scan container images for known vulnerabilities (CVEs) and generate Software Bills of Materials (SBOMs)**. However, they approach container security from completely different angles: **Docker Scout is a native, ecosystem-driven tool focused on image remediation**, while **Trivy is a versatile, open-source multi-target scanner**.

Here is a side-by-side comparison to help you understand what they are and how they differ.

**Direct Comparison**

| **Feature**          | **Docker Scout**                                                                                                                   | **Trivy (by Aqua Security)**                                                                                               |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Core Philosophy**  | Embedded Docker ecosystem utility focused on **remediation and real-time policy**.                                                 | Swiss Army knife **open-source scanner** for cloud-native infrastructure.                                                  |
| **Primary Scope**    | Container images and software supply chains.                                                                                       | Container images, filesystems, Git repos, IaC (Terraform, Kubernetes configs), secrets, and licenses.                      |
| **Installation**     | Built-in to [**Docker Desktop**](https://docs.docker.com/scout/) and the `docker` CLI.                                             | Standalone single binary (can also run via Docker container).                                                              |
| **Data & Speed**     | Evaluates metadata snapshots via **live cloud API calls**; instantly updates when new CVEs are published without needing re-scans. | Downloads and caches the **vulnerability database locally**; blistering speed on subsequent offline runs.                  |
| **Standout Feature** | **Context-aware fix recommendations** (e.g., "Change your base image to X to drop CVEs from 150 to 4").                            | **Multi-target scanning**; scans your whole repository for misconfigurations and exposed secrets alongside your container. |
| **Licensing**        | Commercial product with a free tier (integrated into Docker subscription tiers).                                                   | **100% Open Source** (Apache 2.0).                                                                                         |

### **30. What is `docker system prune`?**
**Answer:** The **`docker system prune`** command is **a global cleanup utility used to reclaim disk space by deleting unused Docker data**. Over time, Docker builds up clutter from stopped containers, old build caches, and untagged images, which can heavily consume storage.

**What it Deletes by Default**

When you run `docker system prune`, it safely targets only **unreferenced or stopped** resources. It will ask for confirmation before removing:
- **Stopped containers:** All containers that are not currently running.
- **Unused networks:** Networks not connected to at least one container.
- **Dangling images:** Layers that no longer have a tag and are not associated with any container.
- **Build cache:** Temporary files generated during the `docker build` process.

*Note: By default, it **will not** delete volumes to prevent accidental data loss.* 

### **31. What is a Docker Plugin?**
**Answer:** A **Docker Plugin** is an out-of-process extension that adds specialized capabilities to the Docker Engine. They allow developers and system administrators to integrate third-party technologies directly into core Docker operations without altering the underlying Docker source code.

### **32. What is `iptables` in Docker?**
**Answer:** In **Docker**, `iptables` is a Linux kernel utility that the Docker Engine uses automatically to manage container networking, traffic routing, and security isolation.
When you create networks, run containers, or publish ports, Docker dynamically writes `iptables` rules behind the scenes to make everything work transparently

### **33. What is Docker daemon (`dockerd`)?**
**Answer:** The **Docker daemon (`dockerd`)** is **the persistent background service that acts as the core engine of Docker**. It is responsible for building, running, and managing Docker objects like containers, images, networks, and storage volumes.

**How it Works**

Docker uses a client-server architecture. The Docker CLI (`docker`) acts as the client (frontend), while the Docker daemon (`dockerd`) acts as the server (backend).

1. **The Request:** You type a command in your terminal, such as `docker run nginx`.
2. **The API Call:** The Docker client translates your command into a **REST API call** and sends it to the Docker daemon over a local UNIX socket or a network connection.
3. **Execution:** The daemon receives the request, pulls the required image from a registry (like **Docker Hub** if it isn't available locally), and coordinates with low-level runtimes (like `containerd` and `runc`) to execute the container.

**Key Responsibilities of `dockerd`**

- **Container Lifecycle:** Handles the creation, starting, stopping, monitoring, and deletion of containers.
- **Image Management:** Manages local container blueprints and interacts with remote registries to pull or push images.
- **Resource Allocation:** Sets up isolated file systems, environments, storage volumes, and network routing for individual containers.
- **System Configuration:** Reads configurations from a centralized JSON file (`/etc/docker/daemon.json` on Linux) to customize runtimes, storage drivers, and log settings.

### **34. What is Docker Socket (`/var/run/docker.sock`)?**
**Answer:** The **Docker socket** (`/var/run/docker.sock`) is the primary, UNIX-based network socket that serves as the API endpoint for the Docker daemon (`dockerd`).
When you run any Docker command (like `docker run` or `docker ps`), the **Docker client-line interface (CLI)** sends that request directly through this socket file using local **Inter-Process Communication (IPC)**. The Docker daemon then receives the message, processes it, and manages your containers accordingly

### **35. What are the core components of the Kubernetes Master / Control Plane?**
**Answer:**
**Kube-api-server → (For all communications)**
- This api-server interacts directly with the user (i.e we apply .yml or .json manifest to kube-api-server)
- This kube-api-server is meant to scale automatically as per load.
- Kube-api-server is the front end of the control plane.

**etcd**
- Stores metadata and status of the cluster.
- etcd is a consistent and high-available store (key-value-store)
- Source of touch for cluster state (info about the state of the cluster)

**etcd has the following features**
1. Fully Replicated → The entire state is available on every node in the cluster.
2. Secure → Implements automatic TLS with optional client-certificate authentication.
3. Fast → Benchmarked at 10,000 writes per second.

**Kube-scheduler (action)**
- When users request the creation & management of Pods, Kube-scheduler is going to take action on these requests.
- Handles POD creation and Management.
- Kube-scheduler match/assigns any node to create and run pods.
- A scheduler watches for newly created pods that have no node assigned. For every pod that the scheduler discovers, the scheduler becomes responsible for finding the best node for that pod to run.
- The scheduler gets the information for hardware configuration from configuration files and schedules the Pods on nodes accordingly.

**Controller-Manager**
- Make sure the actual state of the cluster matches the desired state.
- Two possible choices for controller manager
1. If K8s is on the cloud, then it will be a cloud controller manager.
2. If K8s is on non-cloud, then it will be kube-controller-manager.


### **62. What are the core components of a Kubernetes Worker Node?**
**Answer:**
1. **`kubelet`:** The node agent that communicates with the API server, watches PodSpecs assigned to the node, and commands the CRI runtime to start containers.
2. **`kube-proxy`:** Manages network routing and iptables/IPVS packet filtering rules on each node to implement Kubernetes `Service` abstractions.
3. **Container Runtime (CRI):** The runtime (containerd/CRI-O) executing container processes.

### **63. Explain `etcd` internals and how it maintains data consistency.**
**Answer:** `etcd` is an append-only distributed key-value store implementing the **Raft consensus algorithm**. It requires an odd number of nodes (3 or 5) to form a quorum ($\lfloor N/2 \rfloor + 1$). It writes updates to a Write-Ahead Log (WAL) before committing to memory, providing linearizable reads and strict serializability.

### **64. What is a Kubernetes Pod?**
**Answer:** The smallest deployable computing unit in Kubernetes. A Pod encapsulates one or more co-located containers that share the exact same Network namespace (IP address and port space), IPC namespace, and shared storage volumes.

### **65. What is the Pause Container (`infra` container)?**
**Answer:** A minimal container running an infinite sleep loop spawned first in every Pod. It establishes and holds the shared Network, IPC, and UTS namespaces. Application containers join the pause container's namespaces (`--net=container:pause`), allowing containers in the pod to communicate over `localhost`.

### **66. What is a Kubernetes Deployment vs ReplicaSet vs Pod?**
**Answer:**
- **Pod:** The running instance of containers.
- **ReplicaSet:** Ensures a specified number of identical Pod replicas are running at any given time.
- **Deployment:** A higher-level controller managing declarative rolling updates, rollbacks, and version history for ReplicaSets.

### **67. What is a StatefulSet and how does it differ from a Deployment?**
**Answer:** StatefulSet manages stateful applications requiring:
1. Stable, unique network identifiers (`pod-0`, `pod-1`).
2. Ordered, sequential deployment and rolling updates.
3. Dedicated, persistent volume bindings via `volumeClaimTemplates` that persist across pod rescheduling.

### **68. What is a DaemonSet?**
**Answer:** Ensures that all (or selected) worker nodes run exactly one copy of a specific Pod. Used for node-level agents (Cilium CNI, Fluent Bit log shippers, Prometheus `node_exporter`).

### **69. What is a Kubernetes Job vs CronJob?**
**Answer:**
- **Job:** Spawns one or more pods and ensures a specified number of them terminate successfully (run-to-completion batch tasks).
- **CronJob:** Schedules and executes `Job` objects periodically based on a standard cron expression.

### **70. Explain Kubernetes Service Types (ClusterIP, NodePort, LoadBalancer, ExternalName).**
**Answer:**
- **ClusterIP (Default):** Exposes the Service on an internal cluster IP, accessible *only* within the cluster.
- **NodePort:** Exposes the Service on a static high port (`30000-32767`) on every worker node's physical IP.
- **LoadBalancer:** Provisions an external cloud load balancer (e.g., AWS NLB/ALB) and routes traffic to NodePort/ClusterIP.
- **ExternalName:** Maps the Service to a CNAME DNS record (e.g., `my-db.external.com`) without proxying.

### **71. What is a Headless Service (`clusterIP: None`)?**
**Answer:** A Service that does not allocate a ClusterIP. CoreDNS returns the individual `A` records of all backing Pod IPs directly, allowing clients to establish direct peer-to-peer connections (used for StatefulSets, Kafka, and Cassandra).

### **72. What is Kubernetes Ingress vs Ingress Controller?**
**Answer:**
- **Ingress:** A declarative API resource defining Layer 7 HTTP/HTTPS routing rules, TLS termination, and host/path routing.
- **Ingress Controller:** The actual reverse proxy daemon (Nginx Ingress, Envoy, Traefik) that watches Ingress resources and configures routing rules.

### **73. What is Kubernetes Gateway API?**
**Answer:** The modern successor to Ingress providing role-oriented resource separation:
- `GatewayClass` (Cluster Operator defines infrastructure).
- `Gateway` (Platform Engineer configures listeners, ports, TLS).
- `HTTPRoute` / `GRPCRoute` (Developer defines routing paths and traffic splitting).

### **74. Explain Kubernetes Namespaces and their isolation boundaries.**
**Answer:** Virtual clusters within a physical cluster providing logical scoping for object names, RBAC policies, and ResourceQuotas. Namespaces do **not** provide network isolation by default (requires NetworkPolicies).

### **75. What is a ConfigMap vs a Secret?**
**Answer:**
- **ConfigMap:** Stores non-confidential configuration data in key-value pairs mounted as environment variables or volume files.
- **Secret:** Stores sensitive data (passwords, tokens, keys) encoded in base64. Stored encrypted at rest in `etcd` when KMS encryption is enabled.

### **76. What is a PersistentVolume (PV) vs PersistentVolumeClaim (PVC)?**
**Answer:**
- **PV:** A storage resource provisioned in the cluster (e.g., AWS EBS volume) by an administrator or StorageClass.
- **PVC:** A request for storage by a user specifying size and access modes (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`).

### **77. What is Dynamic Volume Provisioning (StorageClass)?**
**Answer:** A mechanism where Kubernetes automatically provisions underlying cloud storage volumes (e.g., creates an AWS gp3 volume via CSI) on-demand when a developer creates a PVC referencing a `StorageClass`.

### **78. Explain Kubernetes Storage Access Modes.**
**Answer:**
- **RWO (ReadWriteOnce):** Volume can be mounted as read-write by a single worker node.
- **ROX (ReadOnlyMany):** Volume can be mounted as read-only by multiple worker nodes simultaneously.
- **RWX (ReadWriteMany):** Volume can be mounted as read-write by many worker nodes concurrently (e.g., AWS EFS, NFS).
- **RWOP (ReadWriteOncePod):** Volume can be mounted as read-write by a single Pod instance only.

### **79. What is a Kubernetes ServiceAccount (SA)?**
**Answer:** An identity assigned to Pods to authenticate against the Kubernetes API Server. Tokens are mounted automatically at `/var/run/secrets/kubernetes.io/serviceaccount/token`.

### **80. Explain Kubernetes RBAC: Role vs ClusterRole, RoleBinding vs ClusterRoleBinding.**
**Answer:**
- **Role:** Grants API permissions (verbs: `get`, `list`, `create`) scoped strictly to a single namespace.
- **ClusterRole:** Grants cluster-wide permissions across all namespaces and cluster-scoped resources (Nodes, PVs).
- **RoleBinding:** Assigns a Role or ClusterRole to subjects within a specific namespace.
- **ClusterRoleBinding:** Assigns a ClusterRole to subjects cluster-wide.

### **81. What is an Admission Controller?**
**Answer:** Plugins that intercept API Server requests after authentication and authorization, but *before* the object is persisted to `etcd`.
- **Mutating Admission Controllers:** Modify objects (e.g., injecting sidecars).
- **Validating Admission Controllers:** Enforce security policies and accept or reject objects (e.g., Kyverno, OPA).

### **82. Explain the difference between Liveness, Readiness, and Startup Probes.**
**Answer:**
- **Startup Probe:** Validates if legacy slow-booting applications have started. Disables liveness and readiness checks until it passes.
- **Liveness Probe:** Checks if the application process is running and healthy. If it fails, `kubelet` restarts the container.
- **Readiness Probe:** Checks if the container is ready to accept incoming network traffic. If it fails, the Pod's IP is removed from Service endpoints.

### **83. What are Init Containers?**
**Answer:** Specialized containers that run and complete sequentially *before* primary application containers start. Used for blocking tasks (waiting for database availability, running migrations).

### **84. What are Ephemeral Containers?**
**Answer:** Temporary containers injected into existing running Pods for interactive debugging and network troubleshooting (`kubectl debug`) without restarting the Pod.

### **85. What is a Pod Disruption Budget (PDB)?**
**Answer:** A policy limiting the number of Pod replicas of an application that can be unavailable simultaneously during voluntary disruptions (node draining, cluster upgrades).

### **86. What are Resource Requests vs Limits?**
**Answer:**
- **Requests:** The guaranteed minimum CPU and memory allocated for the Pod. Used by `kube-scheduler` to place the Pod on a node.
- **Limits:** The hard upper bound of CPU and memory the Pod is allowed to consume. CPU exceeding limits causes CFS throttling; memory exceeding limits triggers OOMKill.

### **87. What are Kubernetes Quality of Service (QoS) Classes?**
**Answer:**
1. **Guaranteed:** Every container has equal CPU and Memory Requests and Limits. Evicted last during resource exhaustion.
2. **Burstable:** Requests are set and lower than Limits. Evicted when Guaranteed pods require resources.
3. **BestEffort:** Zero requests and limits set. Evicted first during node memory pressure.

### **88. What is Node Affinity and Anti-Affinity?**
**Answer:**
- **Node Affinity:** Constrains which nodes a Pod can schedule on based on node labels (`requiredDuringScheduling...` or `preferredDuringScheduling...`).
- **Pod Anti-Affinity:** Prevents multiple replicas of the same service from scheduling on the same node or AZ to ensure high availability.

### **89. What are Taints and Tolerations?**
**Answer:**
- **Taint:** Applied to a **Node** to repel Pods (`key=value:NoSchedule`, `NoExecute`).
- **Toleration:** Applied to a **Pod** allowing it to schedule on matching tainted nodes (e.g., scheduling GPU workloads on dedicated GPU nodes).

### **90. What is Topology Spread Constraints?**
**Answer:** Rules that control how Pods are evenly spread across failure domains (Availability Zones, racks, nodes) based on label selectors (`maxSkew: 1`).

### **91. What is the Kubelet Eviction Manager?**
**Answer:** A proactive subsystem in kubelet that monitors node memory, disk, and PID usage. When thresholds are breached (`memory.available < 100Mi`), it evicts lower-priority pods to prevent node failure.

### **92. What is PriorityClass and Preemption?**
**Answer:** Defines the relative importance of Pods (`value: 1000000`). If a high-priority Pod cannot schedule due to lack of resources, the scheduler preempts (evicts) lower-priority pods to free up capacity.

### **93. What is a Horizontal Pod Autoscaler (HPA)?**
**Answer:** A control loop that automatically adjusts the number of Pod replicas in a Deployment based on observed CPU/memory utilization or custom external metrics.

### **94. What is a Vertical Pod Autoscaler (VPA)?**
**Answer:** A controller that automatically recommends or updates CPU and memory resource requests/limits based on historical runtime analysis.

### **95. What is KEDA (Kubernetes Event-driven Autoscaling)?**
**Answer:** An autoscaler that scales Kubernetes workloads from 0 to hundreds based on external event sources (Kafka consumer lag, AWS SQS queues, RabbitMQ, Prometheus metrics).

### **96. What is Karpenter?**
**Answer:** An open-source, high-performance node autoscaler developed by AWS that bypasses native Auto Scaling Groups, provisioning right-sized EC2 instances directly via EC2 Fleet API in seconds.

### **97. What is Karpenter Node Consolidation?**
**Answer:** An automated feature where Karpenter continuously evaluates cluster compute waste, drains pods from underutilized nodes, and replaces them with cheaper, right-sized single instances.

### **98. What is Cluster Autoscaler (CA)?**
**Answer:** A traditional autoscaler that adds or removes worker nodes by modifying the desired capacity of underlying cloud Auto Scaling Groups (ASGs) when Pods enter `Pending` state.

### **99. What is CoreDNS in Kubernetes?**
**Answer:** The internal DNS server running as a cluster service that resolves internal service names (`my-svc.my-ns.svc.cluster.local`) and external domains for Pods.

### **100. What is NodeLocal DNSCache?**
**Answer:** A DaemonSet that runs a local DNS caching agent on each worker node, reducing CoreDNS network queries, UDP packet drops, and glibc `ndots:5` latency storms.

### **101. What is the Kubernetes Endpoints vs EndpointSlice API?**
**Answer:**
- **Endpoints:** A monolithic object tracking all pod IPs backing a service; updating 1 pod in a 5,000-pod service rewritten the entire object, overloading etcd.
- **EndpointSlice:** Splits backing pod endpoints into scalable chunks of 100 endpoints each, vastly improving cluster scalability.

### **102. What is `kube-proxy` IPVS mode vs iptables mode?**
**Answer:**
- **iptables:** Linear sequential rule evaluation ($O(N)$). As services reach thousands, packet processing latency degrades significantly.
- **IPVS (IP Virtual Server):** Linux kernel hash table lookup ($O(1)$). Provides consistent sub-millisecond throughput regardless of service scale.

### **103. What is Cilium eBPF CNI?**
**Answer:** A next-generation CNI plugin that replaces `kube-proxy` and iptables with eBPF programs loaded directly into Linux kernel socket layers, providing $O(1)$ routing, Layer 7 visibility, and transparent encryption.

### **104. What is a Kubernetes NetworkPolicy?**
**Answer:** A declarative firewall resource that controls traffic flow between Pods at Layer 3 and Layer 4 using label selectors.

### **105. What is a Default-Deny NetworkPolicy?**
**Answer:** A policy selecting all pods in a namespace with empty ingress/egress rules, blocking all incoming and outgoing network traffic until explicitly whitelisted.

### **106. What is Pod Security Admission (PSA)?**
**Answer:** The built-in replacement for legacy PodSecurityPolicies (PSP) that enforces three security profiles (**Privileged**, **Baseline**, **Restricted**) via namespace labels.

### **107. What is OPA Gatekeeper in Kubernetes?**
**Answer:** An admission controller that enforces custom organizational governance policies written in Rego across Kubernetes manifests.

### **108. What is Kyverno?**
**Answer:** A Kubernetes-native policy engine written in pure YAML that validates, mutates, and generates resources and verifies container image signatures.

### **109. What is Helm and what are Helm Hooks?**
**Answer:** The package manager for Kubernetes. Helm Hooks (`pre-install`, `post-upgrade`, `pre-delete`) allow executing specific jobs (database migrations) at defined lifecycle points during chart releases.

### **110. What is Helm Chart Dependency Management (`Chart.lock`)?**
**Answer:** Defining subcharts in `Chart.yaml` and locking their exact semantic versions and tarball hashes in `Chart.lock`.

### **111. What is Helm Values Schema Validation (`values.schema.json`)?**
**Answer:** A JSON Schema file bundled inside a Helm chart that validates user-provided `values.yaml` inputs before template rendering.

### **112. What is Kustomize?**
**Answer:** A template-free configuration customization tool built directly into `kubectl` (`kubectl apply -k`) that uses declarative overlays on top of base YAML files.

### **113. What is Kustomize Strategic Merge Patch?**
**Answer:** A patching mechanism that merges overlay YAML blocks into base manifests based on resource names and keys rather than overwriting arrays.

### **114. What is a Kubernetes Custom Resource Definition (CRD)?**
**Answer:** An API extension mechanism that allows developers to define custom object kinds (e.g., `kind: KafkaTopic`) that the Kubernetes API Server stores and manages like native resources.

### **115. What is a Kubernetes Custom Controller (Operator Pattern)?**
**Answer:** A software pattern combining CRDs with a custom control loop that watches state and executes domain-specific automation (managing database failovers, taking automated backups).

### **116. What is the Operator Framework / Kubebuilder?**
**Answer:** Go SDKs and scaffolding tools providing code generators, webhook handlers, and controllers for authoring production Kubernetes operators.

### **117. What is Leader Election in Kubernetes Controllers?**
**Answer:** A consensus mechanism using `Lease` API objects in `coordination.k8s.io` to ensure only one active replica of a controller manager executes reconciliation loops at a time.

### **118. What is etcd Compaction and Defragmentation?**
**Answer:**
- **Compaction:** Discards historical revision logs from etcd storage.
- **Defragmentation:** Reclaims disk space and reorganizes on-disk database pages to prevent reaching etcd's 8GB storage quota.

### **119. What is Kubernetes API Versioning (`v1alpha1`, `v1beta1`, `v1`)?**
**Answer:**
- `alpha`: Disabled by default; experimental; may be dropped without notice.
- `beta`: Enabled by default; tested; schema backward-compatibility guaranteed until deprecation.
- `v1` (GA): Stable; long-term support and backward compatibility.

### **120. What is Kubernetes Deprecation Policy?**
**Answer:** Stable APIs cannot be removed without being deprecated for at least 12 months (or 3 consecutive minor releases).

### **121. What is API Server Aggregation Layer?**
**Answer:** Allows extending the Kubernetes API by registering independent secondary API servers (e.g., Metrics Server) behind the main API Server.

### **122. What is the Kubernetes Metrics Server?**
**Answer:** An in-memory, cluster-wide aggregator of resource usage data (CPU, memory) scraped from Kubelets, consumed by `kubectl top` and HPA.

### **123. What is Prometheus Operator?**
**Answer:** An operator that automates deploying and managing Prometheus monitoring stacks on Kubernetes using CRDs (`ServiceMonitor`, `PrometheusRule`).

### **124. What is a `ServiceMonitor` in Prometheus Operator?**
**Answer:** A declarative CRD that discovers and configures Prometheus scrape targets based on Kubernetes Service labels and endpoints.

### **125. What is External Secrets Operator (ESO)?**
**Answer:** An operator that syncs secrets from external systems (AWS Secrets Manager, HashiCorp Vault) into native Kubernetes `Secret` resources.

### **126. What is cert-manager?**
**Answer:** A Kubernetes add-on that automates the issuance and renewal of TLS certificates from Let's Encrypt, HashiCorp Vault, and private PKIs.

### **127. What is Kubernetes Garbage Collection (Cascading Deletion)?**
**Answer:** The mechanism where deleting an owner resource (Deployment) automatically deletes all its dependent children (ReplicaSets, Pods) via `ownerReferences`.

### **128. What is Finalizers in Kubernetes?**
**Answer:** Pre-delete hooks in resource metadata that prevent an object from being deleted from etcd until external cleanup operations (releasing cloud storage) complete.

### **129. What is a DaemonSet Rolling Update Strategy?**
**Answer:** `RollingUpdate` with `maxUnavailable` (e.g., `maxUnavailable: 1`) updates DaemonSet pods on worker nodes one at a time.

### **130. What is Kubernetes Node Drain vs Cordon?**
**Answer:**
- **Cordon (`kubectl cordon`):** Marks the node as unschedulable; existing pods remain running.
- **Drain (`kubectl drain`):** Cordons the node and safely evicts all running pods (respecting PDBs) so the node can be rebooted or terminated.

### **131. What is `kubectl exec` vs `kubectl attach`?**
**Answer:**
- **`exec`:** Spawns a *new* process inside the container.
- **`attach`:** Attaches standard input/output streams to the *existing* running primary process (PID 1).

### **132. What is `kubectl port-forward`?**
**Answer:** Establishes a direct TCP tunnel from a local workstation port to a specific Pod or Service port inside the private Kubernetes cluster.

### **133. What is a Mutating Webhook Failure Policy (`Ignore` vs `Fail`)?**
**Answer:**
- **`Ignore`:** If the webhook server is unreachable, the API Server admits the resource anyway.
- **`Fail`:** If the webhook is unreachable, the API Server rejects the resource (mandatory for security admission controllers).

### **134. What is Kubernetes Graceful Termination Period (`terminationGracePeriodSeconds`)?**
**Answer:** The time Kubernetes allows a Pod to cleanly shut down (drain connections, complete transactions) after sending `SIGTERM` before forcibly killing it with `SIGKILL` (default: 30 seconds).

### **135. What is a `preStop` Lifecycle Hook?**
**Answer:** A script or HTTP call executed inside the container *before* the `SIGTERM` signal is sent, used to initiate graceful connection draining.

### **136. What is Kubernetes Server-Side Apply (SSA)?**
**Answer:** A field-level management mechanism where the API Server tracks which controller or user owns specific YAML fields via `managedFields`, preventing accidental field overwrites.

### **137. What is Subpath Volume Mount in Kubernetes?**
**Answer:** Mounting a specific sub-directory or single file from a volume into a container path rather than mounting the entire root volume directory.

### **138. What is Projected Volumes in Kubernetes?**
**Answer:** Combining multiple volume sources (Secrets, ConfigMaps, DownwardAPI, ServiceAccountTokens) into a single unified directory mount inside the container.

### **139. What is Downward API in Kubernetes?**
**Answer:** Exposing Pod and container metadata (Pod IP, Node name, namespace, resource limits) to the application via environment variables or volume files.

### **140. What is Kubernetes Event (`kubectl get events`)?**
**Answer:** Ephemeral records stored in etcd (retained for 1 hour) capturing lifecycle state changes, scheduling decisions, warnings, and errors across cluster objects.

---

## 🔴 **Part 3: Advanced Architectures & Production Troubleshooting (Questions 141–250)**

### **141. Scenario: A Pod is stuck in `CrashLoopBackOff`. Walk through your step-by-step diagnostic workflow.**
**Answer:**
1. Check Pod status and restart count: `kubectl get pod <name> -o wide`.
2. Inspect lifecycle events and exit codes: `kubectl describe pod <name>`.
3. Check application logs: `kubectl logs <name>`.
4. If the container crashed instantly, check previous crash logs: `kubectl logs <name> --previous`.
5. Common Root Causes: Missing environment variables/secrets, database connection timeouts, wrong command/entrypoint syntax, unhandled exception in initialization code.

### **142. Scenario: A Pod is terminated with `Exit Code 137`. What happened and how do you fix it?**
**Answer:**
- **Reason:** The container was killed by `SIGKILL` ($128 + 9 = 137$) sent by the Linux kernel Out-Of-Memory (OOM) Killer because physical memory usage exceeded the cgroup `limits.memory` threshold.
- **Diagnostics:** Verify via `kubectl describe pod` (look for `Last State: Terminated, Reason: OOMKilled`).
- **Fix:** Increase container memory limits, fix Java JVM heap flags (`-XX:MaxRAMPercentage=75.0`), or profile application memory leaks using continuous profiling tools.

### **143. Scenario: A Pod is terminated with `Exit Code 143`. What happened?**
**Answer:** The container received a graceful `SIGTERM` signal ($128 + 15 = 143$) sent by Kubernetes during a rolling update, node drain, or HPA scale-down, and cleanly exited within the `terminationGracePeriodSeconds` window.

### **144. Scenario: A Pod is stuck in `ImagePullBackOff` / `ErrImagePull`. What are the common causes?**
**Answer:**
1. Incorrect image tag or typo in image repository URL.
2. Missing or misconfigured `imagePullSecrets` for authenticating to private registries.
3. Network timeout or firewall blocking egress from worker node to registry.
4. Hitting public Docker Hub rate limits (fixed via ECR Pull Through Cache).

### **145. Scenario: A Pod is stuck in `Pending` state. How do you identify the cause?**
**Answer:** Run `kubectl describe pod <name>` and check `Events`:
1. **Insufficient CPU / Memory:** No single worker node has enough unreserved capacity to satisfy the Pod's `requests`.
2. **NodeAffinity / Taints:** Pod lacks tolerations for tainted worker nodes.
3. **PVC Unbound:** PersistentVolumeClaim cannot bind to any available PV or StorageClass provisioner fails.
4. **NodeSelector Mismatch:** No nodes match the requested labels.

### **146. Scenario: A Pod is stuck in `Terminating` state for hours. How do you safely resolve it?**
**Answer:**
1. Check for blocking **Finalizers**: `kubectl get pod <name> -o jsonpath='{.metadata.finalizers}'`.
2. Check if a CSI volume unmount is hanging on a dead worker node.
3. If node is dead and volume is safe, remove finalizer or force delete: `kubectl delete pod <name> --grace-period=0 --force`.

### **147. Scenario: Explain the Linux `ndots:5` DNS resolution issue in Kubernetes and how it causes latency storms.**
**Answer:**
- By default, `/etc/resolv.conf` inside pods sets `options ndots:5`.
- If a query has fewer than 5 dots (e.g., `api.stripe.com` has 2 dots), the resolver sequentially appends all search domains:
  1. `api.stripe.com.default.svc.cluster.local` (NXDOMAIN)
  2. `api.stripe.com.svc.cluster.local` (NXDOMAIN)
  3. `api.stripe.com.cluster.local` (NXDOMAIN)
  4. `api.stripe.com` (SUCCESS)
- **Impact:** Generates 3 failed queries per external lookup, overwhelming CoreDNS.
- **Fix:** Deploy **NodeLocal DNSCache**, set `ndots:2` in pod `dnsConfig`, or append a trailing dot (`api.stripe.com.`).

### **148. Scenario: How do you achieve Zero-Downtime deployments for high-throughput HTTP services during rolling updates?**
**Answer:**
1. **Add `preStop` Sleep:** Ingress controllers take 1–3 seconds to update iptables endpoints. Add a `preStop` hook (`sleep 5`) so the old pod continues servicing in-flight requests while being removed from endpoints.
2. **Handle `SIGTERM` Gracefully:** Ensure the application process intercepts `SIGTERM` and drains active HTTP connections.
3. **Configure Accurate Readiness Probes:** Ensure readiness probe fails only after connection draining starts.
4. **Tune Deployment Strategy:** Set `maxSurge: 25%` and `maxUnavailable: 0`.

### **149. What is Istio Service Mesh and what are its core architectural components?**
**Answer:** An open-source service mesh providing traffic management, zero-trust security (mTLS), and observability:
- **`istiod` (Control Plane):** Translates declarative routing rules and distributes certificates to proxies.
- **Envoy Proxy (Data Plane):** High-performance sidecar proxy intercepting all inbound and outbound network traffic.

### **150. What is Istio Ambient Mesh?**
**Answer:** A sidecarless architecture splitting mesh processing into:
1. **`ztunnel` (Zero Trust Tunnel):** Per-node Layer 4 daemon handling mutual TLS encryption.
2. **Waypoint Proxies:** Optional per-namespace Envoy instances handling Layer 7 routing and authorization, reducing memory overhead by over 70%.

### **151. What is Linkerd and how does it compare to Istio?**
**Answer:** A lightweight, ultralow-latency service mesh written in Rust (data plane micro-proxy) and Go (control plane), designed for simplicity and minimal CPU/memory footprint compared to Envoy-based meshes.

### **152. What is Cilium Service Mesh?**
**Answer:** Sidecarless service mesh leveraging Linux kernel eBPF to manage Layer 7 routing, mTLS, and observability directly in the kernel, bypassing userspace proxy hops for Layer 4 traffic.

### **153. What is Cilium Hubble?**
**Answer:** An observability platform running on top of Cilium and eBPF providing real-time service dependency graphs, network flow logs, and HTTP/DNS latency metrics.

### **154. What is Cilium Cluster Mesh?**
**Answer:** Connects multiple independent Kubernetes clusters at the network layer, providing cross-cluster pod IP routing, global service discovery, and cross-cluster failover.

### **155. What is Kubernetes Control Plane High Availability?**
**Answer:** Running at least 3 stacked master nodes across independent Availability Zones, deploying a highly available `etcd` quorum, and placing an external Layer 4 Load Balancer (NLB) in front of the API Servers.

### **156. What is etcd Split-Brain and how is it prevented?**
**Answer:** When network partitions isolate etcd nodes into two groups that both elect leaders, corrupting data. Prevented by requiring an odd number of nodes (3 or 5) and enforcing strict majority quorum ($\lfloor N/2 \rfloor + 1$).

### **157. What is etcd Snapshot and Restore procedure?**
**Answer:**
- **Snapshot:** `etcdctl snapshot save /backup/etcd-snapshot.db`
- **Restore:** `etcdctl snapshot restore /backup/etcd-snapshot.db --data-dir=/var/lib/etcd-restored`

### **158. What is Kubernetes API Priority and Fairness (APF)?**
**Answer:** Protects the API Server from request floods by classifying incoming requests into priority levels and flow schemas, queuing and shedding non-essential requests to guarantee critical control loops continue running.

### **159. What is `kubelet` Certificate Rotation?**
**Answer:** Automatic renewal of client and server TLS certificates used by Kubelets to authenticate against the API Server before expiration, managed via CSRs.

### **160. What is a Custom Kubelet Configuration (`kubelet-config.yaml`)?**
**Answer:** A declarative configuration file managing system resource reservations (`--system-reserved`, `--kube-reserved`), container log sizes, and eviction thresholds.

### **161. What are System and Kube Resource Reservations?**
**Answer:**
- `system-reserved`: Reserves CPU and RAM for OS daemons (systemd, sshd, udev).
- `kube-reserved`: Reserves CPU and RAM for Kubernetes daemons (kubelet, containerd).
- *Without reservations, application pods can consume 100% of node RAM, crashing the kubelet.*

### **162. What is Kubernetes Pod Topology Spread Constraint vs Pod Anti-Affinity?**
**Answer:** Anti-Affinity is binary (schedule or do not schedule on the same node); Topology Spread Constraints allow configuring proportional distribution (`maxSkew: 1`) across zones.

### **163. What is a Volume Snapshot (CSI Snapshotter)?**
**Answer:** A Kubernetes CRD (`VolumeSnapshot`) that triggers point-in-time storage array snapshots of persistent volumes via the underlying CSI driver.

### **164. What is a Volume Expansion in Kubernetes?**
**Answer:** Increasing the capacity of an existing PersistentVolumeClaim dynamically (`spec.resources.requests.storage: 100Gi`) if the StorageClass has `allowVolumeExpansion: true`.

### **165. What is Generic Ephemeral Volumes?**
**Answer:** Ephemeral storage backed by dynamic volume provisioners that create dedicated scratch disks for pods and delete them automatically when the pod terminates.

### **166. What is Kubernetes Secret Encryption at Rest?**
**Answer:** Configuring an `EncryptionConfiguration` file on the API Server using a KMS provider (AWS KMS, HashiCorp Vault) to encrypt Secret data before writing to etcd.

### **167. What is Kubernetes Audit Logging?**
**Answer:** Chronological security records capturing every request made to the API Server (who requested what, when, from which IP, and whether it was authorized).

### **168. What is `kube-bench`?**
**Answer:** An open-source tool that checks whether a Kubernetes cluster is configured securely according to the Center for Internet Security (CIS) Kubernetes Benchmark.

### **169. What is `kube-hunter`?**
**Answer:** An open-source penetration testing tool that hunts for security vulnerabilities and exposed ports in live Kubernetes clusters.

### **170. What is a Container Escape via `hostPath` Volume Mount?**
**Answer:** Mounting the host root filesystem (`hostPath: /`) into a container allows an attacker to modify `/etc/shadow`, install cron backdoors on the host, or escape namespaces.

### **171. What is a Container Escape via Docker Socket Mount?**
**Answer:** Mounting `/var/run/docker.sock` allows code inside the container to command the host Docker daemon to spawn privileged sibling containers with host root access.

### **172. What is Kubernetes Workload Identity Federation (AWS IRSA / GCP Workload Identity / Azure Pod Identity)?**
**Answer:** Projecting OIDC-signed ServiceAccount tokens into Pods, allowing them to exchange tokens with cloud STS for short-lived IAM credentials without static access keys.

### **173. What is AWS EKS Pod Identity (New Standard)?**
**Answer:** AWS EKS native agent-based identity mapping that binds Kubernetes ServiceAccounts directly to AWS IAM roles without needing OIDC provider trust configurations.

### **174. What is Crossplane in Kubernetes?**
**Answer:** A framework that turns Kubernetes into a universal control plane, allowing platform teams to manage cloud infrastructure (S3 buckets, RDS databases) using native Kubernetes CRDs.

### **175. What is `vcluster` (Virtual Cluster)?**
**Answer:** Running fully isolated virtual Kubernetes control planes inside namespaces of an underlying host cluster, providing multi-tenancy with separate API servers and CRD support.

### **176. What is Kubernetes Cluster API (CAPI)?**
**Answer:** A Kubernetes sub-project that uses declarative APIs to automate provisioning, upgrading, and operating multiple Kubernetes clusters across AWS, Azure, GCP, and vSphere.

### **177. What is CoreDNS Plugin Architecture?**
**Answer:** Modular plugins (e.g., `kubernetes`, `forward`, `cache`, `errors`, `health`) configured in the `Corefile` that determine how DNS requests are processed.

### **178. What is CoreDNS Autoscaling?**
**Answer:** Dynamically scaling CoreDNS pod replicas using `cluster-proportional-autoscaler` based on the total number of worker nodes and cores in the cluster.

### **179. What is Kubernetes Ingress Path Types (`Exact` vs `Prefix` vs `ImplementationSpecific`)?**
**Answer:**
- `Exact`: Matches URL path exactly case-sensitively (`/api/v1`).
- `Prefix`: Matches URL paths sharing a prefix (`/api` matches `/api/users` and `/api/orders`).

### **180. What is Envoy Proxy Filter Chain?**
**Answer:** Sequential processing stages (TLS Inspector, HTTP Connection Manager, Rate Limit, RBAC, Router) executed for each network connection in Envoy.

### **181. What is Open Policy Agent (OPA) Gatekeeper Mutation Webhook?**
**Answer:** Automatically injecting required metadata (labels, annotations, resource requests) into Kubernetes manifests upon submission.

### **182. What is Kyverno Auto-Generation of Pod Policies?**
**Answer:** Kyverno rules targeting Pods automatically generate identical validation rules for higher-level controllers (Deployments, StatefulSets, DaemonSets).

### **183. What is Telepresence for Kubernetes Development?**
**Answer:** A tool that connects a local workstation development process directly into a remote Kubernetes cluster network, proxying cluster traffic to local code.

### **184. What is Skaffold?**
**Answer:** A Google CLI tool that automates the continuous local build, push, and deployment loop to local (Minikube/Kind) or remote Kubernetes clusters.

### **185. What is Tilt for Kubernetes?**
**Answer:** A local development environment tool that monitors file changes and executes fast multi-container sync updates directly to running pods.

### **186. What is Kind (Kubernetes in Docker)?**
**Answer:** A tool for running local multi-node Kubernetes clusters where each node is a Docker container, heavily used in CI testing.

### **187. What is K3s?**
**Answer:** A lightweight, fully compliant certified Kubernetes distribution packaged as a single $< 100\text{MB}$ binary, replacing etcd with SQLite for edge and IoT computing.

### **188. What is MicroK8s?**
**Answer:** A zero-ops, lightweight upstream Kubernetes distribution packaged as a snap for developer workstations and edge devices.

### **189. What is Minikube?**
**Answer:** A local single-node Kubernetes tool designed for learning and local testing running inside a VM or container.

### **190. What is Kubernetes API Deprecation Lifecycle?**
**Answer:** APIs are transitioned from `v1alpha1` $\rightarrow$ `v1beta1` $\rightarrow$ `v1`, with deprecation warnings emitted in API responses before final removal in subsequent releases.

### **191. What is `krew` in Kubernetes?**
**Answer:** The official plugin manager for `kubectl` allowing engineers to discover and install community extensions (`kubectl ctx`, `kubectl ns`, `kubectl neat`).

### **192. What is `k9s`?**
**Answer:** A terminal-based UI that provides a real-time dashboard to monitor, manage, and debug Kubernetes cluster resources.

### **193. What is `kubecolor`?**
**Answer:** A wrapper for `kubectl` that colorizes CLI terminal outputs for pods, services, and events to improve readability.

### **194. What is Karpenter Node Expiration (`ttlSecondsUntilExpired`)?**
**Answer:** Automatically terminating and recycling worker nodes after a defined lifespan (e.g., 30 days) to enforce mandatory OS security patching.

### **195. What is Karpenter Disruption Budgets?**
**Answer:** Policies that limit how many nodes Karpenter can terminate or consolidate simultaneously during voluntary cluster optimization.

### **196. What is Kubernetes Dual-Stack Networking?**
**Answer:** Allocating both IPv4 and IPv6 addresses to Pods and Services across the entire cluster.

### **197. What is MetalLB?**
**Answer:** A bare-metal load-balancer implementation for Kubernetes clusters running on-premise without cloud provider LoadBalancer integration.

### **198. What is Cilium BGP Control Plane?**
**Answer:** Allowing Cilium to advertise Pod and Service CIDR IP blocks directly to physical data center Top-of-Rack (ToR) BGP routers.

### **199. What is Kubernetes CSI Volume Cloning?**
**Answer:** Creating a new PersistentVolume populated with the exact data duplicate of an existing PVC on the same storage array.

### **200. What is Kubernetes CSI Ephemeral Inline Volumes?**
**Answer:** Attaching ephemeral storage directly inside the `spec.volumes` block without needing standalone PVC manifests.

### **201. Scenario: You deploy a new version of an app and CPU usage spikes to 100%, causing pods to crash. How do you triage?**
**Answer:**
1. Check HPA and deployment events: `kubectl describe hpa <name>`.
2. Grab CPU profile or thread dump from a running pod: `kubectl exec -it <pod> -- jstack 1` or use continuous profiling (Pyroscope).
3. Roll back deployment immediately: `kubectl rollout undo deployment/<name>`.
4. Investigate root cause: Infinite loops, un-indexed database queries holding locks, or regex backtracking.

### **202. Scenario: CoreDNS pods are crashing with `OOMKilled`. How do you recover and permanently fix?**
**Answer:**
1. Scale up CoreDNS memory limits in `kube-dns` deployment.
2. Enable **NodeLocal DNSCache** DaemonSet to absorb 80% of DNS queries on worker nodes.
3. Deploy `cluster-proportional-autoscaler` to scale CoreDNS replicas dynamically with cluster node count.
4. Fix `ndots:5` configuration in application PodSpecs.

### **203. Scenario: Worker nodes are intermittently running out of disk space on `/var/lib/containerd`. How do you resolve it?**
**Answer:**
1. Clean up unused images immediately: `crictl rmi --prune`.
2. Configure image garbage collection thresholds in `kubelet-config.yaml`:
   ```yaml
   imageGCHighThresholdPercent: 80
   imageGCLowThresholdPercent: 65
   ```
3. Set container log rotation limits: `containerLogMaxSize: 50Mi` and `containerLogMaxFiles: 3`.

### **204. Scenario: A developer accidentally deletes a production Namespace. How does Kubernetes behave and how do you prevent this?**
**Answer:**
- **Behavior:** Kubernetes deletes *all* resources within the namespace (Pods, Services, PVCs, Secrets) via cascading deletion.
- **Prevention:**
  1. Deny `delete namespaces` via RBAC.
  2. Enforce Kyverno / OPA Gatekeeper validating admission webhooks blocking namespace deletion for production namespaces.
  3. Set `deletionProtection: true` on critical cloud resources via Crossplane/Terraform.

### **205. Scenario: Inter-pod communication fails between nodes, but works between pods on the same node. What is the root cause?**
**Answer:**
1. **Security Group / Firewall:** Cloud Security Group is blocking cross-node encapsulation traffic (Geneve/VXLAN port 6081 or 8472).
2. **CNI Node Routing:** CNI overlay routing table on the host is misconfigured or missing route entries to the peer node's Pod CIDR.
3. **MTU Mismatch:** Overlay network MTU (1450 bytes) exceeds host network MTU (1500 bytes), dropping fragmented packets.

### **206. Scenario: Kubernetes API Server responds with HTTP 429 Too Many Requests. How do you triage?**
**Answer:**
1. Check API Priority and Fairness metrics in Prometheus (`apiserver_flow_control_rejected_requests_total`).
2. Identify misbehaving microservices or buggy controllers polling the API in tight loops without informers/watchers.
3. Scale up API Server replicas and tune `max-requests-inflight` settings.

### **207. Scenario: What happens if `etcd` disk write latency exceeds 10ms?**
**Answer:** etcd leader heartbeats time out, triggering frequent leader elections, dropping consensus, and causing API Server requests to hang and Kubelets to fail status updates. Requires dedicated high-IOPS NVMe storage.

### **208. Scenario: How do you migrate 1,000 workloads from legacy Ingress to Kubernetes Gateway API with zero downtime?**
**Answer:**
1. Deploy Gateway API CRDs and Gateway Controller (Envoy Gateway / Cilium).
2. Deploy `Gateway` instance sharing the same external Load Balancer IP or DNS.
3. Incrementally deploy `HTTPRoute` resources alongside existing `Ingress` resources.
4. Shift DNS traffic gradually using Route 53 weighted records.
5. Decommission Ingress resources once 100% traffic is verified on Gateway API.

### **209. Scenario: How do you troubleshoot a pod whose logs show nothing and process terminates immediately?**
**Answer:**
1. Run with interactive override: `kubectl run debug --image=<image> -it -- /bin/sh`.
2. Inspect exit code: `kubectl describe pod <name>` (look for Exit Code 127 = missing binary, or Exit Code 126 = permission denied).
3. Verify library dependencies (`ldd binary_name`).

### **210. Scenario: How do you rotate all Kubernetes cluster TLS certificates safely?**
**Answer:**
1. On control plane nodes: `kubeadm certs check-expiration`.
2. Renew certificates: `kubeadm certs renew all`.
3. Restart control plane static pods (`kube-apiserver`, `kube-controller-manager`, `kube-scheduler`).
4. Update `~/.kube/config` with renewed client credentials.

### **211. What is the Kubernetes Node Problem Detector (NPD)?**
**Answer:** A DaemonSet that monitors node health issues (kernel deadlocks, corrupted filesystems, hardware errors) and reports them as Node Conditions to the API Server.

### **212. What is Kube-Janitor?**
**Answer:** A cleanup controller that automatically deletes expired, orphaned, or non-production Kubernetes resources based on TTL annotations (`janitor/ttl: 7d`).

### **213. What is Popeye for Kubernetes?**
**Answer:** A cluster sanitizer CLI tool that scans live clusters and reports misconfigurations, dead resources, and over-allocated requests.

### **214. What is Reloader for Kubernetes?**
**Answer:** A controller that automatically watches ConfigMaps and Secrets and triggers rolling restarts on dependent Deployments when secret values change.

### **215. What is Goldilocks in Kubernetes?**
**Answer:** A utility that uses VPA in recommendation mode to generate optimal CPU and memory resource request/limit baselines.

### **216. What is Kubernetes Descheduler?**
**Answer:** A cluster optimizer that evicts running Pods that no longer satisfy scheduling criteria (e.g., node underutilization, broken affinity rules, high pod count) so the scheduler can balance them.

### **217. What is Kubernetes Chaos Mesh?**
**Answer:** A cloud-native chaos engineering platform for Kubernetes that injects faults (killing pods, adding network latency, corrupting I/O) via CRDs.

### **218. What is Kube-State-Metrics (KSM)?**
**Answer:** An add-on that listens to the Kubernetes API Server and generates Prometheus metrics about the health and state of objects (deployment replicas, pod phases, resource limits).

### **219. What is KubeClustered vs Host-Network Mode?**
**Answer:** Standard pods get isolated private IPs from the CNI; Host-Network pods (`hostNetwork: true`) bind directly to the host OS network interface, bypassing CNI virtualization.

### **220. What is Kubernetes Container Runtime Class (`RuntimeClass`)?**
**Answer:** A feature allowing different Pods in the same cluster to use different container runtimes (e.g., standard pods use `runc`, untrusted pods use `gVisor` / `kata`).

### **221. What is Sysctl in Kubernetes Pods?**
**Answer:** Configuring kernel parameters per pod namespace (e.g., `net.core.somaxconn=1024`) via `spec.securityContext.sysctls`.

### **222. What is Kubernetes Volume Binding Mode (`Immediate` vs `WaitForFirstConsumer`)?**
**Answer:**
- `Immediate`: PV is provisioned immediately upon PVC creation.
- `WaitForFirstConsumer`: PV is provisioned *only* after a Pod using the PVC is scheduled, ensuring the storage volume is created in the exact Availability Zone where the Pod lands.

### **223. What is Kubernetes Storage Topology Awareness?**
**Answer:** Ensuring CSI storage drivers allocate block storage volumes in the exact cloud Availability Zone matching the scheduled worker node.

### **224. What is Kubernetes Ephemeral Storage Requests/Limits?**
**Answer:** Setting requests and limits for container writable layers and emptyDir volumes (`ephemeral-storage: 2Gi`) to prevent pods from filling node root disks.

### **225. What is Kubernetes Secret Store CSI Driver?**
**Answer:** A CSI driver that mounts secrets from AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault directly into pod filesystems as in-memory files without storing them in etcd.

### **226. What is Kubernetes Custom Resource Conversion Webhook?**
**Answer:** A webhook that converts Custom Resources between different schema versions (e.g., `v1alpha1` $\rightarrow$ `v1`) on-the-fly during API reads/writes.

### **227. What is Dynamic Client vs Typed Client in Kubernetes Go SDK?**
**Answer:**
- **Typed Client (`clientset`):** Works with compiled static Kubernetes Go structs.
- **Dynamic Client:** Works with unstructured JSON objects (`unstructured.Unstructured`), essential for managing arbitrary CRDs.

### **228. What is Informer / SharedInformer in Kubernetes Controllers?**
**Answer:** A local in-memory cache of API Server objects that watches etcd events and triggers event handler callbacks (`OnAdd`, `OnUpdate`, `OnDelete`) without polling the API Server.

### **229. What is Kubernetes Workqueue in Controller development?**
**Answer:** A thread-safe, rate-limiting queue that buffers reconcile requests and deduplicates multiple updates for the same object key.

### **230. What is a Reconcile Loop in Kubernetes?**
**Answer:** An idempotent function: `Reconcile(Request) -> (Result, error)` that compares current cluster state against desired state in Git/CRD and executes CRUD operations to bring them into alignment.

### **231. What is Optimistic Concurrency Control (`resourceVersion`)?**
**Answer:** Every object in etcd has a `resourceVersion`. When updating an object, the API Server rejects writes if the client's `resourceVersion` does not match etcd's current version (HTTP 409 Conflict).

### **232. What is Kubernetes Eventual Consistency in Control Loops?**
**Answer:** Controllers do not expect immediate state synchronization; they continuously retry failed operations until the system reaches desired state.

### **233. What is Kubernetes Subresource (`/status`, `/scale`)?**
**Answer:** Dedicated API endpoints for updating specific parts of an object (e.g., updating `/status` does not trigger spec mutating admission webhooks).

### **234. What is Kubernetes OwnerReferences and BlockOwnerDeletion?**
**Answer:** Metadata pointing a child object to its parent controller, ensuring garbage collection deletes child pods when parent deployments are deleted.

### **235. What is a Headless Service SRV Record?**
**Answer:** CoreDNS returns `SRV` records containing port numbers and hostnames for each named port of backing pods in a Headless Service.

### **236. What is Kubernetes Service Session Affinity (`ClientIP`)?**
**Answer:** Directing requests from the same client IP to the same backing pod replica for a configurable timeout duration (`sessionAffinityConfig.clientIP.timeoutSeconds`).

### **237. What is ExternalTrafficPolicy (`Cluster` vs `Local`)?**
**Answer:**
- `Cluster` (Default): Node receiving traffic SNATs packet and forwards to any node running backing pods (extra network hop, obscures client IP).
- `Local`: Node routes traffic *only* to local pods on the same node (preserves client source IP, zero extra hop, requires health check node ports).

### **238. What is Kubernetes Traffic Distribution (`PreferClose`)?**
**Answer:** Routing Service traffic to endpoints that are topologically closest to the caller (same node $\rightarrow$ same zone $\rightarrow$ same region), minimizing cross-AZ data transfer costs.

### **239. What is Kubernetes IPAM (IP Address Management)?**
**Answer:** The subsystem in CNI plugins responsible for allocating and tracking available IP address blocks for nodes and pods.

### **240. What is AWS VPC CNI Prefix Delegation?**
**Answer:** Allocating IPv4 `/28` subnets (16 IPs) to each ENI instead of individual secondary IPs, increasing the maximum number of pods per EC2 instance by over 4x.

### **241. What is Azure CNI vs Kubenet?**
**Answer:**
- **Azure CNI:** Every pod gets a real IP from the Azure VNet (fast, but consumes large VNet CIDR space).
- **Kubenet:** Pods get private IPs behind NAT; routes configured in Azure route tables.

### **242. What is Calico CNI eBPF Data Plane?**
**Answer:** Replaces standard Linux iptables routing with high-performance eBPF programs, providing native WireGuard encryption and scalable NetworkPolicies.

### **243. What is Cilium Transparent Encryption (WireGuard)?**
**Answer:** Automatically encrypting all node-to-node and pod-to-pod network traffic using kernel WireGuard with zero sidecar proxy overhead.

### **244. What is Kubernetes Node Autoscaling Overprovisioning?**
**Answer:** Running dummy pause pods with low `PriorityClass` that hold reserve compute capacity. When real pods schedule, they preempt the pause pods, launching immediately while nodes scale in the background.

### **245. What is Kubernetes Mixed Instance Policy in Karpenter?**
**Answer:** Configuring Karpenter to dynamically mix diverse EC2 instance types, sizes, and generations across Spot and On-Demand pools to maximize availability and minimize cost.

### **246. What is Kubernetes Multi-Cluster Service Discovery (MCS API)?**
**Answer:** A standardized Kubernetes API (`ServiceExport`, `ServiceImport`) allowing services in Cluster A to discover and communicate with services in Cluster B using `my-svc.my-ns.svc.clusterset.local`.

### **247. What is Submariner for Multi-Cluster Networking?**
**Answer:** An open-source tool creating direct IPsec/WireGuard VPN tunnels between pods in disparate Kubernetes clusters across clouds and data centers.

### **248. What is Kyverno Image Signature Verification with Sigstore?**
**Answer:** An admission policy rule that verifies container image cryptographic signatures against Rekor transparency logs before allowing pods to schedule.

### **249. What is Falco eBPF System Call Monitoring?**
**Answer:** A CNCF security tool that intercepts kernel system calls (`execve`, `openat`, `socket`) to detect anomalous runtime behavior (shell spawning, privilege escalation) inside containers.

### **250. What is Kubernetes Hard Multi-Tenancy Architecture?**
**Answer:** Combining **`vcluster`** (isolated control planes), **Cilium NetworkPolicies** (isolated L3-L7 networks), **gVisor / Kata Containers** (sandboxed kernel runtimes), and **ResourceQuotas** to run untrusted multi-tenant workloads safely on a shared physical cluster.