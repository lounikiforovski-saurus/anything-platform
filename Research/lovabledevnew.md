# Lovable.dev — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026

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

### Models
- **Primary:** Claude (Anthropic) — specific version not publicly documented
- Model selection NOT exposed to users
- Agent Mode uses a more capable model/configuration than standard chat

### Hydration Pattern
- **Plan Mode** — separate planning step before code generation
  - AI analyzes prompt + codebase
  - Generates structured plan
  - User reviews/approves
  - Then AI executes
- **Agent Mode** — autonomous multi-step execution
  - Plans internally, executes multiple actions in sequence
  - Handles database, API, frontend, deployment
  - More tokens per interaction but more complete output
- **File references** — @mentions specific files to include as context
- **Knowledge base** — persistent project documentation injected into prompts

### Token Economy
- **Message-based credits** — each AI interaction costs credits
- Plans/enterprise tiers have higher credit allocations
- Security scans are **free** (don't consume credits)
- Agent Mode likely consumes more credits per interaction (multi-step)

---

## Preview Compute Environment

### How Preview Works
- **Server-side rendering** — NOT WebContainers
- Live preview panel shows real-time app rendering
- Connected to Supabase for real data interactions
- Preview URLs shareable before publishing
- Hot reload on code changes

### Key Difference from Bolt
Lovable's preview connects to real databases and APIs — forms save real data, auth works, API calls fire. Bolt's WebContainers can't do this. The tradeoff: Lovable's preview is slower to start but more complete; Bolt's is faster but frontend-only.

---

## The Diffing Engine

### How Changes Are Applied
- AI generates changes shown as diffs in the chat
- Full-file generation for new files
- Diff-based edits for existing files
- Visual editor generates targeted CSS/Tailwind changes (more surgical)

### Version Control
- **Built-in version history** — view all past states
- **Restore any version** — click to restore
- **GitHub integration** — automatic sync to GitHub repo
  - Every AI change commits to GitHub
  - Proper git history with meaningful commit messages
  - Branch management supported
- **Rollback** — production deployment rollback available

---

# PHASE 3: Maintainability & Guardrail Teardown

## Preventing Spaghetti Code

### Enforced Stack
- **React + TypeScript + Vite** — primary stack
- **Tailwind CSS** — enforced styling framework
- **shadcn/ui** — default component library (Radix primitives)
- **Supabase SDK** — enforced data layer
- Stack is consistent but NOT user-configurable (no Vue, Angular, etc.)

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

## Pricing Model

### Tier Structure
| Tier | Price | Messages | Key Features |
|------|-------|----------|-------------|
| **Free** | $0 | 5/day | 1 project, basic features |
| **Starter** | $20/month | Increased | Multiple projects, custom domains |
| **Launch** | $50/month | Higher | Agent Mode, priority support, Lovable Cloud |
| **Scale** | $100/month | Highest | Team features, security center |
| **Enterprise** | Custom | Unlimited | SSO, dedicated support, SLA, audit logs |

### Hidden Limits
1. **Message credits** — complex Agent Mode interactions use more credits
2. **Supabase limits** — inherited from connected Supabase project tier
3. **Lovable Cloud limits** — not fully documented yet (new service)
4. **Security scans** — free, but Code Security Review is manual (not automatic)

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
- Lovable pricing: https://lovable.dev/pricing
- Lovable security docs: https://docs.lovable.dev/features/security
- Lovable agent mode docs: https://docs.lovable.dev/features/agent-mode
- Lovable cloud docs: https://docs.lovable.dev/integrations/cloud
