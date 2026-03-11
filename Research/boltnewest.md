# Bolt.new — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026
### Last Updated: March 9, 2026

---

# COMPANY PROFILE

| Field | Detail |
|-------|--------|
| **Parent Company** | StackBlitz, Inc. |
| **Founded** | Eric Simons (CEO) & Albert Pai (CTO) |
| **Total Funding** | $7.9M seed round (April 2022) |
| **Lead Investor** | Greylock |
| **Other Investors** | GV (Google Ventures), Tom Preston-Werner (GitHub co-founder) |
| **Employee Count** | ~20 (2022, quadrupled headcount in 2021) |
| **Revenue** | ~$40M ARR (per community analysis) |
| **StackBlitz Users** | 2M+ monthly developers (2022) |

### Key Company Facts
- Eric Simons famously squatted at AOL headquarters at age 19 while building his first company (Thinkster)
- StackBlitz grew 10x enterprise revenue in 2021
- AWS bill reportedly "a couple hundred bucks" due to edge computing architecture (WebContainers run client-side)
- Developers from 2,000+ companies used the platform during beta
- **2025 hackathon**: 130,000+ registered participants ("largest hackathon ever held"), $1M+ in prizes, in-person events in 7 cities

### User Base Composition
- **67% non-developers** according to community analysis
- Primary: PMs, solo founders, designers needing quick prototypes
- Secondary: developers for rapid scaffolding before moving to Cursor/VS Code

---

# PHASE 1: Product & UX Friction Audit

## Exhaustive UI Matrix

### AI Chat Interface
- **Prompt input** — full-width text input at bottom of screen for natural language prompts
- **Model selector** — users can choose between Claude Agent (default, powered by Claude Sonnet 4.6) or v1 Agent (legacy, multi-model)
- **Plan Mode** — toggle that makes AI plan before executing. Generates structured approach before writing code.
- **Design Systems** — attach brand guidelines (colors, fonts, spacing, components) that persist across all generations. Claims "98% fewer errors" with design systems enabled.
- **Project Knowledge** — attach reference documents (PRDs, specs, API docs) that the AI reads for context
- **Figma Import** — paste Figma frames directly into chat to have AI convert designs to code
- **Image/Screenshot Reference** — upload images for the AI to replicate
- **Suggested prompts** — contextual suggestions below input based on current project state
- **Chat history** — full conversation log with expandable code diffs per message
- **Undo/redo** — per-message rollback of AI changes

### Preview Panel
- **Live preview** — real-time rendering of the generated app
- **Responsive toggle** — desktop/tablet/mobile viewport sizes
- **Split-screen** — code editor + preview side by side
- **Full preview mode** — expand preview to full screen
- **Hot Module Replacement** — changes appear in <100ms via WebContainers + Vite HMR
- **Console output** — terminal/dev console visible during preview for debugging
- **Open in new tab** — preview the app in a separate browser tab with unique URL

### Code Editor
- **File tree** — full project file structure with create/rename/delete
- **Syntax highlighting** — full language support
- **Multi-file editing** — tabs for multiple open files
- **Terminal** — integrated terminal powered by WebContainers (runs in browser)
- **Package manager** — npm/yarn commands run in-browser via WebContainers
- **Git integration** — connect to GitHub, push/pull code

### Project Settings
- **Project Knowledge** — attach docs, specs, guidelines
- **Design System** — brand configuration
- **Integrations** — Supabase, Netlify, GitHub, Figma
- **Environment variables** — for API keys and secrets
- **Agent selection** — Claude Agent vs v1 Agent (legacy)
- **Default database** — Bolt Database vs Supabase

### Bolt Cloud (Hosting & Backend)
- **One-click hosting** — Share (restricted) or Publish (public) buttons
- **Custom domains** — buy domains directly in Bolt or connect existing
- **SSL** — automatic HTTPS on all deployments
- **Bolt Database** — unlimited Postgres databases per project (powered by Supabase under the hood)
- **Authentication** — built-in user signups, logins, password resets, roles & permissions
- **Edge Functions** — serverless functions for APIs and integrations
- **Static deployments** — for simple sites without backend
- **Analytics** — basic traffic analytics on published sites

### Team Features
- **Bolt for Teams** — shared workspace for team collaboration
- **Team Templates** — save project configurations as reusable templates
- **Shared billing** — team-level token pool
- **Member management** — invite/remove team members
- **Template library** — curated starter templates

---

## JTBD Friction Map

### Flow 1: Connecting a Custom Domain

1. Open project (1 click)
2. Click "Domains & Hosting" or gear icon → All project settings (2 clicks)
3. **Option A: Buy domain in Bolt**
   - Search for available domain (1 action)
   - Purchase domain (1 click + payment)
   - Domain auto-connects — ZERO DNS configuration needed
   - **Total: 4 clicks. Cognitive leaps: 0**
4. **Option B: Connect existing domain**
   - Enter domain name (1 action)
   - Copy CNAME/A record provided by Bolt
   - **COGNITIVE LEAP:** Go to external DNS provider, create record
   - Return and verify (1 click)
   - **Total: ~5 clicks + DNS. Cognitive leaps: 1**

**Friction rating: LOW (Option A) / MEDIUM (Option B)**
Bolt's in-app domain purchasing eliminates the biggest friction point. This is the gold standard — no other competitor offers domain purchase within the builder.

### Flow 2: Setting Up a Database Table

1. Start a project or open existing (1 click)
2. Type prompt: "Add a users table with name, email, role, and avatar" (1 action)
3. AI generates schema, creates Bolt Database table, connects it — all automatic
4. View database in Bolt Cloud dashboard (1 click)

**OR manually:**
1. Open project settings → Bolt Database
2. Database UI shows tables, click to manage
3. Bolt Database is Supabase under the hood — full SQL access via Supabase Studio

**Total clicks via AI: 1 prompt**
**Cognitive leaps: 0**
**Friction rating: VERY LOW** — Bolt Database auto-provisions with no external account needed

---

## The "Aha!" Moment

**The "Aha!" is the speed of the preview.**

When a user types their first prompt and sees a full application render in the preview panel in under 2 seconds — with working navigation, styled components, and responsive layout — the speed is viscerally shocking. This is faster than any competitor because of WebContainers: the code compiles and renders in the browser, not on a server.

**The second "Aha!" is the terminal.** Power users see a real terminal running in their browser — `npm install`, `vite dev`, console output — and realize this isn't a toy. It's a real development environment.

**The third "Aha!" is Publish.** One click → live URL. For non-technical users, the transition from prompt to live website in under 5 minutes is transformative.

---

# PHASE 2: "Own The Metal" Architecture Blueprint

## BaaS Reliance

### Database Layer — Bolt Database
- **Bolt Database is Supabase under the hood** — confirmed in documentation ("Powered by trusted platforms like... Supabase")
- As of Sept 30, 2025, new projects use Bolt Database by default (managed Supabase)
- Users can still choose raw Supabase integration as an alternative
- **Unlimited databases per project** on paid plans
- Postgres SQL access via Supabase Studio
- Migrations managed by the AI (generates SQL migration files)
- Users can "claim" Bolt databases in their Supabase account for direct management

### Authentication
- **Built-in auth via Bolt Cloud** — signups, logins, password resets
- Roles and permissions system
- Powered by Supabase Auth (GoTrue) under the hood
- No external auth provider needed

### File Storage
- Handled via Supabase Storage under the hood
- File uploads work in generated apps

### Hosting
- **Bolt Cloud** — built-in hosting included in all tiers (including free)
- **CORRECTION (March 9):** Earlier analysis stated this was "powered by Netlify." Updated research shows Bolt has moved to a custom hosting solution. The original Netlify reference may have been historical or inaccurate.
- Custom domains with auto-SSL
- Website hosting included on all plans (up to 333K requests on free, 1M on Pro)
- Edge computing architecture keeps costs low

### Key Insight for HostPapa
**Bolt depends on Supabase for database/auth/storage** (branded as "Bolt Database"). The hosting layer appears to be owned/custom now, but the data layer is still externally dependent. HostPapa can offer full ownership by self-hosting Supabase and controlling the entire stack.

---

## Infrastructure Cost Architecture — The "Couple Hundred Bucks" AWS Bill

### Why Bolt's Infrastructure Is Nearly Free
Eric Simons has publicly stated their AWS bill is "a couple hundred bucks." This sounds impossible for a platform serving millions of users, but the architecture explains it:

**What runs on the user's browser (cost: $0):**
- The entire Node.js runtime (WebContainers)
- npm package installation
- Vite dev server + HMR
- File system operations
- Build/compile steps
- Preview rendering

**What StackBlitz actually pays for:**
- CDN distribution of static assets (WebContainers OS binary ~1MB, templates)
- API gateway for authentication
- Lambda functions for deployment orchestration
- S3 for template storage
- That's it.

**What they DON'T need (that competitors pay heavily for):**
- EC2/GCE instances for dev environments (Replit, CodeSpaces model)
- Container orchestration (ECS/EKS)
- Load balancers for dev environments
- Persistent storage volumes per user
- Compute scaling with user count

**The insight:** Every other AI builder runs a server-side sandbox per user session. Bolt runs the sandbox on the user's own CPU/RAM. This means Bolt's compute costs are **O(1)** — they don't scale with user count. Replit/CodeSpaces/Lovable compute costs are **O(n)** — they scale linearly (or worse) with users.

**Revenue vs cost:** $40M ARR with near-zero compute costs means their primary expense is AI model usage (Anthropic API calls), which scales with revenue since it's usage-gated behind tokens. This is an unusually clean SaaS margin structure.

**Source:** [StackBlitz Blog — WebContainers](https://blog.stackblitz.com/posts/introducing-webcontainers/), [Latent Space Podcast](https://www.latent.space/p/bolt)

### Open Source Architecture Insights (bolt.diy)

**bolt.diy** is the community fork that reveals Bolt's core architecture patterns:

| Component | Technology |
|-----------|-----------|
| Frontend | Remix (React-based) |
| Backend | Node.js + TypeScript |
| AI SDK | Vercel AI SDK (multi-provider abstraction) |
| Dev Environment | WebContainers API |
| Stream Processing | Chunked response handling for real-time code gen |
| LLM Providers | 19+ supported (OpenAI, Anthropic, Google, Ollama, Groq...) |

Key architectural patterns revealed:
- **Stream processing:** Real-time code generation via chunked LLM responses rendered progressively
- **Recovery mechanisms:** Interrupted stream recovery for connection drops mid-generation
- **Provider abstraction:** Modular system makes adding new LLM providers trivial
- **Error recovery:** Self-healing loops with retry logic

**Source:** [bolt.diy GitHub](https://github.com/stackblitz-labs/bolt.diy)

---

## LLM Orchestration

### Models (Verified March 2026)
- **Claude Sonnet 4.6** — primary model, confirmed by Anthropic customer case study. Chosen specifically for **zero-shot code generation** capability — no RAG system required.
- **Claude Opus 4.6** — available for users who toggle in editor
- **Claude Haiku** — lighter tasks / component switching
- **v1 Agent (legacy)** — multi-model, uses various LLMs. Only supports Supabase (not Bolt Database). Being phased out.

**Source:** [Anthropic Customer Story — StackBlitz](https://claude.com/customers/stackblitz)

### Why Claude Specifically
Eric Simons (Latent Space podcast): Claude 3.5 Sonnet represented an **"order of magnitude difference"** in infrastructure requirements vs other models. Other models required complex RAG systems, extensive prompt engineering, and retrieval pipelines. Claude could generate full-stack apps zero-shot — meaning Bolt's AI pipeline is dramatically simpler (and cheaper) than competitors who need to build RAG infrastructure.

**Source:** [Latent Space Podcast with Eric Simons](https://www.latent.space/p/bolt)

### AI Pipeline Architecture (from bolt.diy open source analysis)

**System Prompt Structure:**
- Task breakdown system — complex requests decomposed into concrete file operations
- Context injection — current file system state + error logs fed as context
- Error recovery loop — automatic detection and correction (see Self-Healing Loop below)
- Artifact generation following Claude's structured output format

**Model Configuration** (from bolt.diy source code):
```typescript
MAX_TOKENS = 128000  // Context window
PROVIDER_COMPLETION_LIMITS = {
  Anthropic: 64000,  // Claude models — much higher than competitors
  // Most other providers cap at 4-8K completions
}
```
That 64K completion limit is significant — it means Bolt can generate entire multi-file applications in a single model call where competitors need multiple rounds.

**Source:** [bolt.diy constants.ts](https://github.com/stackblitz-labs/bolt.diy/blob/main/app/lib/.server/llm/constants.ts)

**Code Generation Flow:**
1. User prompt → natural language intent
2. Task decomposition → break into file-level operations
3. Context injection → current file state + terminal/console error logs
4. Code generation → full-stack scaffolding with 64K completion budget
5. Error instrumentation → WebContainer kernel-level error capture (unique to Bolt — only possible because they control the OS)
6. Self-healing loop → auto-correction via error log feedback
7. Preview render → in-memory Vite HMR, <100ms

**Multi-Provider Support** (via bolt.diy):
- 19+ LLM providers supported in the open-source fork: OpenAI, Anthropic, Google, Ollama, Groq, etc.
- **Vercel AI SDK** for provider abstraction
- Stream processing for real-time code generation — chunked responses rendered progressively
- The commercial Bolt.new is locked to Anthropic; bolt.diy proves the architecture is provider-agnostic

### Hydration Pattern
- **Plan Mode** — separate planning step before generation. AI analyzes prompt + codebase → structured plan → user reviews → AI executes. Reduces wasted tokens on wrong approaches.
- **Design Systems** — persistent brand context (colors, fonts, spacing, components) injected into every prompt. Constrains the design space → reduces hallucination. Claims "98% fewer errors" with design systems enabled.
- **Project Knowledge** — uploaded docs (PRDs, API specs, brand guides) injected as context for domain-specific awareness.

### Token Economy
- **Token-based pricing** — each AI interaction consumes Bolt tokens (abstracted credit unit, not raw LLM tokens)
- Different plans have different allocations
- Plan Mode likely uses fewer tokens (planning step is cheaper inference)
- **Unused tokens roll over** to next month (Pro and Teams) — unique among competitors

---

## Preview Compute Environment — WebContainers

### What WebContainers Actually Are
WebContainers are a **custom-built operating system** (~1MB binary) compiled to WebAssembly, running entirely in the browser. This is NOT a Docker container or a VM — it's a purpose-built OS kernel subset that implements enough of POSIX to run Node.js natively in a browser tab.

**Source:** [StackBlitz Blog — Introducing WebContainers](https://blog.stackblitz.com/posts/introducing-webcontainers/)

### Core Architecture

**Syscall Emulation Layer:**
- Maps Linux system calls to browser APIs (Web Workers, SharedArrayBuffer, Service Workers)
- Implements a minimal POSIX-compatible process model
- Custom process orchestration using Web Workers for concurrency
- Strips everything a dev environment doesn't need: audio drivers, graphics drivers, most of the traditional OS
- Result: ~1MB total vs 60-100MB for a minimal Docker container

**Filesystem:**
- **In-memory virtual filesystem** — all file operations happen in browser memory
- **IndexedDB persistence layer** — survives page refreshes and session recovery
- No actual disk I/O — everything is virtualized reads/writes to memory
- **File System Access API** (Chrome only) — enables direct read/write to local filesystem when user grants permission, creating a desktop-like experience
- File operations are instrumented at the kernel level for error tracking (this is what enables the self-healing loop)

**Networking:**
- **Service Worker proxy** handles all HTTP/HTTPS and WebSocket traffic
- Local dev servers get programmatic URLs routed through the Service Worker — **lower latency than actual localhost** because it's in-memory routing, not a TCP roundtrip
- **No raw TCP/UDP sockets** — this is a hard browser limitation. WebContainers can make HTTP requests and open WebSocket connections, but cannot listen for incoming TCP connections or do raw socket operations
- This means: no database servers, no custom protocol servers, no SSH. All backend functionality must be handled by external services (Supabase)

**Node.js Runtime:**
- Full Node.js execution environment in-browser
- Native npm/pnpm/yarn support with **5x faster installs** vs local (optimized virtual filesystem eliminates disk I/O bottleneck)
- **20% faster builds** vs local development (same reason — in-memory operations)
- ESM module support (transpiler-based, native ESM in progress)
- **Native C/C++ modules: LIMITED** — many won't work because there's no actual system to link against
- **No postinstall scripts** — security sandbox prevents arbitrary code execution during install
- WebAssembly modules fully supported

**Source:** [Latent Space Podcast with Eric Simons](https://www.latent.space/p/bolt), [Reddit r/javascript WebContainers discussion](https://www.reddit.com/r/javascript/comments/nhdfkj/)

### Browser Requirements & Isolation
```http
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```
- **SharedArrayBuffer required** — COOP/COEP headers mandatory
- Each project gets a unique subdomain (e.g., `xyz.local.webcontainer.io`) for domain isolation
- Chrome: "Block Third Party Cookies" breaks functionality
- Brave: Shields blocks Service Workers by default
- Edge: "Strict" security mode disables WebAssembly
- Memory ceiling: browser-dependent, roughly ~500MB-1GB per tab

### Performance Characteristics
- **Boot time:** Milliseconds (OS binary is ~1MB, cached aggressively by browser)
- **HMR latency:** <100ms (in-memory Vite dev server, no network hop)
- **npm install:** 5x faster than local (no disk I/O)
- **Build:** 20% faster than local (same reason)
- Startup: <100ms vs 500ms-5s for server-based container approaches (Replit, CodeSpaces)

### Hard Limitations
| What | Status | Why |
|------|--------|-----|
| Python/Go/Rust/Java | ❌ | Only Node.js runtime implemented |
| Raw TCP/UDP sockets | ❌ | Browser sandbox prevents this |
| Native C/C++ modules | ⚠️ Limited | No native system to link against |
| Database servers | ❌ | Can't listen for connections |
| File uploads >browser memory | ❌ | No real disk |
| postinstall scripts | ❌ | Security sandbox blocks them |

### Licensing & Competitive Moat
- **⚠️ WebContainers SDK is MIT licensed** — github.com/stackblitz/webcontainer-core (MIT License, Copyright 2021 StackBlitz)
- 748+ open issues on GitHub — active but has rough edges
- The SDK is open but the **real moat is 7+ years of R&D** and a partnership with the Chrome team
- Building on top of WebContainers is free; building something *as good as* WebContainers would take years
- Bolt's actual moat: AI integration quality, UX polish, and the infra cost advantage (below)

### Key Insight for HostPapa
**WebContainers are MIT licensed — HostPapa can use them directly.** But the real question is whether you want a client-side architecture (WebContainers, $0 compute cost, frontend-only) or a server-side architecture (Docker/VM sandboxes, compute cost per user, but full-stack capability). WebContainers trade backend flexibility for near-zero infrastructure cost. HostPapa could adopt WebContainers for preview and pair with self-hosted Supabase for the backend — getting Bolt's cost structure with owned infrastructure.

---

## The Diffing Engine

### How Bolt Applies Code Changes (from bolt.diy source)

**File operation model** (from source code):
```typescript
interface File {
  type: 'file';
  content: string;
  isBinary: boolean;
  isLocked?: boolean;      // Prevents AI from modifying
  lockedByFolder?: string;  // Folder-level locks
}
```

**Generation patterns:**
- **New files:** Full-file generation — AI writes the entire file in one completion
- **Existing files:** Diff-based edits shown as expandable diffs in chat
- **Ignore system:** Standard patterns excluded from AI context:
  ```
  node_modules/**, .git/**, dist/**, build/**, .next/**, coverage/**
  ```
- Changes applied directly to the WebContainers in-memory virtual filesystem → Vite HMR picks them up → preview updates in <100ms

**Source:** [bolt.diy source code](https://github.com/stackblitz-labs/bolt.diy)

### Version Control
- **Undo/redo per AI message** — each chat interaction is a rollback point
- **GitHub integration** — push to repo for proper git history
- **Revert changes** — can revert specific AI actions
- No built-in branching (use GitHub for that)
- File-level change tracking in chat interface

---

# PHASE 3: Maintainability & Guardrail Teardown

## Preventing Spaghetti Code

### Enforced Stack
- **Primary:** React + Vite + Tailwind (for Claude Agent)
- **Also supported:** Next.js, Astro, Vue, Svelte, Angular, plain HTML/JS
- v1 Agent supports even more frameworks
- **Unlike Base44, Bolt does NOT force a single framework** — users can choose
- This is more flexible but increases hallucination risk (AI must be good at all frameworks)

### Design System Enforcement
- **Design Systems feature** is Bolt's primary code quality tool
- When configured, the AI generates components that follow the brand guidelines
- Colors, fonts, spacing, component styles are all constrained
- This dramatically reduces inconsistency across generated components
- Teams can share design systems via Team Templates

### Code Quality
- No automated linting in the builder
- No ESLint/Prettier enforcement
- No complexity analysis
- No component size warnings
- Code quality depends on the AI model (Claude Sonnet 4.6 produces good React code)

---

## The Self-Healing Loop

### How It Actually Works (Technical Detail)

This is Bolt's most architecturally interesting feature, and it's only possible because they control the OS.

**The mechanism:**
1. AI generates code → written to WebContainers virtual filesystem
2. Vite dev server (running in WebContainers) attempts to compile
3. **Kernel-level instrumentation** captures errors at multiple layers:
   - **Process-level:** Node.js runtime errors, uncaught exceptions
   - **Build-level:** Vite/webpack compilation failures, missing imports, type errors
   - **Console-level:** Browser console errors from the preview iframe
   - **File system-level:** Missing files, broken imports detected at FS access time
4. Error logs are fed back to Claude as context in the next completion
5. AI generates a targeted fix → applied to filesystem → cycle repeats
6. Loop continues until clean compilation or max retries exhausted

**Why this is unique to Bolt:** Traditional container-based approaches (Replit, CodeSpaces) run a standard OS where error instrumentation requires bolting on monitoring tools. Bolt controls the kernel — they can intercept every syscall, every process spawn, every file read. The error surface is instrumented at the OS level, not the application level.

**Source:** [Latent Space Podcast with Eric Simons](https://www.latent.space/p/bolt) — "We can instrument at every level... process, runtime, build system, file system. That's impossible with traditional containers because of OS variability."

### What It Catches vs What It Misses
| Error Type | Auto-Fixed? | Why |
|------------|-------------|-----|
| Syntax errors | ✅ Yes | Detected at compile time |
| Missing imports | ✅ Yes | Vite reports missing modules |
| Type errors | ✅ Yes | TypeScript compiler catches these |
| Runtime exceptions | ✅ Yes | Process-level instrumentation |
| Logical errors | ❌ No | Code compiles fine, just does wrong thing |
| Accessibility issues | ❌ No | Not in error instrumentation scope |
| Performance problems | ❌ No | Not in error instrumentation scope |
| Security vulnerabilities | ❌ No | No security scanner (unlike Lovable) |

### "98% Fewer Errors" Claim
Likely means 98% of first-attempt compilation/runtime errors are auto-fixed before the user sees them. This is plausible given the instrumentation depth — most coding errors ARE syntax/import/type errors that a good LLM can fix with the error message as context. The claim is about compilation errors, not application correctness.

---

## Version Control

### Rollback
- Per-message undo/redo
- Revert specific AI actions
- GitHub push for full git history

### What's Available
- GitHub integration — push/pull/branch
- File change tracking in chat
- Rollback to any previous AI message

### What's NOT Available
- No built-in git (depends on GitHub integration)
- No visual diff viewer
- No merge conflict resolution
- No snapshot/named save points within Bolt itself

---

# PHASE 4: GTM & Telco Partner Strategy

## Pricing Model (Verified March 2026)

### Tier Structure

| Tier | Price | Token Allocation | Key Limits |
|------|-------|-----------------|------------|
| **Free** | $0 | 300K daily / 1M monthly | Bolt branding, 10MB upload, 333K web requests |
| **Pro** | $25/month | 10M/month (no daily cap) | No branding, 100MB upload, 1M requests, custom domains, unused tokens roll over |
| **Teams** | $30/user/month | Pooled team tokens | Everything in Pro + centralized billing, admin controls, private NPM registries, design system knowledge |
| **Enterprise** | Custom | Custom | SSO, audit logs, compliance, dedicated manager, 24/7 priority, custom SLAs, data governance |

### Token Economy
- Token-based (not credit-based like Lovable) — each AI interaction consumes tokens
- Complex prompts consume significantly more tokens than simple ones
- **Unused tokens roll over** to next month (Pro and Teams) — unique among competitors
- No visible per-interaction cost breakdown to users
- Image editing with AI included on Pro+

### Hidden Limits
1. **WebContainers memory** — browser-imposed limit (~500MB), not disclosed
2. **Bolt Database rows** — "unlimited" but likely has soft limits at scale (Supabase under the hood)
3. **Web request caps** — 333K (free), 1M (Pro) — could be limiting for popular apps
4. **File upload limits** — 10MB (free), 100MB (Pro)
5. **Token burn rate** — unpredictable, complex prompts use significantly more

---

## B2B2C Channel Readiness

### Team & Enterprise Features
- **Bolt for Teams** — shared workspace, pooled tokens, team templates
- **Enterprise tier** — SSO, admin controls, priority support
- Team Templates — save and share project configurations

### White-Label Assessment
- **NOT white-label ready:**
  - Builder UI is Bolt-branded throughout
  - Published apps go to Bolt Cloud (Netlify) — Bolt branding in infrastructure
  - No embeddable builder widget
  - No API for programmatic project creation
  - No custom branding options for the builder itself

### Channel Partner Program
- **No documented partner/reseller program**
- Enterprise tier exists but is direct-to-customer
- No multi-tenant billing for sub-accounts
- No API-level management tools for partners

---

## Positioning & Persona

### Hero Copy
- **"Build apps with AI"** / **"From prompt to production"**
- Emphasis on speed: "in minutes, not months"
- Heavy emphasis on WebContainers technology as differentiator

### Target Persona
**Primary: Technical-adjacent creators** — designers, product managers, solo founders who understand tech but don't code daily. More technical than Base44's target.

**Secondary: Developers for rapid prototyping** — use Bolt to scaffold projects, then continue in VS Code/Cursor. GitHub integration supports this workflow.

---

# PHASE 5: Enterprise Compliance & Accessibility

## WCAG 2.2 AA Compliance

### Builder UI
- Complex SPA with multiple panels — likely NOT fully keyboard-accessible
- No accessibility documentation
- WebContainers terminal may not be screen-reader compatible
- No VPAT published

### Generated Code
- Depends on framework and AI model quality
- Claude Sonnet 4.6 generates reasonable semantic HTML
- No automated accessibility checking on generated output
- Design Systems could theoretically enforce accessibility rules but this isn't documented

---

## Tenant Isolation

- **Workspace-based isolation** — each team has separate workspace
- Project-level access controls
- Bolt Database uses Supabase's project isolation (schema-per-project)
- No documented data residency controls
- SSO available on Enterprise tier

---

## Certifications

| Certification | Status |
|--------------|--------|
| SOC 2 Type II | ❓ Not publicly documented |
| ISO 27001 | ❌ Not mentioned |
| GDPR | ⚠️ Privacy policy exists, no DPA template visible |
| HIPAA | ❌ Not applicable |
| VPAT | ❌ Not published |
| FedRAMP | ❌ Not applicable |

### Data Processing
- Code and prompts processed by Anthropic (Claude)
- Data stored on Supabase (managed) and Netlify (hosting)
- Three third-party processors: Anthropic, Supabase, Netlify
- No data sovereignty controls

---

# PHASE 6: Churn & Scalability Ceiling

## Code Ejection

### Export Options
- **GitHub push** — full project pushed to GitHub, clean repo structure
- **Download project** — download as zip
- Code is standard React/Vite/Next.js — portable
- **However:** Bolt Database connections use Supabase SDK — if ejecting, you need your own Supabase instance
- Auth/storage tied to Bolt Cloud (Supabase-managed) — must migrate

### Community Complaints
1. **"Token burn is unpredictable"** — users don't know how many tokens a prompt will consume
2. **"WebContainers crash on complex projects"** — memory limits in browser cause instability
3. **"v1 Agent to Claude Agent migration was rough"** — breaking changes in agent behavior
4. **"Can't do backend-heavy apps"** — WebContainers don't run Python, databases are limited
5. **"GitHub sync is one-way"** — push works, but pulling external changes back into Bolt is limited

---

## The Logic Wall

| Level | Capability |
|-------|-----------|
| ✅ Works great | Static sites, SPAs, landing pages, dashboards, CRUD apps |
| ⚠️ Works with effort | Auth flows, database CRUD, API integrations, multi-page apps |
| ❌ Struggles | Complex backend logic, real-time features, multi-tenant apps |
| ❌ Fails | Python/Go backends, native mobile, complex DevOps, enterprise integrations |

### Where Users Leave
- **Cursor/VS Code** — developers who outgrow the AI builder
- **Lovable** — users wanting better Supabase integration
- **Replit** — users needing non-JavaScript backends (Python, etc.)

---

## Community Sentiment & Real-World Reception

### What the Community Says (Reddit r/webdev, r/boltnewbuilder)

**Senior Dev quote:**
> "Bolt kind of tries to do everything - frontend, backend, database. And long story short... It is jack of all trades, master of none. The backend code it generates? Let's just say there's a reason 67% of their users aren't developers."

**On production readiness:**
> "Anyone else notice how all the Bolt success stories are MVPs and demos, while v0 powers actual production apps? That's not coincidence."

**Positioning comparison:**
> "v0 for anything serious, Bolt for prototypes, hackathon demos and initial scoping layouts"

**Market label:** Community increasingly describes Bolt as "Wix for the AI age" — powerful for non-developers, not trusted for production by professionals.

### What Users Praise
- Prototyping speed — fastest time from idea to working demo
- Full-stack generation in one prompt (vs v0 which is component-only)
- Beginner accessibility — lowest barrier to entry
- Integrated hosting on all tiers including free
- Token rollover (unused tokens carry to next month)

### What Users Criticize
- Code quality degrades with complex applications
- Token burn rate is unpredictable and opaque
- WebContainers crash on large/complex projects (browser memory limits)
- Backend code quality significantly below frontend quality
- GitHub sync is effectively one-way (push works, pull back into Bolt is limited)
- v1 Agent → Claude Agent migration broke existing workflows

---

# EXHAUSTIVE FEATURE INDEX

## AI & Generation

| Feature | Description | Details |
|---------|-------------|---------|
| **Claude Agent** | Default AI agent powered by Claude Sonnet 4.6 | Best for complex apps. Supports Bolt Database, multi-file edits, auto-error-correction. Replaced v1 Agent as default. |
| **v1 Agent (Legacy)** | Original multi-model AI agent | Uses various LLMs. Only supports Supabase (not Bolt Database). Being phased out. Chat history resets when switching agents. |
| **Plan Mode** | AI plans before executing | Toggle on/off. AI generates structured approach, user reviews, then AI executes. Reduces wasted tokens on wrong approaches. |
| **Design Systems** | Brand guidelines that persist across all generations | Upload colors, fonts, spacing, component styles. AI constrains output to match. Claims "98% fewer errors." Shareable via Team Templates. |
| **Project Knowledge** | Attach reference documents for AI context | Upload PRDs, API docs, brand guides. AI reads these before generating. Supports multiple documents per project. |
| **Figma Import** | Convert Figma frames to code | Paste Figma frame URL or drag screenshot into chat. AI converts design to React/HTML code. Works with existing projects. |
| **Image/Screenshot Reference** | Upload images for AI to replicate | Drag and drop images into chat. AI analyzes and generates matching UI. |
| **Suggested Prompts** | Contextual AI suggestions | AI suggests next steps based on current project state. Appear below chat input. |
| **Auto Error Correction** | Self-healing code generation loop | AI detects compilation/runtime errors from WebContainers terminal, auto-generates fixes, retries. |

## Code Editor

| Feature | Description | Details |
|---------|-------------|---------|
| **File Explorer** | Full project file tree | Create, rename, delete files and folders. Drag and drop to reorganize. |
| **Syntax Highlighting** | Multi-language syntax coloring | Supports JavaScript, TypeScript, HTML, CSS, JSON, Markdown, and more. |
| **Multi-Tab Editing** | Open multiple files simultaneously | Tab bar with close, reorder. Switch between files. |
| **Terminal** | Integrated terminal in browser | Powered by WebContainers. Run npm, node, git commands. Full terminal emulator. |
| **Split View** | Code + Preview side by side | Resizable panels. Can focus on code, preview, or both. |
| **Undo/Redo** | Per-message AI change rollback | Each AI message is a rollback point. Undo reverts all changes from that message. |
| **Diff View** | Code changes per AI message | Expandable diffs in chat showing exactly what changed. |

## Preview & Runtime

| Feature | Description | Details |
|---------|-------------|---------|
| **WebContainers Preview** | In-browser Node.js runtime | Full Vite dev server running in browser. <100ms hot reload. Zero server cost. |
| **Responsive Preview** | Desktop/Tablet/Mobile viewports | Toggle between viewport sizes. Preview adapts in real-time. |
| **Full-Screen Preview** | Expand preview to full browser tab | Preview URL can be opened in new tab. Shareable preview links. |
| **Console Output** | Dev console in preview panel | See console.log, errors, warnings. Helps debug generated code. |
| **Hot Module Replacement** | Instant code updates in preview | Changes appear in <100ms without full page reload. |

## Bolt Cloud — Hosting

| Feature | Description | Details |
|---------|-------------|---------|
| **One-Click Publish** | Publish app to production | Click "Publish" → live URL. No configuration needed. Powered by Netlify. |
| **Share Mode** | Restricted access publishing | "Share" button creates URL with limited access. Not publicly discoverable. |
| **Custom Domains** | Connect your own domain | Enter domain, configure DNS (CNAME), auto-SSL. |
| **Buy Domains** | Purchase domains in Bolt | Search and buy domains without leaving Bolt. Auto-configures DNS and SSL. Zero DNS friction. |
| **Auto SSL** | HTTPS on all deployments | Let's Encrypt certificates auto-provisioned by Netlify. |
| **Static Hosting** | Fast CDN-hosted static sites | For sites without backend. Global CDN via Netlify. |
| **Analytics** | Basic traffic analytics | Page views, visitors on published sites. |

## Bolt Cloud — Database

| Feature | Description | Details |
|---------|-------------|---------|
| **Bolt Database** | Managed Postgres database | Unlimited databases per project. Powered by Supabase. Auto-provisioned when AI generates data models. |
| **Database Dashboard** | Visual database management | View tables, rows, run queries. Supabase Studio interface. |
| **SQL Access** | Direct SQL query execution | Full Postgres SQL via Supabase Studio. |
| **Auto-Schema Generation** | AI creates database schema from prompts | Describe your data model in natural language → AI generates SQL migrations and applies them. |
| **Supabase Integration** | Alternative to Bolt Database | Connect your own Supabase project. Full Supabase features (RLS, Edge Functions, Storage). |
| **Database Migration** | Migrate Bolt DB to Supabase | Claim Bolt databases in your Supabase account for direct management. |

## Bolt Cloud — Authentication

| Feature | Description | Details |
|---------|-------------|---------|
| **User Signups** | Built-in user registration | Email/password signups with email verification. |
| **User Login** | Authentication flows | Login with email/password. Session management. |
| **Password Reset** | Self-service password recovery | Reset via email link. |
| **Roles & Permissions** | Access control system | Create custom roles, assign to users, enforce permissions in app. |
| **User Management** | Admin user dashboard | View, edit, delete users. Assign roles. |

## Bolt Cloud — Edge Functions

| Feature | Description | Details |
|---------|-------------|---------|
| **Serverless Functions** | Backend API endpoints | Powered by Supabase Edge Functions (Deno runtime). For API routes, webhooks, server-side logic. |
| **Stripe Integration** | Payment processing | Edge Functions handle Stripe webhooks, checkout sessions, subscription management. |
| **Third-Party APIs** | External API communication | Edge Functions proxy API calls, keeping keys server-side. |

## Integrations

| Feature | Description | Details |
|---------|-------------|---------|
| **GitHub** | Version control integration | Push projects to GitHub. Pull changes (limited). Branch management. |
| **Supabase** | Database/auth provider | Connect Supabase account for database, auth, storage, edge functions. Alternative to Bolt Database. |
| **Netlify** | Hosting provider | Powers Bolt Cloud hosting. Users don't interact directly. |
| **Figma** | Design tool integration | Import Figma frames into projects. AI converts designs to code. |
| **Stripe** | Payment processing | Via Edge Functions. Checkout, subscriptions, webhooks. |

## Supported Technologies

| Technology | Status | Notes |
|------------|--------|-------|
| **React + Vite** | ✅ Primary | Default for Claude Agent. Best AI support. |
| **Next.js** | ✅ Supported | App Router supported. No Supabase integration (Bolt DB only). |
| **Astro** | ✅ Supported | Static site generator. Good for content sites. |
| **Vue** | ✅ Supported | Via Vite. |
| **Svelte** | ✅ Supported | Via Vite/SvelteKit. |
| **Angular** | ✅ Supported | Full Angular CLI support. |
| **Plain HTML/JS** | ✅ Supported | For simple static sites. |
| **Python** | ❌ Not supported | WebContainers only run Node.js/JavaScript. |
| **Go/Rust/Java** | ❌ Not supported | WebContainers limitation. |

## Team & Enterprise

| Feature | Description | Details |
|---------|-------------|---------|
| **Team Workspace** | Shared team environment | Pooled tokens, shared projects, collaborative editing. |
| **Team Templates** | Reusable project configurations | Save starter templates with design systems, project knowledge, and code. Share across team. |
| **Shared Billing** | Team-level payment | Single bill for entire team. Token pool shared among members. |
| **Member Management** | Add/remove team members | Invite by email. Role-based access (admin, member). |
| **SSO** | Single Sign-On | Enterprise tier. SAML/OIDC (details not public). |
| **Admin Controls** | Enterprise management | Enterprise tier. Details not publicly documented. |
| **Priority Support** | Dedicated support channel | Enterprise tier. |

## Account & Settings

| Feature | Description | Details |
|---------|-------------|---------|
| **Personal Settings** | Account configuration | Email, password, profile. |
| **Project Settings** | Per-project configuration | Agent selection, database choice, integrations, environment variables, project knowledge, design system. |
| **Token Usage** | Usage monitoring | View token consumption per project and overall. |
| **Billing Management** | Subscription and payment | Upgrade/downgrade plans, update payment method, view invoices. |

---

**Sources:**
- Bolt.new documentation: https://support.bolt.new
- Bolt.new blog: https://bolt.new/blog
- Bolt.new pricing: https://bolt.new/pricing
- StackBlitz WebContainers: https://webcontainers.io
- StackBlitz WebContainers docs: https://stackblitz.com/docs/platform/webcontainers
