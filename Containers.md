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
- This api-server interacts directly with the user (i.e we apply `.yml` or `.json` manifest to kube-api-server)
- This `kube-api-server` is meant to scale automatically as per load.
- `Kube-api-server` is the front end of the control plane.

**etcd**
- Stores metadata and status of the cluster.
- `etcd` is a consistent and high-available store (key-value-store)
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
  - If K8s is on the cloud, then it will be a cloud controller manager.
  - If K8s is on non-cloud, then it will be kube-controller-manager.


### **36. What are the core components of a Kubernetes Worker Node?**
**Answer:**
1. **`kubelet`:** 
- The agent running on the node.
- Listens to Kubernetes master (eg- Pod creation request).
- Use port 10255.
- Send success/Fail reports to master.

2. **`kube-proxy`:** 
- Assign IP to each pod.
- It is required to assign IP addresses to Pods (dynamic)
- Kube-proxy runs on each node & this makes sure that each pod will
get its unique IP Address.
- These 3 components collectively consist of ‘node’.

3. **Container Runtime (CRI):** 
- Works with kubelet
- Pulling images
- Start/Stop Containers
- Exposing containers on ports specified in the manifest.

### **37. What is a Kubernetes Pod?**
**Answer:** All containers in Kubernetes are contained within Pods. The simplest component of the Kubernetes architecture is a pod. You may think of them as your container's equivalent of a wrapper. Each pod receives a unique IP address that it can use to communicate with other pods in the cluster.

Pods typically only contain one container, however, if several containers are required to share resources, they may do so. A Pod's multiple containers can connect with one another over localhost if there are multiple containers in it.

Since Kubernetes pods are "mortal," they can be generated and deleted as needed by the application. For instance, the backend of a web application may experience a spike in CPU utilization. Eight new Pods would be formed if the cluster scaled up the number of backend Pods from two to ten as a result of this circumstance. The Pods may scale back to two after the traffic dies down, in which case eight pods would be destroyed.


### **38. What is the Pause Container (`infra` container)?**
**Answer:** A **Pause Container** (also known as an **infra container** or **sandbox container**) is a tiny, hidden helper container that Kubernetes automatically starts first in every Pod.
It does not run any application code. Instead, it acts as the parent sandbox that holds and manages shared Linux namespaces (such as Network, IPC, and optionally PID) for all other application containers inside the same Pod.

### **39. What is a Kubernetes Deployment vs ReplicaSet vs Pod?**
**Answer:**
- **Pod:** A Pod represents a single instance of your application. It can contain one or more containers that work together, sharing the same network and storage. Most commonly, you’ll have one container per Pod.
- **ReplicaSet:** A ReplicaSet solves the reliability problem by maintaining a specified number of Pod replicas at all times. If you want 3 instances of your web server running, a ReplicaSet will ensure exactly 3 Pods exist, creating new ones if any fail.
- **Deployment:** Deployments sit above ReplicaSets and provide the features you need for production applications: rolling updates, rollbacks, and scaling. When you create a Deployment, it automatically creates and manages a ReplicaSet for you.

**In practice, you’ll typically work with Deployments and let Kubernetes handle the underlying ReplicaSets and Pods automatically.**

### **40. What is a StatefulSet and how does it differ from a Deployment?**
**Answer:** A **StatefulSet** is a Kubernetes workload API object used to manage stateful applications (such as databases or distributed message queues) that require persistent storage and unique, stable network identities.

**Key Differences: StatefulSet vs. Deployment**

| **Feature**                   | **Deployment**                                          | **StatefulSet**                                                                                      |
| ----------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Primary Use Case**          | Stateless applications (e.g., web servers, REST APIs)   | Stateful applications (e.g., MySQL, Kafka, Redis)                                                    |
| **Pod Identity**              | Anonymous and interchangeable (random hash suffixes)    | Unique, sticky, and predictable (ordinal indexes like `web-0`, `web-1`)                              |
| **Network Identity**          | Shares a common service endpoint; no unique per-pod DNS | Uses Headless Services to give each pod a unique DNS hostname (`$(pod-name).$(service-name)`)        |
| **Storage**                   | Typically ephemeral or shared across pods               | Dedicated **PersistentVolumeClaim (PVC)** mapped individually to each pod via `volumeClaimTemplates` |
| **Scaling & Lifecycle Order** | Created, scaled, and deleted in parallel (any order)    | Created, scaled, and deleted strictly **sequentially** in order of their ordinal index               |

### **41. What is a DaemonSet?**
**Answer:** A Kubernetes DaemonSet is a resource that ensures a specific pod runs on every node in a Kubernetes cluster or on a subset of nodes that meet certain conditions.

DaemonSets are mainly used for background services that must be present on each node, such as:
- Log collectors
- Monitoring agents
- Security or networking components

### **42. What is a Kubernetes Job vs CronJob?**
**Answer:**
#### **Kubernetes Job:** 
A Kubernetes Job is used to create one or more pods that perform a specific task and then terminate. Jobs are ideal for tasks that need to be run to completion, such as batch processing, data migration, or any task that needs to be executed once.

Key Features of Jobs:
- **Task Completion:** A Job ensures that a specified number of pods successfully terminate.
- **Pod Restart Policy:** If a pod fails, the Job controller can create new pods to replace the failed ones until the task is completed.

#### **Kubernetes CronJob:**
 A Kubernetes CronJob is a specialized type of Job that runs on a schedule. CronJobs are useful for tasks that need to be performed periodically, such as backups, report generation, or system maintenance.

Key Features of CronJobs:
- **Scheduled Execution:** CronJobs run at specified times, defined using Cron syntax.
- **Time-based Triggers:** You can schedule tasks to run daily, weekly, monthly, etc.

### **43. Explain Kubernetes Service Types (ClusterIP, NodePort, LoadBalancer, ExternalName).**
**Answer:**
#### 1. **ClusterIP (Default)**
- **Purpose:** Exposes the service within the cluster.
- **Access:** Internal (cluster-internal IP only).
- **Use Case:** For internal communication between pods (e.g., microservices communicating with each other).

#### 2. **NodePort**
- **Purpose:** Exposes the service on a static port on each node’s IP.
- **Access:** External (through NodeIP:NodePort).
- **Use Case:** Quick and simple way to expose services externally, typically for development or testing.

#### 3. **LoadBalancer**
- **Purpose:** Exposes the service externally using a cloud provider’s load balancer.
- **Access:** External (public IP via cloud provider).
- **Use Case:** Suitable for production workloads, where a single IP is required for external clients to access services.

#### 4. **ExternalName**
- **Purpose:** Maps a service to an external DNS name.
- **Access:** External (resolves DNS to an external address).
- **Use Case:** Used for redirecting traffic to an external service, e.g., a managed database or API outside the Kubernetes cluster.

### **44. What is Kubernetes Ingress vs Ingress Controller?**
**Answer:**
- **Ingress:** An Ingress is a Kubernetes resource that allows traffic to come into your Kubernetes cluster. The external traffic could be via HTTP or HTTPS to a service running within your Kubernetes cluster. The service is exposed by your Ingress to allow inbound traffic.

  - Ingress provides externally available URLs, performs load balancing, terminates SSL/TLS, and offers name-based virtual hosting.

- **Ingress Controller:** An Ingress controller is a daemon running in a Pod that watches the /ingresses endpoint on the API server. When a new endpoint is created, the daemon uses the configured set of rules to allow traffic into a service.

  - A controller uses Ingress Rules to handle traffic to and from outside the cluster.

### **45. Explain Kubernetes Namespaces and their isolation boundaries.**
**Answer:** Namespaces in Kubernetes provide a way to logically divide cluster resources into virtual clusters. They offer a scope for Kubernetes objects, allowing multiple virtual clusters to coexist within the same physical cluster. This segregation helps in organizing and managing applications and resources efficiently.

#### Purpose and Functionality:
- **Isolation:** Namespaces enable teams to create isolated environments within a Kubernetes cluster. Each Namespace acts as a virtual cluster, providing a segregated space for deploying and managing resources.
- **Resource Quotas:** By assigning resource quotas to Namespaces, administrators can control and limit the amount of CPU, memory, and other resources consumed by the workloads within that Namespace.
- **Access Control:** Namespaces facilitate role-based access control (RBAC), allowing administrators to define fine-grained access policies for different teams or users. This ensures that each team has access only to the resources within their designated Namespace.
- **Scoping:** Kubernetes objects such as pods, services, deployments, and replica sets are scoped to a Namespace. This means that objects within a Namespace are uniquely identified by their names, allowing for better organization and management.
- **Environment Segregation:** Namespaces can be used to segregate environments such as development, staging, and production within the same Kubernetes cluster. This helps in maintaining consistency and isolation between different stages of the application lifecycle.

### **46. What is a ConfigMap vs a Secret?**
**Answer:**
- **ConfigMap:** Stores non-confidential configuration data in key-value pairs mounted as environment variables or volume files.
- **Secret:** Stores sensitive data (passwords, tokens, keys) encoded in base64. Stored encrypted at rest in `etcd` when KMS encryption is enabled.

**Key Differences at a Glance**

| **Feature**               | **ConfigMap**                                                | **Secret**                                                                                                     |
| ------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Data Type**             | Non-sensitive data (e.g., port numbers, feature flags, URLs) | Sensitive data (e.g., passwords, API keys, private keys, TLS certs)                                            |
| **Storage Format**        | Stored in plain text                                         | Stored with Base64 encoding (not true encryption on its own, though encryption at rest can be enabled in etcd) |
| **Access Control (RBAC)** | Can be broadly shared via standard role permissions          | Restricted by default; treated with higher sensitivity in RBAC rules to prevent privilege escalation           |
| **Size Limit**            | Maximum of 1 MiB                                             | Maximum of 1 MiB                                                                                               |
| **Injection Methods**     | Environment variables or mounted volume files                | Environment variables or mounted volume files                                                                  |

### **47. What is a PersistentVolume (PV) vs PersistentVolumeClaim (PVC)?**
**Answer:**
- **Persistent Volumes:** In Kubernetes, **Persistent Volumes (PVs)** are storage resources provisioned by administrators to provide persistent storage for applications running in pods. These volumes exist independently of the pod lifecycle and can be dynamically provisioned or statically defined.
- **Persistent Volume Claims:** **Persistent Volume Claims (PVCs)**, on the other hand, are requests made by applications for storage resources. PVCs abstract away the details of the underlying storage implementation, allowing developers to request storage based on their requirements without needing to know the specifics of how it’s provisioned.


### **48. What is Dynamic Volume Provisioning (StorageClass)?**
**Answer:** Dynamic volume provisioning in Kubernetes enables on-the-fly creation of storage volumes without manual intervention. When a Pod requests storage, Kubernetes provisions a volume dynamically using the underlying storage infrastructure, such as AWS EBS, GCP Persistent Disk, or local storage.

### **49. Explain Kubernetes Storage Access Modes.**
**Answer:**
**Kubernetes Storage Access Modes** define the read and write patterns allowed when mounting a `PersistentVolume` (PV) through a `PersistentVolumeClaim` (PVC). They act as a contract between your workload and the underlying storage provider, determining how many nodes or pods can concurrently access a volume.

Kubernetes supports **four main access modes**:

**1. ReadWriteOnce (RWO)**

- **Definition:** The volume can be mounted with read and write permissions, but by **only a single cluster node** at a time. 
- **Behavior:** Multiple pods running on that *same* node can read from and write to the volume simultaneously. However, pods scheduled on different nodes cannot attach to it. 
- **Typical Use Case:** Block storage systems (e.g., AWS EBS, Google Cloud Persistent Disk) and production databases or queues that require a single writer to avoid split-brain data corruption.

**2. ReadWriteOncePod (RWOP)**

- **Definition:** The volume can be mounted with read and write permissions by **exactly one pod cluster-wide**.
- **Behavior:** Unlike RWO (which restricts at the node level), RWOP strictly enforces single-writer access at the pod level. Even if another pod is on the same node, it will be barred from attaching to this volume. 
- **Typical Use Case:** High-safety stateful workloads where data protection demands an absolute single-writer layout, protecting against concurrent writes during overlapping pod reschedules or rolling updates.

**3. ReadOnlyMany (ROX)**

- **Definition:** The volume can be mounted as **read-only simultaneously by many nodes**. 
- **Behavior:** Multiple pods across different nodes can read the data at the same time, but no workload can modify or write data back to the volume. 
- **Typical Use Case:** Distributing static assets, configuration assets, machine learning models, or application binaries to scaling microservices.

**4. ReadWriteMany (RWX)**

- **Definition:** The volume can be mounted as **read-write concurrently by many nodes**.
- **Behavior:** Pods on any node across the cluster can read and write to the exact same storage asset at the same time.
- **Typical Use Case:** Shared file storage architectures, content management systems (like WordPress sharing a media library directory), or big data processing workflows. 

### **50. Explain Kubernetes RBAC: Role vs ClusterRole, RoleBinding vs ClusterRoleBinding.**
**Answer:**
**Kubernetes RBAC (Role-Based Access Control)** regulates access to resources based on four primary objects. Role and ClusterRole define what actions can be done, while RoleBinding and ClusterRoleBinding define who can do them

**1. Defining Permissions: Role vs. ClusterRole**

Both objects define a set of additive rules (there are no "deny" rules in Kubernetes RBAC) specifying which **verbs** (e.g., `get`, `list`, `create`) can be performed on which **resources** (e.g., `pods`, `services`). The primary difference is their **scope**:

| **Feature**          | **Role**                                                                                                                     | **ClusterRole**                                                                                                                                |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scope**            | **Namespace-scoped**. Must be created inside a specific namespace.                                                           | **Cluster-scoped**. Lives outside individual namespaces.                                                                                       |
| **Target Resources** | Only targets resources within its designated namespace (e.g., Pods, Deployments).                                            | Can target namespaced resources, **cluster-scoped resources** (e.g., Nodes, PersistentVolumes), and non-resource endpoints (e.g., `/healthz`). |
| **Primary Use Case** | Granting team or app permissions inside an isolated environment (e.g., allowing a developer to view pods in `frontend-dev`). | Defining broad system roles (e.g., cluster admins, node viewers) or creating reusable permission templates.                                    |

**2. Granting Permissions: RoleBinding vs. ClusterRoleBinding**

A role on its own grants no access. A **binding** maps the role's permissions to **Subjects** (Users, Groups, or ServiceAccounts).

```text
[ Subject ] ────────► [ Binding Object ] ────────► [ Role Object ]
(User/Group/SA)       (RoleBinding or CRBinding)      (Role or ClusterRole)
```

**RoleBinding**

- **Scope:** Tied to a specific namespace.
- **Behavior:** It grants the permissions defined in a role *only within the namespace where the RoleBinding is created*. 
- **Flexibility:** A `RoleBinding` can reference a local `Role`, **or it can reference a `ClusterRole`**.
  - *Tip:* Referencing a `ClusterRole` via a `RoleBinding` is a best practice for reusability. It applies the cluster-wide template strictly within the limits of that single namespace. 

**ClusterRoleBinding**

- **Scope:** Cluster-wide.
- **Behavior:** It lifts all namespace boundaries. It grants permissions across **all namespaces** in the cluster, as well as to any cluster-scoped resources defined in the role.
- **Flexibility:** A `ClusterRoleBinding` **can only reference a `ClusterRole`**. It cannot reference a standard `Role`. 

### **51. Explain the difference between Liveness, Readiness, and Startup Probes.**
**Answer:**
**1. Startup Probe**

- **Purpose:** Checks if the application inside the container has successfully started up. 
- **Behavior:** When configured, it disables **Liveness** and **Readiness** probes until it passes. This gives slow applications ample time to load heavy configurations, warm up caches, or run database migrations without being killed prematurely.
- **Failure Handling:** If it fails to pass within its designated timeframe, the `kubelet` kills the container and triggers its restart policy. Once it succeeds **once**, it shuts off for the lifespan of the container, and the other probes take over. 

**2. Liveness Probe**

- **Purpose:** Determines if the container is currently alive and functioning. 
- **Behavior:** It runs periodically throughout the entire lifecycle of the container. It is designed to catch scenarios where the application is technically "running" (the process hasn't crashed) but is completely unresponsive or deadlocked.
- **Failure Handling:** If it fails, Kubernetes assumes the application is unrecoverable, **destroys the container, and spins up a new one**. 
- *Best Practice Note:* Never point a liveness probe to external dependencies (like an external database). If the database goes down, your liveness probe will fail, causing Kubernetes to restart an otherwise healthy container unnecessarily, which can trigger a cascading cluster failure. 

**3. Readiness Probe**

- **Purpose:** Determines if the container is ready to accept user requests and traffic. 
- **Behavior:** Like the liveness probe, it runs continuously. However, an application might be alive (passing its liveness check) but temporarily unable to serve traffic—such as when it is experiencing an intense volume of background tasks, or losing connection to a mandatory backend database. 
- **Failure Handling:** **It does not restart the container**. Instead, Kubernetes simply strips the Pod from the load balancer / Service endpoints. Traffic will bypass this Pod until the readiness probe passes again. 

### **52. What are Init Containers?**
**Answer:** **Init Containers** are specialized, lightweight containers that run and complete their tasks before the main application containers start inside a Kubernetes Pod. They act as preparatory gates, ensuring that all prerequisites and environmental configurations are successfully met before the actual application goes live.

### **53. What are Ephemeral Containers?**
**Answer:** An ephemeral container is a short-lived container that is added dynamically to an already-running Kubernetes pod, primarily used for troubleshooting and interactive debugging.

#### **Key Characteristics**

- **Temporary Nature:** They are not part of the original application deployment specification and do not restart automatically after exiting.
- **No Resource Guarantees:** They share the pod's existing resource allocations and cannot be assigned custom resource limits or network ports.
- **Shared Namespaces:** They can share namespaces (like the process or network namespace) with the main application container to inspect live processes and files.

### **54. What is Node Affinity and Anti-Affinity?**
**Answer:**
Node affinity and anti-affinity are **advanced scheduling features in Kubernetes that control where Pods are placed across worker nodes based on labels**. They act as an expressive, powerful alternative to the basic `nodeSelector` field. 

**Node Affinity (Attraction)**

Node affinity **attracts** Pods to specific nodes or groups of nodes that carry matching labels (such as hardware types, SSD storage, or GPU availability). 

It comes in two primary types:

- **`requiredDuringSchedulingIgnoredDuringExecution` (Hard Rule):** The scheduler cannot place the Pod unless the node matches the rule. If no matching node is available, the Pod stays in a `Pending` state. 
- **`preferredDuringSchedulingIgnoredDuringExecution` (Soft Rule):** The scheduler tries to find a node that matches the preference. If no match is found, it will still schedule the Pod on a non-matching node. You can assign weights (1–100) to prioritize certain preferences over others.

Supported operators for matching label expressions include `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, and `Lt`.

**Node Anti-Affinity (Repulsion)**

Node anti-affinity **repels** Pods, instructing the scheduler to avoid placing Pods on nodes that possess specific labels. 

- **Purpose:** It prevents workloads from crowding onto the same physical machines or fault domains, enforcing high availability, redundancy, and workload isolation. 
- **Behavior:** Just like affinity, it supports both *required* (hard) and *preferred* (soft) enforcement modes. *(Note: In Kubernetes terminology, strict node-level separation is also commonly handled via **Pod Anti-Affinity** or **Taints and Tolerations**, which prevent specific Pods from sharing a node).*

**Key Differences from `nodeSelector`**

| **Feature**     | **`nodeSelector`**       | **Node Affinity / Anti-Affinity**                    |
| --------------- | ------------------------ | ---------------------------------------------------- |
| **Logic**       | Rigid `AND` logic only   | Expressive operators (`In`, `NotIn`, `Exists`, etc.) |
| **Rules**       | Hard requirements only   | Both hard (`required`) and soft (`preferred`) rules  |
| **Flexibility** | Basic key-value matching | Multiple terms combined with `OR`, weighted scoring  |

### **55. What is a Horizontal Pod Autoscaler (HPA)?**
**Answer:** A **Horizontal Pod Autoscaler (HPA)** is a Kubernetes component that automatically updates workload resources—such as a Deployment or StatefulSet—to increase or decrease the number of pod replicas based on demand metrics.

**How It Works**

- **Metric Monitoring:** The HPA controller queries metrics (like CPU utilization, memory usage, custom metrics, or external APIs) at regular intervals via the metrics server.
- **Automatic Scaling:** If observed resource usage exceeds your target threshold, HPA scales *out* by adding more pod replicas. If usage drops below the threshold, it scales *in* by removing excess pods down to a configured minimum.
- **Control Loop & Stability:** It uses a control loop with a default tolerance and stabilization windows to prevent rapid, unnecessary scaling changes (known as "flapping").

**Key Benefits**

- **Cost Efficiency:** Automatically scales down during low-traffic periods to save cluster resources.
- **High Availability:** Automatically scales up during traffic spikes to maintain application performance and responsiveness.
- **No Manual Intervention:** Replaces manual `kubectl scale` commands with automated tracking.

### **56. What is a Vertical Pod Autoscaler (VPA)?**
**Answer:** A **Vertical Pod Autoscaler (VPA)** is a Kubernetes tool that automatically adjusts the CPU and memory resource requests and limits of containers inside a pod based on their actual historical usage.

**How VPA Works**

- **Right-Sizing:** Instead of using static guesses, the VPA observes real-world consumption (such as CPU throttling or memory spikes) and resizes the pod to prevent Out-Of-Memory (OOM) errors or wasted resources.
- **VPA vs. HPA:** While a **Horizontal Pod Autoscaler (HPA)** scales an application *out* by adding more pod replicas, a VPA scales an application *up or down* by changing the compute resources assigned to existing pods.

**Main Components**

A standard VPA deployment relies on three core parts:

- **Recommender:** Monitors historical resource usage and computes ideal CPU and memory targets.
- **Updater:** Checks whether running pods match the recommendations and evicts pods that need updating.
- **Admission Controller:** Intercepts pod creation requests and injects the proper resource values into new pods.

**Operation Modes**

VPA can be configured to run in different modes depending on automation needs:

- **Off:** Generates and records resource recommendations without applying changes.
- **Initial:** Assigns resource requests only when a pod is first created.
- **Recreate / Auto:** Automatically updates pod specs and applies new resource values (typically by evicting and recreating the pod, or resizing in-place on supported versions).

### **57. What is Helm and what are Helm Hooks?**
**Answer:** **Helm** is a package manager for Kubernetes. It simplifies the process of defining, installing, upgrading, and managing applications on a Kubernetes cluster.

**Helm Hooks** are mechanism triggers that allow developers to intervene or execute specific actions at strategic points during an application's release lifecycle.

Normally, when you deploy a chart, Helm applies all the manifests to your cluster simultaneously. Hooks alter this behavior by letting you trigger specific Kubernetes resources (most commonly Jobs or Pods) before or after critical lifecycle events.

**Common Use Cases**

- `pre-install`: Running a database migration script or validating external cluster dependencies before launching the main application pods.
- `pre-upgrade`: Taking a database backup before upgrading an application to a newer version.
- `post-install` / `post-upgrade`: Seeding a database with initial data or sending a Slack notification that a deployment succeeded.
- `post-delete`: Cleaning up external resources or cleaning up persistent volumes that are not automatically removed by Kubernetes.
