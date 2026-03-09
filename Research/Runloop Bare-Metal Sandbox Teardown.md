# Runloop AI: Architectural Teardown of Bare-Metal Sandboxing for Agentic Workloads

The rapid evolution of autonomous artificial intelligence, specifically in the domain of software engineering agents, has precipitated a fundamental architectural crisis in cloud infrastructure. Traditional orchestration environments—typified by shared-kernel Linux containers and asynchronous Kubernetes reconciliation loops—were engineered for long-running, predictable microservices.1 They are structurally inadequate for hosting non-deterministic, highly volatile agentic workloads that require the ability to execute arbitrary code, manipulate file systems, and bind network ports instantaneously.1 To mitigate the severe security risks of executing untrusted AI-generated code while maintaining the millisecond responsiveness required for interactive reasoning loops, infrastructure must shift toward highly optimized, bare-metal virtualization.1

This architectural teardown comprehensively reverse-engineers the infrastructure, orchestration, and security methodologies employed by Runloop, an enterprise-grade AI infrastructure platform explicitly designed for scaling agentic sandboxes. By dissecting Runloop's "defense-in-depth" isolation strategies, its implementation of asynchronous Suspend & Resume mechanics, its pre-warmed snapshotting architectures, and its native Model Context Protocol (MCP) integrations, this report provides a granular blueprint for constructing a sovereign, high-density AI execution platform on bare-metal hardware.

## 1. Defense-in-Depth Isolation: Beyond Shared Kernels

The absolute prerequisite for any platform offering "AI sandboxing as a service" is the mathematical guarantee of workload isolation. When an external AI agent is granted access to a terminal environment to test generated code, the platform assumes maximum risk; the agent (or a malicious prompt hijacking the agent) possesses the theoretical capability to execute container escapes, probe internal networks, and compromise the host infrastructure.

### The Vulnerability of OCI Containers

The initial approach to isolating code execution typically involves standard Open Container Initiative (OCI) runtimes, predominantly Docker. While Docker provides excellent deployment velocity and ecosystem familiarity, its isolation boundaries rely entirely on Linux kernel features—specifically namespaces and control groups (cgroups).1 The fatal architectural flaw is that all containers on a host share the identical underlying Linux kernel. If an AI agent generates a payload that exploits an unpatched kernel vulnerability, it can break out of its namespace and gain root access to the physical host, instantly compromising every other tenant's sandbox residing on that bare-metal machine.1 Consequently, relying exclusively on standard Docker containers for executing untrusted agentic code is categorized as an unacceptable enterprise security posture.1

### Runloop's Two-Layer Architecture: MicroVMs + Containers

To neutralize this threat, Runloop implements a rigorous "defense-in-depth" architecture that fundamentally divorces the execution environment from the host operating system. The platform does not rely on containers alone; it wraps every individual execution environment within a hardware-virtualized micro-Virtual Machine (microVM).2

While Runloop does not explicitly disclose the exact hypervisor underpinning its proprietary stack in its public-facing marketing, architectural parallels strongly suggest the utilization of KVM-backed microVM technologies, similar in profile to AWS Firecracker or highly tuned instances of Cloud Hypervisor / Kata Containers.1 This approach instantiates a dedicated, mathematically isolated guest kernel for every single sandbox.3 To compromise the host, an attacker would be forced to execute a chained exploit: successfully breaking out of the container runtime, and subsequently finding a zero-day vulnerability in the hypervisor boundary to escape the microVM.1

However, a raw microVM is often devoid of the tooling developers expect. Runloop bridges this UX gap by running a standard container environment *inside* the microVM.2 This dual-layer architecture provides the absolute security guarantee of hardware virtualization (the microVM) combined with the extreme flexibility and developer familiarity of OCI images (the container). Users can bring any standard Docker image, and Runloop's orchestrator will securely wrap it in a microVM envelope before execution on the bare-metal fleet.2

| **Isolation Metric** | **Standard Docker** | **Runloop Sandbox** |
| --- | --- | --- |
| **Boundary Mechanism** | Software (Namespaces) | Hardware (KVM MicroVM) + Software |
| **Kernel Architecture** | Shared Host Kernel | Dedicated Guest Kernel per Sandbox |
| **Security Posture** | Low (Vulnerable to Kernel Exploits) | Maximum (Defense-in-Depth) |
| **Startup Latency** | Milliseconds | Sub-second (Optimized) |

### Egress Controls and Network Isolation

Isolation must extend beyond CPU and memory to encompass network topography. An untrusted agent cannot be allowed to arbitrarily scan the host provider's internal Virtual Private Cloud (VPC) or execute distributed denial-of-service (DDoS) attacks against external targets.

Runloop enforces strict, programmatic network boundaries at the sandbox level. Utilizing outbound routing configurations, the platform restricts sandbox egress traffic exclusively to the public internet, completely severing any potential connection to the platform's internal management networks or adjacent tenant sandboxes. Furthermore, the architecture supports granular allow-lists and deny-lists, enabling administrators to explicitly restrict an agent's network access to a handful of approved external APIs (e.g., github.com and pypi.org), neutralizing the threat of data exfiltration via malicious HTTP POST requests.

## 2. Orchestration and Bare-Metal Density Optimization

Achieving security via microVMs is mathematically sound, but it introduces a severe economic and performance challenge. Booting a traditional virtual machine is a high-latency operation, often requiring several seconds to initialize the kernel, mount file systems, and allocate memory. For an interactive AI agent that requires instantaneous execution feedback to maintain its cognitive reasoning loop, a multi-second delay is disastrous. Furthermore, maintaining thousands of persistent VMs actively running in RAM destroys the resource density of the bare-metal hosts, rendering the business model economically unviable.

Runloop solves the latency and density problems through sophisticated orchestration mechanics: Pre-warmed Snapshotting and Asynchronous Suspend/Resume lifecycles.

### Bypassing Cold Starts: The Pre-Warmed Snapshot Engine

To achieve the sub-second provisioning times necessary for fluid AI interaction, Runloop's orchestration engine bypasses the traditional "cold boot" sequence entirely. The platform does not download an image, unpack the filesystem, and boot the kernel linearly when a user API request is received.

Instead, the architecture relies heavily on VM snapshotting.4 When a developer defines a custom sandbox environment (e.g., an environment pre-loaded with specific Python libraries and a specialized database schema), the Runloop backend builds the image, boots the microVM, initializes the operating system, and brings the container to a state of execution readiness.4 Crucially, at this exact moment, the orchestrator triggers a deep system snapshot, capturing the exact state of the microVM's active memory (RAM) and its virtual disk.4

The orchestrator then places copies of this "pre-warmed" snapshot into a dormant, standby pool distributed across the bare-metal worker nodes. When an agent requests a new sandbox via the Runloop API, the orchestrator does not boot a new machine; it simply routes the request to the nearest bare-metal node, which instantly resumes the snapshot from memory.4 By deserializing the pre-existing RAM state rather than executing a full boot sequence, the platform achieves sandbox creation times ranging from 500 milliseconds to 2 seconds, effectively masking the hypervisor tax from the end user.4

![](data:image/png;base64...)

### Maximizing Hardware ROI: The Suspend & Resume Lifecycle

The defining economic characteristic of AI agent workloads is their bursty, unpredictable nature. An agent may execute a flurry of Python scripts for three seconds, and then remain entirely idle for ten minutes while waiting for human input or while the upstream LLM (like Claude or GPT-4) generates the next block of reasoning text. Leaving a microVM actively spinning and holding CPU allocations during these idle periods is a massive waste of bare-metal capital expenditure.

Runloop directly addresses this inefficiency through its intelligent "Suspend & Resume" capability, which forms the core of its sandbox lifecycle management.5

The Runloop orchestrator continuously monitors the active processes, network I/O, and API interaction of every running sandbox. If a sandbox detects a period of absolute inactivity (exceeding a configurable timeout threshold, typically measured in minutes), the orchestrator autonomously issues a Suspend command.5

The Suspend operation is a hyper-optimized checkpointing mechanism. The orchestrator instantaneously flushes the active state of the microVM's RAM and CPU registers directly to persistent storage (typically high-speed, local NVMe drives attached to the bare-metal host).5 Once the state is securely written to disk, the orchestrator aggressively terminates the active microVM process, immediately returning the physical CPU threads and RAM capacity back to the bare-metal host's available resource pool.5

While suspended, the user is billed at a significantly reduced rate—often only fractionally for the storage consumed, entirely eliminating the compute cost.5 When a new event triggers the environment—such as a developer sending a new command via the CLI, or the LLM finally returning its output via the API—the orchestrator detects the ingress request, locates the suspended snapshot on the disk, and issues a Resume command.5 The microVM is instantly repopulated into RAM, and the agent continues executing exactly where it paused, oblivious to the fact that its environment was temporarily frozen.5 This architecture allows Runloop to safely and aggressively overprovision its bare-metal servers, hosting thousands of concurrent agent sessions on a fraction of the hardware that traditional static VM allocation would require.

| **Lifecycle State** | **Trigger Mechanism** | **Compute (CPU/RAM) Status** | **Storage Status** | **Resumption Latency** |
| --- | --- | --- | --- | --- |
| **Running** | API Creation / Start Command | Actively Allocated | Active on Local Disk | N/A |
| **Suspended** | API Suspend / Idle Timeout | Flushed / Released to Host | State Saved to NVMe | Sub-second (Warm Resume) |
| **Terminated** | API Delete / TTL Expiration | Released to Host | Data Destroyed | Requires New Provisioning |

## 3. Protocol Integration: Native Model Context Protocol (MCP)

For an AI sandbox to be functional, the external reasoning agent (e.g., Claude, running on Anthropic's servers) must be able to securely communicate with, command, and extract data from the isolated Runloop microVM. Historically, this required developers to write complex, bespoke "glue code" to map the LLM's outputs to specific API endpoints within the sandbox.

Runloop eliminates this friction by building its entire control plane around the Model Context Protocol (MCP).6 MCP is an open-source standard, originally architected by Anthropic, designed to provide a universal, secure client-server architecture for integrating AI models with external tools and datasets.7

### The Architecture of the Runloop MCP Server

Runloop does not merely support MCP as an afterthought; it provides a highly optimized, fully managed MCP Server specifically engineered for interacting with its sandboxes.8 This server acts as the definitive, secure bridge between the external LLM client and the internal bare-metal execution environment.

When an engineer configures an AI assistant (such as Cursor, Windsurf, or Claude Desktop) to connect to the Runloop environment, they simply provide the assistant with the Runloop MCP Server configuration, passing their proprietary Runloop API key as an authentication header.9

Once authenticated, the Runloop MCP Server utilizes the protocol's "progressive disclosure" mechanisms to expose three primary, highly structured tools directly into the LLM's context window:

1. **run\_bash:** This tool allows the external LLM to inject and execute arbitrary bash commands (e.g., `npm install`, `python script.py`) directly within the secure microVM sandbox. The tool returns the `stdout`, `stderr`, and the exit code synchronously back to the LLM, enabling the agent to read errors and self-correct its code autonomously.8
2. **read\_file:** This tool grants the LLM the capability to inspect the contents of specific files residing within the sandbox's virtual filesystem, pulling critical context (like application logs or existing source code) into its reasoning loop without requiring a human to copy and paste the data.8
3. **write\_file:** This tool enables the LLM to programmatically author or modify code, configuration files, or data structures directly onto the sandbox's disk, facilitating true, hands-free automated software development.8

### Security Downscoping via MCP

The utilization of MCP is critical for maintaining the integrity of the bare-metal control plane. By routing all agentic commands through the structured JSON-RPC framework of the MCP Server, Runloop ensures that the LLM cannot directly interact with the underlying host infrastructure. The MCP Server acts as a strict validation gateway; it authenticates the request, verifies that the command maps to an authorized tool (e.g., run\_bash), and then proxies that command exclusively into the designated, isolated microVM via secure internal channels. This architecture mathematically guarantees that even a severely hallucinating or compromised LLM is strictly bound to the capabilities explicitly exposed by the MCP definitions.

Furthermore, Runloop's architecture allows developers to deploy their own custom, application-specific MCP servers *inside* the Runloop sandbox.10 By running the custom MCP server within the microVM envelope, developers can securely expose proprietary internal databases or legacy enterprise APIs to the LLM. The AI agent connects to this internal MCP server, pulling the sensitive corporate data into its context window, executing its reasoning, and finalizing the output without the raw enterprise data ever being exposed to the public internet or leaving the secure perimeter of the Runloop sandbox.10

## 4. The Development Lifecycle: Repositories and Persistence

A successful agentic execution platform must integrate flawlessly with existing software engineering workflows. Sandboxes that cannot persist data or interact with version control are relegated to simple toy environments. Runloop addresses this by building robust filesystem mounting and GitHub synchronization protocols directly into the orchestration layer.12

### GitHub Integration and Automated Provisioning

To enable autonomous agents to operate on real-world codebases, the platform features a mechanism known as "Repo Connect".13 This feature fundamentally alters how environments are initialized.

Rather than forcing a user to manually boot a generic Ubuntu sandbox, manually authenticate with Git, and manually clone a repository, Repo Connect automates the entire pipeline. When a developer connects their GitHub account via the Runloop dashboard, they authorize the platform to act on their behalf. To create an environment for a specific project, the developer simply passes the GitHub repository URL to the Runloop API during the sandbox creation request.13

The orchestrator intercepts this request and automatically clones the specified repository directly into the root filesystem of the microVM during the boot sequence.13 This ensures that the moment the AI agent connects to the sandbox via the MCP server, the entire codebase is already present on disk, indexed, and ready for manipulation, saving critical context tokens and reducing initialization friction.14

### Volumes and Persistent Workspaces

While the microVMs themselves are ephemeral and subject to the Suspend & Resume lifecycle, the data they generate must be durable. Runloop separates compute from storage by implementing persistent network-attached Volumes.

Developers can programmatically define storage Volumes via the API and attach them to specific sandbox instances. When an agent executes a data processing script that downloads a massive 50GB dataset, or compiles a complex binary that requires hours of processing, that resulting data is written to the mounted Volume. Because the Volume is architecturally decoupled from the ephemeral microVM, the sandbox can be terminated entirely, and the Volume can be subsequently mounted to a brand new, pristine sandbox at a later date. This guarantees that valuable agentic outputs, trained models, and modified codebases are safely preserved on the bare-metal storage arrays, independent of the volatile lifecycle of the execution environment.

## 5. Strategic Blueprint for Bare-Metal AI Infrastructure

The deep architectural analysis of Runloop provides a definitive, exportable blueprint for organizations—specifically hosting providers and infrastructure entities like Hostopia—seeking to build internal, highly scalable execution engines for AI agents.

To successfully support autonomous AI on bare metal, infrastructure must abandon standard shared-kernel containerization in favor of the defense-in-depth isolation provided by KVM-backed microVMs. This is the only architecture that provides the strict, hardware-enforced boundaries required to mitigate the systemic risks of executing unverified, LLM-generated code in a multi-tenant environment.

Furthermore, to ensure economic viability and maximize hardware density, the orchestration plane cannot rely on static provisioning. The control plane must implement pre-warmed snapshotting to bypass hypervisor boot latency, achieving the sub-second responsiveness demanded by interactive agents. Simultaneously, the platform must aggressively implement Suspend & Resume mechanics, continuously monitoring sandbox activity and instantly flushing idle environments to disk. This reclaims critical CPU and RAM resources, allowing a single bare-metal server to host an exponentially larger density of agents.

Finally, the platform's control interface must be fully standardized on the Model Context Protocol (MCP). By deploying dedicated MCP servers to act as the sole communication bridge between the isolated microVMs and the external LLM reasoning engines, infrastructure providers can guarantee strict command validation, secure credential isolation, and seamless interoperability with the broader ecosystem of advanced AI coding assistants.

#### Works cited

1. How to sandbox AI agents in 2026: MicroVMs, gVisor & isolation strategies | Blog, accessed February 22, 2026, <https://northflank.com/blog/how-to-sandbox-ai-agents>
2. E2B vs Modal vs Fly.io Sprites for AI code execution sandboxes | Blog - Northflank, accessed February 22, 2026, <https://northflank.com/blog/e2b-vs-modal-vs-fly-io-sprites>
3. The Complete Guide to Sandboxing Autonomous Agents: Tools, Frameworks, and Safety Essentials - IKANGAI, accessed February 22, 2026, <https://www.ikangai.com/the-complete-guide-to-sandboxing-autonomous-agents-tools-frameworks-and-safety-essentials/>
4. Architecture | Daytona, accessed February 22, 2026, <https://www.daytona.io/docs/en/architecture/>
5. Runloop - Your AI Agent Accelerator, accessed February 22, 2026, <https://runloop.ai/>
6. Model Context Protocol (MCP) - Understanding the Game-Changer - Runloop, accessed February 22, 2026, <https://runloop.ai/blog/model-context-protocol-mcp-understanding-the-game-changer>
7. Why Model Context Protocol (MCP) matters for AI Agents - Runloop, accessed February 22, 2026, <https://runloop.ai/blog/why-mcp-matters-for-ai-agents>
8. Build an MCP Agent - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/tutorials/mcp-agent>
9. Connecting Claude Desktop to Runloop Sandbox - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/tutorials/claude-desktop-sandbox>
10. Build an MCP Server - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/tutorials/mcp-server>
11. Secure runtime for codegen tools: microVMs, sandboxing, and execution at scale | Blog, accessed February 22, 2026, <https://northflank.com/blog/secure-runtime-for-codegen-tools-microvms-sandboxing-and-execution-at-scale>
12. Repositories - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/reference/repositories>
13. Setting Up a Github Repository with a Devbox - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/tutorials/repo-devbox>
14. How to Create and Use a Devbox Workspace - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/tutorials/workspace>
15. restyler/awesome-sandbox: Awesome Code Sandboxing for AI - GitHub, accessed February 22, 2026, <https://github.com/restyler/awesome-sandbox>
16. What's the best code execution sandbox for AI agents in 2026? | Blog - Northflank, accessed February 22, 2026, <https://northflank.com/blog/best-code-execution-sandbox-for-ai-agents>
17. Firecracker vs Docker: The Technical Boundary Between MicroVMs and Containers, accessed February 22, 2026, <https://huggingface.co/blog/agentbox-master/firecracker-vs-docker-tech-boundary>
18. Understanding Firecracker MicroVMs: The Next Evolution in Virtualization - Medium, accessed February 22, 2026, [https://medium.com/@meziounir/understanding-firecracker-microvms-the-next-evolution-in-virtualization-cb9eb8bbeede](https://medium.com/%40meziounir/understanding-firecracker-microvms-the-next-evolution-in-virtualization-cb9eb8bbeede)
19. Daytona vs E2B in 2026: which sandbox for AI code execution? | Blog - Northflank, accessed February 22, 2026, <https://northflank.com/blog/daytona-vs-e2b-ai-code-execution-sandboxes>
20. The OpenHands Software Agent SDK: A Composable and Extensible Foundation for Production Agents - arXiv, accessed February 22, 2026, <https://arxiv.org/html/2511.03690v1>
21. E2B: Sandboxed cloud environments made for AI agents - Hacker News - Y Combinator, accessed February 22, 2026, <https://news.ycombinator.com/item?id=29067493>
22. Repo Connect - Automatic Devbox Setup from a Github Repository - Runloop AI Docs, accessed February 22, 2026, <https://docs.runloop.ai/docs/devboxes/repo-connect>