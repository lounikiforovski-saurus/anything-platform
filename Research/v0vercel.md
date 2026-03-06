# V0 by Vercel — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026

---

# PHASE 1: Product & UX Friction Audit

## Exhaustive UI Matrix

### AI Chat Interface
- **Prompt input** — text area at the bottom of the chat pane. Supports natural language in any language — the AI translates non-English prompts to code. Accepts text, pasted screenshots, uploaded images, and Figma file exports as input.
- **Model selection** — three tiers exposed directly to the user (unique among competitors — no other AI builder lets users pick their model):
  - **v0 Mini** — near-frontier intelligence, optimized for speed. $1/$5 per 1M input/output tokens. Best for quick iterations, simple components, and rapid prototyping where speed matters more than depth.
  - **v0 Pro** — balanced speed and intelligence. $3/$15 per 1M input/output tokens. Default for most work. Best cost-to-quality ratio.
  - **v0 Max** — maximum intelligence for complex work. $5/$25 per 1M input/output tokens. Best for architectural decisions, complex logic, and demanding multi-file changes.
- **Agent capabilities** — v0's AI operates as an autonomous agent, not just a code generator. It can: search the web for current documentation, inspect live sites to understand their structure, automatically diagnose and fix deployment errors, and integrate external tools. Each action shows real-time visual progress indicators so the user knows what the agent is doing.
- **Instructions** — persistent custom rules applied across all chats in a project (replaces the older "Rules" system). Created via the + icon in the prompt bar. Example: "Always use server components when possible" or "Follow our design system tokens from tailwind.config.js." These inject into every prompt automatically, enforcing consistency without manual repetition.
- **File references** — reference specific project files in prompts using @mentions. Focuses the AI on relevant code and reduces hallucination.
- **Image-to-code** — drag/paste screenshots, wireframes, Figma exports, or hand-drawn sketches. AI analyzes layout, colors, spacing, and typography. Each image costs ~1,000 tokens of context. Works with rough mockups through pixel-perfect designs.
- **"Fix with v0"** — when a deployment or preview has errors, a "Fix with v0" button appears. The AI diagnoses the issue (reading error logs, stack traces, and the relevant code) and applies a fix. 20 free uses per day on unedited code. Additional fixes consume credits. This is a closed-loop debugging mechanism — errors surface → AI fixes → redeploy, all without leaving v0.
- **Rich UI feedback** — during agent actions, the preview shows real-time indicators of what's happening: "Searching web...", "Inspecting site...", "Generating code...", "Running build...". No other AI builder provides this level of transparency into agent behavior.

### Full Editor (February 2026 Major Update)
- **VS Code-style editor** — complete code editor built into v0, launched February 2026. Prior to this, v0 was primarily a chat-based tool. The editor adds: file explorer, syntax highlighting (TypeScript, JavaScript, HTML, CSS, JSON, SQL, Markdown), inline editing, and diff view. Editor + AI agent + preview + configuration all in a single workspace.
- **File explorer** — full project file tree. Navigate, create, rename, delete files and folders.
- **Diff view** — per-interaction change visualization. See exactly what the AI changed in each message. Expandable diffs in chat history. This is better than Replit (which doesn't show per-interaction diffs) and comparable to Lovable.
- **Significance** — the February 2026 update transformed v0 from a "chat-based component generator" into a "full development environment." It now competes directly with Replit's IDE rather than just with Bolt/Lovable's chat interfaces.

### Design Mode
- **Activation** — Option+D keyboard shortcut or Design tab. Works on any element in the preview.
- **Element selection** — hover to highlight, click to select. Selected element shows a property panel.
- **Design Panel properties:**
  - **Typography** — font family, size, weight, line height, letter spacing, text alignment, text decoration
  - **Color** — text color and background color via color picker
  - **Layout** — margin and padding controls for all four sides (top, right, bottom, left)
  - **Border** — border color, style, and width
  - **Appearance** — opacity slider, corner radius adjustment
  - **Shadow** — box shadow configuration, multiple shadow support
  - **Content** — direct text editing of any element
- **Tailwind Native** — Design Panel pulls colors, spacing values, and fonts from the project's tailwind.config.js file. Ensures visual tweaks use the project's design tokens, not arbitrary values. This maintains brand consistency across AI-generated and manually-tweaked elements.
- **Free visual tweaks** — ALL Design Panel adjustments (typography, color, spacing, border, shadow, content) cost ZERO tokens. Only prompt-based changes in Design Mode consume credits. This means unlimited visual fine-tuning at no cost. No other AI builder offers this. You could spend hours perfecting spacing and colors without touching your credit balance.
- **Natural language on selected elements** — for complex modifications that can't be done via the panel (e.g., "Make this a dropdown instead of radio buttons"), you can type a prompt while an element is selected. The AI makes the change scoped to that element. This costs tokens.
- **Save button** — click save at the bottom of the preview to commit Design Mode edits. Unsaved changes are indicated visually.

### Git Integration (February 2026 Update)
- **Automatic branching** — v0 creates a dedicated branch per chat (e.g., `v0/main-abc123`). No manual branch management. Each chat's changes are isolated on their own branch.
- **Auto-commits** — every message that changes code automatically creates a git commit with a meaningful commit message. Full change history preserved in git, not just in v0's UI.
- **Protected main** — v0 NEVER pushes directly to main. All changes go through pull requests. This is an enterprise-grade safety mechanism that no other AI builder enforces by default.
- **Pull requests** — create PRs from v0. View PR status (open, merged, closed). Merge PRs without leaving v0. Full GitHub PR workflow including review comments.
- **Branch from branch** — duplicate a branch to work on multiple approaches in parallel. Choose to keep project configuration or start fresh. Useful for A/B testing different implementations.
- **GitHub import** — import ANY GitHub repository into v0. The AI creates a working branch automatically and starts with full project context. This means v0 works on EXISTING codebases, not just greenfield — a capability shared only with Replit and OpenHands among the tools analyzed.
- **Multi-tool collaboration** — team members using Cursor, Claude Code, Windsurf, or other IDE tools can work on the same GitHub repo. Push/pull through GitHub. v0 is one tool among many in the workflow, not a walled garden.

### Projects & Organization
- **Projects** — connected to actual Vercel deployments. Each Project maps to a Vercel production URL. Includes domains, environment variables, deployment settings. Multiple chats can contribute to the same Project — useful for parallel feature development.
- **Folders** — organizational grouping for chats only. No effect on deployments. Purely for human organization.
- **Templates** — community-built starting points. Browse what others have built. Start from proven patterns rather than blank prompts.

---

## JTBD Friction Map

### Flow 1: Building a New App from Scratch
1. Go to v0.app (1 click)
2. Start a new chat (1 click)
3. Describe what you want: "Build a SaaS dashboard with user authentication, Stripe billing, team management, and usage analytics using Next.js and shadcn/ui" (1 action)
4. Optionally select model (Mini/Pro/Max) (0-1 clicks)
5. AI generates working app with live preview
6. Iterate via chat ("Add a dark mode toggle") or Design Mode (click element → adjust in panel)
7. Click Publish (1 click) → deployed to Vercel

**Total clicks: ~4-5**
**Cognitive leaps: 0-1** (model selection, optional)
**Friction rating: LOW**

### Flow 2: Working on an Existing GitHub Codebase
1. Click + → Import from GitHub (2 clicks)
2. Select repository (1 click)
3. v0 imports the project, creates working branch automatically (0 clicks — automatic)
4. Full project context loaded. Chat with AI to make changes.
5. Review changes in diff view
6. Open PR → review → merge (2-3 clicks)

**Total clicks: ~5-7**
**Cognitive leaps: 0** — git workflow is entirely automated
**Friction rating: LOW** — the February 2026 update made this genuinely seamless. This is the best "existing codebase" workflow of any AI builder.

### Flow 3: Fine-Tuning Design Without Credits
1. Activate Design Mode (Option+D or Design tab) (1 click/shortcut)
2. Click any element in preview (1 click)
3. Adjust typography, colors, spacing, borders, shadows in Design Panel (N clicks, free)
4. For complex changes, type a prompt on the selected element (costs credits)
5. Click Save (1 click)

**Total clicks: variable**
**Credit cost: ZERO for panel adjustments**
**Friction rating: VERY LOW** — the free visual editing is a genuinely differentiated UX

---

## The "Aha!" Moment

**The "Aha!" is Design Mode's free visual tweaks.** Click any element, adjust typography, colors, spacing, borders, shadows — all at zero token cost. You can spend hours perfecting visual details without spending a cent beyond your subscription. Every other AI builder charges for every visual change (via chat prompts). v0 separates "design polish" from "code generation" and makes the former free. For designers and perfectionists, this is transformative.

**The second "Aha!" is the git workflow.** v0 automatically creates branches, commits every change with meaningful messages, and never touches main. You create PRs and merge within v0. For teams already on GitHub, this means v0 slides into their existing development workflow rather than replacing it. No other AI builder treats git as a first-class concern this deeply.

**The third "Aha!" is the Vercel infrastructure.** One click and your app is on a global edge network with enterprise-grade CDN, auto-scaling, DDoS protection, Web Analytics, Speed Insights, and deployment protection. Vercel serves some of the highest-traffic sites on the internet (TikTok, Hulu, Under Armour). When you deploy from v0, you're deploying to the same infrastructure. This is production-grade from minute one — not "hosted on a startup's servers."

**The fourth "Aha!" is model selection.** v0 is the only AI builder that lets users explicitly choose their AI model. A quick UI tweak? Use Mini ($1/1M tokens). A complex architecture change? Use Max ($5/1M tokens). This gives users direct control over the quality-cost tradeoff, something every other builder hides behind opaque "credits."

---

# PHASE 2: "Own The Metal" Architecture Blueprint

## Infrastructure Ownership

v0 sits on top of Vercel's infrastructure, which is itself built on cloud providers (primarily AWS). This creates a unique position: v0 doesn't own the metal, but Vercel operates a sophisticated platform layer that functions as proprietary infrastructure.

### What Vercel/v0 Owns
- **Edge Network** — Vercel's global CDN spans 18+ regions worldwide. Content is served from edge nodes closest to the user. This is not Cloudflare or Fastly — it's Vercel's own edge infrastructure (though built on AWS/GCP underneath).
- **Serverless Functions** — Vercel's serverless compute for API routes and server-side rendering. Auto-scaling, zero cold start on edge functions.
- **ISR (Incremental Static Regeneration)** — Vercel's proprietary caching mechanism for Next.js. Pages regenerate in the background while serving cached versions. This is a genuine competitive moat — ISR only works optimally on Vercel.
- **Vercel Sandbox** — lightweight VMs for v0 previews. Runs the full Next.js application including server-side code, API routes, and database connections. What you see in preview IS what you get in production.
- **Build pipeline** — Vercel's build system optimizes Next.js applications automatically (code splitting, image optimization, font optimization, bundle analysis).
- **Analytics** — Web Analytics (page views, unique visitors, top pages) and Speed Insights (Core Web Vitals, TTFB, LCP, CLS) built into the platform.
- **v0's AI models** — v0 Mini, Pro, and Max are proprietary model offerings (likely fine-tuned versions of foundation models, not raw API calls to Claude/GPT). Vercel has invested in making these models specifically excellent at Next.js + React + Tailwind code generation.
- **v0 Model API** — v0's models can be used in external IDEs (Cline, Cursor, Zed). This means the AI engine is a separable product, not locked to the v0 UI.

### What Vercel/v0 Does NOT Own
- **Underlying cloud compute** — Vercel runs on AWS and GCP. They don't own data centers.
- **Database services** — Vercel offers Vercel Postgres, Vercel KV (Redis), and Vercel Blob (object storage), but these are managed wrappers around third-party services (Neon for Postgres, Upstash for KV).
- **No in-app domain registration** — unlike Replit, you can't buy domains within v0. Domain management happens in the Vercel dashboard or an external registrar.
- **No email hosting** — unlike Horizons (Hostinger email) or HostPapa's existing offering. Users need separate email providers.

### Key Strategic Insight for HostPapa
**v0 demonstrates the "platform play" — the AI builder is the acquisition funnel for the hosting platform.** v0 generates Next.js apps that deploy to Vercel. This creates a flywheel: users try v0 → deploy to Vercel → become Vercel customers → pay for hosting, analytics, databases, and domains. HostPapa should think of their AI builder the same way: it's not a standalone product, it's the top of funnel for HostPapa hosting, domains, email, and managed services.

The v0 Model API is also strategically significant. By making their AI models available in external IDEs, Vercel ensures that even developers who don't use the v0 UI still generate Vercel-optimized code. HostPapa could consider a similar strategy: make the AI code generation available via API/plugin, not just via a proprietary builder UI.

---

## BaaS Reliance

### Database Layer
- **No built-in database** — v0 generates code that connects to whatever database the user specifies:
  - **Vercel Postgres** — managed PostgreSQL via Neon. Serverless-friendly connection pooling. Billed based on compute time, storage, and data transfer.
  - **Vercel KV** — managed Redis via Upstash. For caching, sessions, rate limiting.
  - **Vercel Blob** — object storage for files, images, uploads.
  - **External databases** — Supabase, PlanetScale, MongoDB Atlas, Turso, or any database with a connection string. The AI generates appropriate ORM code (Prisma, Drizzle) for whatever you choose.
- **This is the "bring your own database" model.** v0 doesn't force a data layer (unlike Lovable → Supabase, or Horizons → Integrated Backend). The upside: maximum flexibility. The downside: users must choose and configure their own database, adding a cognitive step.

### Authentication
- AI generates auth flows using **NextAuth.js / Auth.js** — the standard Next.js auth library. Supports email/password, OAuth providers (Google, GitHub, Discord, etc.), magic links, and JWT sessions.
- No Vercel-native auth service. Auth is application code, not a platform feature.
- AI follows Auth.js best practices by default, which provides better security baselines than hand-coded auth.

### Payment Processing
- AI generates Stripe integration when requested. Standard approach across all competitors.

---

## LLM Orchestration

### Models (Detailed)
v0 is the only competitor that exposes model selection to end users AND publishes pricing per token:

| Model | Input Price | Cache Write | Cache Read | Output Price | Positioning |
|-------|-----------|-------------|------------|-------------|-------------|
| **v0 Mini** | $1/1M tokens | $1.25/1M | $0.10/1M | $5/1M | Fast, near-frontier. Quick iterations. |
| **v0 Pro** | $3/1M tokens | $3.75/1M | $0.30/1M | $15/1M | Balanced. Default for most work. |
| **v0 Max** | $5/1M tokens | $6.25/1M | $0.50/1M | $25/1M | Maximum intelligence. Complex tasks. |

Key observations:
- **Cache reads are 10-20x cheaper than fresh input.** This means iterative work within a session gets progressively cheaper as context is cached. The first prompt in a session costs more than the tenth.
- **Output tokens cost 5x input tokens.** Code-heavy responses (large file generation) are the most expensive operations.
- **v0's models are likely fine-tuned foundation models** — the pricing structure mirrors Anthropic's Claude pricing ratios, suggesting Claude may be the base model with v0-specific fine-tuning for Next.js/React/Tailwind.

### Hydration Pattern
- **Full project context** — all project files, conversation history, and Instructions are injected into every prompt.
- **Instructions persistence** — custom rules persist across all chats in a project. This is a project-level system prompt that ensures consistency.
- **Agent autonomy** — the AI can search the web for documentation, inspect live sites, and diagnose errors without explicit user instruction. These are tool-use actions within the agent framework.
- **Vercel-specific knowledge** — v0 has deep knowledge of Vercel's platform features (ISR, Edge Functions, Image Optimization, etc.) and generates code that leverages them. This is a competitive moat — v0-generated code is optimized for Vercel in ways that generic AI tools can't match.

### Context Window
- **128,000 tokens input** — large enough for most project contexts.
- **32,000 tokens output** — maximum per response. Sufficient for generating multiple files in one turn.
- **Image inputs** — ~1,000 tokens each. A screenshot or Figma export is a relatively small context cost.

---

## Preview Compute Environment

### How Preview Works (Post-February 2026)
- **Vercel Sandbox** — lightweight VMs running the full Next.js application. Server-side rendering, API routes, database connections, and middleware all work in preview. This replaced the previous browser-based preview.
- **Production accuracy** — the preview IS production. Same Next.js runtime, same environment, same behavior. What you see in v0's preview is what your users will see after deployment. This is the most production-accurate preview of any AI builder.
- **Tradeoff** — slightly slower to start than Bolt's WebContainers (which boot in ~2 seconds) but dramatically more accurate. WebContainers can't run server-side code, database connections, or API routes. Vercel Sandbox can.

---

## The Diffing Engine

### How Changes Are Applied
- AI generates code changes shown in the editor's diff view.
- Full-file generation for new files.
- Diff-based edits for existing files — the AI modifies only the relevant sections.
- Design Mode generates targeted Tailwind class changes (more surgical than chat-based generation).
- Every change is auto-committed to git with a meaningful message.

### Version Control
- **Auto-commits to git** — every code change committed with meaningful messages. Full git history.
- **Automatic branching** — per-chat branches (e.g., v0/main-abc123). Parallel work streams.
- **Protected main** — v0 never pushes to main. Enterprise-grade safety.
- **PR workflow** — create, view, merge PRs within v0. Full GitHub integration.
- **Branch from branch** — duplicate for parallel experiments.
- **Vercel preview deployments** — each branch gets its own preview URL. Shareable with stakeholders.
- **Vercel deployment rollback** — revert production to any previous deployment via the Vercel dashboard.

### Assessment
**Best-in-class version control.** The combination of automatic branching + auto-commits + protected main + in-app PR management + Vercel preview deployments + production rollback provides the most complete version control story of any AI builder. This is genuinely enterprise-ready.

---

# PHASE 3: Maintainability & Guardrail Teardown

## Preventing Spaghetti Code

### Enforced Stack
- **Next.js** — primary framework with best-in-class AI expertise. v0 knows Next.js App Router, Server Components, Server Actions, Route Handlers, middleware, and ISR at a level that no generic AI tool matches. This deep framework knowledge is a competitive moat.
- **React** — component framework. Server and Client components properly distinguished.
- **TypeScript** — enforced. All generated code is TypeScript with proper typing. This catches entire categories of bugs at compile time.
- **Tailwind CSS** — default and primary styling. Design Mode reads/writes Tailwind classes.
- **shadcn/ui** — default component library (Radix primitives underneath). Accessible, composable, well-documented components. The same library Lovable uses.
- **Can import existing repos** — when importing non-Next.js repos, v0 works with the existing stack. AI quality may be lower for non-Next.js frameworks.

### Code Quality Mechanisms
1. **Instructions** — persistent project rules enforce coding standards across all chats. Example: "Use server components by default," "Always validate form inputs with Zod," "Follow the repository's existing naming conventions."
2. **TypeScript** — strict mode catches type errors at compile time.
3. **Diff view** — every change is visible and reviewable. Users can catch quality issues before committing.
4. **"Fix with v0"** — automatic error diagnosis and repair. 20 free/day.
5. **Build pipeline** — Vercel's build step catches build errors, dead code, and import issues. Deployment fails if the build fails — a hard quality gate.
6. **No dedicated security scanners** (unlike Lovable). No dependency auditing in the builder.
7. **No automated testing** generation (unlike Replit's App Testing).

---

# PHASE 4: GTM & Telco Partner Strategy

## Pricing Model

### Tier Structure

| Tier | Monthly Price | Monthly Credits | Projects | Key Differentiators |
|------|-------------|----------------|----------|-------------------|
| **Free** | $0 | $5 | 200 | Basic usage. "Built with v0" badge on deployed apps. |
| **Premium** | $20/month | $20 | Unlimited | Badge removed. Full feature access. Individual use. |
| **Team** | $30/user/month | $30/user | Shared | Shared projects, shared credit pool, basic access controls, centralized billing. Credit pool shared across team — individual monthly credits used first, then shared pool. Shared pool credits expire after 1 year. |
| **Business** | $100/user/month | $30/user | Shared | Everything in Team + **data opt-out** (prompts and code NOT used for model training). This is the critical enterprise privacy tier. |
| **Enterprise** | Custom | Custom | Custom | SAML SSO, full RBAC, priority access, advanced analytics. |

### Token Pricing Detail
| Model | Input/1M | Cache Write/1M | Cache Read/1M | Output/1M |
|-------|---------|---------------|--------------|----------|
| v0 Mini | $1.00 | $1.25 | $0.10 | $5.00 |
| v0 Pro | $3.00 | $3.75 | $0.30 | $15.00 |
| v0 Max | $5.00 | $6.25 | $0.50 | $25.00 |

### Technical Limits
- Context window: 128,000 tokens
- Max output: 32,000 tokens per response
- Image inputs: ~1,000 tokens each
- Monthly credits don't roll over
- Can purchase additional credits on-demand

### Hidden Costs & Limits
1. **Vercel hosting is SEPARATE** — v0 credits pay for AI code generation only. Production hosting on Vercel has its own pricing (free tier → $20/month Pro → $400/month Enterprise). Database, analytics, and domains are additional Vercel charges.
2. **Data opt-out costs $100/user/month** — the Business tier is the minimum for organizations that can't have their code used for model training. This is 5x the Premium price. For a 10-person team, that's $1,000/month for data privacy.
3. **Credits don't roll over** — monthly reset. Unused credits are lost. (Team shared pool credits do persist for 1 year.)
4. **"Built with v0" badge on free tier** — cannot be removed without upgrading to Premium ($20/month).
5. **Output tokens are 5x input tokens** — large file generation is expensive. A complex component that generates 2,000 tokens of output costs 10x more than the input prompt that requested it.

### Cost Comparison with Competitors
| Scenario | v0 Cost | Bolt Cost | Lovable Cost | Horizons Cost |
|----------|---------|-----------|-------------|---------------|
| AI builder only | $20/mo | $20/mo | $50/mo | $6.99/mo |
| + Hosting | +$0-20/mo (Vercel) | +$0-19/mo (Netlify) | Included | Included |
| + Database | +$0-25/mo (Vercel Postgres) | +$0-25/mo (Supabase) | Included (Supabase) | Included |
| + Data privacy | $100/user/mo | Not available | Not documented | N/A |
| **Realistic total** | **$20-145/mo** | **$20-64/mo** | **$50-75/mo** | **$6.99-79.99/mo** |

---

## B2B2C Channel Readiness

### Enterprise Features
- **Team workspaces** — shared projects and credit pools.
- **Access controls** — basic on Team, full RBAC on Enterprise.
- **SAML SSO** — Enterprise tier only.
- **Data opt-out** — Business tier ($100/user/month). Prompts and code not used for training.
- **Centralized billing** — Team and above. Single invoice per organization.
- **Usage analytics** — daily, weekly, monthly usage summaries. Per-event log with date, user, model, cost. Basic on Free/Premium, team-wide on Team+, advanced on Enterprise.

### White-Label Assessment: NOT READY (but API is interesting)
- **Heavily Vercel/v0 branded** — builder UI, deployment URLs, analytics all branded.
- **"Built with v0" badge** — free tier apps show v0 branding.
- **No embeddable builder widget.**
- **No partner/reseller program.**
- **No custom branding options.**

### v0 Model API — The Semi-White-Label Option
v0's models can be used in external IDEs (Cline, Cursor, Zed) via the v0 Model API. This means:
- A partner could build their own UI on top of v0's AI models
- The AI code generation is separable from the builder interface
- This is the closest thing to "white-label AI" among all competitors

**For HostPapa:** using v0's Model API would give you state-of-the-art Next.js code generation without building your own AI. You'd build the UI, hosting, and deployment — v0 provides the brain. The downside: dependency on Vercel's API, no control over model improvements or pricing, and generated code would be optimized for Next.js/Vercel, not necessarily for HostPapa's infrastructure.

---

## Positioning & Persona

### Hero Copy
- "AI agent that helps anyone create real code and full-stack apps"
- Emphasis on "real code" and "production-ready" — positioning against tools that generate prototypes.

### Target Persona
**Primary: Professional development teams** who need AI-accelerated development within existing GitHub/Vercel workflows. The git integration, PR workflow, existing repo import, and team features signal "this is for people already shipping software."

**Secondary: Designers and PMs** who need working prototypes. Design Mode's free visual tweaks and Figma import target people who think visually, not in code.

**Tertiary: Solo developers and indie hackers** on the Premium tier ($20/month) who want the fastest path from idea to production on world-class infrastructure.

---

# PHASE 5: Enterprise Compliance & Accessibility

## Security Posture

### Application-Level Security
- **No code security scanners** in v0 itself.
- **No dependency auditing** in the builder.
- **No pre-publish security gate.**
- **"Fix with v0"** catches runtime errors but not security vulnerabilities.
- **TypeScript strict mode** prevents certain categories of bugs.
- **Vercel build pipeline** catches build errors and can be configured with security checks (ESLint rules, etc.).

### Infrastructure-Level Security (via Vercel)
- **DDoS protection** — Vercel's edge network provides infrastructure-level DDoS mitigation.
- **WAF** — Vercel Firewall available on Enterprise.
- **Deployment protection** — password protection, user/team restriction, preview protection.
- **Auto SSL** — all deployments get HTTPS automatically.
- **Edge Functions** — server-side code runs at the edge, reducing attack surface for origin servers.

### Data Privacy
- **Data opt-out requires Business tier ($100/user/month)** — below this, prompts and code may be used for model training. This is a significant enterprise concern.
- **No explicit "we never use your data" statement on free/Premium tiers** — contrast with Lovable's blanket "We never use your prompts or project data to train the models."

## Certifications (via Vercel)

| Certification | Status | Details |
|--------------|--------|---------|
| SOC 2 Type II | ✅ | Vercel is SOC 2 Type II compliant. Applies to infrastructure hosting v0 apps. |
| ISO 27001 | ✅ | Vercel is ISO 27001 certified. |
| GDPR | ✅ | Vercel is GDPR compliant. DPA available. |
| HIPAA | ⚠️ Enterprise | Vercel offers HIPAA compliance on Enterprise plans with BAA. |
| PCI DSS | ⚠️ Via Stripe | Payment compliance delegated to Stripe when using Stripe integration. |
| VPAT | ❌ | No accessibility conformance report published for v0 or generated output. |

---

# PHASE 6: Churn & Scalability Ceiling

## Code Ejection

### Export Options
- **GitHub is the default** — if you've connected a GitHub repo, your code is already there. Every change is committed. Full git history. You can close your v0 account and your code remains on GitHub.
- **Standard Next.js** — the generated code is standard Next.js + React + TypeScript + Tailwind. There is NOTHING proprietary in the generated code. No v0-specific SDKs, no Vercel-locked APIs (unless you explicitly use Vercel-specific features like ISR or Edge Functions, which are standard Next.js APIs anyway).
- **Vercel is optional** — Next.js apps can be deployed on any Node.js hosting: AWS, GCP, DigitalOcean, Railway, Fly.io, self-hosted. The code is fully portable.
- **No lock-in for database** — v0 doesn't force a database. If you used Vercel Postgres, you can migrate to any PostgreSQL host. If you used Supabase, you keep your Supabase account.

### Ejection Assessment
**Best ejection story of any competitor.** Code is on GitHub by default. Framework is standard. No proprietary SDKs. Hosting is optional. Database is user-chosen. The only "lock-in" is that v0-generated code is optimized for Next.js, which means migrating to Vue or Angular would require a rewrite. But within the Next.js ecosystem, ejection is frictionless.

---

## The Logic Wall

| Complexity Level | Capability | Evidence & Details |
|-----------------|-----------|-------------------|
| ✅ Works great | UI components and design systems | Core competency. Best-in-class React/Tailwind/shadcn generation. |
| ✅ Works great | Full-stack Next.js apps with API routes | Vercel Sandbox previews server-side code. Production deployment handles scaling. |
| ✅ Works great | Data visualizations and interactive UIs | AI SDK integration for AI-powered features. Chart libraries, dynamic content. |
| ✅ Works great | Landing pages and marketing sites | Design Mode for visual polish. ISR for performance. Edge CDN for speed. |
| ⚠️ Works with effort | Complex multi-page apps with state management | v0 Max model needed. Instructions help maintain consistency. Multiple chats for parallel feature development. |
| ⚠️ Works with effort | Real-time features | Next.js supports WebSocket via API routes. Not as natural as a backend-first tool like Replit. |
| ⚠️ Works with effort | Complex authentication flows | NextAuth.js is powerful but configuration-heavy. AI handles standard flows well; custom OAuth providers require iteration. |
| ❌ Struggles | Non-Next.js backends | v0's expertise is Next.js. Python, Go, Ruby backends are not well-supported. Import an existing repo with those backends, sure — but AI generation quality drops. |
| ❌ Fails | Native mobile apps | Web-only output. React Native not supported (despite React expertise). |
| ❌ Fails | Monorepo management | v0 works on single projects, not multi-package monorepos. |

### Competitive Position
v0's Logic Wall is **high for Next.js, moderate for everything else.** Within the Next.js ecosystem, v0 is the most capable tool — period. The deep framework knowledge, Vercel-optimized deployment, and enterprise git workflow make it the best choice for professional Next.js development. Outside Next.js, it's limited. If your app needs a Python backend, Go microservices, or non-React frontend, v0 is the wrong tool.

---

# EXHAUSTIVE FEATURE INDEX

## AI & Generation

| Feature | Description | Details |
|---------|-------------|---------|
| **Chat Mode** | Conversational AI development | Describe what you want in natural language. AI generates working code with live preview. Supports text, images, Figma files as input. Multi-language prompt support — write in any language, AI translates to code. Maintains conversation history as context. Rich UI feedback shows agent actions in real-time. |
| **v0 Mini Model** | Fast, near-frontier intelligence | $1/$5 per 1M input/output tokens. Cache read: $0.10/1M. Optimized for speed. Best for quick iterations, simple components, and rapid prototyping. Lowest cost per interaction. |
| **v0 Pro Model** | Balanced speed and intelligence | $3/$15 per 1M input/output tokens. Cache read: $0.30/1M. Default for most work. Best balance of quality, speed, and cost. Recommended for standard development tasks. |
| **v0 Max Model** | Maximum intelligence | $5/$25 per 1M input/output tokens. Cache read: $0.50/1M. For complex, demanding work: architectural decisions, multi-file refactoring, sophisticated business logic. Highest quality output. |
| **Agent Capabilities** | Autonomous AI actions | Web search for current documentation and APIs. Site inspection for debugging deployed apps. Automatic error fixing ("Fix with v0" — 20 free/day). External tool integration. Visual progress indicators show what agent is doing in real-time. |
| **Instructions** | Persistent project-level rules | Create reusable instruction sets applied across ALL chats in a project. Click + in prompt bar to create. Examples: coding standards, design system rules, accessibility requirements, naming conventions. Inject into every prompt automatically. Replaces old "Rules" system. Ensures consistency across multi-chat projects. |
| **Image-to-Code** | Upload visuals for replication | Drag/paste screenshots, wireframes, Figma exports, hand-drawn sketches. AI analyzes layout, colors, spacing, typography. ~1,000 tokens per image. Works with rough mockups through pixel-perfect designs. |
| **Fix with v0** | Automatic error repair | When deployment or preview has errors, "Fix with v0" button appears. AI reads error logs, stack traces, and relevant code. Diagnoses and applies fix. 20 free uses per day on unedited code. Additional fixes consume credits. Closed-loop debugging: error → diagnosis → fix → redeploy. |
| **Multi-Language Prompts** | Write prompts in any language | AI processes prompts in 100+ languages and generates English code output. No translation step required. Expands accessibility to non-English-speaking developers and teams. |

## Design Mode

| Feature | Description | Details |
|---------|-------------|---------|
| **Element Selection** | Click-to-select in preview | Activate with Option+D or Design tab. Hover to highlight elements. Click to select. Properties panel appears for the selected element. |
| **Typography Controls** | Font editing panel | Font family (pulls from tailwind.config.js), font size, font weight, line height, letter spacing, text alignment, text decoration. All changes generate proper Tailwind classes. ZERO token cost. |
| **Color Controls** | Text and background colors | Color picker for text and background. Pulls colors from tailwind.config.js design tokens for brand consistency. Arbitrary color values also supported. ZERO token cost. |
| **Layout Controls** | Margin and padding | Per-side margin and padding controls (top, right, bottom, left). Visual spacing adjustments. Tailwind spacing scale values. ZERO token cost. |
| **Border Controls** | Border editing | Border color, style (solid, dashed, dotted), and width configuration. ZERO token cost. |
| **Appearance Controls** | Opacity and border radius | Opacity slider (0-100%). Corner radius adjustment (per-corner support). ZERO token cost. |
| **Shadow Controls** | Box shadow editing | Add, customize, or remove box shadows. Multiple shadow support for layered effects. ZERO token cost. |
| **Content Editing** | Direct text editing | Edit text content of any element directly in the Design Panel. Faster than re-prompting for copy changes. ZERO token cost. |
| **Tailwind Native** | Design token integration | Design Panel reads colors, spacing, and fonts from your project's tailwind.config.js. Ensures all visual tweaks use your project's design system, not arbitrary values. Maintains brand consistency across AI-generated and manually-tweaked elements. |
| **Natural Language on Selection** | Prompt-based complex changes | For modifications beyond the panel's capabilities (e.g., "Convert this to a dropdown" or "Add a loading skeleton"), type a prompt while an element is selected. AI makes the change scoped to that element. This DOES cost tokens (unlike panel adjustments). |
| **Save** | Commit visual changes | Click save button at bottom of preview to commit Design Mode edits. Unsaved changes are indicated visually. Saved changes are auto-committed to git. |

## Editor

| Feature | Description | Details |
|---------|-------------|---------|
| **VS Code-Style Editor** | Full code editing environment | Complete file editor built into v0 (launched February 2026). Browse files, edit code, see syntax highlighting. Editor, AI agent, preview, and configuration in one workspace. Transforms v0 from chat tool to development environment. |
| **File Explorer** | Project file tree | Navigate all project files. Standard file operations (create, rename, delete, move). |
| **Syntax Highlighting** | Multi-language support | TypeScript, JavaScript, HTML, CSS, JSON, SQL, Markdown. Proper language detection. |
| **Diff View** | Per-interaction change visualization | See exactly what AI changed per message. Expandable diffs in chat history. Review changes before committing. Better change transparency than Replit (no per-message diffs) or Horizons (no diffs at all). |
| **Inline Editing** | Direct code modification | Edit any file directly in the editor. Changes apply to preview immediately. Manual refinement of AI-generated code without re-prompting. |

## Git Integration

| Feature | Description | Details |
|---------|-------------|---------|
| **Automatic Branching** | Per-chat isolated branches | v0 creates a dedicated branch (e.g., v0/main-abc123) for each chat. No manual branch management. Each chat's changes are isolated. Prevents cross-contamination between feature work. |
| **Auto-Commits** | Every change committed with meaningful messages | Every message that changes code automatically creates a git commit. Full change history preserved in git. Commit messages describe what was changed and why. |
| **Protected Main** | Production branch safety | v0 NEVER pushes directly to main. All changes go through pull requests. Production code stays safe from unreviewed AI changes. Enterprise-grade safety that no other AI builder enforces by default. |
| **Pull Requests** | Full in-app PR workflow | Create PRs from v0. View PR status (open, merged, closed). Merge PRs without leaving v0. Review comments supported. Full GitHub PR workflow. |
| **Branch from Branch** | Parallel work streams | Duplicate a branch to work on multiple approaches. Choose to keep or start fresh with project configuration. Useful for A/B testing implementations, exploring alternatives. |
| **GitHub Import** | Work on existing codebases | Import ANY GitHub repository into v0. v0 creates a working branch automatically. Full project context loaded. AI works on existing code, not just greenfield projects. |
| **Multi-Tool Collaboration** | Cross-IDE workflow | Team members using Cursor, Claude Code, Windsurf, Zed, or any git-compatible tool can work on the same repo. Push/pull through GitHub. v0 is one tool in the workflow, not a walled garden. |

## Deployment (Vercel)

| Feature | Description | Details |
|---------|-------------|---------|
| **One-Click Publish** | Deploy to Vercel's global infrastructure | Click Publish → automatic build, optimization, and deployment to Vercel's edge network. Zero-downtime deployments. Automatic code splitting, image optimization, font optimization, and bundle analysis. |
| **Global Edge Network** | 18+ region CDN | Content served from edge nodes closest to the user. 18+ global locations. Automatic caching. Edge Functions for server-side logic at the edge. Low-latency delivery worldwide. |
| **Preview Deployments** | Per-branch preview URLs | Every branch gets its own preview URL automatically. Test changes before merging to production. Share with stakeholders for review and feedback. Preview URLs persist until the branch is deleted. |
| **Vercel Sandbox** | Production-accurate previews | Lightweight VMs running the full Next.js app. Server-side code, API routes, database connections, and middleware all work in preview. What you see in preview IS what you get in production. Replaces earlier browser-based preview. |
| **Production Deployments** | Enterprise-grade hosting | Auto HTTPS, edge caching, DDoS protection, performance monitoring, analytics integration. Zero-downtime deployments. Automatic rollback on failed builds. |
| **Custom Domains** | Domain configuration | Add custom domains via Vercel dashboard. DNS configuration guidance. Domain redirects (www → apex, etc.). Auto-SSL certificate provisioning. |
| **Environment Variables** | Secrets management | Production, preview, and development environment variable management. Synced between v0 and Vercel. Encrypted at rest. Per-environment configuration (different API keys for production vs preview). |
| **Web Analytics** | Traffic monitoring | Page views, unique visitors, top pages, referrers, geographic distribution. Built into Vercel platform. No Google Analytics setup required. |
| **Speed Insights** | Performance monitoring | Core Web Vitals (LCP, FID, CLS), Time to First Byte, page load times. Real User Monitoring (RUM). Performance trends over time. |
| **Deployment Protection** | Access control | Password protection for preview deployments. User/team restriction. Useful for staging environments, client previews, internal tools. |
| **Instant Rollback** | Revert production | Roll back to any previous production deployment via Vercel dashboard. Instant — no rebuild required. |

## Projects & Organization

| Feature | Description | Details |
|---------|-------------|---------|
| **Projects** | Vercel deployment containers | Each Project maps to a Vercel production URL. Includes domains, environment variables, deployment settings. Multiple chats can contribute to the same Project — useful for parallel feature development with different team members or approaches. |
| **Folders** | Chat organization | Group chats for organization only. No effect on deployments or code. Purely for human navigation. |
| **Templates** | Community starting points | Browse community-built templates. Start from proven patterns. Explore what others are building. |

## Team & Enterprise

| Feature | Description | Details |
|---------|-------------|---------|
| **Shared Projects** | Team collaboration | Team members work on shared projects. Requires Team plan ($30/user/month) or above. Multiple people can work on the same codebase via separate chats/branches. |
| **Shared Credit Pool** | Team-wide credits | Monthly credits are per-user; shared pool credits used when individual credits are exhausted. Shared pool credits expire after 1 year (unlike monthly credits which expire monthly). Team and Enterprise plans. |
| **Access Controls** | Permission management | Basic controls on Team plan. Full RBAC on Enterprise. Control who can create, edit, deploy, and view projects. |
| **Centralized Billing** | Organization invoice | Single invoice for the team. Usage tracking per member. Per-event cost log with date, user, model, and dollar amount. |
| **Data Opt-Out** | Training data exclusion | Business plan ($100/user/month) and Enterprise. Prompts and code are NOT used for model training. Critical for organizations with proprietary code or compliance requirements. |
| **SAML SSO** | Enterprise identity | Enterprise plan only. Integration with identity providers (Okta, Azure AD, etc.). |
| **Usage Analytics** | Spending visibility | Daily, weekly, monthly summaries. Detailed per-event log showing date, user, model used, and cost. Basic on Free/Premium. Team-wide on Team+. Advanced breakdowns on Enterprise. |

## Integrations

| Feature | Description | Details |
|---------|-------------|---------|
| **GitHub** | Deep version control integration | Import repos, automatic branching, auto-commits, PR workflow. Every code change is in git. The deepest GitHub integration of any AI builder. |
| **Vercel** | Hosting & infrastructure | Seamless deployment to global edge network. Environment variables, domains, analytics, monitoring, protection. The AI generates Vercel-optimized code. |
| **Figma** | Design import | Upload Figma files for AI to convert to code. Design-to-code workflow for designers. |
| **v0 Model API** | External IDE integration | Use v0's AI models (Mini, Pro, Max) in Cline, Cursor, and Zed editors. Bring v0's Next.js code generation expertise to your preferred development environment. Pricing same as in-app usage. |
| **Snowflake** | Data applications | Build Python/SQL data apps on Snowflake. Data visualization and analysis. Niche but significant for data-heavy organizations. |

## Supported Technologies

| Technology | Status | Details |
|------------|--------|---------|
| **Next.js** | ✅ Best-in-class | Primary framework. Deepest AI expertise. App Router, Server Components, Server Actions, ISR, Edge Functions. |
| **React** | ✅ Primary | Component framework. Server and Client components properly distinguished. |
| **TypeScript** | ✅ Enforced | All generated code is TypeScript with proper typing. |
| **Tailwind CSS** | ✅ Primary | Default styling. Design Mode reads/writes Tailwind. Design token integration via tailwind.config.js. |
| **shadcn/ui** | ✅ Default | Accessible component library (Radix primitives). Consistent with Lovable's default components. |
| **AI SDK** | ✅ Supported | Vercel's AI SDK for building AI-powered features (chatbots, embeddings, streaming). |
| **Python** | ⚠️ Limited | Data apps via Snowflake integration. Not for general Python backends. |
| **Vue/Svelte/Angular** | ⚠️ Import only | Can work with imported repos but AI generation quality drops significantly outside React. |
| **Native Mobile** | ❌ Not supported | Web-only output. No React Native despite React expertise. |

---

**Sources:**
- v0 docs: https://v0.app/docs
- v0 FAQs: https://v0.app/docs/faqs
- v0 pricing: https://v0.app/docs/pricing
- v0 GitHub integration: https://v0.app/docs/github
- v0 Design Mode: https://v0.app/docs/design-mode
- v0 Deployments: https://v0.app/docs/deployments
- Vercel documentation: https://vercel.com/docs
