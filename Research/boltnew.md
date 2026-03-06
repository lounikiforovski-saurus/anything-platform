# Bolt.new — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026

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
- **Bolt Cloud hosting powered by Netlify** — confirmed in documentation
- Static sites deploy to Netlify CDN
- Server-side apps use Supabase Edge Functions
- Custom domains with auto-SSL via Netlify

### Key Insight for HostPapa
**Bolt depends on TWO managed services: Netlify (hosting) and Supabase (database/auth/storage).** They've branded it as "Bolt Cloud" but it's a white-label wrapper around Netlify + Supabase. This is a vulnerability — Bolt has no control over these services' pricing, availability, or feature roadmap. HostPapa can offer the same experience with true ownership by self-hosting Supabase and using k3s for hosting.

---

## LLM Orchestration

### Models
- **Claude Agent** (default) — powered by Claude Sonnet 4.6 (Anthropic)
  - Best for complex full-stack applications
  - Supports Bolt Database natively
  - Can handle multi-file changes
- **v1 Agent (legacy)** — multi-model, uses various LLMs
  - Only supports Supabase databases (not Bolt Database)
  - Being phased out

### Hydration Pattern
- **Plan Mode** — when enabled, the AI first creates a structured plan before generating code
  - Analyzes the prompt and existing codebase
  - Proposes an approach with specific steps
  - User can review/modify the plan before execution
  - This is Bolt's version of the hydration pattern — separate planning from generation
- **Design Systems** — persistent brand context injected into every prompt
  - Colors, fonts, spacing, component styles
  - Reduces hallucination by constraining the design space
  - Claims "98% fewer errors" with design systems enabled
- **Project Knowledge** — uploaded documents injected as context
  - PRDs, API specs, brand guidelines
  - Provides domain-specific context to the LLM

### Token Economy
- **Token-based pricing** — each AI interaction consumes tokens
- Different plans have different token allocations
- Tokens are NOT the same as LLM tokens — they're Bolt's abstracted credit unit
- Plan Mode likely consumes fewer tokens for the planning step (cheaper model inference)

---

## Preview Compute Environment — WebContainers

### How It Works
- **WebContainers** run a full Node.js runtime inside the browser using WebAssembly + Service Workers
- Developed by StackBlitz (Bolt's parent company) — they own and control this technology
- The entire development server (Vite, npm, Node.js) runs client-side
- No server compute cost for preview
- Startup time: <100ms (vs 500ms-5s for container-based approaches)

### Technical Details
- **Cross-Origin Isolation required** — pages must set COOP and COEP headers
- **Supported browsers:** Chrome, Edge, Brave, Firefox, Safari (with limitations)
- **File system:** Virtual file system in browser memory
- **Package manager:** Full npm support, packages installed in-browser
- **Terminal:** Real terminal emulator connected to WebContainers runtime
- **Networking:** Service Worker-based network proxy for HTTP requests

### Limitations
- **No native binaries** — can't run Python, Go, Rust, etc.
- **No real database connections during preview** — need Supabase/Bolt Database for persistent data
- **Memory limit** — ~500MB in browser (varies by device)
- **No WebSocket server** — WebContainers can't listen for incoming connections
- **Commercial license required** — WebContainers are NOT open source for production use

### Key Insight for HostPapa
WebContainers are Bolt's core competitive advantage and they OWN the technology (StackBlitz). To replicate this, HostPapa would need to license WebContainers from StackBlitz or build an alternative. Alternatives: Firecracker micro-VMs (server-side, 125ms startup) or browser-based sandboxes (limited). The hybrid approach recommended in our blueprint (WebContainers for frontend preview + Firecracker for full-stack) is the optimal strategy.

---

## The Diffing Engine

### How Bolt Applies Code Changes
- AI generates code diffs shown in the chat interface
- Changes are applied to the WebContainers virtual file system
- **Full-file generation** appears to be the primary approach for new files
- **Diff-based edits** for modifications to existing files (shown as expandable diffs in chat)
- Auto-testing and error correction loop — claims "98% less errors" through:
  1. Generate code
  2. Run in WebContainers
  3. Catch errors from terminal/console
  4. Auto-fix and retry

### Version Control
- **Undo/redo per AI message** — each chat interaction is a rollback point
- **GitHub integration** — push to GitHub repo for proper version control
- **Revert changes** — can revert specific AI actions
- No built-in branching (use GitHub for that)
- File-level change tracking in the chat interface

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

### Error Detection & Auto-Fix
- **YES — Bolt has a partial self-healing loop**
- WebContainers provide real-time terminal output
- If the AI-generated code has a compilation error:
  1. Error appears in the terminal/console
  2. The AI agent detects the error automatically
  3. AI generates a fix and applies it
  4. Cycle repeats until the error is resolved or max retries hit
- Claims "98% fewer errors" — likely means 98% of first-attempt errors are auto-fixed before the user notices
- This is significantly better than Base44 (which shows raw errors to users)

### Limitations
- Auto-fix works for compilation/runtime errors (syntax, import, type errors)
- Does NOT detect logical errors (wrong behavior, incorrect data flow)
- Does NOT detect accessibility issues
- Does NOT detect performance problems

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

## Pricing Model

### Tier Structure
| Tier | Price | Tokens | Key Features |
|------|-------|--------|-------------|
| **Free** | $0 | Limited | 1 project, basic features |
| **Pro** | $20/month | Increased | Multiple projects, custom domains, Bolt Cloud |
| **Teams** | $30/user/month | Pooled team tokens | Shared workspace, team templates, design systems |
| **Enterprise** | Custom | Custom | SSO, admin controls, priority support |

### Token Economy
- Tokens consumed per AI interaction (not per-message — complex prompts use more)
- Plan Mode may use fewer tokens for the planning step
- No visible credit cost per individual message
- Teams share a pooled token allocation

### Hidden Limits
1. **WebContainers memory** — browser-imposed limit (~500MB), not disclosed
2. **Bolt Database rows** — "unlimited" but likely has soft limits at scale
3. **Edge Functions** — execution limits inherited from Supabase
4. **Hosting bandwidth** — inherited from Netlify, limits not disclosed
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
