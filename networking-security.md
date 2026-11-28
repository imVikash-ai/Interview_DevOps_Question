## **Networking & DevSecOps - DevOps Interview Questions **

### **1. Explain the 7-Layer OSI Model and the specific layers DevOps engineers interact with daily.**
**Answer:**

The **OSI (Open Systems Interconnection) model** is a conceptual seven-layer framework that **standardizes how data travels across a network**. As data moves from sender to receiver, each layer wraps or unwraps headers while handling a distinct networking job.

#### The 7 OSI Layers

- **Layer 7 – Application:** Provides network services directly to user-facing apps and APIs using protocols like HTTP/HTTPS, DNS, SMTP, and FTP.

- **Layer 6 – Presentation:** Handles data formatting, serialization (JSON, XML), compression, and encryption/decryption (TLS/SSL).

- **Layer 5 – Session:** Establishes, manages, maintains, and closes communication sessions or dialogues between endpoints (e.g., RPC, socket/session tracking).

- **Layer 4 – Transport:** Delivers data end-to-end with ports, segmentation, flow control, and reliability via TCP or speed via UDP.

- **Layer 3 – Network:** Routes packets across different networks using logical IP addresses (IPv4/IPv6) and routing protocols.

- **Layer 2 – Data Link:** Moves frames node-to-node across the same local network segment using MAC addresses, switches, and Ethernet.

- **Layer 1 – Physical:** Transmits raw bits as electrical, optical, or radio signals over cables, fiber, or wireless hardware.

#### Layers DevOps Engineers Interact With Daily

DevOps engineers rarely manage physical cables (Layer 1) or local switches (Layer 2) in cloud environments, focusing instead on **Layers 3, 4, 6, and 7** (with occasional Layer 5 debugging):

- **Layer 7 (Application):** Configuring NGINX, Envoy, or API gateways; routing traffic by host/path; debugging HTTP status codes, DNS records, and service mesh policies.
- **Layer 6 (Presentation):** Provisioning and renewing TLS/SSL certificates (e.g., cert-manager, Let's Encrypt), enforcing HTTPS, and debugging serialization/encoding mismatches.
- **Layer 4 (Transport):** Opening or blocking TCP/UDP ports in security groups and network policies, configuring Layer 4 load balancers, and diagnosing connection resets or timeouts.
- **Layer 3 (Network):** Designing VPCs, subnets, route tables, NAT gateways, VPNs/peering, and Kubernetes pod/service IP ranges.

### **2. Compare TCP vs UDP across connection state, reliability, header overhead, and production use cases.**
**Answer:**

**TCP is connection-oriented and guarantees reliable, ordered delivery with a 20–60 byte header, while UDP is connectionless and "fire-and-forget" with an 8-byte header that prioritizes low latency over guaranteed delivery.**

#### Side-by-Side Comparison

| **Dimension** | **TCP (Transmission Control Protocol)** | **UDP (User Datagram Protocol)** |
|---|---|---|
| **Connection State** | **Connection-oriented:** Uses a 3-way handshake (`SYN`, `SYN-ACK`, `ACK`) to establish state and tracks byte streams. | **Connectionless:** Stateless datagram delivery with no setup handshake or session state. |
| **Reliability** | **Reliable & ordered:** Uses sequence numbers, ACKs, retransmissions, plus flow and congestion control. | **Best-effort:** No delivery guarantees, ordering, ACKs, retransmissions, or flow/congestion control. |
| **Header Overhead** | **20–60 bytes** (variable; includes ports, sequence/ACK numbers, flags, window size, checksum, options). | **8 bytes** (fixed; four 16-bit fields: source port, destination port, length, checksum). |
| **Production Use Cases** | HTTP/HTTPS, SSH, FTP, SMTP, financial transactions, database/API calls. | DNS, DHCP, VoIP, live video/audio streaming, multiplayer gaming, telemetry/sensors. |

#### Key Trade-offs in Production

- **Latency vs. Integrity:** As noted on **GeeksforGeeks**, TCP protects data integrity at the cost of handshake delays and retransmission stalls, whereas UDP avoids delays when stale packets matter less than speed. Most users on **StackOverflow** agree on this core divide between guaranteed delivery for data accuracy and speed for real-time traffic.

- **Traffic Patterns:** According to [**Nitiz Sharma**](https://nitizsharma.com/tcp-vs-udp-differences/), UDP supports broadcast/multicast and serves as the transport foundation for modern protocols like QUIC (which adds connection semantics, encryption, and loss recovery over UDP). As highlighted by **Spiceworks**, UDP leaves reliability and congestion handling to the application layer when needed.

### **3. Walk through the complete step-by-step DNS resolution lifecycle when accessing `api.stripe.com`.**
**Answer:**

When an application makes a request to `api.stripe.com`, the system translates that human-readable hostname into an IP address through a hierarchical **DNS resolution process**. Because `api.stripe.com` relies on dynamic infrastructure and load balancing, this lookup happens every time a cached record expires.

Here is the complete step-by-step lifecycle from your application to Stripe's servers.

#### 1. Local Client & OS Checks

Before sending any traffic over the network, the client device checks local memory and configuration files.

- **Application & Browser Cache:** If the request originates from a browser or an HTTP client with internal DNS caching (like the JVM or Node.js caching libraries), it checks its local memory for a recent, unexpired IP mapping.
- **OS Stub Resolver & Cache:** If the app doesn't have the IP, it hands the query to the operating system's stub resolver, which checks the OS-level DNS cache.
- **Local `hosts` File:** If there is a cache miss, the OS reads the local `hosts` file (`/etc/hosts` on Linux/macOS or `C:\Windows\System32\drivers\etc\hosts` on Windows) to see if `api.stripe.com` is manually mapped to a static IP.

#### 2. The Recursive Resolver

If the IP isn't found locally, the OS stub resolver sends a **recursive query** to your configured DNS Recursive Resolver (typically operated by your ISP, your cloud VPC like AWS `10.0.0.2`, or a public resolver like `8.8.8.8` or `1.1.1.1`).

- **Resolver Cache Check:** The recursive resolver checks its own massive cache. Because `api.stripe.com` is a heavily trafficked endpoint, public resolvers almost always have it cached. If it's a cache hit and the Time-to-Live (TTL) hasn't expired, the resolver immediately returns the IP and skips to Step 6.
- **Iterative Querying:** If it's a cache miss, the recursive resolver takes over the heavy lifting, performing a series of **iterative queries** down the DNS hierarchy.

#### 3. The Root Nameserver (`.`)

The recursive resolver contacts one of the 13 logical **Root Nameserver** clusters (whose IP addresses are hardcoded into the resolver's configuration).

- **Query:** *"Where can I find `api.stripe.com`?"*
- **Response:** The root server doesn't know the IP for Stripe, nor does it store IP mappings. Instead, it looks at the rightmost part of the domain (`.com`) and responds with a referral to the Top-Level Domain (TLD) nameservers responsible for `.com`.

#### 4. The TLD Nameserver (`.com`)

The recursive resolver takes the referral and queries the **`.com` TLD Nameserver** (managed by Verisign).

- **Query:** *"Where can I find `api.stripe.com`?"*
- **Response:** The `.com` TLD server doesn't hold the final IP address either. Instead, it checks its registry for `stripe.com` and returns the **NS (Name Server) records** pointing to Stripe's designated authoritative nameservers (e.g., AWS Route 53 or NS1 endpoints).

#### 5. Stripe's Authoritative Nameserver

The recursive resolver now queries one of **Stripe's Authoritative Nameservers** directly.

- **CNAME & Traffic Routing:** In enterprise architectures like Stripe's, `api.stripe.com` often resolves first to a `CNAME` (Canonical Name) record managed by a traffic management or CDN/edge routing layer to direct your request to the healthiest, geographically closest Stripe edge point of presence (PoP).
- **A Record Resolution:** Stripe's authoritative DNS server evaluates routing health and resolves the chain to a final **A record** containing an IPv4 address. *(Note: Per **[Stripe's IP documentation](https://docs.stripe.com/ips)**, `api.stripe.com` strictly serves IPv4 `A` records and does not return IPv6 `AAAA` records).*
- **TTL Assignment:** Along with the IPv4 address, the authoritative server attaches a short **Time-to-Live (TTL)** value—typically 60 seconds or less—so Stripe can rapidly reroute traffic away from degraded servers without clients getting stuck on stale IPs.

#### 6. Caching & Connection Handoff

- **Resolver Caching:** The recursive resolver receives the IPv4 address, caches it for the duration of the TTL, and sends the answer back to your machine's OS stub resolver.
- **Client Caching & Socket Creation:** Your OS caches the IP locally and passes it to your application.
- **TCP & TLS Handshake:** With the DNS lifecycle complete, your application opens a socket to that IPv4 address on port `443`, initiating the **TCP 3-way handshake** followed by the **TLS handshake** (verifying Stripe's SSL certificate) before finally sending your encrypted HTTP API request.

### **4. Compare Public IP vs Private IP (RFC 1918) vs NAT (Network Address Translation).**
**Answer:**

**Public IPs are globally unique, internet-routable addresses assigned by an ISP; Private IPs (RFC 1918) are non-routable local addresses reused inside private networks; and NAT (Network Address Translation) is the router mechanism that translates between the two so local devices can use the internet.**

#### Side-by-Side Comparison

| **Feature** | **Public IP** | **Private IP (RFC 1918)** | **NAT (Network Address Translation)** |
|---|---|---|---|
| **What It Is** | A globally unique IPv4/IPv6 address for external communication. | A reserved local IPv4 address for internal networks (LANs). | A translation process at a network boundary/router that rewrites IP packet headers. |
| **Address Ranges** | Globally allocated blocks (excludes private & special-use ranges). | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. | Not an address range; maps private IPs to public IPs (often using ports via PAT). |
| **Internet Routability** | Routable across the public internet. | Not routable on the public internet; dropped by public routers. | Enables private-IP devices to reach the internet via a public IP. |
| **Uniqueness** | Globally unique while assigned. | Unique within a local network; reusable across separate networks. | Tracks active sessions/ports in a translation table to keep return traffic distinct. |
| **Who Assigns / Runs It** | Assigned by an ISP (dynamic or static). | Assigned locally by a router/DHCP server or set statically. | Performed by a router, firewall, or gateway (and sometimes ISP CGNAT). |

#### How They Work Together

- **Internal addressing:** Devices like laptops and phones get RFC 1918 addresses (such as `192.168.1.10`) to talk inside your LAN.
- **Outbound translation:** When a device sends internet traffic, the router's NAT—typically **PAT (Port Address Translation)**—replaces the private source IP and port with its public IP and a mapped port.
- **Return routing:** External servers reply to the public IP. The router checks its NAT table and routes the reply back to the exact internal device.
- **Security note:** Private IPs and NAT hide internal addresses from direct external reachability, but they are not a substitute for a firewall or access controls.


### **5. Explain CIDR Notation and subnet sizing calculations.**
**Answer:** 

**Classless Inter-Domain Routing (CIDR)** notation writes an IP address followed by a slash and a prefix length (such as `192.168.1.0/24`), showing how many leading bits form the network portion. In IPv4 (32 bits total), the remaining bits identify individual hosts, and you calculate subnet sizes using powers of 2.

#### How CIDR Notation Works

- **Prefix length (`/n`):** Counts the consecutive `1` bits in a subnet mask from left to right. For example, `/24` means the first 24 bits define the network and `32 - 24 = 8` bits remain for hosts.
- **Classless flexibility:** Unlike older Class A, B, and C boundaries, CIDR lets you pick arbitrary prefix lengths (`0` to `32` in IPv4; `0` to `128` in IPv6) to match actual network needs and reduce address waste. 
- **Subnet masks:** Each prefix maps to a binary mask of `n` ones followed by `32 - n` zeros. For instance, `/26` has 26 ones (`255.255.255.192`). 

#### Subnet Sizing Formulas

For an IPv4 prefix `/n`:

1. **Host bits (`h`):** `32 - n`
2. **Total IP addresses:** `2^h` (or `2^(32 - n)`)
3. **Usable host IPs:** `2^h - 2` for standard subnets (`/0` through `/30`). The first address is the **network address** (all host bits `0`) and the last is the **broadcast address** (all host bits `1`).

   **Exceptions:** `/31` is commonly used for point-to-point links with 2 usable IPs per RFC 3021, and `/32` identifies a single host IP.

#### Common IPv4 Subnet Sizes

| **CIDR** | **Subnet Mask** | **Total IPs** | **Usable Hosts** |
|---|---|---:|---:|
| `/24` | `255.255.255.0` | 256 | 254 |
| `/25` | `255.255.255.128` | 128 | 126 |
| `/26` | `255.255.255.192` | 64 | 62 |
| `/27` | `255.255.255.224` | 32 | 30 |
| `/28` | `255.255.255.240` | 16 | 14 |
| `/29` | `255.255.255.248` | 8 | 6 |
| `/30` | `255.255.255.252` | 4 | 2 |

#### Quick Example

Suppose you have `10.0.1.0/27`:

- Host bits = `32 - 27 = 5`.
- Total IPs = `2^5 = 32` (`10.0.1.0` through `10.0.1.31`).
- Usable hosts = `32 - 2 = 30` (`10.0.1.1` through `10.0.1.30`), since `.0` is the network ID and `.31` is the broadcast address.

### **6. Compare Symmetric vs Asymmetric Encryption.**
**Answer:**

Symmetric and asymmetric encryption are the two core methods used to secure digital data. They **differ primarily in how they handle encryption keys and performance**.

#### Symmetric Encryption

- **Key usage:** Uses a **single shared secret key** for both encrypting and decrypting data. Both parties must possess this same key.
- **Speed & efficiency:** Very fast and computationally lightweight, making it ideal for bulk data encryption (like databases, file storage, and VPNs).
- **Key challenge:** Requires a secure way to share the secret key beforehand, which can create a vulnerability if intercepted.
- **Common algorithms:** AES, DES, 3DES, Blowfish, and RC4.

#### Asymmetric Encryption (Public-Key Cryptography)

- **Key usage:** Uses a **mathematically linked pair of keys**—a **public key** to encrypt data and a **private key** to decrypt it.
- **Speed & efficiency:** Slower and more resource-intensive because it relies on complex mathematical operations (such as large prime numbers or elliptic curves).
- **Key advantage:** Eliminates the need to share a secret key over the network, enhancing security and enabling **digital signatures** for authenticity and non-repudiation.
- **Common algorithms:** RSA, ECC (Elliptic Curve Cryptography), Diffie-Hellman, and DSA.

#### How They Work Together (Hybrid Model)

Most modern secure systems—such as HTTPS/TLS web browsing and secure messaging—combine both methods.

- Asymmetric cryptography is used at the start to safely authenticate participants and establish shared session keys.
- Symmetric encryption then handles fast, bulk data transfer for the rest of the connection.


### **7. Compare Reverse Proxy vs Forward Proxy.**
**Answer:**

A **forward proxy** acts on behalf of clients to manage outbound requests to external servers, while a **reverse proxy** acts on behalf of backend servers to manage incoming requests from clients.

Both sit between clients and servers to filter traffic, cache data, and add security.

#### Side-by-Side Comparison

| **Feature** | **Forward Proxy** | **Reverse Proxy** |
|---|---|---|
| **Represents** | Clients (users or internal networks) | Backend / origin servers |
| **Traffic Flow** | Outbound (Client → Proxy → Internet/Server) | Inbound (Client/Internet → Proxy → Backend) |
| **IP Masking** | Hides client IP addresses from external servers | Hides backend server IP addresses from clients |
| **Primary Goal** | Client privacy, access control, and outbound filtering | Server scalability, security, load balancing, and routing |
| **Where It Sits** | Near clients or on client-side networks | In front of web/application servers (often in a DMZ or edge) |
| **Common Examples** | Squid, Apache Traffic Server, VPNs | Nginx, HAProxy, Cloudflare, AWS Elastic Load Balancer |

#### Key Differences Explained

#### 1. Forward Proxy

- The destination server sees the forward proxy's IP address rather than the client's IP address.
- Organizations use forward proxies to:
  - Block restricted websites.
  - Monitor employee browsing.
  - Cache frequently fetched external pages.
  - Apply outbound access policies.
- Forward proxies are configured on client devices or client-side networks and can tunnel or intercept HTTPS traffic when trusted.

#### 2. Reverse Proxy

- Clients connect to the reverse proxy as if it were the origin server, without knowing which backend server handles the request.
- Reverse proxies are typically managed by site administrators and DevOps teams.
- Common uses include:
  - **Load balancing:** Distributes traffic across multiple backend servers.
  - **Caching:** Stores static content to improve response times.
  - **SSL/TLS termination:** Handles encryption and decryption at the proxy layer.
  - **Security:** Helps block DDoS attacks and malicious traffic.
  - **Routing:** Directs requests to the appropriate backend service.


### **8. What is a Web Application Firewall (WAF)?**
**Answer:** 

A **web application firewall (WAF)** is **a security tool that monitors, filters, and blocks malicious HTTP and HTTPS traffic traveling between a web application and the internet**.

Operating at Layer 7 (the application layer) of the OSI model, it inspects the content of web requests to stop threats before they reach the application or backend servers.

#### How It Works

A WAF typically acts as a reverse proxy in front of a web application. It evaluates incoming requests against security rules (policies) and decides whether to allow, block, or log them.

Common filtering models include:

- **Blocklist (negative model):** Blocks known attack signatures and malicious sources while allowing other traffic.
- **Allowlist (positive model):** Permits only pre-approved, trusted traffic and blocks everything else.
- **Hybrid model:** Combines both blocklist and allowlist rules.

#### Common Usage Examples

Organizations use WAFs to protect websites, mobile backends, and APIs against application-layer threats, including:

- **SQL injection and cross-site scripting (XSS):** Stopping attackers from injecting malicious code or database commands through forms, URLs, or headers.
- **Cross-site request forgery (CSRF) and file inclusion:** Preventing unauthorized actions or unauthorized file access.
- **Application-layer DDoS and abusive bots:** Applying rate limiting and bot rules to keep applications responsive during traffic spikes or scraping attempts.
- **Compliance:** Helping meet regulatory requirements such as PCI DSS for protecting payment and customer data.

#### Deployment Types

- **Cloud-based WAF:** Hosted and managed by a third-party provider (often via DNS or a CDN), offering easy setup and automatic rule updates.
- **Network-based (hardware) WAF:** Installed as local appliances on-premises to minimize latency, though typically more expensive and harder to scale.
- **Host-based WAF:** Integrated directly into the web server or application software, offering high customization while using local server resources.

### **9. What is Zero Trust Architecture (ZTA)?**
**Answer:** 

**Zero Trust Architecture (ZTA)** is **a cybersecurity framework built on the principle of "never trust, always verify," requiring every user, device, and connection to be continuously authenticated and authorized before accessing resources**.

Unlike traditional perimeter-based security—which assumes everything inside a corporate network is safe—ZTA treats every access request as potentially risky, regardless of location.

#### Core Principles

- **Verify explicitly:** Evaluate identity, device health, location, and context continuously before granting or maintaining access.
- **Use least privilege:** Grant only the minimum access needed for a specific task or timeframe.
- **Assume breach:** Design systems to limit lateral movement and contain damage by assuming an attacker may already be inside.

#### Key Components & Pillars

According to frameworks like **NIST SP 800-207** and guidance from **Palo Alto Networks**, ZTA spans **identity, devices, networks, applications/workloads, and data** using:

- **Identity and Access Management (IAM):** Multi-factor authentication (MFA), single sign-on (SSO), and role-based or just-in-time access.
- **Policy Engine & Enforcement:** Central decision logic (policy engine/administrator) paired with enforcement points that allow or block sessions dynamically.
- **Microsegmentation & Encryption:** Isolating network zones and encrypting data in transit and at rest to stop lateral movement.
- **Continuous Monitoring:** Real-time telemetry, endpoint posture checks, and security analytics to detect anomalies.

#### Common Usage Examples

- **Remote and hybrid work:** Letting employees safely reach internal apps from home or public networks without broad VPN access.
- **Cloud and multicloud environments:** Enforcing consistent access policies across SaaS tools, public clouds, and on-premises systems.
- **Ransomware and lateral-movement containment:** Preventing a single compromised laptop or phishing credential from unlocking critical servers or databases.
- **Third-party and contractor access:** Restricting vendors or temporary staff to only the specific apps and data they need.

### **10. What is DevSecOps and the "Shift-Left" philosophy?**
**Answer:** 

**DevSecOps** is a software delivery approach that integrates security practices, automation, and shared responsibility across development, security, and operations teams throughout the entire software development lifecycle (SDLC).

The **"Shift-Left"** philosophy is **a core principle of DevSecOps that moves security testing, threat modeling, and compliance checks earlier in the SDLC—closer to planning, design, and coding—rather than waiting until testing or release**.

#### How DevSecOps and Shift-Left Work Together

- **Earlier defect detection:** Security checks happen while code is written or built, making flaws cheaper and easier to fix than after deployment.
- **Automated guardrails:** Pipelines run automated checks—such as static application security testing (SAST), dependency/SCA scans, and infrastructure-as-code checks—without creating manual bottlenecks.
- **Shared accountability:** Developers, operations, and security share visibility and ownership instead of treating security as a separate final gate.
- **Balanced velocity:** Teams ship updates faster while maintaining continuous monitoring (often paired with "shifting right" in production).

#### Usage Examples

- **Coding & pull requests:** Running static analysis (SAST) or secret/dependency scanners in a CI pipeline (such as GitHub Actions) before code merges.
- **Design & architecture:** Performing threat modeling during planning to spot design risks before writing features.
- **Build & pre-deploy:** Validating container images, APIs, and cloud/infrastructure configurations before staging or production release.

### **11. Compare SAST vs DAST vs IAST vs SCA.**
**Answer:**

**SAST checks your own source code before running, DAST probes a live app from the outside, IAST monitors a running app from the inside, and SCA audits third-party and open-source dependencies.**

#### Comparison at a Glance

| **Feature** | **SAST (Static)** | **DAST (Dynamic)** | **IAST (Interactive)** | **SCA (Composition)** |
|---|---|---|---|---|
| **What it inspects** | Source code or binaries | Running app (black-box) | Running app with sensors (gray-box) | Open-source & third-party packages |
| **Needs running app?** | No | Yes | Yes (during active tests) | No (scans manifests, builds, SBOMs) |
| **SDLC stage** | IDE, pull request, build | QA, staging, pre-release | QA / functional testing | Build & continuous monitoring |
| **Strengths** | Early feedback; pinpoints exact code lines | Finds runtime & config flaws; language-agnostic | Confirms exploitability with code location; fewer false positives | Catches known CVEs, license risks & supply-chain issues |
| **Main limitations** | False positives; misses runtime context | Slower; no code line mapping; depends on test coverage | Needs instrumentation; depends on test execution & language support | Only covers imported dependencies, not custom code |

#### Key Differences

#### 1. SAST (Static Application Security Testing)
- Scans proprietary code for insecure patterns, unsafe functions, and hardcoded secrets before execution.

#### 2. DAST (Dynamic Application Security Testing)
- Sends simulated attacks to deployed endpoints to spot runtime injection, authentication flaws, and server misconfigurations.

#### 3. IAST (Interactive Application Security Testing)
- Uses agents or sensors inside a running application while tests execute to trace data flow and confirm vulnerabilities with code context.

#### 4. SCA (Software Composition Analysis)
- Inventories libraries and dependencies to flag known CVEs, outdated versions, risky licenses, and malicious packages.

### **12. What is HashiCorp Vault and what core capabilities does it provide?**
**Answer:**

**HashiCorp Vault** is **an identity-based secrets and encryption management system that centralizes control over sensitive credentials, keys, certificates, and data across on-premises, cloud, and hybrid environments**.

It helps organizations eliminate secret sprawl and enforce strict access controls via a UI, CLI, or HTTP API.

#### Core Capabilities

- **Centralized Secrets Management:** Stores, accesses, and distributes static secrets—such as passwords, API keys, and tokens—in an encrypted state from a single location.
- **Dynamic Secrets & Automated Rotation:** Generates short-lived, just-in-time credentials on demand (for databases, cloud providers, and services) and automatically revokes them when leases expire.
- **Encryption as a Service (Transit):** Encrypts and decrypts application data in transit and at rest without requiring applications to store or manage encryption keys directly.
- **Certificate & Key Lifecycle Management:** Issues, rotates, and revokes TLS/PKI certificates and manages cryptographic keys across providers.
- **Identity-Based Authentication & Policy Authorization:** Authenticates human users, machines, and workloads through trusted identity providers (such as AWS, Azure, Kubernetes, LDAP, or AppRole) and restricts access using path-based policies.
- **Audit Logging & Compliance:** Records detailed request-response trails of all secret access and administrative actions to support security monitoring and regulatory compliance.

### **13. Explain Kubernetes RBAC (Role, ClusterRole, RoleBinding, ClusterRoleBinding).**
**Answer:**

**Kubernetes Role-Based Access Control (RBAC)** is the built-in authorization mechanism that regulates who can access the Kubernetes API and what actions they can perform.

It splits permissions into **what can be done** (Roles) and **who can do it** (Bindings), operating at either the namespace or cluster level.

#### Quick Comparison

| **Object** | **Scope** | **Purpose** | **Can Reference** |
|---|---|---|---|
| **Role** | Namespace | Defines permissions within a single namespace. | N/A (contains rules) |
| **ClusterRole** | Cluster-wide | Defines permissions across all namespaces or for cluster-level resources. | N/A (contains rules) |
| **RoleBinding** | Namespace | Grants permissions to subjects inside a specific namespace. | A `Role` (same namespace) or a `ClusterRole` |
| **ClusterRoleBinding** | Cluster-wide | Grants permissions to subjects across the entire cluster. | Only a `ClusterRole` |

#### 1. Defining Permissions: Role vs. ClusterRole

Both objects contain `rules` made up of **API Groups** (e.g., `""` for core, `apps`), **Resources** (e.g., `pods`, `deployments`), and **Verbs** (e.g., `get`, `list`, `create`, `delete`).

- **Role:** Sets permissions strictly within one specific namespace. For example, you might create a Role in the `dev` namespace allowing a user to read and restart pods there, without affecting `prod`.
- **ClusterRole:** Sets permissions at the cluster level. It is used for:
  - **Cluster-scoped resources** that do not live in a namespace (like `nodes` or `PersistentVolumes`).
  - **Non-resource endpoints** (like `/healthz`).
  - **Reusable templates** of namespaced permissions (such as read-only access to pods) that you want to apply across many namespaces without duplicating YAML.

#### 2. Assigning Permissions: RoleBinding vs. ClusterRoleBinding

Defining a Role does nothing on its own. Bindings connect **Subjects** (Users, Groups, or ServiceAccounts) to a Role or ClusterRole.

- **RoleBinding:** Grants the permissions defined in a Role to a subject *within a single namespace*.
  - **Common pattern:** A `RoleBinding` in the `staging` namespace can reference a cluster-wide `ClusterRole` (like the built-in `view` role). This grants the user read-only access **only** inside `staging`, keeping permissions reusable.
- **ClusterRoleBinding:** Grants the permissions defined in a `ClusterRole` to a subject *across every namespace and cluster resource*. A `ClusterRoleBinding` cannot reference a standard namespaced `Role`.

### **14. What is Secret Masking and Secret Scanning in Git?**
**Answer:**

**Secret Scanning finds and blocks hardcoded credentials in your repository, while Secret Masking hides sensitive values from appearing in your CI/CD build logs.**

While core Git itself does not include these features natively, both are standard security capabilities built into Git hosting platforms like GitHub and GitLab.

#### Secret Scanning

Secret scanning automatically inspects your codebase and repository history to detect accidentally exposed credentials—such as API keys, passwords, database strings, and private tokens. 

- **How it works.** Scanners use regular expressions (matching known formats like AWS keys), entropy checks (flagging high-randomness strings), and AI models to identify sensitive data.
- **Where it looks.** Beyond active source code, platforms like **GitHub Secret Scanning** and **GitLab Secret Detection** inspect entire commit histories, pull/merge requests, issues, and comments. 
- **Push protection.** Modern secret scanning often runs right as you execute `git push`, blocking the upload before the secret ever enters the remote repository. 

#### Secret Masking

Secret masking (or log redaction) prevents sensitive environment variables from being printed in plain text inside CI/CD pipeline logs during automated builds, tests, or deployments. 

- **How it works.** When a pipeline runs a script that accidentally outputs a stored secret (e.g., via an `echo` command or a verbose error trace), the runner intercepts the string and replaces it with asterisks (`***`) or `[MASKED]`. 
- **Platform implementation.** Both GitHub Actions (which masks repository secrets automatically and supports the `::add-mask::` workflow command) and **GitLab CI/CD** (via masked variables) enforce log hygiene so anyone viewing build logs cannot read the raw credentials. 
- **Limitations.** Masking generally requires secrets to be single-line strings without spaces. If a script encodes or alters the secret (such as Base64 encoding it before printing), the masking filter may fail to recognize and hide it.

#### Key Differences

| **Feature** | **Secret Scanning** | **Secret Masking** |
|---|---|---|
| **Primary Goal** | Stop secrets from living in Git code/history | Stop secrets from appearing in CI/CD logs |
| **When it Runs** | During `git push`, commits, or background scans | During automated pipeline/runner execution |
| **Action Taken** | Blocks the push or triggers a security alert | Replaces the string with `***` or `[MASKED]` |
| **Remediation if Exposed** | Revoke/rotate key and scrub Git history | Revoke/rotate key and delete the job log |

### **15. What is Falco and how does it detect Runtime Threats in Kubernetes?**
**Answer:** 

**Falco** is **an open-source, Cloud Native Computing Foundation (CNCF) graduated runtime security tool that detects abnormal behavior, potential threats, and compliance violations across Kubernetes clusters, containers, hosts, and cloud environments in near real time**. Originally created by Sysdig in 2016, it acts like an intrusion detection system (IDS) for running workloads—catching suspicious activity that static image scanners and admission controllers miss. 

#### How Falco Detects Runtime Threats in Kubernetes

Falco typically runs across Kubernetes nodes (often deployed as a DaemonSet via Helm) and detects threats in four main stages:

1. **Event Collection (Kernel & External Sources):**
   - **Linux System Calls (syscalls):** Containers share the host kernel, so every action—opening a file, spawning a process, or making a network call—triggers syscalls. Falco captures these events from kernel space using an **eBPF** probe or a **kernel module**.
   - **Additional Telemetry:** Through plugins and integrations, Falco also ingests **Kubernetes audit logs** and cloud/application events. 

2. **Context Enrichment:**
   - Raw kernel events are enriched with metadata from the container runtime and the Kubernetes API server—such as container ID, image name, pod/namespace, user, and file or network details—so alerts show *which* workload triggered an event. 

3. **Rule Evaluation:**
   - Falco's user-space engine evaluates incoming enriched events against YAML-based **rules** (along with reusable **macros** and **lists**).
   - It ships with default rules and supports custom rules tailored to your environment or compliance frameworks. 

4. **Alerting & Response:**
   - When an event matches a rule's conditions, Falco emits an alert with a priority, description, and structured fields.
   - Falco focuses on **detection and alerting** rather than blocking syscalls itself. Alerts can be sent to stdout, files, syslog, Prometheus, SIEMs, or tools like **Falcosidekick** to route notifications (such as Slack or PagerDuty) or trigger automated responses (such as via **Falco Talon**). 

#### Common Kubernetes Threats Falco Detects

- **Interactive shells in containers:** A shell (`sh`, `bash`) spawned inside a running container.
- **Sensitive file access:** Reading or modifying sensitive files such as `/etc/shadow` or `/etc/passwd`.
- **Suspicious execution & escapes:** Running binaries from `/tmp`, privilege escalation attempts, or container escape techniques.
- **Unexpected network & API activity:** Outbound connections to unusual destinations, traffic sniffing, or unexpected calls to the Kubernetes API server from inside a container.
- **Malware patterns:** Signatures associated with activity like cryptominers.

### **16. Compare AWS GuardDuty vs AWS Security Hub.**
**Answer:**

**Amazon GuardDuty detects active threats and malicious behavior in real time, while AWS Security Hub aggregates security findings across tools and checks configuration compliance**. They are complementary rather than competing services—GuardDuty spots live attacks, and Security Hub centralizes visibility, posture checks, and response workflows. 

#### Core Differences

| **Feature** | **Amazon GuardDuty** | **AWS Security Hub** |
|---|---|---|
| **Primary role** | Threat detection | Security posture management & finding aggregation |
| **What it looks for** | Active threats, unauthorized access, anomalies, malicious IPs | Misconfigurations, compliance violations, consolidated alerts |
| **Data sources** | CloudTrail, VPC Flow Logs, DNS logs, plus optional S3, EKS, RDS, and Lambda protection plans | Security checks/controls, AWS Config rules, and findings from GuardDuty, Inspector, Macie, and third-party tools |
| **Detection method** | Machine learning, anomaly detection, and threat intelligence | Automated security checks and compliance evaluations against standards (such as CIS, PCI DSS, and NIST) |
| **Pricing model** | Based on volume of log/event data analyzed and enabled protection plans | Based on security checks/resource monitoring and ingested findings (plus underlying service costs like AWS Config) |

#### How They Work Together

- **GuardDuty feeds Security Hub:** GuardDuty generates detailed threat findings and can send them into Security Hub so security teams see live threats alongside posture and vulnerability data in one place.
- **Shared automation:** Both integrate with Amazon EventBridge to trigger automated alerts or remediation workflows.

#### When to Use Each

- **Use Amazon GuardDuty** when you need continuous, low-overhead detection of active intrusions, compromised instances, or suspicious API/network activity. 
- **Use AWS Security Hub** when you need a centralized dashboard, compliance benchmarking, and unified management of findings across AWS services and partner tools. 
- **Use both** for layered defense: GuardDuty catches ongoing attacks, while Security Hub tracks configuration drift and centralizes alerts across your environment.

### **17. What is Trivy and what artifact types does it scan?**
**Answer:** 

**Trivy** is **an open-source, all-in-one security scanner created by Aqua Security that detects vulnerabilities (CVEs), misconfigurations, exposed secrets, and license issues across software artifacts and environments**. You can run it from the command line, inside CI/CD pipelines, or as a Kubernetes operator. 

#### Artifact Types Trivy Scans

Trivy scans several target and artifact types, including:

- **Container images:** Images stored locally (such as Docker, Containerd, or Podman) or in remote registries (like Docker Hub, AWS ECR, Google Container Registry, and Azure Container Registry).
- **Filesystems:** Local directories, root filesystems, and binary or software package artifacts.
- **Git repositories:** Local or remote code repositories, including source code, dependencies, and Infrastructure as Code (IaC) files.
- **Kubernetes clusters:** Live cluster resources and configurations.
- **Virtual machine images:** Disk and VM images.
- **Cloud environments and artifacts:** Supported cloud services and storage targets such as AWS accounts, S3 buckets, and Lambda functions.
- **Software Bills of Materials (SBOMs):** Generates and inspects SBOM formats such as SPDX and CycloneDX.

### **18. Compare SSL Termination vs End-to-End Encryption.**
**Answer:**

**SSL (TLS) termination decrypts incoming encrypted traffic at an intermediary (such as a load balancer or reverse proxy) and often forwards it as unencrypted HTTP internally, whereas end-to-end encryption (E2EE) keeps data encrypted from the sender all the way to the final recipient without exposing plaintext to intermediaries**. 

#### Key Differences

- **Where Decryption Happens**
  - **SSL Termination:** Decrypts at the network edge or load balancer. Internal traffic to backend servers may travel as plain HTTP (unless re-encrypted in a bridging/re-encryption setup).
  - **End-to-End Encryption:** Data remains encrypted across the entire path. In transport/infrastructure contexts, this happens via TLS passthrough (backend decrypts) or re-encryption; in messaging/app contexts, only the sender and recipient hold the decryption keys. 
- **Security & Trust Model**
  - **SSL Termination:** Trusts the intermediary and internal network. If an attacker breaches the internal network or compromises the intermediary, plaintext traffic can be exposed.
  - **End-to-End Encryption:** Stronger protection against eavesdropping and internal breaches. agree that TLS allows intermediate servers to read traffic when terminated, while end-to-end encryption prevents intermediate servers from accessing the contents. 
- **Performance & Overhead**
  - **SSL Termination:** Offloads cryptographic work from backend servers and simplifies certificate management in one place.
  - **End-to-End Encryption:** Can add CPU overhead on backend servers or require managing certificates across more endpoints. 
- **Traffic Inspection & Routing**
  - **SSL Termination:** Lets load balancers or web application firewalls (WAFs) inspect headers, paths, and payloads for Layer 7 routing and threat scanning.
  - **End-to-End Encryption:** Passthrough encryption prevents intermediaries from inspecting or modifying encrypted application payloads, though re-encryption models can balance inspection with backend encryption. 

#### When to Choose Which

- **Choose SSL Termination** when you need Layer 7 routing/inspection, centralized certificate management, and lower backend CPU load on a trusted or properly segmented internal network. 
- **Choose End-to-End Encryption** when handling highly sensitive or regulated data, enforcing strict zero-trust boundaries, or protecting privacy from intermediaries.

### **19. What is Internet Control Message Protocol (ICMP)?**
**Answer:** 

The **Internet Control Message Protocol (ICMP)** is **a connectionless network-layer protocol in the TCP/IP suite used by routers and network devices to report delivery errors and run network diagnostics**. Because the Internet Protocol (IP) itself does not provide error feedback, ICMP notifies senders when packets fail to reach their destination or encounter routing problems. 

#### How It Works

- **Connectionless:** ICMP does not establish a session or use a handshake before sending messages, nor does it use TCP/UDP port numbers.
- **Encapsulation:** ICMP messages ride inside IP datagrams, using **Type** and **Code** fields to specify the message purpose and details.
- **IPv4 and IPv6:** ICMP works alongside IPv4 (ICMPv4) and has an IPv6 counterpart (ICMPv6) that also supports neighbor discovery and Path MTU Discovery. 

#### Common Usage Examples

- **Ping:** Sends **Echo Request** (Type 8) packets and listens for **Echo Reply** (Type 0) packets to test reachability and round-trip latency.
- **Traceroute:** Uses **Time Exceeded** (Type 11) and **Destination Unreachable** (Type 3) messages to map the path and hop delays between devices.
- **Routing & Error Alerts:** Notifies senders of unreachable hosts/networks, expired Time-to-Live (TTL) values, or better routes via **Redirect** (Type 5) messages.

### **20. What is a Subnet Mask and Default Gateway?**
**Answer:**

**A subnet mask defines the boundaries of a local network, while a default gateway serves as the exit point to reach outside networks like the internet**. Together, they tell a device whether data can be sent directly to a local neighbor or must be routed externally.

#### Subnet Mask

A subnet mask is a 32-bit number (such as `255.255.255.0`) that splits an IP address into two distinct parts:

- **Network ID:** Identifies the specific local network or subnet.
- **Host ID:** Identifies the individual device on that network.

**How it works:** When your computer wants to send data to another IP address, it uses the subnet mask to check if the destination shares the same Network ID.

- If it matches, the destination is on the **same local network**, and your device sends the data directly to it.
- If it does not match, the destination is on a **remote network**, and your device hands the data off to the default gateway.

#### Default Gateway

A default gateway is the IP address of the router (such as `192.168.1.1`) that connects your local network to external networks.

- **Role:** It acts as a traffic cop or middleman, forwarding data packets from your local devices out to the wider internet and directing incoming traffic back to the right device.
- **Requirement:** Without a configured default gateway, devices can still communicate with each other locally (like printing to a Wi-Fi printer), but they cannot browse the web or reach outside servers.

#### Quick Analogy

| **Networking Term** | **Real-World Equivalent** | **Purpose** |
|---|---|---|
| **IP Address** | Your street address (`123 Main St`) | Uniquely identifies your house. |
| **Subnet Mask** | Your neighborhood boundary | Tells the mail carrier which houses are in the same neighborhood. |
| **Default Gateway** | The local post office | Handles any mail going outside your neighborhood to other cities. |

### **21. What is DNS CNAME vs A vs AAAA vs ALIAS Record?**
**Answer:**

**A and AAAA records map a domain name directly to an IPv4 or IPv6 address, while CNAME and ALIAS records point one domain name to another domain name**. **CNAME** is a standard DNS alias that cannot sit at a root domain, whereas **ALIAS** is a provider-specific record that acts like a CNAME while allowing root-domain use and coexisting with other records. 

#### Key Differences

- **A Record (Address Record):** Maps a domain or subdomain directly to a 32-bit **IPv4** address (for example, `192.0.2.1`).
- **AAAA Record (Quad-A Record):** Maps a domain or subdomain directly to a 128-bit **IPv6** address (for example, `2001:db8::1`). 
- **CNAME Record (Canonical Name):** Points an alias hostname to another canonical domain name rather than an IP address. It cannot coexist with other records (such as `MX` or `TXT`) on the same name and generally cannot be placed on a root/apex domain (`example.com`). 
- **ALIAS Record (also called ANAME):** Maps a hostname—including a root/apex domain—to another domain name. It is not a standard RFC record type; DNS providers resolve it dynamically into `A` or `AAAA` answers so it can coexist with other records like `MX`.

#### Quick Comparison

| **Record** | **Target** | **Root / Apex (`example.com`)** | **Coexists with Other Records** | **Standard RFC Type** |
|---|---|---|---|---|
| **A** | IPv4 address | Yes | Yes | Yes |
| **AAAA** | IPv6 address | Yes | Yes | Yes |
| **CNAME** | Another domain name | Usually no | No | Yes |
| **ALIAS** | Another domain name | Yes | Yes | No (provider-specific) |

#### Usage Examples & Related Terms

- **A / AAAA:** Pointing `example.com` or `api.example.com` to a dedicated server with a static IPv4 or IPv6 address.
- **CNAME:** Pointing `www.example.com` or `blog.example.com` to a third-party service or `example.com` so IP changes only need one update.
- **ALIAS:** Pointing the bare domain `example.com` to a load balancer, CDN, or cloud app host (such as AWS or Heroku) while keeping `MX` email records on `example.com`.
- **Related terms / synonyms:** `ALIAS` is often called an **ANAME** record or uses **CNAME flattening**; the root domain is also called the **apex** or **naked** domain.

### **22. What is DNS TTL (Time to Live)?**
**Answer:** 

**DNS TTL (Time to Live)** is **a setting on a DNS record (measured in seconds) that tells resolvers and caches how long to store the record before asking the authoritative DNS server for a fresh copy**. 

#### How It Works

- **Caching & Speed:** While the TTL timer is active, local or recursive DNS resolvers serve cached answers instead of re-querying authoritative servers, reducing lookup latency and server load. 
- **Updates & Propagation:** When you edit a DNS record, users may still see the old value until existing cached copies expire. 
- **Practical Caveat:** Resolvers and local caches treat TTL as guidance rather than a strict guarantee, and some providers or applications may cache slightly longer or shorter. 

#### Common Usage Examples

- **High TTL (e.g., 3,600–86,400 seconds / 1–24 hours):** Used for stable records like `MX` or `TXT` records that rarely change, maximizing cache hits and reducing DNS traffic.
- **Low TTL (e.g., 30–300 seconds):** Used before server migrations, during failover/disaster recovery, or for frequently changing records so updates take effect quickly. 
- **Migration Workflow:** Administrators often lower a record's TTL hours or days ahead of a change so old caches expire faster, make the DNS update, and raise the TTL back afterward.

### **23. What is Gitleaks Pre-Commit Hook?**
**Answer:** 

A **Gitleaks pre-commit hook** is **an automated security script that scans your staged code for sensitive information—such as API keys, passwords, and private tokens—and blocks a Git commit before it enters your repository's history**. 

It combines two tools:

- **Gitleaks:** An open-source secret scanner that uses regular expressions and entropy checks to detect hardcoded credentials. 
- **Git Pre-Commit Hook:** A local script that triggers automatically right after you run `git commit`, inspecting your changes before the commit is finalized. 

#### Why It Matters

Once a secret is committed to Git, simply deleting it in a later commit does not remove it; the credential remains recoverable in the Git history unless you perform a disruptive history rewrite. Catching secrets locally stops leaks before they ever reach a remote server like GitHub or GitLab, saving you the cost and emergency effort of revoking and rotating exposed keys.

#### How It Works

1. **Trigger.** You stage your files (`git add`) and run `git commit`. 
2. **Scan.** Before asking for a commit message or saving the snapshot, the hook runs Gitleaks specifically against your staged changes (`gitleaks git --pre-commit --staged`). 
3. **Action.**
   - **If clean:** The commit proceeds normally.
   - **If a secret is found:** Gitleaks halts the commit and outputs the exact file path, line number, and type of secret detected so you can remove it. 

#### Common Ways to Set It Up

- **Using the `pre-commit` framework (Recommended):** Teams typically manage this by adding the official **Gitleaks GitHub repository** to a `.pre-commit-config.yaml` file at the root of their project. This ensures every developer on the team runs the exact same version of Gitleaks. 
- **Native Git Hook:** You can place a custom shell or Python script directly inside your local `.git/hooks/pre-commit` directory to invoke the Gitleaks CLI.

### **24. What is Trufflehog?**
**Answer:**

**TruffleHog** is **a security tool developed by Truffle Security Co.** that scans code, commit history, and cloud resources to find and verify leaked secrets like API keys, passwords, and access tokens.

#### Key Features

- **Deep Scanning:** Searches Git repositories (including commit history and deleted code), GitHub, GitLab, Docker images, S3 buckets, Slack, local filesystems, and CI/CD pipelines across 800+ secret types. 
- **Live Verification:** Tests discovered credentials against the relevant provider's API to check if they are active, helping teams reduce false positives and prioritize live risks.
- **Open-Source and Enterprise:** Available as an open-source command-line tool on **GitHub** alongside enterprise monitoring and analysis features.

#### Common Usage

- **Scan a Git repository:** Run `trufflehog git https://github.com` from the terminal.
- **Scan local files:** Use `trufflehog filesystem /path/to/folder`.
- **Prevent leaks early:** Integrate it into CI/CD pipelines or Git pre-commit hooks to catch secrets before they are pushed.

### **25. Scenario: Your Kubernetes cluster is hit with a container escape attack. How does Defense-in-Depth contain the blast radius?**
**Answer:**

When a container escape happens in Kubernetes, an attacker breaks out of the container boundary onto the underlying host node. **Defense-in-Depth contains the blast radius by placing independent security barriers between the container, the host node, the Kubernetes control plane, and the underlying cloud infrastructure**, ensuring that compromising one layer does not grant cluster-wide or cloud-wide control.

**1. Runtime & Host Isolation (Stopping or Slowing the Breakout)**

Even if an application is exploited, hardened runtime boundaries prevent the attacker from freely controlling the host kernel or filesystem: 

- **Sandboxed Container Runtimes:** Using microVMs or user-space kernels like Kata Containers or gVisor adds a hypervisor or syscall-interception layer so a kernel exploit inside the container cannot directly reach the host OS kernel. 
- **Linux Kernel Hardening:** Enforcing **seccomp**, **AppArmor**, or **SELinux** profiles restricts the system calls and file paths a compromised process can access on the host. 
- **Immutable & Minimal Host OS:** Running stripped-down node operating systems (such as Bottlerocket or Flatcar) with read-only root filesystems removes local compilers, shells, and package managers that attackers rely on post-escape.

**2. Workload & Node Containment (Limiting Lateral Movement on the Node)**

Once an attacker lands on the node, they typically look for secrets, kubelet credentials, or high-privilege "trampoline" pods running on the same machine: 

- **Node & Workload Isolation:** Using node taints, tolerations, and dedicated node pools isolates untrusted or internet-facing workloads from sensitive internal workloads, preventing an attacker from harvesting co-located secrets. 
- **Short-Lived, Scoped Secrets:** Avoiding static secrets mounted on disk and using external secret managers with short TTLs minimizes what an attacker can steal from `/var/lib/kubelet/pods` on the compromised node. 
- **Pod Security Standards (PSS):** Enforcing the `Restricted` profile via Admission Controllers blocks `privileged` pods, `hostPath` mounts, and dangerous Linux capabilities (`SYS_ADMIN`, `SYS_PTRACE`), removing easy escalation paths. 

**3. Cluster Control Plane & Identity Boundaries (Protecting the API Server)**

A compromised node gives the attacker access to the node's **kubelet** identity. Strict identity and RBAC controls stop them from taking over the rest of the cluster:

- **NodeRestriction Admission Controller:** Kubernetes' built-in `NodeRestriction` plugin ensures a compromised kubelet can only modify Pod and Node objects bound specifically to its own node, preventing it from altering workloads elsewhere in the cluster.
- **Least-Privilege Service Accounts:** Disabling default service account token automounting and binding granular RBAC roles to individual workloads prevents an attacker from pivoting using stolen pod tokens.
- **Network Segmentation:** Default-deny **NetworkPolicies** (and CNI-level microsegmentation) block the compromised node or pod from scanning internal cluster services, databases, or administrative dashboards. 

**4. Cloud Infrastructure Boundaries (Preventing Cloud Account Takeover)**

In cloud-hosted clusters, attackers immediately target the cloud provider's control plane to escalate beyond Kubernetes: 

- **Blocking Instance Metadata Service (IMDS) Access:** Enforcing **IMDSv2** (requiring session tokens and a hop limit of 1) or blocking pod access to `169.254.169.254` stops escaped containers from stealing the node's underlying cloud IAM role.
- **Workload Identity Federation:** Mapping granular cloud IAM roles directly to specific Kubernetes Service Accounts (such as AWS IRSA or GCP Workload Identity) ensures that a node compromise does not grant broad permissions to S3 buckets, databases, or cloud APIs. 

**5. Runtime Detection & Automated Response (Cutting the Attack Short)**
Containment also depends on shrinking the time window between breakout and remediation: 

- **eBPF Behavioral Monitoring:** Tools like Falco or Tetragon detect escape indicators in real time—such as unexpected namespace changes (`setns`, `unshare`), host filesystem writes, or processes spawning outside a container's cgroup.
- **Automated Node Quarantine:** Triggering automated workflows to cordon, drain, isolate security groups, and terminate the compromised node evicts the attacker before they can establish persistence.

### **26. Scenario: Design an automated End-to-End Supply Chain Security Pipeline conforming to SLSA Level 3.**
**Answer:**

An **SLSA Level 3** supply chain pipeline **runs builds in ephemeral, isolated environments and generates non-falsifiable, cryptographically signed provenance from a trusted build platform**. Signing keys and attestation logic stay strictly separated from user-defined build steps so build scripts cannot tamper with or forge metadata.

#### 1. Source & Version Control

- **Protected repositories:** Require signed commits, branch protection rules, and peer reviews (such as two-person review for critical changes).
- **Immutable history:** Retain full Git history and audit logs indefinitely to prevent tampering or silent rewrites.
- **Pre-commit & CI checks:** Run automated secret scanning, linting, and dependency policy checks before merging.

#### 2. Hardened, Ephemeral Build Platform

- **Isolated execution:** Provision a fresh container or VM per build run and destroy it immediately after completion to prevent cross-build contamination or persistent malware.
- **Restricted network & access:** Limit outbound network access (ideally using an internal artifact proxy or mirror) and enforce least-privilege IAM roles.
- **Centralized build workflows:** Use platform-enforced or centralized reusable workflows rather than ad-hoc local or self-hosted scripts.

#### 3. SBOM, Scanning & Isolated Provenance Signing

- **Artifact & SBOM generation:** Build the artifact deterministically where possible, compute its cryptographic digest, and generate a Software Bill of Materials (SBOM) in CycloneDX or SPDX format.
- **Vulnerability & policy checks:** Scan dependencies and built artifacts for known vulnerabilities and policy violations.
- **Out-of-band attestation:** Have an external or isolated attester service—not user-controlled build steps—record trusted build parameters, source revision, and artifact digests.
- **Keyless or hardware-backed signing:** Sign provenance (for example, in-toto statements) using short-lived OIDC-issued certificates or isolated KMS/HSM keys so build steps cannot access signing material.

#### 4. Registry & Deployment Verification

- **Immutable storage:** Push signed artifacts, SBOMs, and provenance attestations to a tamper-evident registry (such as an OCI registry with immutability enabled).
- **Admission & deployment gates:** Enforce verification policies at release and runtime—such as a Kubernetes admission controller or deployment gate—that check signature validity, trusted builder identity, source repository, and vulnerability thresholds before allowing deployment.
- **Continuous auditing:** Log all build runs, attestations, and policy denials for ongoing security auditing.

### **63. Scenario: Migrate a legacy application with hardcoded database passwords to HashiCorp Vault Dynamic Secrets with zero downtime.**
**Answer:**
1. Deploy External Secrets Operator (ESO) / Vault Agent Sidecar injecting credentials into an in-memory volume (`/vault/secrets/database.env`).
2. Configure PostgreSQL to accept both legacy static passwords and dynamic Vault-generated user roles concurrently.
3. Update application to read dynamic credentials from `/vault/secrets/database.env`.
4. Deploy pods rolling fashion.
5. Delete legacy static database user after all old pods terminate.

### **64. How do you mitigate TCP SYN Flood DDoS attacks at the Linux Kernel and Load Balancer layers?**
**Answer:**
1. **Enable SYN Cookies:** `sysctl -w net.ipv4.tcp_syncookies=1` (server encodes state into `SYN-ACK` sequence number instead of allocating half-open connection memory).
2. **Tune Kernel Backlog:** `sysctl -w net.ipv4.tcp_max_syn_backlog=4096` and `sysctl -w net.core.somaxconn=4096`.
3. **Cloudflare / CloudFront Anycast:** Absorbs flood traffic at the global edge network before reaching origin servers.

### **65. How do you implement Zero-Trust Microsegmentation in Kubernetes using Cilium and WireGuard Transparent Encryption?**
**Answer:**
- **Cilium eBPF:** Enforces Layer 3 to Layer 7 NetworkPolicies without modifying application pods.
- **WireGuard Encryption:** Set `encryption.type: wireguard` in Cilium Helm values to automatically encrypt all node-to-node and pod-to-pod network traffic in kernel-space with zero proxy sidecar overhead.

### **66. Scenario: An audit discovers that engineers have direct SSH access to production EC2 instances. Implement a Zero-Trust Access Gateway with Teleport or AWS SSM.**
**Answer:**
1. Delete security group rules allowing inbound port 22; disable public IPs.
2. Deploy AWS SSM Managed Instance Core or Teleport Auth/Proxy service.
3. Integrate with corporate Okta / Entra ID SSO with mandatory MFA.
4. Engineers authenticate via SSO to receive short-lived SSH certificates (valid for 8 hours); all interactive sessions are recorded for compliance auditing.

### **67. What is Secrets Sprawl and how do you design an enterprise remediation pipeline?**
**Answer:**
1. **Pre-Commit:** Developers install `pre-commit` running `gitleaks protect` locally.
2. **Push Protection:** GitHub blocks commits containing detected high-entropy keys.
3. **Scheduled Scanning:** Nightly org-wide Trufflehog scans across all repositories.
4. **Automated Key Revocation:** Trigger automated Lambda functions via webhook to deactivate exposed keys within 60 seconds of commit detection.

### **68. What is DNS Tunneling / DNS Exfiltration and how is it blocked?**
**Answer:**
- **Attack:** Malware encodes stolen sensitive data into DNS subdomains (`stolen_data.evil-domain.com`) and queries an attacker-controlled authoritative name server, bypassing standard firewall HTTP filters.
- **Defense:** Enforce AWS Route 53 Resolver DNS Firewall / Pi-hole; detect high volumes of high-entropy subdomain queries and non-standard DNS query types (`TXT`, `NULL`).

### **69. Scenario: Securely connect two Kubernetes clusters in different AWS VPCs without public internet exposure.**
**Answer:**
1. Connect the two VPCs via **AWS Transit Gateway** or **VPC Peering** with non-overlapping CIDR blocks.
2. Use **Cilium Cluster Mesh** or **Submariner** for cross-cluster pod IP routing and service discovery.
3. Enforce mutual TLS (mTLS) across clusters using a shared root SPIFFE/SPIRE trust bundle or Istio Multi-Cluster Mesh.

### **70. What is Kubernetes Pod Security Standards (Privileged vs Baseline vs Restricted)?**
**Answer:**
- **Privileged:** Unrestricted; permits privileged containers, host networking, hostPath volumes.
- **Baseline:** Minimally restrictive default preventing known privilege escalation vectors.
- **Restricted:** Hardened standard mandating non-root execution, dropping all capabilities, read-only root filesystems, and seccomp profiles.

---

## 🔴 **Part 3: Advanced DevSecOps, Cryptography & Threat Detection (Questions 71–200)**

### **71. What is In-Toto Attestation Framework?**
**Answer:** A framework for cryptographically certifying the chain of custody across the entire software supply chain, recording metadata at each step (source commit, build runner, test execution).

### **72. What is Rekor Transparency Log?**
**Answer:** A public, append-only, tamper-evident cryptographic Merkle tree ledger that records software supply chain metadata and Cosign signatures.

### **73. What is Fulcio Certificate Authority?**
**Answer:** A free, public Certificate Authority managed by Sigstore that issues short-lived (10-minute) X.509 certificates linked to OIDC identities (GitHub Actions).

### **74. What is HashiCorp Vault Shamir's Secret Sharing?**
**Answer:** Cryptographic algorithm that splits the master unseal key into $N$ key shares, requiring a minimum threshold of $M$ keys (e.g., 3 of 5) to reconstruct the master key and unseal Vault.

### **75. What is Vault Auto-Unseal?**
**Answer:** Automates unsealing HashiCorp Vault using cloud KMS services (AWS KMS, Azure Key Vault, Google Cloud KMS) without manual human key entry.

### **76. What is Kubernetes KMS v2 Provider?**
**Answer:** High-performance encryption at rest API in Kubernetes 1.28+ that encrypts Secret resources in etcd using envelope encryption with cached DEKs.

### **77. What is AWS IAM Session Policies?**
**Answer:** Advanced policies passed when programmatically assuming an IAM role to further restrict the permissions granted by the role's identity-based policy for that specific session.

### **78. What is Confused Deputy Problem in Cloud IAM?**
**Answer:** A security issue where an entity that doesn't have permission to perform an action coerces a privileged service into performing it (mitigated using `sts:ExternalId` in IAM trust policies).

### **79. What is Certificate Transparency (CT)?**
**Answer:** An open framework of public, append-only logs recording all issued SSL/TLS certificates, preventing CAs from issuing fraudulent certificates secretly.

### **80. What is OCSP Stapling (Online Certificate Status Protocol)?**
**Answer:** A mechanism where the web server queries the CA periodically for certificate revocation status and attaches ("staples") the timestamped OCSP response directly to the TLS handshake, improving client speed and privacy.

### **81. What is HTTP Strict Transport Security (HSTS)?**
**Answer:** A security response header (`Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`) informing client browsers to communicate exclusively over HTTPS, blocking SSL stripping attacks.

### **82. What is Content Security Policy (CSP)?**
**Answer:** An HTTP header restricting the resources (scripts, images, stylesheets) that a browser is allowed to load, mitigating Cross-Site Scripting (XSS) and data injection attacks.

### **83. What is Cross-Origin Resource Sharing (CORS)?**
**Answer:** A browser security mechanism that uses HTTP headers (`Access-Control-Allow-Origin`) to allow or restrict resources requested on a web page from a different domain.

### **84. What is SameSite Cookie Attribute (`Strict`, `Lax`, `None`)?**
**Answer:** Protects against Cross-Site Request Forgery (CSRF) by controlling whether cookies are sent with cross-site requests.

### **85. What is Linux Capabilities Dropping in Containers?**
**Answer:** Explicitly stripping POSIX capabilities (`--cap-drop=ALL`) from container processes to prevent kernel exploitation during escapes.

### **86. What is Linux Seccomp (Secure Computing Mode) BPF Filter?**
**Answer:** Using Berkeley Packet Filters to intercept and allow or deny system calls invoked by container processes (e.g., blocking `reboot`, `sys_chroot`).

### **87. What is AppArmor Profile for Docker?**
**Answer:** A Linux kernel security module enforcing filesystem path access restrictions, networking limits, and raw socket creation rules on container processes.

### **88. What is SELinux Mandatory Access Control (MAC)?**
**Answer:** Assigns security labels (`system_u:system_r:container_t`) to processes and files, enforcing strict kernel authorization rules regardless of Linux root permissions.

### **89. What is eBPF Falco Rule Syntax?**
**Answer:** Declarative YAML rules defining matching conditions over kernel syscalls:
```yaml
- rule: Unauthorized Shell in Container
  desc: Detect shell spawned inside production container
  condition: container and proc.name in (bash, sh, zsh)
  output: Shell spawned (user=%user.name pod=%k8s.pod.name)
  priority: WARNING
```

### **90. What is BGP Route Hijacking?**
**Answer:** An attacker falsely announcing ownership of IP address blocks to upstream BGP routers, intercepting or blackholing internet traffic (mitigated via RPKI).

### **91. What is Resource Public Key Infrastructure (RPKI)?**
**Answer:** A cryptographic framework securing internet BGP routing by validating that an Autonomous System is authorized to announce specific IP address prefixes.

### **92. What is Anycast BGP Routing in Cloudflare?**
**Answer:** Multiple globally distributed edge servers announcing the exact same IP address via BGP, allowing client traffic to route to the topologically nearest data center.

### **93. What is WireGuard VPN Protocol?**
**Answer:** An ultra-fast, modern cryptographic VPN protocol running in Linux kernel-space using state-of-the-art cryptography (Curve25519, ChaCha20-Poly1305, BLAKE2s).

### **94. What is IPsec ESP (Encapsulating Security Payload)?**
**Answer:** An IPsec protocol providing data confidentiality, origin authentication, and anti-replay protection for IP network packets.

### **95. What is IPsec IKEv2 (Internet Key Exchange)?**
**Answer:** The protocol used to establish IPsec security associations (SAs), negotiating encryption algorithms and authenticating endpoints.

### **96. What is Generic Routing Encapsulation (GRE)?**
**Answer:** A tunneling protocol that encapsulates a wide variety of network-layer protocols inside virtual point-to-point links over IP networks.

### **97. What is VXLAN (Virtual Extensible LAN)?**
**Answer:** An overlay network encapsulation protocol that packages Layer 2 Ethernet frames inside Layer 4 UDP packets (port 4789), scaling network segmentation from 4,096 VLANs to 16 million VXLAN segments.

### **98. What is Geneve Protocol in CNI Networking?**
**Answer:** Generic Network Virtualization Encapsulation—a flexible network virtualization protocol used by Cilium and OVN to transmit metadata across overlay networks.

### **99. What is Proxy Protocol (v1 and v2)?**
**Answer:** A protocol prepending a header to TCP connections that conveys client connection information (real client source IP and port) across Layer 4 proxies (NLB/HAProxy) to backend servers.

### **100. What is Server Name Indication (SNI)?**
**Answer:** An extension to the TLS protocol where the client indicates the hostname it is connecting to in the `ClientHello` packet, allowing reverse proxies to serve multiple SSL certificates on a single IP address.

### **101. What is TCP Keepalive?**
**Answer:** Periodic probe packets sent over idle TCP connections to verify that the remote peer is still alive and prevent intermediate firewalls from dropping idle NAT entries.

### **102. What is TCP Window Size and Flow Control?**
**Answer:** The amount of data in bytes that a receiver is willing to buffer without receiving an acknowledgment, preventing fast senders from overwhelming slow receivers.

### **103. What is TCP Congestion Control (BBR vs CUBIC)?**
**Answer:**
- **CUBIC:** Loss-based congestion algorithm; scales back throughput when packet loss occurs.
- **BBR (Bottleneck Bandwidth and RTT):** Model-based congestion algorithm developed by Google that optimizes throughput and minimizes queue latency regardless of packet loss.

### **104. What is Ephemeral Port Exhaustion?**
**Answer:** When high volumes of short-lived outbound connections consume all available OS client ports (`1024-65535`), causing connection failures (`EADDRNOTAVAIL`). Prevented by enabling `tcp_tw_reuse` and using connection pooling.

### **105. What is TIME_WAIT Socket State in TCP?**
**Answer:** The final TCP connection state where the closing socket waits for $2 \times \text{MSL}$ (Maximum Segment Lifetime, usually 60s) to ensure delayed packets in-flight do not interfere with new connections.

### **106. What is SO_REUSEPORT Socket Option?**
**Answer:** Allows multiple application processes/threads on the same machine to bind to the exact same TCP/UDP port, with the Linux kernel balancing incoming connections across processes in kernel-space.

### **107. What is TCP Splicing (Zero-Copy)?**
**Answer:** Transferring data directly between two network sockets in kernel space (`splice()` syscall) without copying data into userspace memory buffers.

### **108. What is Network Interface Card (NIC) Offloading (TSO, LRO, Checksum Offload)?**
**Answer:** Delegating network packet processing tasks (TCP segmentation, checksum computation) to physical NIC hardware to save host CPU cycles.

### **109. What is Single Root I/O Virtualization (SR-IOV)?**
**Answer:** A hardware specification allowing a physical PCIe network card to present itself as multiple virtual network cards (Virtual Functions) directly to VMs/containers, bypassing hypervisor latency.

### **110. What is Data Plane Development Kit (DPDK)?**
**Answer:** A set of libraries that enables fast packet processing by moving network drivers into userspace and using poll-mode drivers, bypassing the Linux kernel network stack.

### **111. What is Linux Socket Filtering with eBPF (sockops)?**
**Answer:** Accelerates local inter-pod communication by redirecting packets directly from sending sockets to receiving sockets within kernel memory, bypassing the entire TCP/IP network stack.

### **112. What is BPF Compiler Collection (BCC)?**
**Answer:** A toolkit for authoring eBPF programs in C and Python to execute system analysis and network traffic inspection.

### **113. What is Calico Felix Agent?**
**Answer:** The daemon running on each node in a Calico CNI cluster that translates declarative Kubernetes NetworkPolicies into Linux iptables rules or eBPF maps.

### **114. What is HashiCorp Vault Agent Auto-Auth?**
**Answer:** Automates authentication of application workloads to Vault using Kubernetes ServiceAccount tokens or cloud IAM instance profiles.

### **115. What is Vault Response Wrapping?**
**Answer:** Encrypts a secret into a single-use token with a short TTL (e.g., 5 minutes). If an attacker intercepts the token, attempting to unwrap it alerts the legitimate recipient that the secret was compromised.

### **116. What is HashiCorp Vault Lease ID and Revocation?**
**Answer:** Every dynamic secret has an associated Lease ID. Applications must renew the lease before expiration; revoking the lease drops the associated database credentials immediately.

### **117. What is HashiCorp Vault Performance Replication vs Disaster Recovery Replication?**
**Answer:**
- **Performance Replication:** Replicates secrets to secondary clusters for local low-latency reads.
- **Disaster Recovery (DR) Replication:** Replicates secrets and configuration to a secondary standby cluster that can be promoted to Primary during total outages.

### **118. What is HashiCorp Sentinel vs OPA Rego?**
**Answer:**
- **Sentinel:** Proprietary HashiCorp policy engine embedded in Terraform Cloud and Vault Enterprise.
- **OPA Rego:** CNCF open-source, general-purpose policy engine supported across Kubernetes, Terraform, and cloud gateways.

### **119. What is Kyverno Generate Rule?**
**Answer:** Automatically generates new Kubernetes resources (e.g., generating default NetworkPolicies, ResourceQuotas, or Secret syncs) whenever a new Namespace is created.

### **120. What is Kyverno Pod Security Standards Validation?**
**Answer:** Pre-configured Kyverno policy packs that enforce CIS Kubernetes Benchmarks and Pod Security Standards in pure YAML.

### **121. What is Trivy Operator for Kubernetes?**
**Answer:** An in-cluster operator that continuously scans running workloads and container images for vulnerabilities, publishing results as native Kubernetes Custom Resources (`VulnerabilityReport`).

### **122. What is Grype SBOM Vulnerability Matching?**
**Answer:** Fast vulnerability scanner developed by Anchore that scans CycloneDX/SPDX SBOM files against upstream CVE databases without pulling container image layers.

### **123. What is Semgrep for SAST?**
**Answer:** A lightweight, fast static analysis tool matching code patterns using syntax tree queries to find security bugs across multiple languages.

### **124. What is CodeQL in GitHub Advanced Security?**
**Answer:** A semantic code analysis engine that treats application source code as a queryable database, discovering data flow paths and SQL injection vulnerabilities.

### **125. What is Dependabot Security Advisory Alert?**
**Answer:** Automated notifications in GitHub alerting developers when a repository dependency has a published CVE, submitting automated PRs with version patches.

### **126. What is GitHub Secret Push Protection?**
**Answer:** Scans commit payloads during `git push` and blocks the push if high-entropy secrets (AWS keys, Slack tokens) are detected.

### **127. What is Trufflehog Canary Tokens?**
**Answer:** Deceptively placed fake credentials (honeytokens) across infrastructure that trigger high-priority alerts to SecOps if an attacker attempts to use them.

### **128. What is Teleport Access Plane?**
**Answer:** An identity-native infrastructure access gateway providing zero-trust SSH, Kubernetes, database, and web access with short-lived X.509/SSH certificates and session recording.

### **129. What is Cloudflare Zero Trust (Access / Tunnel)?**
**Answer:** Replaces legacy corporate VPNs by exposing internal web applications securely through outbound-only encrypted tunnels behind cloud identity providers (Okta, Google Workspace).

### **130. What is AWS Verified Access?**
**Answer:** Provides secure, VPN-less access to internal corporate web applications using AWS IAM and third-party identity and device health signals.

### **131. What is DNS over HTTPS (DoH) vs DNS over TLS (DoT)?**
**Answer:**
- **DoH (Port 443):** Encapsulates DNS queries inside standard HTTPS traffic, blending in with web traffic.
- **DoT (Port 853):** Encrypts DNS queries directly over dedicated TLS connections.

### **132. What is DNS Anycast Routing in Root Servers?**
**Answer:** The 13 root name server IP addresses (`A` through `M`) are distributed across thousands of physical server nodes globally using Anycast BGP routing.

### **133. What is BGP Multi-Exit Discriminator (MED)?**
**Answer:** An optional non-transitive BGP attribute used to suggest to external peers which path is preferred into your Autonomous System when multiple ingress links exist.

### **134. What is BGP AS-Path Prepending?**
**Answer:** Artificially lengthening the AS-Path of advertised BGP routes by repeating your own AS number, making that path less attractive to external routers to establish primary/backup links.

### **135. What is BGP Local Preference?**
**Answer:** A BGP attribute configured locally within an Autonomous System to dictate which outbound exit path is preferred for outbound traffic (higher value is preferred).

### **136. What is IPsec Dead Peer Detection (DPD)?**
**Answer:** Periodic keepalive messages exchanged between IPsec endpoints to verify that the remote VPN gateway is reachable and drop stale security associations during network failures.

### **137. What is IPsec Perfect Forward Secrecy (PFS)?**
**Answer:** Ensures that the compromise of a long-term private key does not compromise past encrypted session keys, generating unique ephemeral Diffie-Hellman keys for every session.

### **138. What is SSL Stripping Attack?**
**Answer:** An attacker intercepts the initial unencrypted HTTP connection and prevents the client from upgrading to HTTPS, communicating with the server over HTTPS while proxying plain HTTP to the client (mitigated by HSTS).

### **139. What is Certificate Revocation List (CRL)?**
**Answer:** A signed, timestamped list published by a Certificate Authority containing serial numbers of certificates that have been revoked before their expiration date.

### **140. What is Let's Encrypt ACME Protocol (Automated Certificate Management Environment)?**
**Answer:** An automated protocol allowing web servers and cert-manager to request, validate (via HTTP-01 or DNS-01 challenges), and install free X.509 TLS certificates automatically.

### **141. What is ACME HTTP-01 vs DNS-01 Challenge?**
**Answer:**
- **HTTP-01:** Let's Encrypt verifies domain ownership by fetching a token hosted on `http://<domain>/.well-known/acme-challenge/`.
- **DNS-01:** Let's Encrypt verifies ownership by checking for a `_acme-challenge.<domain>` TXT record (required for Wildcard certificates).

### **142. What is Elliptic Curve Cryptography (ECC) vs RSA?**
**Answer:** ECC (e.g., ECDSA with Curve25519) provides equivalent cryptographic security to RSA with significantly smaller key sizes (256-bit ECC $\approx$ 3072-bit RSA), resulting in faster TLS handshakes and reduced CPU utilization.

### **143. What is ChaCha20-Poly1305 Cipher Suite?**
**Answer:** A high-speed symmetric stream cipher with authenticated data, optimized for mobile devices and processors lacking dedicated AES hardware acceleration.

### **144. What is Quantum-Resistant Cryptography (Post-Quantum Cryptography)?**
**Answer:** Cryptographic algorithms (e.g., Kyber, Dilithium) designed to withstand attacks by future quantum computers executing Shor's algorithm.

### **145. What is Diffie-Hellman Ephemeral (DHE / ECDHE)?**
**Answer:** Key exchange protocol where client and server exchange public parameters to establish a shared symmetric secret over an untrusted network with Perfect Forward Secrecy.

### **146. What is Hardware Security Module (HSM)?**
**Answer:** A dedicated physical cryptographic processor that securely generates, stores, and manages digital keys inside tamper-resistant hardware boundaries (FIPS 140-2 Level 3).

### **147. What is AWS KMS Multi-Party Computation (MPC)?**
**Answer:** Cryptographic protocol allowing multiple parties to compute a function over their inputs while keeping those inputs private, used in digital asset custody.

### **148. What is Zero-Knowledge Proof (ZKP)?**
**Answer:** A cryptographic method by which one party (prover) can prove to another party (verifier) that a statement is true without revealing any information beyond the validity of the statement.

### **149. What is HashiCorp Vault Key-Value (KV) v2 Secrets Engine?**
**Answer:** Provides versioned key-value storage with support for historical secret revisions, soft deletion, and undeletion.

### **150. What is HashiCorp Vault Cubbyhole Secrets Engine?**
**Answer:** An internal storage area scoped strictly to a specific authentication token, completely inaccessible to any other token.

### **151. What is HashiCorp Vault Token TTL and Max TTL?**
**Answer:**
- **TTL:** The lifespan of a token; can be renewed before expiration.
- **Max TTL:** The hard upper limit after which a token cannot be renewed and is permanently revoked.

### **152. What is HashiCorp Vault Periodic Token?**
**Answer:** A long-running service token that can be renewed indefinitely within its specified renewal interval without reaching a Max TTL limit.

### **153. What is HashiCorp Vault Entity and Alias?**
**Answer:**
- **Entity:** Represents an individual human or machine identity inside Vault.
- **Alias:** Maps identity records from external authentication backends (GitHub, Okta, Kubernetes) to a single unified Vault Entity.

### **154. What is HashiCorp Vault Sentinel Integration?**
**Answer:** Enforcing fine-grained organizational policy rules (Role-Governing Policies - RGPs) evaluated before Vault allows secret read/write operations.

### **155. What is HashiCorp Vault Transform Secrets Engine?**
**Answer:** Provides Format-Preserving Encryption (FPE) and data tokenization, encrypting sensitive fields (credit card numbers) while preserving data formats.

### **156. What is HashiCorp Vault KMIP Secrets Engine?**
**Answer:** Acts as a centralized Key Management Interoperability Protocol (KMIP) server for managing encryption keys used by VMware vSphere and storage arrays.

### **157. What is Kyverno ClusterAdmissionReport?**
**Answer:** A Kubernetes CRD generated by Kyverno listing all live resources in the cluster that violate defined governance policies.

### **158. What is Open Policy Agent Gatekeeper Constraint?**
**Answer:** The declarative instantiation of a `ConstraintTemplate` that targets specific Kubernetes kinds and namespaces with enforcement actions (`deny`, `warn`, `dryrun`).

### **159. What is Falco gRPC Output?**
**Answer:** Streaming runtime threat alerts from Falco directly to centralized SIEM platforms or automated remediation operators via gRPC.

### **160. What is Falco Sidekick?**
**Answer:** An open-source companion daemon that routes Falco security alerts to over 50 output destinations (Slack, PagerDuty, AWS Lambda, Kafka, Datadog).

### **161. What is Kubernetes Security Context `readOnlyRootFilesystem`?**
**Answer:** Enforces that the container's root filesystem is mounted as read-only, preventing attackers from downloading malware or backdoors into `/tmp` or `/bin`.

### **162. What is Kubernetes Security Context `allowPrivilegeEscalation: false`?**
**Answer:** Prevents child processes from gaining more privileges than their parent process (disables `setuid` binary execution).

### **163. What is Linux Network Namespace Veth Pair?**
**Answer:** Virtual Ethernet cable connecting a container's private network namespace to the host bridge or CNI router.

### **164. What is Linux Bridge (`brctl`) in Container Networking?**
**Answer:** A software-based Layer 2 switch created on the host OS to forward Ethernet frames between container veth interfaces and physical NICs.

### **165. What is Linux Iptables Connection Tracking (`conntrack`)?**
**Answer:** A kernel subsystem that tracks the state of all active network connections (NEW, ESTABLISHED, RELATED) to implement stateful firewall filtering and NAT.

### **166. What is Conntrack Table Exhaustion?**
**Answer:** When high concurrency network traffic exceeds the maximum entries in `/proc/sys/net/netfilter/nf_conntrack_max`, causing the Linux kernel to drop new packets.

### **167. What is Linux IP Forwarding (`net.ipv4.ip_forward=1`)?**
**Answer:** A kernel parameter allowing the Linux host to forward network packets between different network interfaces, required for container routing and Kubernetes CNIs.

### **168. What is Linux Reverse Path Filtering (`rp_filter`)?**
**Answer:** A kernel security feature that validates whether incoming packets on an interface arrive from an IP that matches the host's return route, mitigating IP spoofing.

### **169. What is Linux TCP Fast Open (TFO)?**
**Answer:** Enables data exchange directly within the initial TCP `SYN` packet for returning clients with valid TFO cookies, eliminating 1 RTT during connection setup.

### **170. What is Linux TCP SACK (Selective Acknowledgment)?**
**Answer:** Allows a receiver to acknowledge out-of-order packets, enabling the sender to retransmit *only* the specific missing segments rather than the entire window.

### **171. What is Linux Epoll vs Select?**
**Answer:** `select()` scales $O(N)$ with monitored file descriptors; `epoll()` uses event-driven kernel callbacks scaling $O(1)$, enabling servers to handle hundreds of thousands of concurrent connections.

### **172. What is Linux Socket SO_LINGER Option?**
**Answer:** Dictates whether an application process blocks on `close()` to wait for unsent data to transmit, or immediately sends a `RST` packet to terminate the socket.

### **173. What is TLS 1.3 Key Schedule?**
**Answer:** The cryptographic key derivation process utilizing HKDF (HMAC-based Key Derivation Function) to derive handshake and application traffic keys.

### **174. What is Encrypted Server Name Indication (ESNI) / Encrypted Client Hello (ECH)?**
**Answer:** Encrypts the destination hostname in the TLS `ClientHello` packet, preventing eavesdroppers and ISPs from observing which web domain a user is visiting.

### **175. What is HTTP/2 HPACK Header Compression?**
**Answer:** Compresses HTTP headers using Huffman coding and shared client/server dynamic header tables, eliminating redundant header transmission across requests.

### **176. What is HTTP/3 QPACK Header Compression?**
**Answer:** A header compression format designed specifically for QUIC over UDP that eliminates Head-of-Line blocking between independent streams.

### **177. What is DNS Rebinding Attack?**
**Answer:** An attacker tricks a client browser into making requests to an external domain that dynamically changes its DNS `A` record to `127.0.0.1` (localhost) or internal private IPs, bypassing the Same-Origin Policy.

### **178. What is Cross-Site Request Forgery (CSRF)?**
**Answer:** Tricking an authenticated user's browser into submitting unauthorized requests to a trusted web application using existing session cookies.

### **179. What is Server-Side Request Forgery (SSRF)?**
**Answer:** An attacker forces a backend web server to execute network requests to internal systems (e.g., querying `http://169.254.169.254` to steal AWS EC2 metadata credentials).

### **180. What is Command Injection Attack?**
**Answer:** An attacker executes arbitrary shell commands on the host OS by injecting malicious characters (`;`, `&&`, `|`) into un-sanitized user inputs passed to system execution functions.

### **181. What is Path Traversal Attack (Directory Traversal)?**
**Answer:** An attacker uses `../` sequences in input fields to access unauthorized files outside the web root directory (e.g., `/etc/passwd`).

### **182. What is Insecure Deserialization?**
**Answer:** An attacker passes crafted serialized objects (Python pickle, Java serialization) that execute malicious code during object reconstruction.

### **183. What is XML External Entity (XXE) Injection?**
**Answer:** Exploiting vulnerable XML parsers that process external entity references to read local server files or execute internal port scans.

### **184. What is Man-in-the-Middle (MitM) Attack?**
**Answer:** An attacker intercepts and potentially alters communication between two systems without their knowledge (mitigated via mTLS and certificate pinning).

### **185. What is SSL Strip / Downgrade Attack?**
**Answer:** An attacker intercepts TLS negotiation and forces the connection to fall back to insecure, legacy cipher suites or unencrypted HTTP.

### **186. What is Memory Corruption Attack (Buffer Overflow)?**
**Answer:** Writing more data into a memory buffer than allocated, overwriting adjacent memory and executing arbitrary payload code (mitigated via ASLR and memory-safe languages like Rust/Go).

### **187. What is Address Space Layout Randomization (ASLR)?**
**Answer:** A Linux kernel security feature that randomizes the memory addresses of program execution areas (stack, heap, libraries), preventing buffer overflow exploits.

### **188. What is Non-Executable Stack (NX / DEP)?**
**Answer:** Hardware feature marking data memory pages as non-executable, preventing code injected into buffers from executing.

### **189. What is Linux Kernel Module (LKM) Rootkit?**
**Answer:** Malicious code loaded directly into kernel memory that hooks system calls and hides processes, files, and network connections from userland monitoring tools.

### **190. What is eBPF Rootkit?**
**Answer:** Stealthy malware loaded into eBPF kernel hooks that modifies packet payloads or hides processes directly in kernel memory without modifying disk files.

### **191. What is Supply Chain Dependency Confusion Attack?**
**Answer:** An attacker registers a malicious package on a public repository (npm/PyPI) with the exact same name as an internal private package, tricking CI systems into downloading the public version.

### **192. What is Supply Chain Typosquatting Attack?**
**Answer:** Registering package names with slight misspellings of popular packages (e.g., `cross-env` vs `crossenv`) containing embedded cryptominers or backdoors.

### **193. What is Signed Commits in Git (GPG / SSH / S/MIME)?**
**Answer:** Cryptographically signing Git commits using private PGP or SSH keys, verifying that commits originated from an authentic developer identity.

### **194. What is Git Branch Protection Rule?**
**Answer:** Enforcing mandatory code review approvals, passing CI status checks, signed commits, and linear history before allowing merges to protected branches (`main`).

### **195. What is Two-Person Rule in Production Access?**
**Answer:** Requiring two independent authorized individuals to approve any privileged change or access request before execution.

### **196. What is Just-In-Time (JIT) Privileged Access?**
**Answer:** Granting temporary, time-bounded (e.g., 2-hour) elevated production permissions that automatically expire upon task completion.

### **197. What is Honeytoken in DevOps Security?**
**Answer:** Fake API tokens or database accounts intentionally planted in source code or repositories that alert SecOps when accessed, detecting attacker presence.

### **198. What is Zero Trust Device Posture Checking?**
**Answer:** Verifying client device compliance (disk encryption enabled, OS updated, antivirus running) before granting access to corporate resources.

### **199. What is Dynamic Data Masking (DDM)?**
**Answer:** Real-time obfuscation of sensitive database columns (masking credit card numbers to `XXXX-XXXX-XXXX-1234`) based on user querying permissions.

### **200. What is Enterprise DevSecOps Maturity Framework?**
**Answer:**
1. **Level 1 (Reactive):** Ad-hoc vulnerability patching after release.
2. **Level 2 (Shift-Left CI):** Automated SAST, SCA, and container image scanning in pipelines.
3. **Level 3 (Policy as Code):** Kyverno/OPA admission controllers, Cosign image signing, and SLSA Level 3 supply chain security.
4. **Level 4 (Zero Trust & Runtime):** SPIFFE/SPIRE workload attestation, mTLS microsegmentation, and Falco eBPF threat detection.