# Lovable.dev — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026
### Last Updated: March 9, 2026

---

# COMPANY PROFILE

| Field | Detail |
|-------|--------|
| **Legal Name** | Lovable (formerly GPT Engineer, rebranded Dec 2024) |
| **Founded** | 2024 (launched November 18, 2024) |
| **Founders** | Anton Osika (CEO), Fabian Hedin |
| **HQ** | Stockholm, Sweden |
| **Employees** | 15 (Feb 2025), likely 100+ by March 2026 given growth |
| **Valuation** | $6.6B (Series B, December 2025) |
| **Total Funding** | $545M+ across 3 rounds |

### Funding History

| Round | Date | Amount | Valuation | Lead Investors |
|-------|------|--------|-----------|----------------|
| **Seed** | Feb 2025 | $15M | — | Creandum. Angels: Charlie Songhurst (Meta board), Thomas Wolf (Hugging Face) |
| **Series A** | Jul 2025 | $200M | $1.8B | Accel. Also: 20VC, byFounders, Creandum, Hummingbird, Visionaries Club |
| **Series B** | Dec 2025 | $330M | $6.6B | CapitalG + Menlo Ventures (Anthology). Also: NVIDIA (NVentures), Salesforce Ventures, Databricks Ventures, Deutsche Telekom (T.Capital), Atlassian Ventures, HubSpot Ventures, Khosla Ventures, DST Global, EQT Growth |

### User & Revenue Metrics (Verified)

| Metric | Value | Date |
|--------|-------|------|
| **Total projects created** | 25M+ | Dec 2025 |
| **New projects daily** | 100,000+ | Dec 2025 |
| **Visits to Lovable-built sites** | 6M+ daily / 200M+ monthly | Dec 2025 |
| **Paying customers** | 30,000+ | Feb 2025 |
| **ARR** | $17M (3 months after launch) → $100M (Jul 2025) | |

### Enterprise Customers (Named)
- **Zendesk** — reduced prototype time from 6 weeks to 3 hours
- **Deutsche Telekom** — UI projects requiring rapid stakeholder alignment
- **Uber AI** — interactive prototypes
- **Leading ERP platform** — 75% of frontend generated through Lovable
- **Revenue-generating businesses built on Lovable:** Lumoo ($800K ARR), ShiftNex ($1M ARR), QuickTables ($100K+/year)

---

# PHASE 1: Product & UX Friction Audit

## Exhaustive UI Matrix

### AI Chat Interface
- **Prompt input** — bottom text area for natural language prompts
- **Plan Mode** — AI generates a structured plan before coding. User reviews/approves. Reduces wasted iterations.
- **Agent Mode** — full-stack autonomous agent that handles database setup, API connections, Supabase integration, deployment — end-to-end
- **Chat Mode** — toggle between chat (default, iterative development) and code mode
- **Code Mode** — directly edit generated code in the editor
- **Image upload** — drag/paste screenshots or mockups for AI to replicate
- **File references** — reference specific project files in prompts using @mentions
- **Knowledge context** — attach external docs/specs that persist across prompts
- **Suggested prompts** — contextual next-step suggestions
- **AI model** — primarily Claude (Anthropic), with model selection not exposed to users

### Visual Editor
- **Select & Edit** — click any element in the preview to select it
- **Properties panel** — when element selected: edit text, colors, fonts, spacing, layout directly
- **Component tree** — hierarchical view of all components
- **Responsive editing** — edit different breakpoints visually
- **Style overrides** — visual changes generate proper Tailwind classes
- **Real-time preview** — changes appear instantly as you edit

### Code Editor
- **Full file tree** — browse all project files
- **Syntax highlighting** — TypeScript, JavaScript, HTML, CSS, JSON
- **In-line editing** — edit any file directly
- **Code Mode** — switch to full code editor view
- **Diff view** — see what AI changed per interaction

### Project Dashboard
- **Version History** — view all past states, restore any version
- **Security View** — project-level security scan results and findings
- **Supabase Management** — database tables, RLS policies, Edge Functions
- **Settings** — project configuration, integrations, custom domains
- **Team Management** — workspace members, roles, permissions

### Deployment & Hosting (Lovable Cloud)
- **Publish button** — one-click deploy to production
- **Preview deployments** — shareable preview URLs before publishing
- **Custom domains** — connect your own domain with auto-SSL
- **Rollback** — instant rollback to previous deployment
- **Environment variables** — secrets management for API keys

### Security Center (Workspace-Level)
- **Cross-project vulnerability dashboard** — monitor security across all workspace projects
- **RLS Analysis** — automated review of database access policies
- **Database Security Check** — schema + RLS configuration review
- **Code Security Review** — application code vulnerability analysis
- **Dependency Audit** — npm package vulnerability detection
- **Critical issue blocking** — warns before publishing with unresolved security issues

---

## JTBD Friction Map

### Flow 1: Connecting a Custom Domain

Based on Lovable Cloud docs:
1. Open project (1 click)
2. Click Publish or Settings → Custom Domains (2 clicks)
3. Enter domain name (1 action)
4. Lovable provides DNS records (CNAME or A record)
5. **COGNITIVE LEAP:** User goes to DNS provider, creates record
6. Return to Lovable, click Verify (1 click)
7. SSL auto-provisions

**Total clicks in Lovable: ~5**
**Cognitive leaps: 1 (DNS configuration)**
**Friction rating: MEDIUM** — standard DNS friction, no in-app domain purchasing (unlike Bolt)

### Flow 2: Setting Up a Database Table

**With Agent Mode:**
1. Type: "Add a products table with name, price, description, and category" (1 action)
2. Agent Mode automatically:
   - Connects to Supabase (or provisions Lovable Cloud database)
   - Creates the table with proper types
   - Generates RLS policies
   - Creates TypeScript types
   - Connects frontend to backend
3. Done — zero additional steps

**Total clicks: 1 prompt**
**Cognitive leaps: 0**
**Friction rating: VERY LOW** — Agent Mode handles the entire stack

---

## The "Aha!" Moment

**The "Aha!" is Agent Mode handling the full stack automatically.**

Unlike competitors where the AI generates frontend code and you manually set up the database, Lovable's Agent Mode handles everything — it provisions the database, creates tables, writes RLS policies, generates TypeScript types, and connects the frontend. The user describes a feature in natural language and gets a complete, working full-stack implementation.

**The second "Aha!" is the security scanner.** When you try to publish, Lovable automatically scans your code for vulnerabilities, checks RLS policies, audits dependencies, and blocks publishing if critical issues are found. No other AI builder does this. It makes users feel safe.

**The third "Aha!" is the visual editor.** Click any element → change its text, color, spacing in a panel → see changes in real-time. For non-technical users, this is more intuitive than chat-based prompting.

---

# PHASE 2: "Own The Metal" Architecture Blueprint

## BaaS Reliance

### Database Layer
- **Lovable Cloud (new)** — managed database service powered by Supabase
  - Automatic provisioning — Agent Mode sets up databases without user configuration
  - Built-in to Lovable's hosting
  - PostgreSQL with full SQL access
- **Supabase Integration (legacy/alternative)** — connect your own Supabase project
  - Full Supabase features: database, auth, storage, edge functions, realtime
  - User manages Supabase account separately
  - AI generates Supabase-native code (createClient, RLS, Edge Functions)
  - Deep integration: Lovable can create tables, write RLS policies, deploy Edge Functions directly

### Authentication
- **Via Supabase Auth (GoTrue)** — email/password, OAuth providers, magic links
- AI generates auth flows (sign up, login, password reset, protected routes)
- RLS policies enforce data access per authenticated user

### File Storage
- **Via Supabase Storage** — file uploads, image hosting
- AI generates upload components and storage bucket configuration

### Hosting
- **Lovable Cloud** — new hosting infrastructure
  - Managed by Lovable
  - Custom domains with auto-SSL
  - Environment variables / secrets
  - Rollback capability
- **Alternative:** GitHub export → deploy anywhere (Netlify, Vercel, self-hosted)

### Key Insight for HostPapa
**Lovable is transitioning from external Supabase dependency to their own managed cloud.** This validates the "own the stack" approach. However, Lovable Cloud is still built on managed Supabase underneath. HostPapa can go further — self-hosted Supabase on our own metal. The code patterns Lovable generates (Supabase SDK calls) would work identically against self-hosted Supabase.

---

## LLM Orchestration

### Models (Verified March 2026)
- **Default model:** Gemini 3 Flash (as of Jan 2026) — cost-effective for standard tasks
- **Primary premium:** Claude Opus 4.6 (Feb 2026) — "at no additional cost"
- **Previous:** Claude Opus 4.5 (Dec 2025)
- **Also available:** GPT-5.2, Gemini 3 Pro, Nano Banana Pro
- Model selection IS exposed to users on paid plans — users can choose which model to use

### Multi-Provider Infrastructure (Deep Technical Detail)
As revealed in the March 2026 blog post "Routing Billions of Tokens per Minute":

**Scale:** Processing **1B+ tokens/minute** at peak. At average 1.4K tokens/request, that's roughly **43M requests/hour**.

**Provider Architecture:**
- Three providers: Anthropic (direct API), Vertex AI (Google Cloud), AWS Bedrock
- Each provider has its own rate limits, latency characteristics, and failure modes
- Custom load balancer (not off-the-shelf) — likely powered by their Maglev consistent hashing library

**PID Controller Implementation:**
- Classic control theory applied to LLM traffic routing
- One PID controller **per provider**, recalculating every 30 seconds
- Tracks: latency, error rates, throughput per provider
- Scoring formula: `score = successes - 200×errors + 1`
  - 0.5% error threshold triggers weight reduction
  - The `+1` bias prevents permanently blacklisting a provider after transient failures
  - Provider weights calculated greedily: preferred provider gets full availability, next fills remaining

**Sticky Routing for Prompt Caching:**
- **Project-level affinity** — each project gets a cached fallback chain for several minutes
- This preserves prompt caching across sequential agent turns within the same project
- Breaking cache (switching providers mid-project) forces full context reprocessing → higher latency AND higher cost
- Provider rate limits measured in **non-cached tokens** — poor cache behavior can exhaust ALL provider capacity simultaneously
- Sticky routing maintains cache hit rates → **estimated 50-90% token cost reduction** vs naive multi-provider round-robin

**Failover:**
- Three-tier: Provider A → Provider B → Emergency fallback
- Health checks every 30 seconds
- Circuit breaker pattern prevents cascade failures
- **Auto-healing without human intervention** during provider outages
- Multiple fallback chains with **probabilistic provider ordering** (not a single ranked list)

**Source:** [Routing Billions of Tokens per Minute](https://lovable.dev/blog/routing-billions-of-tokens-per-minute)

**Cost Implications:** 1B+ tokens/minute at LLM API pricing implies massive costs. Even at cached rates, this is likely $50K-$200K+/day in API spend. The multi-provider routing isn't just for reliability — it's for **cost optimization** through competitive pricing across providers and maximizing cache hit rates.

### Backend Migration
- **Core platform migrated from Python to Go** (Feb 2025) — the Go rewrite was specifically for the performance demands of the LLM routing layer and API orchestration at this scale

### Hydration Pattern
- **Plan Mode** (Feb 2026) — separate planning step before code generation
  - AI analyzes prompt + codebase
  - Generates structured plan with specific steps
  - User reviews/approves/modifies before execution
  - Reduces wasted iterations on wrong approaches
- **Agent Mode** — autonomous multi-step execution
  - Plans internally, executes multiple actions in sequence
  - Handles database, API, frontend, deployment
  - More credits per interaction but more complete output
- **File references** — @mentions specific files to include as context
- **Cross-project referencing** (Mar 2026) — reference implementations from other projects
- **Knowledge base** — persistent project documentation injected into prompts
- **MCP servers** — custom Model Context Protocol servers for external tool integration (all paid plans)

### AI Pipeline Architecture (from system prompt analysis)

**System Prompt:** 19.8KB — one of the largest known AI builder system prompts, indicating sophisticated prompt engineering.

**Tool System** (from leaked system prompts):
- `search-replace` — primary tool for surgical code edits (exact text matching required)
- `write-file` — create new files
- `rename-file`, `delete-file` — file management
- `read-console-logs` — debugging via console output
- `read-network-requests` — debugging via network tab
- Built-in batching: "ALWAYS batch multiple operations when possible"
- Cardinal rule: "Never read files already in context" — aggressive token optimization

**Context Window Management:**
- Files already read are marked to prevent re-reading (saves tokens)
- Batched file operations to minimize token usage
- Search-replace preferred over full file rewrites (smaller diffs = fewer tokens)
- Automatic context cleanup and relevance filtering
- "useful-context" section checked FIRST before any file reads

**Key difference from Bolt:** Lovable uses text-based search-replace operations for code modification — NOT AST manipulation. This is simpler and more robust but less precise than AST transforms. TypeScript compilation is handled by Vite in the sandbox.

**Source:** [System prompts repository](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools/blob/main/Lovable/Agent%20Prompt.txt)

### Credit Economy
- Each AI interaction costs fractional credits (0.5 to 2.0+ depending on complexity)
- Security scans are **free** (don't consume credits)
- Agent Mode consumes significantly more credits per interaction (multi-step)
- Top-up credits available: $15/50 credits (Pro), $30/50 credits (Business)

---

## Infrastructure Deep Dive

### Discovered Infrastructure Stack (from Lovable's GitHub Organization)

Lovable's GitHub org (`lovablelabs`) reveals their actual infrastructure, much of which is custom-built or acquired:

**1. Database: Neon Postgres (Self-Hosted)**
- Custom Kubernetes operator: [lovablelabs/neon-operator](https://github.com/lovablelabs/neon-operator)
- Manages Neon Postgres clusters on their own infrastructure
- Supports both cloud and on-premises deployment
- Acquired via **Molnett acquisition** (infrastructure company)

**2. Container Orchestration: Kubernetes**
- Evidence: Neon operator, Molnett tooling, all infrastructure repos are K8s-native
- Custom operators for database, KMS, and service management

**3. Key Management: Custom KMS (Valv)**
- [lovablelabs/molnett-valv](https://github.com/lovablelabs/molnett-valv)
- Open-source KMS inspired by Google Cloud KMS
- Built for modern cloud environments
- Manages encryption keys for user data, API secrets, etc.

**4. Load Balancing: Maglev Consistent Hashing**
- [lovablelabs/maglev-hash](https://github.com/lovablelabs/maglev-hash)
- "Maglev consistent hashing with top-K preference lists for replica-aware routing"
- Likely powers the LLM provider routing system (sticky routing for prompt caching)
- Based on Google's Maglev paper — serious infrastructure engineering

**5. Observability: Wide Events**
- [lovablelabs/wide-event](https://github.com/lovablelabs/wide-event)
- "Honeycomb-style wide events — accumulate structured fields throughout request lifecycle"
- Rust-based tracing and monitoring
- Production-grade observability for debugging at 1B+ tokens/minute scale

**6. Schema Migrations: pg-schema-diff**
- [lovablelabs/pg-schema-diff](https://github.com/lovablelabs/pg-schema-diff)
- Postgres schema diffing for automated migration generation
- This is what powers the "describe your data → AI generates schema → auto-apply migrations" pipeline

**7. Backend: Migrated from Python to Go (Feb 2025)**
- Significant engineering investment for performance at scale
- Go handles the LLM routing, request orchestration, and API layer

### What This Reveals
Lovable is NOT just a Claude wrapper with a UI. They're building vertically:
- **Own database infrastructure** (Neon operator, not just using Supabase's hosted offering)
- **Own key management** (not relying on cloud provider KMS)
- **Own load balancing** (Maglev — same approach Google uses)
- **Own observability** (Rust-based, built for their specific scale)
- **Own migration tooling** (schema diffing for AI-generated databases)

This is a $6.6B company building like a $6.6B company. The Molnett acquisition gave them a team that knows how to build cloud infrastructure from scratch.

**Source:** [Lovable GitHub Organization](https://github.com/lovablelabs)

---

## Preview Compute Environment

### Architecture: Custom Iframe Sandbox (NOT WebContainers)

Lovable uses a **frontend-only iframe sandbox** — this is a deliberate architectural choice, not a limitation they haven't gotten around to fixing.

**What it is:**
- Custom iframe-based live preview on the right side of the interface
- Runs React + Vite + Tailwind CSS + TypeScript in an isolated iframe
- Real-time code updates reflected immediately in the preview
- Connected to real Supabase instances — forms save real data, auth works, API calls fire

**What it is NOT:**
- Not WebContainers (Bolt's approach — browser-native Node.js runtime)
- Not Sandpack (CodeSandbox's bundler-based approach)
- Not a server-side VM (Replit's approach)

**Hard limitations of this design:**
- **Cannot run backend code directly** — no Node.js, Python, Ruby, etc. in the sandbox
- **Cannot run build processes, testing suites, or dev servers** within the preview
- **No terminal** — unlike Bolt, there's no in-browser terminal for npm commands
- All backend functionality routed through Supabase (Edge Functions, database, auth, storage)

**Evidence from system prompts** (19.8KB, analyzed via leaked prompt repository):
- System prompts explicitly state the sandbox "cannot run backend code directly"
- "Cannot run Python, Node.js, Ruby, etc."
- Framework locked to "React, Vite, Tailwind CSS, TypeScript" only
- "Not possible to support Angular, Vue, Svelte, Next.js, native mobile apps"

**Source:** [System prompt analysis](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools/blob/main/Lovable/Agent%20Prompt.txt)

### How Open Source Clones Approach This Problem
Analyzing 4 different Lovable clones reveals the sandbox is the hardest architectural decision:

| Clone | Sandbox Approach | Tradeoff |
|-------|-----------------|----------|
| **poor-mans-lovable** | Docker containers per app (ports 3100, 3101...) | Full isolation, slower, compute cost per user |
| **Adorable (freestyle-sh)** | Freestyle cloud VMs with git-backed persistence | Full dev environment, expensive |
| **beam-cloud clone** | Beam compute sandbox + WebSocket streaming | Specialized for AI workloads |
| **open-lovable (firecrawl)** | Vercel Sandbox or E2B sandbox (configurable) | Managed services, multi-provider |

Every clone chose a different approach — this confirms there's no obvious "right answer." Lovable chose the simplest option (iframe) and compensated for backend limitations by deep Supabase integration.

### Key Difference from Bolt
| | Bolt | Lovable |
|---|------|---------|
| Sandbox type | WebContainers (browser-native OS) | Iframe (frontend-only) |
| Backend code | ❌ (Node.js only, no servers) | ❌ (Supabase Edge Functions only) |
| Terminal | ✅ Real in-browser terminal | ❌ No terminal |
| Real data in preview | ⚠️ Limited (no persistent DB connections) | ✅ Connected to real Supabase |
| Boot time | <100ms | Depends on Vite build |
| Compute cost | $0 (runs on user's CPU) | Server-side rendering cost |
| Framework support | React, Next.js, Vue, Svelte, Angular | React only |

**Source:** [poor-mans-lovable](https://github.com/restyler/poor-mans-lovable), [Adorable](https://github.com/freestyle-sh/Adorable), [beam-cloud clone](https://github.com/beam-cloud/lovable-clone), [open-lovable](https://github.com/firecrawl/open-lovable)

---

## The Diffing Engine

### How Changes Are Applied (from system prompt analysis)

**Primary tool: `search-replace`** — surgical text-based edits
- Exact text matching required: `oldText must match exactly (including whitespace)`
- This is NOT AST-level manipulation — it's literal string matching and replacement
- Preferred over full file rewrites because it generates smaller diffs → fewer tokens → cheaper

**Fallback tools:**
- `write-file` — for creating new files (full content generation)
- `rename-file` — file renaming operations
- `delete-file` — file removal

**Batching requirement:** System prompt enforces "ALWAYS batch multiple operations when possible" — multiple file edits bundled into a single tool call to minimize token overhead.

**Atomic operations** with rollback capability — each change set can be reverted independently.

**Visual editor path:** When users edit via the visual editor (clicking elements, changing properties), changes generate targeted Tailwind class modifications — more surgical than AI-generated rewrites.

**Source:** [System prompt analysis](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools/blob/main/Lovable/Agent%20Prompt.txt)

### Database Schema Diffing
For database changes, Lovable uses their custom **pg-schema-diff** tool:
1. User describes data requirements in natural language
2. AI generates target Supabase schema definitions
3. pg-schema-diff compares current schema → target schema → generates migration SQL
4. Migration auto-applied to project's Supabase database
5. TypeScript types auto-generated from new schema
6. API routes automatically created for CRUD operations

**Source:** [lovablelabs/pg-schema-diff](https://github.com/lovablelabs/pg-schema-diff)

### Version Control
- **Built-in version history** — every state saved, one-click restore any version
- **GitHub integration** — every AI change commits with meaningful commit messages
- Branch management supported via GitHub
- **Deployment rollback** — revert production to previous version instantly
- **Best-in-class** among AI builders: built-in history + GitHub sync + deployment rollback covers every recovery scenario

---

# PHASE 3: Maintainability & Guardrail Teardown

## Preventing Spaghetti Code

### Enforced Stack (Verified Versions — March 2026)
- **React 18 + TypeScript** — all generated code is TypeScript by default
- **Vite** — build tool with HMR, ES modules
- **Tailwind CSS v4** — latest version, with PostCSS configuration
- **shadcn/ui** — default component library (Radix UI primitives, class-variance-authority, clsx, tailwind-merge)
- **React Router** — routing (standard implementation)
- **Vitest** — testing framework, included in template since Jan 2026
- **Supabase SDK** — enforced data layer
- Stack is consistent but NOT user-configurable (no Vue, Angular, etc.)

### Verified File Structure Pattern
```
project-name/
├── src/
│   ├── components/     # React components (.tsx)
│   ├── hooks/          # Custom React hooks
│   ├── pages/          # Route components
│   ├── lib/            # Utility libraries
│   ├── types/          # TypeScript type definitions
│   └── index.css       # Tailwind imports
├── public/             # Static assets
├── package.json
├── vite.config.ts
├── postcss.config.js
├── tailwind.config.ts
└── .lovable/
    └── plan.md         # Saved plan from Plan Mode
```

### State Management
- React hooks (useState, useEffect) for local state
- Supabase client for server state
- Custom hooks for data fetching patterns
- Real-time subscriptions for live data via Supabase

### Code Quality Tools
- **Security scanners** — 4 automated scanners check code quality
- **Code Security Review** — analyzes application code for vulnerability patterns
- **Dependency Audit** — checks for vulnerable npm packages
- **TypeScript strict mode** — generated code uses TypeScript
- **No automated linting** in the builder itself (but TypeScript catches many issues)

### Agent Mode Quality
- Agent Mode generates more structured, modular code than chat mode
- Handles separation of concerns (frontend components, API calls, database operations)
- Generates TypeScript types from database schema

---

## The Self-Healing Loop

### Testing Features
- **Built-in testing capabilities** — Lovable can generate and run tests
- **Security scans before publish** — automatic vulnerability check
- **RLS policy validation** — checks database access rules
- Agent Mode includes error detection and retry logic

### Pre-Publish Guardrails
- Security scanners run automatically before publishing
- Critical issues prompt user review
- Can block publishing until issues resolved
- This is the closest thing to a "safety net" in any AI builder

---

## Version Control

### Capabilities
- **Full version history** — every state saved
- **One-click restore** — restore any version
- **GitHub sync** — proper git integration
- **Meaningful commits** — AI creates descriptive commit messages
- **Branch support** — via GitHub integration
- **Deployment rollback** — revert production to previous version

### Assessment
**Best-in-class version control among AI builders.** The combination of built-in version history + GitHub sync + deployment rollback provides recovery options at every level. Significantly better than Base44 (prompt-level undo only) and Bolt (per-message undo + GitHub).

---

# PHASE 4: GTM & Telco Partner Strategy

## Pricing Model — Usage-Based Credits

Unlike competitors with fixed tiers, Lovable uses a **sliding credit scale**. Users buy a credit allocation, not a plan name.

### Free Plan
- 5 daily credits, max 30/month
- Workspace collaboration (unlimited members)
- Private projects

### Pro Plans (Verified March 2026)

| Credits/Month | Monthly | Annual (per month) |
|---------------|---------|-------------------|
| 100 | $25 | $21 |
| 200 | $50 | $42 |
| 400 | $100 | $84 |
| 800 | $200 | $167 |
| 1,200 | $294 | $245 |
| 2,000 | $480 | $400 |
| 3,000 | $705 | $588 |
| 5,000 | $1,125 | $938 |
| 10,000 | $2,250 | $1,875 |

### Business Plans
Double the Pro price for the same credit count. Adds enterprise features (SSO, SCIM provisioning, audit logs, dedicated support).

### Credit Economics
- "Make the button gray" = 0.50 credits
- "Remove the footer" = 0.90 credits
- "Add authentication" = 1.20 credits
- "Build a landing page with images" = 2.00 credits
- **Top-ups**: $15 per 50 credits (Pro), $30 per 50 credits (Business)

### Hidden Limits & Observations
1. **Credit consumption is unpredictable** — complex Agent Mode interactions use significantly more credits than simple edits
2. **Business tier is 2x Pro pricing** for the same credits — pure margin on enterprise features
3. **Supabase limits** — inherited from connected Supabase project tier
4. **Security scans are free** — don't consume credits (smart retention play)
5. **At scale, this gets expensive fast** — a team burning 3,000 credits/month pays $705/mo (Pro) or $1,410/mo (Business)

---

## B2B2C Channel Readiness

### Enterprise Features
- **Workspace management** — team collaboration
- **Security Center** — cross-project vulnerability monitoring
- **Role-based access** — workspace owner, admin, member, viewer
- **SSO** — enterprise tier (SAML/OIDC implied)
- **Audit capabilities** — security scan history

### White-Label Assessment
- **NOT white-label ready**
- Lovable branding throughout builder UI
- No embeddable builder widget
- No custom branding options
- No partner program documented

### Channel Partner Program
- **No documented partner/reseller program**
- Enterprise tier is direct-to-customer
- No multi-tenant billing for sub-accounts
- No API for programmatic project management

---

## Positioning & Persona

### Hero Copy
- **"The Full-Stack AI Engineer"** / **"Build software with AI"**
- Emphasis on "full-stack" — not just frontend
- Positioning against hiring developers

### Target Persona
**Primary: Technical-adjacent founders and product managers** who need full-stack apps but don't want to manage infrastructure. More sophisticated than Base44's target — they understand databases and auth.

**Secondary: Developers for rapid prototyping** — GitHub export makes ejection clean.

---

# PHASE 5: Enterprise Compliance & Accessibility

## Security Features (UNIQUE DIFFERENTIATOR)

Lovable has the most comprehensive security tooling of any AI builder:

### 4 Automated Security Scanners
1. **RLS Analysis** — reviews database access policies and row-level security rules
2. **Database Security Check** — reviews schema + RLS configuration together
3. **Code Security Review** — analyzes code for common vulnerability patterns
4. **Dependency Audit** — checks npm packages for known CVEs

### Pre-Publish Security Gate
- Scanners run automatically before publishing
- Critical issues generate warnings
- Users prompted to review before proceeding
- Can proceed despite warnings (not a hard block)

### Workspace Security Center
- Admins monitor security across ALL projects
- Cross-project vulnerability dashboard
- Identify which projects are affected by new CVEs
- Centralized security management

### API Key Protection
- Automatic detection of API keys pasted in chat
- Guides users to use Secrets storage instead of hardcoding
- Generates Edge Functions for server-side API calls

---

## WCAG Compliance
- **Generated code uses shadcn/ui** (Radix) which has strong accessibility foundations
- **However:** No automated WCAG checking on generated output
- No VPAT published
- Builder UI accessibility not documented

## Certifications

| Certification | Status |
|--------------|--------|
| SOC 2 Type II | ❓ Not publicly documented |
| ISO 27001 | ❌ Not mentioned |
| GDPR | ⚠️ Privacy policy exists |
| HIPAA | ❌ Not applicable |
| VPAT | ❌ Not published |

---

# PHASE 6: Churn & Scalability Ceiling

## Code Ejection

### Export Options
- **GitHub sync** — clean, well-structured repo with proper git history
- Code is standard React + TypeScript + Vite + Supabase — highly portable
- Generated code follows community conventions
- Ejection is the cleanest among all AI builders

### Where Code Quality Excels
- TypeScript types generated from database schema
- RLS policies properly structured
- Component separation is reasonable
- Supabase SDK usage follows best practices

### Where Code Quality Struggles
- Complex business logic still requires iteration
- Styling can become inconsistent over many iterations
- State management in large apps gets messy

---

## The Logic Wall

| Level | Capability |
|-------|-----------|
| ✅ Works great | Full CRUD apps, auth flows, dashboards, landing pages |
| ✅ Works great | Database with RLS, protected routes, user management |
| ⚠️ Works with effort | Complex forms, multi-step workflows, real-time features |
| ❌ Struggles | Complex business logic, custom auth flows, enterprise integrations |
| ❌ Fails | Non-JavaScript backends, native mobile, complex DevOps |

---

## Community Sentiment & Real-World Reception

### Enterprise Case Studies (Verified)
- **Zendesk:** Prototype time reduced from 6 weeks → 3 hours
- **Global ridesharing platform:** Design concept testing from 6 weeks → 5 days
- **One PM:** 30-minute prototype vs. 3-month traditional timeline
- **Healthcare:** Patient journey visualization app in production
- **Enterprise HCM:** Rebuilt onboarding workflow tools in days vs. months

### Community Discussion Themes (Reddit r/webdev, r/nocode)
- "Do People Really Just Create An Entire App just Vibe Coding?" — mixed reception from traditional devs
- "Are AI tools like Lovable and Bolt a trap for Non-Coders?" — ongoing concern about lock-in
- "Building a client website with Lovable" — freelancers testing it for client work
- Active comparison threads: Lovable vs Bolt vs v0 — Lovable generally seen as more full-stack capable

### What Users Praise
- Full-stack capability (database + auth + frontend in one prompt)
- Agent Mode handling the boring setup work
- Clean code ejection to GitHub
- Security scanning (unique among competitors)

### What Users Criticize
- Credit consumption is unpredictable — hard to budget
- "Vibe coding" produces code that's hard to maintain at scale
- Platform lock-in concerns, especially for Lovable Cloud projects
- Complex business logic still requires developer intervention

---

### Competitive Position
Lovable's Logic Wall is HIGHER than Base44's because:
1. Real Supabase integration (SQL, RLS, Edge Functions) vs proprietary data layer
2. Agent Mode handles multi-step setup autonomously
3. GitHub export lets developers continue in VS Code when they hit limits
4. TypeScript provides compile-time error catching

---

# EXHAUSTIVE FEATURE INDEX

## AI & Generation

| Feature | Description | Details |
|---------|-------------|---------|
| **Chat Mode** | Iterative AI development via conversation | Default mode. Send prompts, AI generates code changes. See diffs per message. Supports image upload, file references, knowledge context. |
| **Agent Mode** | Autonomous full-stack AI agent | Handles multi-step tasks: database setup, API connections, auth flows, deployment. Plans internally, executes sequence of actions. Higher credit cost but more complete output. |
| **Plan Mode** | AI plans before executing | Generates structured approach. User reviews/modifies before execution. Reduces wasted iterations on wrong approaches. |
| **Code Mode** | Direct code editing | Switch to code editor view. Edit files directly. Changes reflected in preview. |
| **Image Upload** | Screenshot/mockup to code conversion | Drag or paste images into chat. AI analyzes and generates matching UI. Supports Figma screenshots, hand-drawn wireframes, existing app screenshots. |
| **File References** | @mention project files in prompts | Reference specific files to include as AI context. Reduces hallucination by focusing on relevant code. |
| **Knowledge Base** | Persistent project documentation | Attach external docs, specs, guidelines. Persist across all prompts. Provides domain context. |
| **Suggested Prompts** | Contextual next-step suggestions | AI suggests logical next actions based on current project state. |

## Visual Editor

| Feature | Description | Details |
|---------|-------------|---------|
| **Select & Edit** | Click-to-edit UI elements | Click any element in preview to select. Properties panel appears. Direct manipulation without prompting. |
| **Properties Panel** | Visual property editing | Edit text content, colors, fonts, spacing, layout, borders, shadows. Changes generate Tailwind classes. |
| **Component Tree** | Hierarchical component view | Browse all components in tree structure. Select nested components. |
| **Responsive Editing** | Breakpoint-specific visual edits | Edit different viewport sizes independently. Changes scoped to breakpoint. |
| **Style System** | Design token management | Colors, fonts, spacing defined as reusable tokens. Visual editor respects token system. |
| **Real-Time Preview** | Instant visual feedback | Changes appear immediately in preview. No build step. |

## Code Editor

| Feature | Description | Details |
|---------|-------------|---------|
| **File Explorer** | Project file tree | Browse all files. Create, rename, delete. |
| **Syntax Highlighting** | Multi-language support | TypeScript, JavaScript, HTML, CSS, JSON, SQL, Markdown. |
| **Inline Editing** | Direct code modification | Edit any file. Changes apply to preview. |
| **Diff View** | Per-interaction change visualization | See exactly what AI changed. Expandable diffs in chat history. |

## Database (Supabase Integration)

| Feature | Description | Details |
|---------|-------------|---------|
| **Auto-Provisioning** | Agent Mode creates databases automatically | Describe data needs in natural language → Agent creates tables, types, RLS policies. |
| **Table Management** | Create, modify, delete tables | Via AI prompts or Supabase Studio. Full SQL access. |
| **Row-Level Security** | Per-user data access control | AI generates RLS policies. Security scanner validates them. Postgres RLS enforcement. |
| **Edge Functions** | Server-side logic | AI generates Deno-based Edge Functions for API routes, webhooks, server-side processing. Deploys to Supabase. |
| **TypeScript Types** | Auto-generated database types | Types generated from database schema. Type-safe frontend-to-backend. |
| **Realtime** | Live data subscriptions | Supabase Realtime integration for live updates. AI can generate realtime-enabled components. |
| **Storage** | File upload and hosting | Supabase Storage for images, documents. AI generates upload components. |
| **Migrations** | Schema version control | AI generates SQL migration files. Applied automatically. |

## Security

| Feature | Description | Details |
|---------|-------------|---------|
| **RLS Analysis** | Database access policy scanner | Automated review of row-level security rules. Identifies overly permissive policies, missing access checks. Runs on file changes and before publish. Free (no credits). |
| **Database Security Check** | Schema + RLS combined review | Reviews schema design and access rules together. Catches issues not visible when reviewing rules in isolation. Runs per session after RLS analysis. Free. |
| **Code Security Review** | Application code vulnerability scanner | Analyzes code for common vulnerability patterns (XSS, injection, insecure auth flows, etc.). Manual trigger via "Update" button. Free. |
| **Dependency Audit** | npm package vulnerability scanner | Checks project dependencies for known CVEs. Runs when dependencies change. Shows affected package, vulnerability type, remediation steps. Free. |
| **Security View** | Project-level security dashboard | View all scan results for a single project. Drill into findings. Take action on vulnerabilities. |
| **Security Center** | Workspace-level vulnerability dashboard | Admins monitor security across ALL projects. Identify projects affected by new CVEs. Centralized security management. |
| **Pre-Publish Gate** | Security scan before deployment | Automatic scans run when publish dialog opens. Critical issues prompt review. Not a hard block but strongly discouraged to skip. |
| **API Key Detection** | Automatic credential protection | Detects API keys pasted in chat. Guides to Secrets storage. Prevents hardcoding credentials in frontend code. |
| **Secrets Management** | Secure API key storage | Store credentials securely. Accessible via Edge Functions server-side. Not exposed in client code. |

## Version Control & History

| Feature | Description | Details |
|---------|-------------|---------|
| **Version History** | Complete state timeline | Every project state saved. Browse all past versions. |
| **One-Click Restore** | Instant rollback to any version | Select any past version → restore. No data loss. |
| **GitHub Sync** | Automatic git integration | Every AI change commits to GitHub. Proper git history. Meaningful commit messages. |
| **Branch Support** | Git branching via GitHub | Create and switch branches via GitHub integration. |
| **Deployment Rollback** | Revert production deployment | Roll back published app to previous version. Instant. |

## Hosting & Deployment (Lovable Cloud)

| Feature | Description | Details |
|---------|-------------|---------|
| **One-Click Publish** | Deploy to production | Click Publish → live URL. Auto-build and deploy. |
| **Preview Deployments** | Shareable pre-production URLs | Preview app before publishing. Share URL with stakeholders for review. |
| **Custom Domains** | Connect your own domain | Enter domain → configure DNS → auto-SSL. CNAME or A record. |
| **Auto SSL** | HTTPS on all deployments | TLS certificates auto-provisioned and renewed. |
| **Environment Variables** | Secrets for production | Store API keys and config for production environment. |
| **Rollback** | Instant production rollback | Revert to previous deployment with one click. |

## Integrations

| Feature | Description | Details |
|---------|-------------|---------|
| **Supabase** | Database, auth, storage, functions | Deep native integration. AI generates Supabase-native code. Auto-provisioning via Agent Mode. |
| **GitHub** | Version control | Automatic sync. Every change committed. Clean repo structure. |
| **Stripe** | Payment processing | AI generates Stripe integration code. Checkout, subscriptions, webhooks via Edge Functions. |
| **Resend** | Email sending | AI generates email sending via Resend API + Edge Functions. |
| **Lovable Cloud** | Managed hosting | Built-in hosting with custom domains, SSL, environment variables. |

## Team & Collaboration

| Feature | Description | Details |
|---------|-------------|---------|
| **Workspaces** | Team environments | Shared workspace for team projects. |
| **Roles** | Role-based access control | Owner, Admin, Member, Viewer. Different permissions per role. |
| **Security Center** | Admin security dashboard | Workspace-level vulnerability monitoring across all projects. |
| **Shared Projects** | Collaborative development | Multiple team members can work on same project (not simultaneous editing). |

## Supported Technologies

| Technology | Status | Notes |
|------------|--------|-------|
| **React + Vite** | ✅ Primary | Default and best-supported stack. |
| **TypeScript** | ✅ Enforced | All generated code is TypeScript. |
| **Tailwind CSS** | ✅ Enforced | All styling via Tailwind utilities. |
| **shadcn/ui (Radix)** | ✅ Default | Accessible component library. |
| **Supabase** | ✅ Deep integration | Database, auth, storage, functions, realtime. |
| **Next.js** | ❌ Not supported | React + Vite only. |
| **Vue/Svelte/Angular** | ❌ Not supported | React only. |
| **Python/Go** | ❌ Not supported | JavaScript/TypeScript only. |

---

---

# UPDATE — March 9, 2026: LLM Load Balancer Deep Dive

**Source:** [Routing Billions of Tokens per Minute](https://lovable.dev/blog/routing-billions-of-tokens-per-minute) (Published Mar 4, 2026)

## Key Revelations

### Scale
- Processing **1B+ tokens/minute** at peak traffic
- Multi-provider setup: Anthropic (primary), Vertex (Google Cloud), Bedrock (AWS)

### Architecture: Custom LLM Load Balancer
- **Multiple fallback chains** with probabilistic provider ordering (not a single ranked list)
- **Project-level affinity** — each project gets a cached fallback chain for several minutes, preserving prompt caching across agent turns
- **PID controller** per provider recalculates availability every 30 seconds
- Scoring formula: `score = successes - 200×errors + 1` (0.5% error threshold triggers weight reduction)
- +1 bias prevents permanently blacklisting providers after transient failures
- Provider weights calculated greedily: preferred provider gets its full availability, next provider fills remaining capacity, etc.

### Why Prompt Caching Matters
- Lovable's agent does exploration before code generation, so initial context is reused across subsequent calls
- Breaking cache (by switching providers mid-project) forces full context reprocessing → higher latency, higher cost
- Provider rate limits are measured in **non-cached tokens** — poor cache behavior can exhaust all provider capacity simultaneously
- Sticky routing keeps cache hit rates high → estimated 50-90% token cost reduction vs naive multi-provider

### Competitive Implications
- **Revised assessment:** Lovable is NOT just "Claude with a UI" — they have genuine infrastructure depth
- They're building vertically: own LLM routing + Lovable Cloud hosting + reducing Supabase dependency
- This matches the "own the metal" strategy from our blueprint, executed at billion-token scale
- Auto-healing without human intervention during provider outages = significant reliability advantage
- **Threat level: INCREASED** — competing requires solving the same prompt caching + multi-provider problem

### What They Revealed (potentially too much)
- Exact scoring formula and error threshold
- Preferred provider ordering strategy
- Probabilistic sampling approach for fallback chains
- This is a recruiting play — blog ends with careers link

---

**Sources:**
- Lovable documentation: https://docs.lovable.dev
- Lovable docs index (llms.txt): https://docs.lovable.dev/llms.txt
- Lovable pricing & credit system: https://docs.lovable.dev/introduction/plans-and-credits
- Lovable security docs: https://docs.lovable.dev/features/security
- Lovable agent mode docs: https://docs.lovable.dev/features/agent-mode
- Lovable cloud docs: https://docs.lovable.dev/integrations/cloud
- Lovable Series B announcement: https://lovable.dev/blog/series-b
- Lovable Series A announcement: https://lovable.dev/blog/200m-series-a-fundraise
- Lovable changelog: https://docs.lovable.dev/changelog
- LLM load balancer blog: https://lovable.dev/blog/routing-billions-of-tokens-per-minute
- Community discussions: Reddit r/webdev, r/nocode
