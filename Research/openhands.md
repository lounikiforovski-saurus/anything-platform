# OpenHands — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026

---

# Fundamental Positioning Note

OpenHands (formerly OpenDevin) is **categorically different** from every other tool in this analysis. Bolt, Lovable, Horizons, Replit, and v0 are all products — commercial AI app builders with proprietary UIs, hosted infrastructure, and subscription pricing. OpenHands is an **open-source AI software development agent framework** released under the MIT license.

The distinction matters for HostPapa's strategy:
- Bolt, Lovable, v0 → **competitors to benchmark against**
- OpenHands → **potential foundation to build ON**

HostPapa cannot white-label Bolt or Lovable. But HostPapa could build their own AI builder using OpenHands as the agent engine, deploying on HostPapa infrastructure, with HostPapa branding. The MIT license explicitly permits this.

---

# PHASE 1: Product & UX Friction Audit

## Exhaustive UI Matrix

### Three Deployment Modes

OpenHands operates in three distinct modes, each with different UI:

#### 1. Local GUI (Self-Hosted)
- **Chat interface** — left panel for conversational interaction with the AI agent. Describe tasks in natural language. Agent responds with explanations of what it's doing, interspersed with actions (file edits, commands, web browsing).
- **Workspace file browser** — right panel showing all files in the sandboxed workspace. Click to view/edit files. Standard file operations (create, rename, delete).
- **Terminal output** — live feed of commands the agent runs. The user sees exactly what bash commands are executed, their output, and any errors. Full transparency into agent behavior.
- **Browser view** — when the agent browses the web (for documentation, debugging, or site inspection), the user can see the browser state. Not just URLs — the actual rendered page.
- **Jupyter notebook** — agent can create and execute Python notebooks. Output (including plots, DataFrames, and rich media) displayed in the UI. Useful for data science and analysis tasks.
- **Settings panel** — configure: LLM provider (OpenAI, Anthropic, Google, local), model name, API key, base URL, search engine (Tavily API), agent type (CodeAct is default). Advanced options for custom model configurations.
- **File upload** — upload files to the agent's workspace for context, test data, or reference material.
- **Workspace download** — download a ZIP archive of the current workspace at any point.

#### 2. CLI Mode
- **Terminal-based interaction** — `openhands` command starts a terminal chat session. Same agent capabilities as the GUI — file editing, command execution, web browsing — but entirely text-based.
- **Headless mode** — scriptable, non-interactive execution. Run agent tasks from CI/CD pipelines, cron jobs, or automation scripts. Example: `openhands -t "Fix the failing test in tests/auth_test.py"` → agent works, produces output, exits.
- **Cloud integration** — CLI can manage OpenHands Cloud conversations. Create, continue, and list conversations from the terminal.

#### 3. OpenHands Cloud (Commercial SaaS)
- **Hosted GUI** — same interface as local GUI but hosted by OpenHands (All Hands AI, Inc.). No Docker setup required. Sign in with GitHub.
- **Source control integration** — deep integration with GitHub, GitLab, and Bitbucket. Import repos, create branches, push changes, create PRs — all from the agent.
- **Project management integration** — Slack, Jira, Linear connections. Agent can read issues, create tasks, and post updates.
- **Suggested tasks** — OpenHands analyzes your recent repos and suggests tasks the agent can tackle (bug fixes, improvements, refactoring). Like a "task backlog" populated by AI analysis.
- **Multi-user support** — team collaboration on conversations.
- **RBAC** — role-based access control for teams.
- **Conversation sharing** — share agent sessions with teammates for review, knowledge transfer, or debugging.
- **Usage reporting** — track token consumption, conversation counts, and costs across users and projects.
- **Budget enforcement** — set spending limits per user or organization. Prevents runaway API costs during long autonomous sessions.

---

## JTBD Friction Map

### Flow 1: Fixing a GitHub Issue via OpenHands Cloud
1. Sign in with GitHub (1 click, one-time)
2. Select repository from list (1 click)
3. OpenHands shows suggested tasks from recent issues/PRs (0 clicks — automatic)
4. Select a suggested task OR describe your own: "Fix issue #142 — login form doesn't validate email format" (1 action)
5. Agent clones repo, reads the codebase, identifies the relevant files, implements the fix, runs tests, and creates a pull request (0 clicks — fully autonomous)
6. Review the PR on GitHub. Merge if satisfied. (1-2 clicks)

**Total clicks: ~4-5**
**Wait time: 2-15 minutes depending on complexity**
**Cognitive leaps: 1** (reviewing the PR to ensure quality)
**Friction rating: LOW for developers** — the GitHub-native workflow eliminates context switching

### Flow 2: Self-Hosting OpenHands Locally
1. Install Docker (prerequisite)
2. Install OpenHands: `uv tool install openhands` OR `pip install openhands` OR `docker run -it ...` (1 command)
3. Run: `openhands serve` → opens http://localhost:3000 (1 command)
4. Configure: LLM provider → select Anthropic/OpenAI/Google → paste API key (3 actions)
5. Optionally configure search engine (Tavily API key) (1 action)
6. Start chatting with the agent (1 action)

**Total setup time: ~10 minutes (assuming Docker is installed)**
**Cognitive leaps: 2** (Docker familiarity, LLM API key acquisition)
**Friction rating: MEDIUM-HIGH for non-technical users, LOW for developers**
**Ongoing cost: LLM API charges only (no OpenHands subscription)**

### Flow 3: Using OpenHands as a GitHub Action (Fully Automated)
1. Add OpenHands GitHub Action to your repo's workflow YAML (1 edit)
2. Configure: which issues to trigger on (label-based), which LLM to use, API key as secret (3 configurations)
3. When someone labels an issue with the configured label (e.g., "openhands"), the agent automatically:
   - Reads the issue description
   - Clones the repo
   - Implements a fix
   - Runs tests
   - Creates a PR
4. Review and merge the PR (1-2 clicks)

**Total setup: ~15 minutes one-time**
**Per-issue effort: 0 clicks** (label the issue → PR appears)
**Friction rating: LOW after initial setup** — this is the most autonomous development workflow in any tool analyzed

---

## The "Aha!" Moment

**The "Aha!" is that this is a real developer.** OpenHands doesn't generate code from templates or patterns. It has a sandboxed Linux environment where it can: run arbitrary bash commands, install any package, edit any file, browse the web, run tests, manage git, and create PRs. It reasons about your codebase, plans an approach, executes it, validates the results, and delivers a PR. No other tool in this analysis (except Replit with Max Autonomy) offers this level of autonomous, multi-step software development.

**The second "Aha!" is the open-source self-hosting.** Run it on YOUR servers, with YOUR LLM API keys, on YOUR codebase. Zero vendor lock-in. No data leaves your infrastructure. The agent's code is MIT-licensed — you can modify it, embed it, and redistribute it. For organizations with strict data sovereignty requirements, this is the only viable option among the tools analyzed.

**The third "Aha!" is the GitHub Action integration.** Label an issue → agent creates a PR. This is software development on autopilot. For teams that maintain large codebases with routine bugs and small features, this transforms the development workflow from "developer reads issue → developer writes code → developer creates PR" to "developer labels issue → reviews PR." The agent does the work; the human does the review.

**The fourth "Aha!" is the Agent SDK.** OpenHands isn't just an agent — it's a framework for building agents. The Python SDK lets you define custom agents with specialized behaviors, integrate custom tools via MCP (Model Context Protocol), and scale to thousands of parallel agent instances. This is the foundation for building a HostPapa-branded AI builder.

---

# PHASE 2: "Own The Metal" Architecture Blueprint

## Architecture Deep Dive

### The Sandbox
Every OpenHands conversation runs in an isolated Docker container. This container is the agent's "computer" — a full Linux environment with:
- **Shell access** — bash with root privileges. The agent can run ANY command: install packages (apt, pip, npm, cargo), compile code, run tests, start servers, curl APIs, manage files.
- **Filesystem** — complete Linux filesystem. The agent reads, creates, edits, and deletes files. Local code can be mounted into the sandbox via Docker volume mounts — the agent works on your actual codebase, not a copy.
- **Network access** — outbound network for web browsing, API calls, package downloads, and git operations. The agent can clone repos, push to GitHub, fetch documentation, and interact with external services.
- **Browser** — headless browser for web interaction. The agent can browse documentation, inspect deployed sites, fill forms, and take screenshots. Used for research during development and for testing web applications.
- **Jupyter** — integrated Jupyter kernel for Python execution. The agent can run data analysis, generate visualizations, and iterate on data science tasks with rich output.
- **GPU support** — nvidia-docker integration for ML workloads. The agent can train models, run inference, and process data using GPU acceleration.

### Agent Architecture (CodeAct)
- **CodeAct** is the default agent type. It operates by generating "actions" (bash commands, file edits, browser actions, Jupyter cells) based on the conversation context and codebase state.
- The agent follows a think → plan → act → observe → reflect loop:
  1. **Think** — analyze the task and current project state
  2. **Plan** — determine which files to read, what commands to run
  3. **Act** — execute actions (edit files, run commands, browse web)
  4. **Observe** — read the output of actions (terminal output, file contents, browser state)
  5. **Reflect** — assess whether the task is complete or needs more work
- This loop continues until the agent determines the task is complete or encounters a blocker requiring human input.

### Microagents
- **Repository microagents** — `.openhands/microagents/` directory in your repo. These are Markdown files that provide coding guidelines, architecture decisions, and project-specific knowledge to the agent. Analogous to `.cursorrules` or v0's Instructions, but file-based and repo-specific.
- **Knowledge microagents** — background information injected into agent context. API documentation, library usage patterns, deployment procedures.
- **Trigger-based** — microagents can be triggered by specific keywords or file patterns, providing context-sensitive guidance to the agent.

### MCP (Model Context Protocol) Integration
- **Tool extensibility** — MCP allows adding custom tools to the agent. Default: `fetch` MCP server for web content retrieval. Custom MCP servers can add: database access, API integrations, deployment tools, monitoring, or any capability you can expose via MCP.
- **Significance for HostPapa** — MCP would be the mechanism for integrating HostPapa-specific tools: "deploy to HostPapa hosting," "create HostPapa database," "configure HostPapa DNS." Build custom MCP servers that give the agent native access to HostPapa infrastructure.

---

## BaaS Reliance

### OpenHands provides NO backend services.
- **No database** — the agent can INSTALL and CONFIGURE databases (PostgreSQL, MySQL, SQLite, MongoDB, Redis) in its sandbox. But there's no managed database service.
- **No auth service** — the agent IMPLEMENTS auth in your code. No auth platform.
- **No file storage** — workspace files exist in the Docker container. No object storage service.
- **No hosting** — OpenHands does not deploy apps. The agent can WRITE deployment configurations (Dockerfiles, CI/CD pipelines, Kubernetes manifests, serverless configs) but doesn't operate hosting infrastructure.
- **No CDN, SSL, domains, or email.**

### What This Means for HostPapa
OpenHands is the agent brain. HostPapa provides the body:
- **HostPapa hosting** → where the agent deploys apps
- **HostPapa database** → what the agent connects apps to
- **HostPapa domains** → what the agent configures DNS for
- **HostPapa email** → what the agent sets up for users
- **HostPapa SSL** → automatic for all deployments

The OpenHands agent, equipped with HostPapa-specific MCP tools, could build apps that deploy directly to HostPapa infrastructure. The user describes their app → agent builds it → agent deploys to HostPapa → user has a live app on HostPapa hosting with HostPapa domain and HostPapa email. This is the "Hostinger Horizons" model built with open-source components.

---

## LLM Orchestration

### Bring Your Own Model (BYOM)
OpenHands supports virtually any LLM:
- **Anthropic Claude** — Claude 3.5 Sonnet, Claude 3 Opus, etc. Recommended for code-heavy tasks.
- **OpenAI GPT** — GPT-4, GPT-4 Turbo, GPT-4o. Well-supported.
- **Google Gemini** — Gemini Pro, Gemini Ultra. Supported.
- **OpenHands LLM Provider** — OpenHands' own API with "competitive pricing" and "state-of-the-art agentic coding models." Available via Cloud API keys. This suggests OpenHands may be fine-tuning or routing to optimized models specifically for coding agent workloads.
- **Local models** — Ollama, LM Studio, llama.cpp, vLLM. Run models on your own hardware with no API costs and no data leaving your network. Quality varies significantly by model — smaller local models may not be suitable for complex coding tasks.
- **Custom base URL** — point to any OpenAI-compatible API endpoint. Supports proxies, custom deployments, and enterprise API gateways.

### Model Selection Impact
Unlike v0 (which fine-tunes models for Next.js) or Replit (which optimizes for their IDE), OpenHands is model-agnostic. The quality of the agent's output depends directly on the model you choose:
- **Claude 3.5 Sonnet** — best overall for coding tasks (based on SWE-bench evaluations)
- **GPT-4** — strong but slightly behind Claude for code generation
- **Local models (7B-13B)** — adequate for simple tasks; struggle with complex multi-file changes
- **Local models (70B+)** — approaching API model quality but require significant GPU resources

### Token Economy
- **Self-hosted:** you pay LLM API costs directly. No OpenHands markup. A complex coding session (reading codebase + planning + implementing + testing) might consume 50,000-200,000 tokens. At Claude 3.5 Sonnet pricing ($3/$15 per 1M in/out), that's roughly $0.10-$1.00 per task.
- **OpenHands Cloud:** OpenHands LLM provider with "competitive pricing." Exact rates not publicly documented. Budget enforcement prevents runaway costs.
- **Self-hosted with local models:** $0 per task (hardware costs only). Ideal for organizations processing high volumes of routine tasks.

### Critic (Experimental)
- Automatic task success prediction — the agent evaluates whether its own work succeeded or needs revision. Available for OpenHands LLM provider users. This is conceptually similar to Replit's App Testing but operates at the code/test level rather than the browser/UI level.

---

## Sandbox Environment (Deep Technical)

### Container Specification
- **Base image:** OpenHands provides preconfigured Docker images with common development tools preinstalled (Python, Node.js, git, common system libraries). Custom images can be specified for specialized environments.
- **Resource allocation:** configurable CPU, memory, and storage limits per container. Default settings are suitable for most development tasks.
- **Lifecycle:** containers are created per conversation and persist for the duration of the session. Between sessions, workspace state is preserved (on Cloud) or lost (self-hosted, unless workspace directory is mounted).
- **Networking:** outbound network access enabled by default. The agent can fetch web content, clone repos, download packages, and call APIs. Inbound access (running a web server the user can access) is supported through port mapping.

### Local Code Mounting
When running self-hosted, you can mount local directories into the sandbox:
```
docker run -v /path/to/your/code:/workspace ...
```
The agent works on your actual codebase. Changes persist to your local filesystem. This means:
- No code upload/download step
- Changes are immediately visible in your local IDE
- Git operations in the sandbox affect your actual repo
- The agent can run your existing test suite, build pipeline, and deployment scripts

### GPU Support
For ML/AI workloads:
```
docker run --gpus all ...
```
The agent can train models, run inference, process large datasets, and use CUDA-accelerated libraries. This is unique among AI builders — no other tool in this analysis supports GPU workloads.

---

# PHASE 3: Maintainability & Guardrail Teardown

## Preventing Spaghetti Code

### Stack
- **Any stack, any language, any framework.** OpenHands is entirely agnostic. It works with whatever your project uses. Python, JavaScript, TypeScript, Go, Rust, Java, C++, Ruby — if it runs on Linux, the agent can work with it.
- Quality depends on: (1) the underlying LLM's expertise with the language/framework, (2) the microagents you've defined, and (3) the existing codebase patterns the agent learns from.

### Code Quality Mechanisms
1. **Microagents** — repo-specific coding guidelines in `.openhands/microagents/`. Tell the agent: "Use Prettier for formatting," "Follow Google's Python style guide," "All API endpoints require authentication middleware." The agent reads these before every task.
2. **Existing codebase patterns** — the agent reads your existing code before making changes. If your codebase uses certain patterns (repository pattern, service layer, etc.), the agent tends to follow them. Consistency emerges from context, not enforcement.
3. **Test execution** — the agent can run your existing test suite after making changes. If tests fail, it revises. This is a natural quality gate — if you have good tests, the agent will produce code that passes them.
4. **Security analyzers** — the REST API lists a `list-security-analyzers` endpoint, suggesting built-in or pluggable security analysis capabilities. Documentation is sparse.
5. **No enforced linting, formatting, or type checking** — unlike v0 (TypeScript enforced) or Lovable (TypeScript + shadcn/ui). Quality enforcement is the user's responsibility through microagents and CI/CD.

---

## Version Control

### Capabilities
- **Full git** — the agent performs standard git operations: clone, branch, commit, push, pull, diff, merge. This is not a simplified "version history" — it's real git in a real terminal.
- **GitHub Actions integration** — automated issue-to-PR workflow. Label an issue → agent creates a PR. This is the most autonomous git workflow of any tool analyzed.
- **GitHub/GitLab/Bitbucket** — Cloud supports all three major git platforms.
- **PR creation** — agent creates pull requests with descriptive titles and bodies explaining what was changed and why.
- **Branch management** — agent creates feature branches per task, following standard branching conventions.

### Assessment
**Best version control for existing codebases.** OpenHands doesn't need its own version control system — it uses YOURS. The agent works within your existing git workflow, respecting branch naming conventions, commit message formats, and PR templates. For teams with established development practices, this is the least disruptive tool.

---

# PHASE 4: GTM & Telco Partner Strategy

## Pricing Model

### Self-Hosted (Open Source)
| Component | Cost |
|-----------|------|
| OpenHands software | **Free** (MIT license) |
| LLM API keys | User pays directly ($0.10-$1.00 per typical task with Claude) |
| Docker infrastructure | User provides (laptop, server, cloud VM) |
| **Total for a solo developer** | **$0-50/month** depending on usage volume |
| **Total for a team of 10** | **$0-500/month** depending on usage volume |

### OpenHands Cloud
| Tier | Details |
|------|---------|
| **Free** | Sign in with GitHub. Limited usage. Experience the platform. |
| **Paid** | Usage-based. Exact pricing not publicly documented. OpenHands LLM provider at "competitive" rates. |
| **Enterprise** | Self-hosted in your VPC via Kubernetes. Source-available license (different from MIT for enterprise features). Extended support. Research team access. |

### Enterprise Self-Hosted
- Kubernetes deployment in your own VPC
- Source-available license (not MIT — additional terms for enterprise features)
- Extended support from the OpenHands team
- Access to research team for custom agent development
- 1-month trial available

### Cost Comparison with Competitors
| Scenario | OpenHands Self-Hosted | v0 Premium | Replit Core | Horizons Starter |
|----------|---------------------|-----------|------------|-----------------|
| Monthly platform cost | $0 | $20 | $25 | $13.99 |
| AI costs (moderate use) | ~$20 (API) | Included ($20 credits) | Included ($20 credits) | Included (70 credits) |
| Hosting | User provides | +$0-20 (Vercel) | +$5-50 | Included |
| Database | User provides | +$0-25 | Included | Included |
| **Total** | **~$20/month** | **$20-65/month** | **$30-75/month** | **$13.99/month** |

Self-hosted OpenHands is the cheapest option for developers who already have infrastructure and LLM API keys. The cost advantage grows with team size — one self-hosted instance serves the entire team, while per-user SaaS tools scale linearly.

---

## B2B2C Channel Readiness

### White-Label / Embedding Assessment: THE MOST EMBEDDABLE OPTION

This is the critical strategic insight for HostPapa:

- **MIT license (core)** — HostPapa can embed, modify, and redistribute OpenHands' agent engine without license fees or revenue sharing. The MIT license is the most permissive OSS license available.
- **Agent SDK (Python)** — a composable library for building custom agents. Define specialized behaviors, integrate custom tools, scale to thousands of parallel instances. HostPapa could build a "HostPapa App Builder" that uses OpenHands' agent SDK under the hood.
- **REST API** — full programmatic control over the agent. Create conversations, send messages, manage workspaces, list files. HostPapa could build a custom web UI that communicates with OpenHands' API backend.
- **MCP extensibility** — add HostPapa-specific tools via Model Context Protocol. "Deploy to HostPapa," "Create HostPapa database," "Configure HostPapa DNS" could be native agent capabilities.
- **Custom Docker images** — pre-configure the sandbox with HostPapa CLI tools, SDKs, and deployment scripts.
- **No UI white-label** — the React GUI is open-source but not designed for embedding. HostPapa would build their own frontend on top of the SDK/API. This is more work but results in a fully branded experience.

### What HostPapa Would Need to Build
1. **Custom frontend** — a web UI for HostPapa customers (similar to Horizons or Bolt's interface) that communicates with OpenHands' API.
2. **HostPapa MCP servers** — tools that let the agent interact with HostPapa infrastructure (hosting, domains, databases, email).
3. **Managed sandbox infrastructure** — run Docker containers for each user session. Could use Kubernetes, ECS, or Fly.io.
4. **LLM routing** — integrate with Anthropic/OpenAI APIs or self-host models. Rate limiting, cost management, model selection per task.
5. **User management** — authentication, billing, usage tracking, project management.
6. **Template library** — pre-built starting points for common app types (business site, e-commerce, SaaS).

### Build vs. Buy Assessment
| Approach | Effort | Control | Cost | Time to Market |
|----------|--------|---------|------|----------------|
| Build on OpenHands | High (6-12 months) | Maximum | Low ongoing (MIT, API costs only) | 6-12 months |
| White-label Bolt/Lovable | N/A (not available) | None | N/A | N/A |
| Partner with Replit | Medium (API integration) | Low | Revenue share or licensing | 3-6 months |
| Build from scratch | Very high (18-24 months) | Maximum | High (LLM fine-tuning, agent dev) | 18-24 months |

**Recommendation:** OpenHands is the most viable foundation for HostPapa's AI builder. The MIT license, agent SDK, REST API, and MCP extensibility provide all the building blocks. The main investment is the custom frontend, HostPapa infrastructure integration, and user management layer.

---

## Positioning & Persona

### Hero Copy
- "AI-driven software development"
- "The open-source AI software developer"
- Positioning as the open-source alternative to Devin (Cognition Labs' $2B AI developer)

### Target Persona
**Primary: Software development teams** who want AI-assisted coding on their existing codebases. Not beginners building their first app — teams with existing repos, test suites, and deployment pipelines.

**Secondary: Platform builders** who want to embed AI coding capabilities into their own products. This is HostPapa's use case.

**Tertiary: Individual developers and open-source contributors** who want a free, self-hosted AI coding agent.

---

# PHASE 5: Enterprise Compliance & Accessibility

## Security Posture

### Application-Level Security
- **Sandboxed execution** — all agent actions run in isolated Docker containers. The agent cannot access the host system outside the mounted workspace.
- **Network isolation** — configurable. Can restrict outbound network access for air-gapped environments.
- **Open source auditability** — the entire codebase is public on GitHub. Any organization can audit the agent's behavior, sandbox security, and data handling.
- **Security analyzers** — API endpoint exists for security analysis (documentation sparse).
- **No default security scanning** of generated code — unlike Lovable's 4-scanner suite.

### Data Privacy (Self-Hosted)
- **No data leaves your infrastructure** (when self-hosted with local models). This is the strongest data privacy story of any tool analyzed.
- **LLM API calls** are the only data egress point when using cloud LLMs. The prompts sent to OpenAI/Anthropic/Google include your code context. For sensitive codebases, use local models (Ollama, vLLM) to eliminate this.
- **No telemetry** — configurable. Self-hosted instances can be fully air-gapped.

### Data Privacy (Cloud)
- OpenHands Cloud stores conversation data on their servers. Data handling policies not extensively documented.
- Enterprise self-hosted (Kubernetes in your VPC) provides Cloud features with self-hosted data control.

## Certifications

| Certification | Status | Details |
|--------------|--------|---------|
| SOC 2 Type II | ❓ | Cloud may have; not publicly documented. Self-hosted: N/A (your infrastructure). |
| ISO 27001 | ❌ | Not documented. |
| GDPR | ⚠️ | Self-hosted: you control all data, GDPR compliance is your responsibility. Cloud: standard privacy policy. |
| HIPAA | ⚠️ | Self-hosted with local models: possible with your own controls (no PHI leaves your infrastructure). Cloud: not documented. |
| VPAT | ❌ | Not published. |

### Compliance Advantage
For regulated industries (healthcare, finance, government), OpenHands self-hosted with local models is the ONLY tool in this analysis that can guarantee zero data egress. No code, no prompts, no telemetry ever leaves the organization's network. This is a genuine competitive differentiator.

---

# PHASE 6: Churn & Scalability Ceiling

## Code Ejection

### There Is No Lock-In
- The agent works on YOUR code, in YOUR repo, on YOUR infrastructure.
- Every change is a git commit in your repository.
- No proprietary file formats, SDKs, or data formats.
- If you stop using OpenHands, your code, repo, and infrastructure are exactly as they were — minus one tool in your workflow.
- **Zero ejection friction. Zero lock-in. By design.**

---

## The Logic Wall

| Complexity Level | Capability | Evidence & Details |
|-----------------|-----------|-------------------|
| ✅ Works great | Bug fixes on existing codebases | Core use case. GitHub Action integration automates issue → PR. SWE-bench evaluations show strong performance. |
| ✅ Works great | Small-to-medium feature additions | Agent reads codebase context, plans implementation, writes code, runs tests. |
| ✅ Works great | Test writing and documentation | Agent can generate unit tests, integration tests, and documentation based on existing code. |
| ✅ Works great | Greenfield project scaffolding | Agent can create new projects from scratch with proper structure, dependencies, and configuration. |
| ✅ Works great | Refactoring and code cleanup | Agent can restructure code, rename variables, extract functions, and improve patterns. |
| ⚠️ Works with effort | Complex multi-file refactoring | Limited by LLM context window. Large codebases require careful task decomposition. |
| ⚠️ Works with effort | Architecture changes | Agent can implement architectural changes but may not make optimal design decisions without strong microagent guidance. |
| ⚠️ Works with effort | Full-stack app building from scratch | Can do it but lacks the specialized UI (preview, Design Mode) of dedicated builders. Output is code + terminal, not a visual app preview. |
| ❌ Struggles | Very large codebases (>100K lines) | Context window limits mean the agent can't hold the entire codebase in memory. Must work file-by-file with limited global awareness. |
| ❌ Struggles | Novel algorithms and research code | LLMs are trained on existing code. Genuinely novel solutions require human creativity. |
| ❌ Struggles | Complex DevOps and infrastructure | Can write Dockerfiles and CI/CD configs but may not handle complex cloud architectures reliably. |

### Competitive Position
OpenHands' Logic Wall is defined by the underlying LLM's capability, not by platform limitations. The sandbox has no constraints — any language, any tool, any framework. The ceiling is the AI's reasoning ability. As LLMs improve, OpenHands' capabilities improve automatically — no platform changes required.

This is fundamentally different from Bolt (limited by WebContainers), Lovable (limited to React/Supabase), Horizons (limited to web apps on Hostinger), or v0 (limited to Next.js). OpenHands is limited only by the model — and models get better every quarter.

---

# EXHAUSTIVE FEATURE INDEX

## Agent Core

| Feature | Description | Details |
|---------|-------------|---------|
| **CodeAct Agent** | Primary autonomous coding agent | Plans, implements, tests, and delivers code changes autonomously. Think → plan → act → observe → reflect loop. Handles multi-step tasks with self-correction. Works on any codebase in any language. Sandboxed in Docker for safety. |
| **Bash Execution** | Full shell command capability | Agent runs arbitrary bash commands in the sandbox. Install packages (apt, pip, npm, cargo), run scripts, execute tests, manage git, start servers, curl APIs, process data. Root access. Full Linux environment. |
| **File Editing** | Direct file manipulation | Agent reads, creates, edits, and deletes files. Surgical line-level edits or full-file rewrites. Works on any file type (code, config, documentation, data). Change tracking via git. |
| **Web Browsing** | Internet access for research and testing | Headless browser for documentation lookup, API reference, debugging deployed sites, and testing web applications. Agent sees rendered pages (not just HTML). Can interact with web UIs — click, type, navigate. |
| **Jupyter Notebooks** | Data science and analysis | Agent creates and executes Jupyter notebooks. Data analysis, visualization (matplotlib, plotly), ML model development. Rich output (plots, DataFrames, images) displayed in the UI. |
| **Microagents** | Repository-specific AI instructions | Place `.openhands/microagents/` files in your repo. Knowledge microagents (background info about the project) and repository microagents (coding guidelines, architecture decisions). Agent reads these for project-specific context. Analogous to .cursorrules or v0 Instructions but file-based and version-controlled. |
| **MCP Integration** | Model Context Protocol tool extensibility | Add custom tools via MCP servers. Default: fetch server for web content. Custom servers can add: database access, deployment tools, API integrations, monitoring, custom business logic. The mechanism for extending the agent with platform-specific capabilities (e.g., HostPapa infrastructure tools). |
| **Search Engine** | Web search during development | Tavily API integration for search. Agent can search for documentation, solutions, and current information during development. Configurable in settings. Disabled if no API key provided. |
| **Critic (Experimental)** | Automatic task success evaluation | Agent evaluates whether its own work succeeded or needs revision. Available for OpenHands LLM provider users. Reduces the need for human review of simple tasks. |
| **File Upload** | Import files to workspace | Upload files to the agent's sandbox. Provide context documents, test data, configuration files, design assets. Available in GUI and Cloud. |
| **Workspace Download** | Export workspace as ZIP | Download the complete workspace at any point. Full file export for backup, sharing, or migration. |

## Deployment Modes

| Feature | Description | Details |
|---------|-------------|---------|
| **Local GUI** | Self-hosted browser interface | `openhands serve` starts a React SPA on localhost:3000. REST API backend. Full workspace browser, terminal view, chat, Jupyter output, browser view. Familiar to Devin/Jules users. No account required. No data sent to OpenHands servers (when using third-party LLM APIs). |
| **CLI** | Terminal-based interaction | `openhands` command for terminal chat. Same agent capabilities as GUI. Headless mode for scripting and automation. `openhands serve` for GUI mode. Cloud integration for managing Cloud conversations from terminal. |
| **OpenHands Cloud** | Hosted commercial SaaS | Sign in with GitHub. No Docker setup. Additional integrations (Slack, Jira, Linear, GitLab, Bitbucket). Multi-user, RBAC, usage reporting, budget enforcement. Suggested tasks from repo analysis. Free tier available. |
| **Enterprise Self-Hosted** | Kubernetes VPC deployment | Deploy OpenHands in your own infrastructure. Source-available license (additional terms beyond MIT for enterprise features). Extended support. Research team access. 1-month trial available. |
| **Docker** | Container-based local deployment | Single `docker run` command to start. Mount local code via volumes. GPU support via `--gpus all` (nvidia-docker). Version pinning available. Multiple image variants for different use cases. |
| **Agent SDK** | Python composable library | Define custom agents in code. Compose actions and observations. Run locally or scale to thousands in cloud. MIT-licensed. The building block for platform integration. |

## Source Control Integration

| Feature | Description | Details |
|---------|-------------|---------|
| **GitHub Integration** | Deep GitHub support | Cloud: import repos, create branches, commits, PRs. Suggested tasks from recent issues and PRs. Repository microagent discovery. Permissions-based access. Self-hosted: full git operations via terminal. |
| **GitLab Integration** | GitLab repository support | Cloud supports GitLab repositories with same agent capabilities. |
| **Bitbucket Integration** | Bitbucket repository support | Cloud supports Bitbucket repositories. |
| **GitHub Actions** | Automated issue-to-PR pipeline | Configure OpenHands as a GitHub Action in your workflow YAML. Label an issue → agent clones repo, reads issue, implements fix, runs tests, creates PR. Fully autonomous CI/CD integration. The most automated development workflow of any tool analyzed. |
| **Standard Git** | Full git capabilities | Agent performs all standard git operations: clone, branch, commit, push, pull, diff, merge, rebase, stash. Works with any git remote. Respects existing branch naming conventions and commit message formats. |

## LLM Configuration

| Feature | Description | Details |
|---------|-------------|---------|
| **Multi-Provider** | Use any LLM API | Anthropic (Claude), OpenAI (GPT), Google (Gemini), and more. Select provider and model in settings. API key management per provider. |
| **OpenHands LLM Provider** | Competitive-pricing native API | "State-of-the-art agentic coding models" at competitive pricing. Available via OpenHands Cloud. May include fine-tuned models optimized for agent workloads. |
| **Local Model Support** | Self-hosted LLMs | Ollama, LM Studio, llama.cpp, vLLM support. Any value as API key (not validated). Zero data egress. For privacy-sensitive environments and high-volume batch processing. |
| **Custom Base URL** | Enterprise API gateways | Configure custom LLM API endpoints. Support for corporate proxies, custom model deployments, and enterprise API gateways (Azure OpenAI, AWS Bedrock, etc.). |

## Workspace & Sandbox

| Feature | Description | Details |
|---------|-------------|---------|
| **Docker Sandbox** | Isolated execution environment | Each conversation gets an isolated Docker container. Full Linux environment with root access. Install any package. Run any language. Network access. Browser. Jupyter. Container persists for conversation duration. |
| **Local Code Mounting** | Work on your actual codebase | Mount local directories into the sandbox via Docker volumes. Agent changes persist to your local filesystem. No upload/download step. Changes visible in your local IDE immediately. |
| **GPU Support** | ML/AI workload acceleration | nvidia-docker integration. CUDA-accelerated libraries. Model training, inference, data processing. Unique capability among AI builders — no other tool supports GPU workloads. |
| **Custom Docker Images** | Pre-configured environments | Specify custom Docker images for specialized development environments. Pre-install tools, SDKs, frameworks. Useful for enterprise environments with specific requirements. |
| **Network Configuration** | Outbound/inbound access control | Configurable network access. Outbound: enabled by default for web, git, packages. Can restrict for air-gapped environments. Inbound: port mapping for testing web applications. |

## Cloud-Specific Features

| Feature | Description | Details |
|---------|-------------|---------|
| **Multi-User** | Team collaboration | Multiple team members on a shared OpenHands Cloud instance. |
| **RBAC** | Role-based access control | Permissions management for conversations, repos, and settings. |
| **Conversation Sharing** | Share agent sessions | Share conversations for review, knowledge transfer, or collaborative debugging. |
| **Usage Reporting** | Cost and activity tracking | Token consumption, conversation counts, time spent per user and project. Budget monitoring. |
| **Budget Enforcement** | Spending limits | Set caps per user or organization. Prevent unexpected cost overruns during long autonomous sessions. Critical for enterprise cost management. |
| **Suggested Tasks** | AI-identified work items | OpenHands analyzes recent repos and suggests tasks (bugs, improvements, refactoring) the agent can tackle. Reduces the "what should I work on?" decision for developers. |
| **Project Management Integration** | Jira, Linear, Slack | Connect agent work to project management workflows. Read issues, create tasks, post updates. |

## API & Extensibility

| Feature | Description | Details |
|---------|-------------|---------|
| **REST API (V1)** | Full programmatic control | Create conversations, add events, list messages, search, manage files. WebSocket support for real-time communication. Comprehensive API for building custom integrations and UIs. |
| **Agent SDK (Python)** | Build custom agents | Composable Python library. Define agent behavior with custom action/observation loops. Integrate custom tools. Run single agent locally or scale to thousands in cloud. MIT-licensed. The foundation for building platform-specific AI builders. |
| **MCP Extensibility** | Custom tool integration | Model Context Protocol support for adding arbitrary tools. Default fetch server. Custom servers for databases, deployment, monitoring, APIs. The mechanism for HostPapa infrastructure integration. |
| **Chrome Extension** | Browser-based quick access | Chrome extension for accessing OpenHands from any browser tab. Quick task creation. |
| **Benchmarks** | Agent evaluation | Standardized benchmarks for evaluating agent performance. SWE-bench compatible. Useful for comparing model quality and tuning agent behavior. |

## Supported Technologies

| Technology | Status | Details |
|------------|--------|---------|
| **Any Language** | ✅ Full support | Anything installable in a Linux Docker container. |
| **Python** | ✅ Excellent | Primary SDK language. Jupyter support. Data science ecosystem. |
| **JavaScript/TypeScript** | ✅ Excellent | Full Node.js ecosystem. React, Next.js, Express, etc. |
| **Go, Rust, Java, C/C++** | ✅ Supported | System-level package installation via apt/brew. |
| **Ruby, PHP, Scala, Kotlin** | ✅ Supported | Any language with a Linux toolchain. |
| **Docker/Kubernetes** | ✅ Supported | Agent can create Dockerfiles, docker-compose configs, and Kubernetes manifests. |
| **ML/AI Workloads** | ✅ With GPU | PyTorch, TensorFlow, Hugging Face, CUDA — with nvidia-docker support. |
| **Any Framework** | ✅ Supported | Not opinionated. Works with whatever your project uses. |

---

**Sources:**
- OpenHands documentation: https://docs.openhands.dev
- OpenHands introduction: https://docs.openhands.dev/overview/introduction
- OpenHands local setup: https://docs.openhands.dev/openhands/usage/run-openhands/local-setup
- OpenHands docs index: https://docs.openhands.dev/llms.txt
- OpenHands GitHub: https://github.com/All-Hands-AI/OpenHands
- OpenHands Cloud: https://app.all-hands.dev
- MIT License: https://github.com/All-Hands-AI/OpenHands/blob/main/LICENSE
