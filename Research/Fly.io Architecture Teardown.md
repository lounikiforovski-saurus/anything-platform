# Fly.io Sprites\_Sandbox\_Architecture.md

## Executive Synthesis: The Bare-Metal Sandboxing Paradigm

The architectural blueprint of Fly.io's "Sprites" represents a fundamental divergence from traditional, ephemeral, container-based sandboxing models utilized in modern cloud computing. Designed explicitly to host highly autonomous, potentially untrusted Artificial Intelligence (AI) agents, the Sprites architecture discards the standard Open Container Initiative (OCI) image provisioning model in favor of persistent, stateful micro-virtual machines (microVMs).1 This paradigm shift is engineered to solve a critical bottleneck in AI execution: the latency, computational overhead, and state-loss associated with rebuilding development environments for every individual execution cycle.4

The traditional approach to secure multi-tenant code execution relies on spinning up ephemeral Docker containers or utilizing user-space kernels.1 However, this model introduces significant cold-start latencies and limits the persistence of state, forcing agents to repeatedly download dependencies, reconfigure environments, and bootstrap runtimes.4 To achieve secure, high-density code execution on bare metal with sub-second cold starts and permanent state retention, the Sprites architecture relies on three foundational engineering pillars: hardware-virtualized isolation via the Linux Kernel Virtual Machine (KVM) and Firecracker, a highly customized tiered storage stack utilizing Litestream and S3-compatible object storage, and an unconventional "inside-out" orchestration model coordinated by a custom gossip protocol known as Corrosion.3

This exhaustive teardown dissects the specific isolation mechanisms, state persistence economics, and control plane integrations utilized to achieve this advanced bare-metal execution environment. The objective is to provide Hostopia/HostPapa with the precise architectural schemas required to replicate and scale an internal AI application platform capable of sustaining untrusted code execution without compromising host integrity or execution speed.

## 1. The Isolation Mechanism: MicroVMs vs. Traditional Containers

The state of the art in agent isolation has historically relied upon read-only sandboxes.9 However, autonomous AI agents, such as Anthropic's Claude, operate optimally when provided with full systemic access to a computational environment, requiring the ability to install dependencies, manipulate filesystems, and bind network ports.4 This requirement necessitates an isolation boundary that is significantly more robust than traditional Linux namespaces and cgroups.

### Hardware-Level Virtualization via Firecracker

To execute untrusted AI-generated code securely, the underlying Hostopia infrastructure must guarantee strict workload isolation without sacrificing boot speed or bare-metal performance. The Fly.io Sprites architecture achieves this through the deployment of Firecracker, an open-source virtual machine monitor (VMM) originally developed by Amazon Web Services to power AWS Lambda and AWS Fargate.7 Firecracker utilizes KVM to provision lightweight microVMs, granting each individual Sprite its own dedicated, unshared Linux guest kernel.1

The distinction between isolation models dictates the security posture of the entire bare-metal fleet. Platforms such as Modal rely on gVisor, which implements a user-space kernel that intercepts and filters system calls before passing them to the shared host kernel.1 While effective for standard containerized workloads, this approach introduces system call overhead and does not provide true hardware boundary isolation. Alternative multi-tenant architectures, such as those utilized by Northflank, deploy Kata Containers integrated with Cloud Hypervisor to achieve isolation.12 However, the Sprites architecture exclusively utilizes Firecracker to minimize the attack surface by stripping away unnecessary legacy device drivers and capabilities found in full-scale hypervisors like QEMU.11

The integration of Firecracker on bare-metal servers involves several low-level engineering implementations to ensure high density and low latency:

* **Minimalist Device Model:** Firecracker provisions only the most essential virtual devices to the guest operating system, primarily relying on virtio-net for network interfacing and virtio-block for storage management.7 By minimizing the emulated hardware footprint, the microVM achieves an astonishingly low memory overhead of less than 5 MiB per instance prior to application payload execution.13
* **Hardware Resource Dedication:** The host operating system relies on strict CPU pinning and resource allocation strategies to ensure that physical CPU cores process instructions for only one microVM at a time.14 This architecture prevents "steal time" contention, ensuring deterministic performance for the AI agent while mitigating the risk of processor-level side-channel attacks across the shared bare-metal hardware.14
* **Capacity Boundaries and Fleet Distribution:** The virtualized applications run on dedicated physical servers equipped with 8 to 32 physical CPU cores and 32GB to 256GB of RAM.14 A single Sprite microVM is inherently capped at a maximum configuration of 8 vCPUs and 8GB of RAM.6 Furthermore, the architecture is currently optimized exclusively for CPU workloads; GPU pass-through capabilities (such as those required for NVIDIA L4 or A100 provisioning) are not supported within the Firecracker-based Sprite instances, necessitating separate orchestration pipelines for hardware acceleration.12

![](data:image/png;base64...)

### Comparative Isolation Analysis for AI Agent Execution

The decision to utilize Firecracker dictates the platform's operational characteristics. A comparative analysis of leading AI sandbox isolation methodologies reveals the strategic trade-offs inherent in the Fly.io architectural blueprint.

| **Platform / Technology** | **Isolation Mechanism** | **Kernel Architecture** | **GPU Acceleration Support** | **State Persistence Philosophy** |
| --- | --- | --- | --- | --- |
| **Standard Docker/OCI** | Linux Namespaces / cgroups | Shared Host Kernel | High (NVIDIA Runtime) | Ephemeral / Volume Dependent |
| **Modal (gVisor)** | System Call Interception | User-Space Kernel | Low to None (Primarily CPU) | Ephemeral |
| **Northflank Sandboxes** | Kata Containers / Cloud Hypervisor | Dedicated Guest Kernel | High (L4, A100, H100, H200) | Configurable (Ephemeral or Stateful) 1 |
| **Fly.io Sprites (Firecracker)** | KVM Hardware Virtualization | Dedicated Guest Kernel | None (CPU Only) 12 | Strictly Stateful ("Disposable Computers") 1 |

### The Abandonment of OCI and the Introduction of Warm Worker Pools

A standard microVM provisioned by orchestration layers typically requires the host machine to pull an OCI-compliant container image (e.g., a Docker image) from a remote registry, unpack the filesystem layers, and mount them to a root filesystem before the guest kernel can initiate the boot sequence.17 This standard process introduces highly variable cold-start latencies. Fetching a large image across geographic regions can take seconds or even minutes, violating the sub-second latency constraints demanded by interactive AI agents.17

To eliminate this latency, the Sprites architecture fundamentally rejects the use of custom Docker or OCI container images.1 Instead of pulling unique, user-defined images for every execution environment, every single Sprite across the global fleet initializes from a standardized, identical base Linux container.3 This uniform base image comes pre-installed with the essential runtime dependencies required by AI coding agents, specifically including Python 3.13, Node.js 22.20, the Codex Command Line Interface (CLI), the Gemini CLI, and Anthropic's Claude Code.4

This architectural standardization unlocks a highly aggressive pre-allocation strategy known as "worker pooling".18 Because the bare-metal worker nodes know the exact immutable base image that any subsequent Sprite will require, the host orchestrator pre-boots pools of "empty" microVMs that idle in a dormant state on the host hardware.3 This methodology bypasses the network transit time associated with downloading massive image tarballs, effectively amortizing the creation cost before the user or the API ever requests the compute instance.17

### The Programmatic Boot Sequence: Engineering Sub-Second Cold Starts

The step-by-step programmatic boot sequence that achieves near-instantaneous code execution bypasses the traditional multi-step provisioning pipeline entirely. To understand how Sprites achieve creation times of 1.0 to 2.0 seconds and resume times of roughly 300 milliseconds, one must deconstruct the standard Fly Machines orchestration flow and contrast it with the optimized Sprites flow.3

In standard infrastructure, virtual machine creation is a prolonged, geographically constrained operation. If a developer issues a creation command from a local API proxy, that proxy must communicate with a strongly consistent centralized database to verify account standing, perform preflight authorization checks, and retrieve the immutable OCI image URL.17 Following authorization, the API server must broadcast a network message (historically via NATS) to available hosts in the target region, requesting them to reserve computing capacity.17 The hosts reply with offers, the API selects a host, the central database registers the new instance, and the selected host finally begins the prolonged process of downloading the image from an S3-backed repository.17 Fly.io engineers refer to this extensive lifecycle as the "92 step process".17

The Sprites architecture subverts this entire latency chain through pre-allocation and network-edge optimization:

1. **Request Ingestion & Edge Proxying:** An execution request (via the REST API or the sprite create CLI command) is received by the Fly Proxy at the closest geographic edge node, utilizing Anycast IP routing to minimize initial packet transit time.8
2. **API Preflight and Amortization:** Because the OCI pull has been eliminated by the standard base image, the local API server circumvents the prolonged capacity reservation dialogue. It instantly claims a pre-allocated, empty microVM from the warm pool located on a regional bare-metal worker.3
3. **Local Execution and Namespace Initialization:** The worker node assigns the specific sandbox identity to the pre-booted microVM. Because the Firecracker instance is already compiled and waiting on the physical hardware, the environment is handed over to the user via an active exec session in approximately 1 to 2 seconds for a total cold creation.6
4. **Instantaneous Boot Communication:** When a start command is issued to an existing, sleeping machine, the API server sends a direct message to the specific host owning the machine ID, bypassing central coordination entirely. If the user and the host are geographically proximate, this direct start command can execute in as little as 10 to 50 milliseconds.17

## 2. State Persistence & Storage Economics

Traditional code execution environments for AI agents rely on ephemeral containers, which enforce a strict, stateless architecture.9 If an agent hallucinated a command, made a syntax error, or corrupted a dependency tree, the ephemeral environment was destroyed, and the next session was forced to start from scratch, rebuilding the development environment, downloading packages, and authenticating APIs anew.4 The Sprites architecture takes the exact opposite approach, providing persistent, highly durable "disposable computers" that maintain continuous state across prolonged periods of inactivity.16

### The Hibernation Loop and Microsecond Checkpointing

To balance the exorbitant cost of persistent, dedicated compute with the necessity of stateful development environments, the architecture implements an aggressive, non-configurable "scale-to-zero" hibernation loop.4 After exactly 30 seconds of inactivity—defined by a lack of active HTTP ingress routing or active interactive exec shell sessions—the Sprite is automatically commanded to sleep.4 During this dormant state, CPU and RAM billing immediately ceases; the user is billed exclusively for the storage blocks consumed.4 Consequently, the microVM transitions through the platform's lifecycle states, moving from started to suspending and finally to suspended.22

This suspension process is not a graceful Operating System shutdown sequence. It is an instantaneous, transactional checkpointing mechanism powered by Firecracker's inherent snapshot capabilities.1

The exact methodology for capturing this state involves two simultaneous operations:

* **RAM Checkpointing:** The entirety of the microVM's active memory state—including running process trees, variable states, and kernel space allocations—is serialized and flushed directly to persistent local storage.20 When the machine is later commanded to wake, the Firecracker VMM deserializes this snapshot back into physical RAM, circumventing the Linux OS boot sequence entirely. This "warm resume" restores the sandbox to full operational status in approximately 100 to 500 milliseconds.20
* **Disk Checkpointing and Copy-on-Write Filesystems:** Simultaneously, a transactional snapshot of the Ext4 filesystem is captured.6 The storage stack employs a highly optimized copy-on-write implementation. This ensures that the snapshot captures only the writable overlay—the precise delta of modifications, installed packages, and file changes made by the AI agent—rather than duplicating the entire immutable 100GB base image.4 This design is explicitly described as "TRIM friendly," meaning that as the agent deletes temporary files, the allocated blocks are freed, and the ongoing storage bill is dynamically reduced.4

The checkpoint mechanism is exposed directly to the user and the autonomous agent via a clean REST API (POST /v1/sprites/{name}/checkpoint) and the internal CLI tool, sprite-env.4 The system automatically maintains a localized history of the environment, mounting the last five system checkpoints directly into the microVM's filesystem at the path /.sprite/checkpoints.4 This allows for direct file recovery without requiring a total system restoration. If an execution sequence fails catastrophically or corrupts the root environment, an API call (e.g., sprite-env checkpoints restore v1) allows the user to instantaneously roll back the entire RAM and disk state to a pristine condition in under a second.1 The architecture even leverages pre-installed capabilities to teach agents like Claude Code how to self-manage these checkpoints, creating an autonomous, self-healing development loop.4

### Storage Backing: Decoupling Data Gravity

Providing 100GB of persistent, durable storage to every rapidly scaling microVM presents a significant distributed systems challenge.2 Relying solely on local, attached Non-Volatile Memory Express (NVMe) arrays would lead to extreme data gravity. If the state was tied exclusively to physical hardware, a failure of the underlying bare-metal server would result in catastrophic data loss for the user.3 Conversely, relying entirely on network-attached storage or remote S3-compatible object storage for live filesystem operations would introduce unacceptable read/write latency, suffocating the performance of basic command-line utilities and package managers.3

To solve this latency-durability dichotomy, the Sprites storage blueprint relies on a heavily modified, tiered storage stack based on the JuiceFS architectural model.3 This model fundamentally decouples block data payloads from filesystem metadata mapping.

The architecture is bifurcated into two distinct layers:

1. **The Durable Root (Data Chunks):** The absolute, immutable source of truth for all data written within a Sprite is an S3-compatible object storage layer (often utilizing Tigris for S3 compatibility).2 Files manipulated by the agent are chunked and stored as raw, immutable blobs on the remote object store.3 Object storage provides near-infinite scalability, geographic redundancy, and supreme data durability, preventing loss during host hardware failure.2
2. **The Fast Read-Through Cache (NVMe):** To mitigate the latency of network-attached object storage, a sparse 100GB Ext4 volume is mounted to the Firecracker microVM from the bare-metal host's physically attached NVMe drives.1 Crucially, this local drive functions exclusively as a read-through cache.3 When a process attempts to access a file, the storage stack pulls the requisite chunks from the remote object store and caches them locally on the NVMe drive to eliminate read amplification for subsequent accesses.3 The physical state of the NVMe drive is inherently ephemeral; if the host machine experiences a critical fault, the cache is destroyed, but the underlying chunk data remains perfectly preserved in the S3 bucket.3

![](data:image/png;base64...)

### SQLite and Litestream: The Metadata Synchronization Engine

If the physical file data resides in S3 and the local NVMe array acts only as an ephemeral cache, the microVM requires a highly performant, continuously available mechanism to map virtual file paths to object storage chunks. In standard, unoptimized JuiceFS deployments, this crucial metadata mapping is managed by an external database instance, commonly Redis.26 The Sprites architecture completely redesigns this mechanism, replacing the external Redis dependency with a localized, highly optimized SQLite database embedded directly within the storage stack of the individual microVM.3

This localized SQLite database serves as the absolute "block map" for the filesystem, keeping a real-time record of which in-use storage blocks correspond to which files.16 Because the map must survive the lifecycle of the microVM and any potential hardware failures, it is made highly durable using an open-source tool known as **Litestream**.3 Litestream operates as a background process that continuously monitors the SQLite Write-Ahead Log (WAL).26 As the AI agent writes data and the SQLite metadata map updates, Litestream streams these incremental frame changes directly to the same S3-compatible object storage that holds the payload file chunks.3

This architecture introduces a massive operational advantage during cold-start or host-migration scenarios. If the physical hardware hosting the Sprite fails, the Fly.io orchestrator immediately provisions a new Sprite instance on a different physical server in the fleet.27 During the sub-second boot process, the new microVM utilizes a recently developed feature called the **Litestream Writable Virtual File System (VFS)**.28

Historically, migrating a database meant the system had to pause and download the entire multi-megabyte metadata file from S3 before the system could boot.28 Litestream VFS eliminates this bottleneck. It allows the booting Sprite to execute point-in-time SQLite queries "hot," reading the metadata directly off the remote object storage blobs without downloading the entire database first.28 This enables the filesystem to mount instantly. The NVMe cache is then repopulated asynchronously as the AI agent begins requesting file reads, ensuring that the environment feels persistently available despite migrating across distinct physical hardware.3 Furthermore, recent advancements in Litestream (v0.5.0) have introduced point-in-time recovery and a new LTX file format specifically designed for efficient transaction interchange, solidifying the reliability of this storage layer.29

## 3. Kubernetes / Bare-Metal Orchestration: Bypassing Consensus

Achieving sub-second orchestration and state synchronization across thousands of globally distributed bare-metal servers requires bypassing the heavy abstraction layers typically associated with Kubernetes or traditional centralized control planes.

While the broader Fly.io ecosystem offers an orchestration product called Fly Kubernetes (FKS)—which utilizes k3s, Custom Resource Definitions (CRDs), and a Virtual Kubelet architecture to map standard Kubernetes Pods to Firecracker Fly Machines—the Sprites platform intentionally avoids this Kubernetes overhead entirely.30 The standard FKS model relies on a complex stack including CoreDNS, persistent SQLite mapping (via kine), and standard Kubernetes declaration reconciliation loops.33 Sprites discard this declarative YAML complexity in favor of a proprietary, localized, and highly distributed bare-metal control plane designed specifically for raw speed.10

### Inside-Out Orchestration: Reversing the Host-Guest Paradigm

In traditional cloud hosting environments, user applications and virtual machines are managed by two strictly separated layers: the host operating system, which orchestrates workloads, monitors state, and handles networking, and the guest VM, which blindly executes the payload.3 The Sprites architecture radically inverses this dynamic through a concept documented as "Inside-Out Orchestration".2

Because the Sprite environment is deeply customized and uniform across the fleet, the most critical orchestration and management components are relocated directly into the guest microVM's root namespace.3

* **The Service Manager:** The process supervisor responsible for managing user code, handling automatic process restarts upon waking from the hibernation loop, and executing initialization commands resides entirely inside the VM boundary.3
* **Dynamic Network Binding:** If an untrusted agent executes a command that attempts to bind a local server to \*:8080 (a common requirement for web development agents), the internal orchestrator detects this action within the root namespace.3 It then automatically negotiates port forwarding and establishes public internet access via HTTP over the Fly Proxy, without requiring any declarative external host intervention or complicated ingress YAML configurations.3
* **Storage Management Integration:** The entire checkpoint/restore loop execution, the SQLite metadata management, and the Litestream synchronization processes run completely from within the microVM boundary rather than on the host hypervisor.3

By placing the orchestration burden inside the guest, the bare-metal host is relieved of massive computational overhead. The host's primary responsibility is reduced to simply providing raw physical CPU, provisioning memory, and routing incoming WireGuard network packets to the microVM interface.3

### Control Plane Integration: Corrosion and Gossip-Based State

Because the Sprites platform avoids a centralized PostgreSQL or etcd database for live orchestration state (which would introduce insurmountable geographic latency constraints during rapid scale-up/scale-down cycles), it relies entirely on a custom-built, Rust-based service discovery and state synchronization system known as **Corrosion**.28

Corrosion operates as a globally replicated SQLite database utilizing a highly distributed mesh network architecture, representing a stark departure from traditional distributed consensus models utilized by standard orchestrators:

* **Rejection of Raft and Centralized Consensus:** Mainstream orchestration systems like Kubernetes (backed by etcd) or Nomad (backed by Consul) rely heavily on distributed consensus algorithms like Raft.34 Raft requires strict leader election and cluster quorum to commit state changes.37 At a global scale with thousands of nodes, maintaining this quorum creates severe latency bottlenecks, as every state change (e.g., a microVM waking from sleep) must be verified by a majority of nodes globally.38 Corrosion rejects this model entirely, operating without distributed consensus, central servers, or database locking mechanisms.34
* **The SWIM Gossip Protocol:** To achieve rapid cluster membership awareness and global state propagation, Corrosion utilizes the Scalable Weakly-consistent Infection-style Process Group Membership (SWIM) protocol.34 Every bare-metal worker node and edge proxy in the fleet acts as an independent node in the network.40 Instead of broadcasting state changes to the entire cluster simultaneously, each node continuously exchanges state heartbeats with a small, randomized subset of neighboring nodes.34 If a node fails to respond, it is marked as "suspect," preventing network partition cascades.34 This peer-to-peer infection-style routing propagates orchestrator state globally in under a second.34
* **QUIC Transport and CRDT Synchronization:** Data packet exchanges between the thousands of individual nodes are handled via the QUIC transport protocol, prioritizing high-speed, low-latency transmission over standard TCP connections.38 To ensure that the massive, decentralized SQLite database across all nodes eventually converges to an identical state without merge conflicts, Corrosion leverages Conflict-free Replicated Data Types (CRDTs).38 Updates to the database rows are merged using logical timestamps (causal ordering) rather than wall-clock time, implementing a highly efficient last-write-wins (LWW) resolution mechanism for the orchestration state.34

When a Sprite is initialized, wakes from hibernation, or is assigned a unique public HTTPS URL, the microVM injects this state update directly into the local worker's instance of Corrosion. This row update is collected, bundled into a batched update packet, and gossiped across the global fleet.3 Because every worker server acts as the absolute source of truth for its own resident microVMs, there is no need for globally coordinated orchestration approval.34

### Comparative Orchestration Frameworks

The deployment of Corrosion marks a profound shift in bare-metal state management. The table below illustrates the differences between traditional consensus-based orchestration (like Kubernetes/Nomad) and the decentralized gossip protocol utilized by the Sprites architecture.

| **Orchestration Metric** | **Traditional (Consensus-Based)** | **Sprites Architecture (Corrosion)** |
| --- | --- | --- |
| **State Storage Engine** | etcd (Key-Value) or Consul | Localized SQLite |
| **Data Synchronization** | Raft Consensus (Quorum Required) | CRDTs (Conflict-free Replicated Data Types) 38 |
| **Network Protocol** | TCP / gRPC | SWIM Gossip Protocol + QUIC Transport 34 |
| **Conflict Resolution** | Strict Leader Election | Logical Timestamps (Last-Write-Wins) 34 |
| **Source of Truth** | Centralized Control Plane | Decentralized (Worker Node Authority) 34 |

### Egress Networking and The WireGuard Mesh

The final layer of the bare-metal architecture focuses on network isolation, proxy integration, and dynamic routing. Because Sprites execute untrusted, third-party code and autonomous agent scripts, network egress must be tightly controlled at the hypervisor level to prevent malicious activity or internal network probing.

The platform utilizes a comprehensive internal mesh network powered by WireGuard, known internally as "6PN" (IPv6 Private Network), combined with eBPF-based private networking mechanisms to secure traffic.8

* **The Fly Proxy:** Every physical edge node and bare-metal worker across the globe runs a proprietary Rust-based, TLS-terminating Anycast proxy known as the Fly Proxy.8 When an end-user or an API request enters the network via Anycast routing, the Fly Proxy located at the nearest edge server inspects the request headers, redirects HTTP to HTTPS, and queries its localized Corrosion database replica to find the exact physical location of the target Sprite microVM.8 The proxy then routes the TCP connection over an encrypted WireGuard backhaul tunnel directly to the worker server hosting the Firecracker instance.8
* **Auto-Waking via Proxy Interception:** The Proxy is deeply integrated with the hibernation loop. If a request targets a Sprite that has entered its 30-second sleep state, the Fly Proxy detects the suspended status in the Corrosion database. It intercepts and holds the HTTP request, and issues an immediate wake command to the specific bare-metal worker.8 The worker restores the RAM snapshot in ~300ms, and the proxy seamlessly forwards the held request, resulting in a near-imperceptible delay for the end user.4
* **Layer 3 Network Policies:** Egress control is managed programmatically at the network layer to limit the blast radius of compromised agents. Users can define strict DNS-based allow/deny lists, restricting the microVM from accessing anything outside of strictly approved domains. By default, environments are provisioned with an LLM-friendly list allowing access only to necessary package registries and GitHub, preventing the microVM from initiating arbitrary outbound connections.4

## Strategic Implications for Hostopia/HostPapa Infrastructure

The engineering paradigms executed within the Sprites architecture provide a highly precise, replicable blueprint for Hostopia/HostPapa's internal AI platform development. Extracting these methodologies reveals a profound shift away from traditional cloud-native tooling when dealing with hostile, autonomous, or highly dense workloads.

The extraction underscores that **standard OCI container lifecycles and traditional Kubernetes orchestration models (K3s/CRDs) are fundamentally misaligned with the latency and density requirements of persistent AI agents.** Relying on container image registries guarantees slow cold starts. Relying on etcd and Raft consensus guarantees control plane latency and regional scaling bottlenecks.

To achieve true bare-metal efficiency for AI agents, an architecture must implement **hardware-level virtualization** via KVM and Firecracker to ensure definitive security boundaries. It must leverage **standardized base images and warm worker pools** to eliminate OCI provisioning latency. It requires a **decentralized, gossip-based control plane** (mirroring Corrosion via SWIM and CRDTs) to bypass quorum-based orchestration limits. Finally, the decoupling of state via **read-through NVMe caching and SQLite/Litestream metadata replication** proves that it is technologically possible to provide massive, stateful persistent disks to ephemeral compute instances without succumbing to the fatal risks of localized data gravity. Implementing these specific architectural blueprints allows for the secure, instantaneous execution of dynamic agent code with the persistence, durability, and operational feel of a dedicated physical computer.

#### Works cited

1. E2B vs Modal vs Fly.io Sprites for AI code execution sandboxes | Blog - Northflank, accessed February 22, 2026, <https://northflank.com/blog/e2b-vs-modal-vs-fly-io-sprites>
2. The Design & Implementation of Sprites | daily.dev, accessed February 22, 2026, <https://app.daily.dev/posts/the-design-implementation-of-sprites-7gy2ixoa4>
3. The Design & Implementation of Sprites - Fly.io, accessed February 22, 2026, <https://fly.io/blog/design-and-implementation/>
4. Fly's new Sprites.dev addresses both developer sandboxes and API sandboxes at the same time - Simon Willison's Weblog, accessed February 22, 2026, <https://simonwillison.net/2026/Jan/9/sprites-dev/>
5. Fly.io introduces Sprites: lightweight, persistent VMs to isolate agentic AI - DevClass.com, accessed February 22, 2026, <https://www.devclass.com/ai-ml/2026/01/13/flyio-introduces-sprites-lightweight-persistent-vms-to-isolate-agentic-ai/4079557>
6. sandbox-environments.md - ghuntley/how-to-ralph-wiggum - GitHub, accessed February 22, 2026, <https://github.com/ghuntley/how-to-ralph-wiggum/blob/main/references/sandbox-environments.md>
7. What is AWS Firecracker? The microVM technology, explained | Blog - Northflank, accessed February 22, 2026, <https://northflank.com/blog/what-is-aws-firecracker>
8. Fly Proxy · Fly Docs - Fly.io, accessed February 22, 2026, <https://fly.io/docs/reference/fly-proxy/>
9. Code And Let Live · The Fly Blog, accessed February 22, 2026, <https://fly.io/blog/code-and-let-live/>
10. Fly.io puts AI agents in VMs, not containers - Techzine Global, accessed February 22, 2026, <https://www.techzine.eu/news/devops/137884/fly-io-puts-ai-agents-in-vms-not-containers/>
11. Firecracker vs QEMU — E2B Blog, accessed February 22, 2026, <https://e2b.dev/blog/firecracker-vs-qemu>
12. Top Fly.io Sprites alternatives for secure AI code execution and sandboxed environments, accessed February 22, 2026, <https://northflank.com/blog/top-fly-io-sprites-alternatives-for-secure-ai-code-execution-and-sandboxed-environments>
13. Firecracker, accessed February 22, 2026, <https://firecracker-microvm.github.io/>
14. The Fly.io Architecture · Fly Docs, accessed February 22, 2026, <https://fly.io/docs/reference/architecture/>
15. Top Blaxel alternatives for AI sandbox and agent infrastructure in 2026 | Blog - Northflank, accessed February 22, 2026, <https://northflank.com/blog/top-blaxel-alternatives-for-ai-sandbox-and-agent-infrastructure>
16. The Design & Implementation of Sprites - Simon Willison's Weblog, accessed February 22, 2026, <https://simonwillison.net/2026/Jan/15/the-design-implementation-of-sprites/>
17. Fly Machines: an API for fast-booting VMs · The Fly Blog - Fly.io, accessed February 22, 2026, <https://fly.io/blog/fly-machines/>
18. Simon Willison on fly, accessed February 22, 2026, <https://simonwillison.net/tags/fly/>
19. Sprites (Fly.io) | Ry Walker Research, accessed February 22, 2026, <https://rywalker.com/research/sprites>
20. Machine Suspend and Resume - Fly.io, accessed February 22, 2026, <https://fly.io/docs/reference/suspend-resume/>
21. serverless-claude-code-platform-comparison.md - GitHub Gist, accessed February 22, 2026, <https://gist.github.com/alexfazio/dcf2f253d346d8ed2702935b57184582>
22. Machine states and lifecycle · Fly Docs, accessed February 22, 2026, <https://fly.io/docs/machines/machine-states/>
23. AI Agent Sandboxes Compared | Ry Walker Research, accessed February 22, 2026, <https://rywalker.com/research/ai-agent-sandboxes>
24. Code and Let Live - Hacker News, accessed February 22, 2026, <https://news.ycombinator.com/item?id=46557825>
25. App Availability and Resiliency · Fly Docs - Fly.io, accessed February 22, 2026, <https://fly.io/docs/apps/app-availability/>
26. xleliu/mystars: Update my stars by github actions, accessed February 22, 2026, <https://github.com/xleliu/mystars>
27. Simon Willison on thomas-ptacek, accessed February 22, 2026, <https://simonwillison.net/tags/thomas-ptacek/>
28. Litestream Writable VFS · The Fly Blog - Fly.io, accessed February 22, 2026, <https://fly.io/blog/litestream-writable-vfs/>
29. Fly.io - Bluesky, accessed February 22, 2026, <https://bsky.app/profile/fly.io>
30. superfly/fly-k3s: Provision a k3s cluster on top of fly machines - GitHub, accessed February 22, 2026, <https://github.com/superfly/fly-k3s>
31. Fly Kubernetes features · Fly Docs, accessed February 22, 2026, <https://fly.io/docs/kubernetes/fks-features/>
32. Fly Kubernetes does more now · The Fly Blog - Fly.io, accessed February 22, 2026, <https://fly.io/blog/fks-beta-live/>
33. Introducing Fly Kubernetes · The Fly Blog, accessed February 22, 2026, <https://fly.io/blog/fks/>
34. Corrosion · The Fly Blog, accessed February 22, 2026, <https://fly.io/blog/corrosion/>
35. parking\_lot: ffffffffffffffff... · The Fly Blog - Fly.io, accessed February 22, 2026, <https://fly.io/blog/parking-lot-ffffffffffffffff/>
36. Corrosion - Hacker News, accessed February 22, 2026, <https://news.ycombinator.com/item?id=45680583>
37. Carving The Scheduler Out Of Our Orchestrator - Fly.io, accessed February 22, 2026, <https://fly.io/blog/carving-the-scheduler-out-of-our-orchestrator/>
38. Fast Eventual Consistency: Inside Corrosion, the Distributed System Powering Fly.io - InfoQ, accessed February 22, 2026, <https://www.infoq.com/news/2025/04/corrosion-distributed-system-fly/>
39. Fast Eventual Consistency: Inside Corrosion, the Distributed System Powering Fly.io, accessed February 22, 2026, <https://qconlondon.com/presentation/apr2025/fast-eventual-consistency-inside-corrosion-distributed-system-powering-flyio>
40. Infra Log - Fly, accessed February 22, 2026, <https://fly.io/infra-log/>
41. 2024-11-30 · Infra Log - Fly.io, accessed February 22, 2026, <https://fly.io/infra-log/2024-11-30/>
42. Reliability: It's Not Great - Fly.io Community, accessed February 22, 2026, <https://community.fly.io/t/reliability-its-not-great/11253>
43. Fly Kubernetes - Hacker News, accessed February 22, 2026, <https://news.ycombinator.com/item?id=38685393>
44. initial load after a period of no requests - Fly.io Community, accessed February 22, 2026, <https://community.fly.io/t/initial-load-after-a-period-of-no-requests/17370>