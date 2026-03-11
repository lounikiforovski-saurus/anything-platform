# Hostinger Horizons — Full Enterprise Teardown
### For Hostopia / HostPapa — March 2026
### Last Updated: March 9, 2026

---

# COMPANY PROFILE

| Field | Detail |
|-------|--------|
| **Parent Company** | Hostinger International, Ltd. |
| **Founded** | 2004 (Horizons launched March 2025) |
| **HQ** | Kaunas, Lithuania |
| **Employees** | 1,000+ across 54 countries |
| **Customers** | 4.6M businesses across 150+ countries |
| **2025 Revenue** | €275.4M (+51% YoY) |
| **Funding** | Bootstrapped — no external venture capital |
| **Recognition** | FT 1000 fastest-growing companies for 6 consecutive years; 2nd place in FT & Statista Long-term Growth Champions: Europe 2026 |

### Financial Trajectory (Verified)

| Year | Revenue | Growth | Customers | Notes |
|------|---------|--------|-----------|-------|
| 2022 | €69.6M | — | 1.5M | |
| 2023 | €110.2M | +57% | 2.4M | First EBITDA profit (€2.4M) |
| 2024 | €182.4M | +65% | 3.5M | |
| 2025 | €275.4M | +51% | 4.6M | 4th consecutive year of 50%+ growth |

**CAGR:** 58% since 2022.

### Horizons Metrics
- **Launched:** March 2025
- **1M+ users** tried Horizons in first year
- **800,000 Horizons customers** by end of 2025
- **Usage breakdown:** 49% business/portfolio sites, 30% custom apps/AI tools, 10% ecommerce, 6% content/learning, 5% SaaS dashboards

### Strategic Significance for HostPapa
Hostinger is the **only AI builder competitor that is itself a hosting company**. They proved that a hosting company CAN layer an AI builder on top of existing infrastructure. They own the compute, CDN, DNS, email, and security — the AI builder is just the generation layer on top. This is the exact model HostPapa should pursue, designed for channel from day one.

---

# PHASE 1: Product & UX Friction Audit

## Exhaustive UI Matrix

### AI Chat Interface
- **Prompt input** — bottom-aligned text area for natural language descriptions. Supports 80+ languages, including voice input on Starter tier and above. The AI interprets intent and generates both frontend and backend code simultaneously.
- **Image-to-code** — upload screenshots, wireframes, hand-drawn sketches, or polished designs. The AI analyzes visual layout, element positioning, color palette, and typography, then generates a working web page matching the input. Available on Starter ($13.99/mo) and above — the Explorer tier ($6.99/mo) is text-only.
- **Voice prompting** — speak your requirements instead of typing. Available on Starter and above. Useful for rapid iteration when describing layout changes or feature additions.
- **Select-and-edit mode** — click elements in the live preview to edit text and images directly without consuming AI credits. This is a visual editing layer on top of the AI-generated code. Only available on Starter and above.
- **Free chat mode** — a separate non-credit-consuming chat for guidance, help, and learning. Available on Starter and above. Provides AI assistance for understanding what to build and how, without depleting message credits.
- **Auto error fixing** — when the generated app has bugs, Horizons automatically detects and offers to fix them. Reduces the manual debugging loop.
- **Context persistence** — the AI maintains conversation history within each project session, allowing iterative refinement without re-explaining prior context.
- **Suggested follow-ups** — after each generation, the AI suggests logical next steps (add payments, improve design, add user accounts).

### Code Editor
- **Full source code access** — view and edit all generated files directly. Available on Hobbyist ($39.99/mo) and above — notably NOT on Explorer or Starter tiers. This is a significant gate: cheaper plans can only interact via chat and visual editing.
- **Syntax highlighting** — standard web language support (HTML, CSS, JavaScript).
- **Live preview sync** — code changes reflect immediately in the preview panel.
- **File tree** — browse all project files. Create, rename, delete.
- **Custom JavaScript injection** — add custom logic or integrate third-party APIs by editing code directly.
- **No terminal access** — unlike Replit, there is no command-line interface. Package management is AI-mediated.

### Project Management
- **Project dashboard** — grid view of all created projects. Explorer: 1 project. Starter: up to 25. Hobbyist: up to 50. Hustler: up to 50 (with early feature access).
- **Version history** — restore any previous version of a project with one click. Available on all plans. Hostinger does not document how many versions are retained.
- **Project duplication** — clone a project to use as a template. Available on Hobbyist and above.
- **Collaboration** — invite team members to work on projects. Available on Starter and above.
- **Built-in analytics** — track visitor traffic to published projects. Available on Starter and above.

### Publishing & Hosting
- **One-click publish** — deploy the app to Hostinger's production infrastructure with a single click. No build configuration, no CI/CD pipeline.
- **Temporary domain** — free *.horizons.hostinger.com subdomain provided automatically on all plans.
- **Custom domains** — connect your own domain. Free domain included for 1 year on Starter, Hobbyist, and Hustler plans. Explorer has no free domain.
- **Auto SSL** — TLS certificates auto-provisioned for all published projects, including custom domains.
- **Hostinger CDN** — content delivered via Hostinger's in-house CDN across 4 continents. Not a third-party CDN like Cloudflare or Fastly — Hostinger operates their own edge network.
- **99.9% uptime SLA** — contractual uptime guarantee, typical of Hostinger's hosting products.
- **Security stack** — free SSL, built-in firewall, DDoS protection, automated malware scanning included on all plans. No additional security configuration required.
- **Cloud hosting upgrade path** — projects can be upgraded to Hostinger's cloud hosting tier for up to 20x more speed, storage, and capacity when traffic demands grow.
- **Business email** — included with plans. Explorer: 1 mailbox per website for 1 year. Starter: 2 mailboxes. Hobbyist/Hustler: 5 mailboxes.

---

## JTBD Friction Map

### Flow 1: Idea to Live App (First-Time User)
1. Navigate to hostinger.com/horizons (1 click)
2. Click "Start for free" — requires Hostinger account creation (email/Google/GitHub) (2-3 clicks)
3. Describe your app in natural language: "Create an invoice generator with client fields, line items, auto tax calculation, and PDF export" (1 action)
4. AI generates the complete app with live preview (~30s-2min depending on complexity)
5. Iterate via chat: "Add 15% tax calculation" / "Make the header blue" / "Add Stripe payments" (1+ actions per refinement)
6. Click Publish (1 click)
7. App is live on temporary subdomain

**Total clicks to live app: ~5-6**
**Time to first deploy: 5-15 minutes**
**Cognitive leaps: 0** — the entire flow is prompt-driven
**Friction rating: VERY LOW** — the simplest path from idea to production URL of any competitor tested

### Flow 2: Adding a Payment Integration (Stripe)
1. In an existing project chat, type: "Add Stripe payment processing for subscriptions" (1 action)
2. AI generates the Stripe integration code
3. User needs to provide Stripe API keys
4. **COGNITIVE LEAP:** User must create a Stripe account and obtain API keys externally
5. Paste API keys into the project configuration
6. Test the payment flow in preview
7. Republish (1 click)

**Total clicks: ~4-5**
**Cognitive leaps: 1 (external Stripe setup)**
**Friction rating: MEDIUM** — same Stripe friction as every competitor

### Flow 3: Connecting a Custom Domain
1. Open project settings (1 click)
2. Navigate to domain settings (1 click)
3. Either select free domain (included in Starter+) or enter existing domain (1 action)
4. **COGNITIVE LEAP:** If using external domain, configure DNS records at registrar
5. Verify and publish (1 click)

**Total clicks: ~4**
**Cognitive leaps: 0-1** (0 if using included free domain)
**Friction rating: LOW** — the free domain inclusion eliminates DNS friction for most users

---

## The "Aha!" Moment

**The "Aha!" is the zero-config hosting.** With Bolt, you deploy to Netlify (separate account). With Lovable, you deploy to Lovable Cloud (additional setup). With v0, you deploy to Vercel (separate platform). With Horizons, you click Publish and it's live on Hostinger's infrastructure — domain, SSL, CDN, firewall, and email all included in the plan you're already paying for. There is no second platform. No DNS configuration. No deployment pipeline. For a user who has never deployed a web app before, this is profoundly simpler than every alternative.

**The second "Aha!" is the price-to-value ratio.** For $6.99/month (Explorer), you get an AI builder + hosting + SSL + CDN + DDoS protection + firewall + email. Bolt charges $20/month for the builder alone with no hosting. Lovable charges $20/month for the builder with basic hosting. The cheapest equivalent stack from Bolt + Netlify + Cloudflare + email hosting exceeds $40/month. Hostinger's bundle pricing structurally undercuts standalone AI builders.

**The third "Aha!" is the SEO/AI visibility automation.** When you publish, Horizons automatically generates sitemap.xml, robots.txt, AND llms.txt. The llms.txt file is a machine-readable description of the site for AI tools like ChatGPT, Claude, Gemini, and Perplexity. No other AI builder does this. For SMBs who need both Google and AI discoverability, this is a genuine competitive edge.

---

# PHASE 2: "Own The Metal" Architecture Blueprint

## Infrastructure Ownership

This is the most strategically significant insight for HostPapa: **Hostinger owns their entire infrastructure stack.** Unlike Bolt (deploys to Netlify/Vercel), Lovable (deploys to Lovable Cloud, built on managed Supabase), or v0 (deploys to Vercel), Hostinger Horizons deploys to Hostinger's own servers, CDN, and data centers.

### What Hostinger Owns
- **Data centers** — Hostinger operates data centers globally (US, UK, Netherlands, Lithuania, Singapore, India, Brazil)
- **CDN** — in-house content delivery network across 4 continents (not Cloudflare/Fastly)
- **Compute** — shared hosting, VPS, and cloud hosting all on Hostinger-managed infrastructure
- **DNS** — Hostinger operates their own nameservers and domain registration
- **Email** — in-house email hosting (not Google Workspace by default)
- **Security** — own firewall, DDoS protection, malware scanning services

### What Hostinger Does NOT Own
- **LLM infrastructure** — uses third-party "most advanced large language models" (specific models/providers not disclosed). Their AI models page states: "We regularly evaluate the best AI models available, updating our systems so you always have access to the most stable and capable tools." This means they likely multi-vendor (OpenAI, Anthropic, Google) with model routing.
- **Database for user apps** — Horizons uses what they call "Integrated backend" for user accounts, logins, and data storage. The specific database engine is not publicly documented. Based on tutorials mentioning Supabase integration as optional ("With integrations like Supabase, users can save and access their data later"), the "Integrated backend" appears to be a simpler, Hostinger-native persistence layer — possibly a managed PostgreSQL or SQLite service behind their hosting infrastructure.

### Key Strategic Insight
**Hostinger is the proof-of-concept that a hosting company CAN build an AI app builder on its own infrastructure.** The model is: take your existing hosting stack (compute, CDN, SSL, domains, email) and layer an AI builder on top. The AI generates the code; the hosting stack runs it. The result is the lowest-friction, lowest-cost AI builder because there's no third-party deployment target.

**HostPapa already has hosting infrastructure, domain registration, email, and SSL.** The missing piece is the AI builder layer. Hostinger proves this model works and has product-market fit — they launched February 2025 and are already running aggressive pricing campaigns.

---

## BaaS Reliance

### Integrated Backend (Updated March 2026 — Major Revision)

**⚠️ PREVIOUS ANALYSIS WAS SPECULATIVE. Now verified from Hostinger's February 2026 blog post and product pages.**

The Integrated Backend launched in **February 2026** as Horizons' biggest upgrade since launch. It is a **custom-built, proprietary solution — NOT Firebase, NOT Supabase, NOT any third-party BaaS.** It runs entirely on Hostinger's own infrastructure.

**What it includes:**
- **Databases** — built-in persistent data storage (specific database engine still undisclosed, but runs on Hostinger infrastructure)
- **Authentication** — email/password, OTP, SSO via Google, Apple, Facebook, Microsoft
- **File Storage** — built-in file uploads and hosting, no external dependencies
- **Email Service** — integrated email sending for auth flows, notifications, and custom messages
- **Test vs. Live environments** — clear separation for safe development

**Key characteristics:**
- Free for all new users — included at no extra cost
- No external accounts or services needed — everything stays in one platform
- AI generates the backend automatically when you describe what you want
- Existing Supabase users can continue using Supabase or migrate (migration requires project recreation)

**This is a genuine competitive moat.** Every other AI builder (Bolt, Lovable, v0, Replit) depends on external services for the backend. Hostinger owns the entire stack. This means:
1. No third-party costs passed to users
2. No dependency on Supabase/Firebase pricing changes
3. Single billing relationship for everything
4. Simpler user experience (no "connect your Supabase account" step)

### Authentication (Verified)
- **Email/password** — standard auth flow
- **OTP** — one-time password support
- **SSO providers:** Google, Apple, Facebook, Microsoft
- Significantly more auth options than the original analysis suggested

### Payment Processing
- **Stripe** — confirmed integration with checkout, subscriptions, payment flows
- **AdSense** — Google ad monetization integration
- **Built-in ecommerce** — native store functionality without external services

### Third-Party API Integration
- AI can connect to any API you describe — user provides endpoint and credentials, AI generates connection code
- "Need something unique? Just tell Horizons what to connect, and the AI guides you through the setup process."

---

## LLM Orchestration

### Models Used (Verified March 2026 — Major Update)

**⚠️ PREVIOUS ANALYSIS SAID "not publicly disclosed." Hostinger has since published a detailed blog post on their LLM strategy ("Balancing Horizons LLMs").**

Hostinger uses a **multi-agent, multi-model architecture** with specialized AI agents collaborating on each project:

| Model | Provider | Role | Notes |
|-------|----------|------|-------|
| **Gemini 3** | Google | Coding tasks + error fixes | Autofix success rate: 80% (up from 50% with Gemini 2.5). Slow but accurate for complex tasks. |
| **Claude Sonnet 4.5** | Anthropic | Initial prompting | Primary model for most generation. Fast and reliable. |
| **Claude Opus 4.5** | Anthropic | Complex applications | Used for advanced reasoning. SWE-bench Verified score: 80.9% |
| **GPT-5 / 5.1 / 5.2** | OpenAI | Supplementary | Part of the mix but not primary |

### Multi-Agent Architecture
Not just multi-model — Horizons uses **specialized AI agents** that collaborate:
- **Communication Agent** — asks clarifying questions when prompts are ambiguous
- **Planning Agent** (added Jan 2026) — organizes and sequences tasks before execution
- **Code Generation Agent(s)** — model selected based on task complexity

### Performance Optimizations
- **25% reduction** in total response time through model optimization
- **Background error checking** reduced from 40 seconds to 12 seconds
- Right model for right task: Sonnet 4.5 for initial prompts, Gemini 3 for fixes, Opus 4.5 for complex architecture
- Continuous benchmarking across "dozens of major LLMs"
- **Privacy commitment confirmed:** "We never use your prompts or project data to train the models."

### Internal Quality Control
- Custom internal scoring system for landing page generation quality
- "Real-world performance over benchmark scores" philosophy
- Regular model evaluation and switching without user disruption

### Token Economy
- **AI credits** — each plan includes a monthly credit allocation:
  - Explorer: 30 credits/month
  - Starter: 70 credits/month
  - Hobbyist: 200 credits/month
  - Hustler: 400 credits/month
- **1 credit = 1 AI message** (implied but not explicitly confirmed)
- **Free actions that don't consume credits:**
  - Select-and-edit mode (text/image editing in preview)
  - Free chat mode (guidance/help)
  - Publishing/republishing
- **Credit top-ups** — additional credits purchasable anytime on Starter and above without upgrading plan tier. Explorer cannot top up.
- **30-day money-back guarantee** — applies to accounts with fewer than 30 credits used

---

## Preview Compute Environment

### How Preview Works
- **Server-rendered preview** — the app preview is rendered server-side and displayed in a panel beside the chat
- **Live data connections** — preview connects to Integrated Backend for real data interactions. Forms save real data, login flows work, payment processing can be tested.
- **Mobile-responsive preview** — test responsive layouts across device sizes
- **Hot reload** — changes from AI or code editor reflect immediately in preview
- **Sandbox testing** — documented in the FAQ: "Before going live, you can preview and test your web app to make sure everything works as expected. With Hostinger Horizons, you can test your app in a sandbox environment and quickly fix issues by asking AI to make adjustments."

### What We Don't Know
- Whether the preview runs on the same infrastructure as production (likely yes, given Hostinger's hosting integration)
- Whether the preview has network isolation or shares the production environment
- Cold start times vs. Bolt's WebContainers (~2s) vs. Replit's VMs (~5-10s)

---

## The Diffing Engine

### How Changes Are Applied
- AI generates code changes per chat message
- New files: full-file generation
- Existing files: AI determines scope of changes (not documented whether this is diff-based or full-file replacement)
- Visual edits (select-and-edit): targeted changes to specific elements without AI involvement
- Code editor edits: direct file modification

### Version Control
- **Built-in version history** — restore any previous version with one click
- **NO GitHub integration** — this is a significant gap. Code lives only in Horizons unless manually downloaded.
- **NO git-based version control** — no branches, no commits, no diff history
- **NO code export/download on Explorer/Starter** — code editor (and presumably code access) only available on Hobbyist ($39.99/mo) and above
- **Project duplication** — available on Hobbyist+, functions as a manual "branch"

### Assessment
**Version control is Horizons' weakest area.** No git, no GitHub, no code export on lower tiers. Users on Explorer ($6.99/mo) and Starter ($13.99/mo) are fully locked in — they can't see or download their own code. This is a deliberate lock-in strategy, but it also means these users can't hire a developer to continue their project.

---

# PHASE 3: Maintainability & Guardrail Teardown

## Preventing Spaghetti Code

### Generated Stack
- Hostinger does not publicly document which frontend framework the AI generates. Based on the tutorial content describing component-based architecture and the code editor mention of "splitting components for clarity," the generated code is likely:
  - **React or vanilla JavaScript** for frontend
  - **Tailwind or custom CSS** for styling
  - **Proprietary backend** connected to Integrated Backend service
- Stack is NOT user-configurable — you cannot choose Vue, Angular, or alternative frameworks
- The output is optimized for Hostinger's hosting platform, which means it may use Hostinger-specific patterns or APIs that reduce portability

### Code Quality Signals
- **Auto error fixing** — AI detects and repairs bugs automatically
- **No linting or formatting** tools documented
- **No security scanners** (unlike Lovable's 4-scanner approach)
- **No automated testing** framework
- Code quality depends entirely on the underlying LLM's output quality

### When Code Degrades
- The tutorials explicitly acknowledge degradation over many iterations: "When ready, click Publish" appears early in each tutorial flow, suggesting Hostinger optimizes for quick publish cycles rather than extended refinement
- No documented guidance on refactoring or code organization as apps grow complex
- No component library enforcement (like Lovable's shadcn/ui or v0's shadcn/ui)

---

## Self-Healing Mechanisms

### Error Detection & Recovery
- **Auto error fixing** — documented as a feature: "If something breaks, Horizons helps detect and fix issues automatically"
- **AI-assisted debugging** — describe the bug in chat, AI diagnoses and fixes
- **Version restore** — if a change breaks the app, restore to any previous working version
- **No automated testing** — no browser-based testing (unlike Replit's App Testing) or unit test generation

### Pre-Publish Guardrails
- **No security scanning** before publish (unlike Lovable)
- **No build-time error checking** documented
- **No dependency auditing**
- Default security comes from the hosting layer (SSL, firewall, DDoS protection, malware scanning) rather than application-level scanning

---

# PHASE 4: GTM & Telco Partner Strategy

## Pricing Model

### Exact Pricing (as of March 2026)

| Tier | Monthly Price | Promo Price | AI Credits/Mo | Projects | Code Editor | Key Differentiators |
|------|--------------|-------------|---------------|----------|-------------|-------------------|
| **Explorer** | $9.99/mo | **$6.99/mo** | 30 | 1 | ❌ No | Text-only prompting. Basic support. 1 mailbox. No free domain. No credit top-ups. No collaboration. |
| **Starter** (Most Popular) | $19.99/mo | **$13.99/mo** | 70 | 25 | ❌ No | Image + voice prompting. Select-and-edit. Free chat mode. Analytics. Collaboration. Credit top-ups. Free domain (1yr). 2 mailboxes. Priority 24/7 support. E-commerce (subscriptions + products). |
| **Hobbyist** | $55.99/mo | **$39.99/mo** | 200 | 50 | ✅ Yes | Code editor access. Project duplication/templates. 5 mailboxes. |
| **Hustler** | $99.99/mo | **$79.99/mo** | 400 | 50 | ✅ Yes | Early access to new features. 5 mailboxes. |

**Annual commitment required** — promo prices shown are monthly rate for 12-month commitment:
- Explorer: $83.88/year ($6.99/mo × 12, with +2 months free)
- Starter: $167.88/year ($13.99/mo × 12, with +2 months free)
- Hobbyist: $479.88/year ($39.99/mo × 12, with +2 months free)
- Hustler: $959.88/year ($79.99/mo × 12, with +2 months free)

Renewal at full monthly price after first term.

### Hidden Costs & Limits
1. **Explorer is severely constrained** — text-only prompting, 1 project, no image/voice input, no select-and-edit, no analytics, no collaboration, no credit top-ups, no free domain, no code access. It's effectively a trial tier.
2. **Code access costs $39.99/mo minimum** — Hobbyist tier required for code editor. This means users on Explorer ($6.99) and Starter ($13.99) literally cannot see their source code, let alone download or export it.
3. **No code export documented** — even with the code editor, there's no "Download project" or "Push to GitHub" feature documented anywhere. Users can edit code in-browser but cannot take it with them.
4. **30-credit money-back limit** — the 30-day refund guarantee only applies if fewer than 30 credits have been used, which means heavy initial users may lose refund eligibility quickly.
5. **Annual lock-in** — promo pricing requires 12-month commitment. Monthly pricing not prominently shown.
6. **Prices exclude VAT** — stated at bottom of pricing page.

### Cost Comparison vs. Competitors

| Product | Entry Price | Credits | Hosting Included | Code Access | Domain Included |
|---------|-----------|---------|------------------|-------------|-----------------|
| **Horizons Explorer** | $6.99/mo | 30/mo | ✅ Yes (CDN, SSL, firewall) | ❌ No | ❌ No |
| **Horizons Starter** | $13.99/mo | 70/mo | ✅ Yes | ❌ No | ✅ Free 1yr |
| **Horizons Hobbyist** | $39.99/mo | 200/mo | ✅ Yes | ✅ Yes | ✅ Free 1yr |
| **Bolt Pro** | $20/mo | ~$20 tokens | ❌ No (Netlify/Vercel) | ✅ Yes | ❌ No |
| **Lovable Launch** | $50/mo | Higher | ⚠️ Lovable Cloud | ✅ Yes | ❌ No |
| **v0 Premium** | $20/mo | $20 credits | ❌ No (Vercel separate) | ✅ Yes | ❌ No |
| **Replit Core** | ~$20/mo | $20 credits | ✅ Yes (Autoscale) | ✅ Yes | ❌ No |

---

## B2B2C Channel Readiness

### White-Label Assessment: NOT READY
- **Heavily Hostinger-branded** — the builder, hosting, domains, email, and support are all branded as Hostinger
- **No embeddable builder widget** — cannot iframe or embed Horizons into a partner portal
- **No partner/reseller program documented** — Hostinger has an affiliate program but no white-label channel
- **No multi-tenant billing API** — cannot create sub-accounts programmatically
- **No custom branding options** — cannot replace Hostinger branding with partner branding
- **No API for project management** — cannot programmatically create, configure, or deploy projects

### What Hostinger DID Get Right for Channel
- **Bundled pricing model** — hosting + builder + domain + email in one price. This is the exact model HostPapa should pursue.
- **Built-in email** — mailboxes included, not a separate upsell. Reduces the number of products a reseller would need to bundle.
- **Infrastructure ownership** — deploying to owned infrastructure means margins are controlled, not dependent on AWS/GCP markup.

### HostPapa Opportunity
Hostinger proved the business model but built it as a direct-to-consumer product. HostPapa can build the SAME model (AI builder on owned infrastructure) but design it for channel from day one:
- White-label capable
- Multi-tenant billing via partner API
- Custom branding per reseller
- Programmatic project/domain/email provisioning

---

## Positioning & Persona

### Hero Copy
- "Launch a no-code site or web app with AI"
- "Prompt your app ideas to life"
- "Make a move with our biggest deals"

### Target Persona
**Primary: Non-technical entrepreneurs and small business owners** who want a web app (not just a website) but have zero coding ability. The pricing, messaging, and feature gates all target this audience. The Explorer tier at $6.99/mo is clearly aimed at absolute beginners with a single project idea.

**Secondary: Freelancers and agencies** building tools for clients. The Starter tier supports 25 projects with collaboration, analytics, and e-commerce — this is an agency-oriented package.

**Tertiary: Existing Hostinger customers** who already have hosting and want to add AI app building to their existing accounts. The Horizons blog launch post mentions their 3.5 million customers as the initial distribution.

---

# PHASE 5: Enterprise Compliance & Accessibility

## Security Posture

### Application-Level Security
- **No code security scanners** — unlike Lovable's 4-scanner suite, Horizons has zero application-level security scanning
- **No dependency auditing** — no npm/package vulnerability detection
- **No pre-publish security gate** — no blocking or warning before deploying vulnerable code
- **API key detection** — not documented. Users might paste API keys into chat without protection.

### Infrastructure-Level Security
- **Free SSL** on all projects (auto-provisioned)
- **Built-in firewall** — Hostinger's server-level firewall
- **DDoS protection** — infrastructure-level DDoS mitigation
- **Automated malware scanning** — server-level malware detection
- **99.9% uptime SLA** — contractual availability guarantee

### Data Privacy
- "We never use your prompts or project data to train the models" — clear data usage policy
- Data residency policies not documented per region
- GDPR compliance inherited from Hostinger (EU-based company, headquarters in Lithuania)

## Certifications

| Certification | Status | Details |
|--------------|--------|---------|
| SOC 2 Type II | ❌ Not documented | Hostinger has not published SOC 2 reports |
| ISO 27001 | ❌ Not documented | No ISO certification mentioned |
| GDPR | ✅ Compliant | Hostinger is EU-based (Lithuania), subject to GDPR. Privacy policy and cookie policy published. |
| HIPAA | ❌ Not applicable | No healthcare compliance features |
| PCI DSS | ⚠️ Via Stripe | Payment compliance delegated to Stripe integration |
| VPAT | ❌ Not published | No accessibility conformance report |

## Accessibility
- **No documented WCAG compliance** for generated output
- No accessibility testing tools in the builder
- Generated code accessibility depends on LLM output quality
- Builder UI accessibility not documented

---

# PHASE 6: Churn & Scalability Ceiling

## Code Ejection

### Can Users Leave?
- **Explorer/Starter tiers: NO** — code editor not available. Users cannot see, copy, or export their source code. Complete vendor lock-in.
- **Hobbyist/Hustler tiers: PARTIALLY** — code editor available, so users can view and manually copy code. But there's no "Download project" or GitHub export feature documented.
- **No GitHub integration** — no push-to-repo, no sync, no version control export
- **No CLI or API** — no programmatic way to extract projects

### Ejection Assessment
**Worst ejection story of any competitor reviewed.** Bolt and v0 have GitHub integration. Lovable auto-syncs to GitHub with meaningful commit messages. Replit has full git support plus downloads. Horizons has... nothing. Users who outgrow the platform must manually recreate their apps elsewhere.

### Lock-In Strategy
This is clearly deliberate. By withholding code access on cheaper tiers and offering no export on any tier, Hostinger ensures:
1. Users stay on Horizons for the lifetime of their project
2. Migration cost is prohibitively high (manual recreation)
3. Users upgrade to higher tiers to access their own code

For HostPapa, this is a cautionary model. Heavy lock-in drives short-term retention but generates resentment and negative word-of-mouth among technical users who discover they can't leave.

---

## The Logic Wall

| Complexity Level | Capability | Evidence |
|-----------------|-----------|----------|
| ✅ Works great | Landing pages, portfolios, business sites | Core use case. Every tutorial demonstrates this. |
| ✅ Works great | Simple interactive tools (quizzes, calculators, trackers) | Heavily promoted — invoice generators, finance tools, quiz builders, goal trackers |
| ✅ Works great | Forms with data persistence | "Add user accounts, logins, data storage" is a tier feature |
| ⚠️ Works with effort | CRUD applications with auth | Depends on Integrated Backend capabilities. Tutorials show it working but with iteration. |
| ⚠️ Works with effort | E-commerce (Stripe integration) | Available on Starter+. AI generates Stripe code but user must configure Stripe account and test payment flows. |
| ❌ Struggles | Complex business logic | No terminal, no package manager, limited debugging. AI can generate complex logic but fixing bugs in it requires the code editor (Hobbyist+ only). |
| ❌ Struggles | Real-time features (WebSocket, live updates) | No evidence of WebSocket or real-time capability in Integrated Backend |
| ❌ Fails | Multi-service architectures | No microservice support. No container orchestration. Single-app deployment model. |
| ❌ Fails | Non-web outputs (mobile apps, desktop, CLI) | Explicitly stated: "Hostinger Horizons AI app creator is designed for web-based app development, not native mobile apps." |
| ❌ Fails | Apps requiring specific backend languages | No Python, Go, Ruby, or Java backend support. JavaScript/TypeScript only. |

---

## Community Sentiment & Real-World Reception

### Testimonials (from official sources — independent reviews are sparse)

> "Vibe coding compresses development by 45%, with tools like Hostinger Horizons turning natural language into working prototypes in hours instead of weeks." — @markdiantonio

> "Hostinger Horizons may be the most efficient vibe code tool for building million dollar apps." — @web3wikis

> "Am wondering how Hostinger pulled off Horizons... It's crazy how fast it allows anyone deploy front and back end." — @rezmeram

> "It is like having a top-tier web developer/software developer ready to create whatever you can imagine." — @spillow82

### Independent Community Feedback
- **Reddit discussions are sparse** — Hostinger Horizons doesn't have the developer community presence that Bolt/Lovable have
- Found basic questions about migrating from Horizons to WordPress (suggesting some users outgrow it)
- Safety/lock-in concerns mentioned but not detailed in available threads
- The lack of critical developer discussion is itself a signal: Horizons targets non-developers who don't post on Reddit

### ⚠️ Caveat on Sentiment Data
Most available feedback comes from Hostinger's own blog and curated testimonials. Independent reviews and critical analysis of Horizons are significantly harder to find than for Bolt or Lovable. This limits confidence in the community sentiment assessment.

---

### Comparative Position
Horizons' Logic Wall is **lower than Bolt, Lovable, v0, and Replit** but **higher than traditional no-code builders** (Wix, Squarespace). It occupies a middle ground: more capable than drag-and-drop website builders, less capable than full-stack AI coding tools. This positioning makes sense given the target persona (non-technical users who need more than a website but less than a SaaS platform).

---

# EXHAUSTIVE FEATURE INDEX

## AI & Generation

| Feature | Description | Details |
|---------|-------------|---------|
| **Natural Language Prompting** | Describe your app in plain language | Core interaction model. Enter a text description of what you want to build — structure, features, design, content. AI generates complete frontend and backend code simultaneously. Supports 80+ languages for input, so non-English speakers can build in their native language. Available on all plans. |
| **Image-to-Code** | Upload visual media for AI replication | Drag and drop screenshots, wireframes, hand-drawn sketches, or polished designs into the chat. AI analyzes visual layout, element positioning, color palette, typography, and spacing, then generates a working web page matching the input. Works with rough sketches (pencil on paper) through pixel-perfect screenshots. Available on Starter ($13.99/mo) and above only — Explorer is text-only. |
| **Voice Prompting** | Speak your requirements | Audio input converted to text and processed as a prompt. Useful for rapid iteration when describing spatial/visual changes. Available on Starter and above. |
| **Select-and-Edit Mode** | Click elements to edit directly | Click any text or image element in the live preview to select it. Edit content directly in place — change text, swap images. These edits do NOT consume AI credits, making them unlimited. Changes generate proper code updates. Available on Starter and above. |
| **Free Chat Mode** | Non-credit AI guidance | A separate chat mode for asking questions, getting guidance, and learning — without using message credits. Ask "How should I structure my invoice app?" or "What features should a CRM have?" without depleting credits. Available on Starter and above. |
| **Auto Error Fixing** | Automatic bug detection and repair | When generated code has errors, Horizons detects them and offers automatic fixes. Reduces the manual debug-describe-fix loop. The user doesn't need to identify the bug — the system catches it. |
| **AI Credit System** | Message-based usage metering | Each AI interaction (prompt that generates or modifies code) consumes 1 credit from the monthly allocation. Explorer: 30/mo, Starter: 70/mo, Hobbyist: 200/mo, Hustler: 400/mo. Credits that don't consume: select-and-edit, free chat mode, publishing, previewing. Additional credits purchasable on Starter+ without tier upgrade. 30-day money-back guarantee limited to accounts with <30 credits used. |
| **Context Persistence** | Conversation memory within sessions | AI maintains full conversation history within each project session. Iterative refinements don't require re-explaining prior context. "Add a dark mode toggle" works because the AI knows the full project state. |
| **Suggested Follow-Ups** | AI-recommended next steps | After each generation, the AI suggests logical improvements: "Add user authentication," "Connect payment processing," "Improve mobile responsiveness." Reduces the "what should I do next?" decision fatigue for non-technical users. |

## Code Editor & File Management

| Feature | Description | Details |
|---------|-------------|---------|
| **Code Editor** | Full source code editing | View and edit all generated source files in a browser-based code editor. Syntax highlighting for HTML, CSS, JavaScript. Live preview sync — changes reflect immediately. Available on Hobbyist ($39.99/mo) and above ONLY. Explorer and Starter users cannot see or edit code. |
| **File Explorer** | Project file tree | Browse all project files in a hierarchical tree view. Create, rename, and delete files and folders. Standard file operations. Available only with code editor (Hobbyist+). |
| **Custom JavaScript** | Inject custom logic | Add custom JavaScript, integrate third-party APIs, or implement complex business logic by editing code directly. No restrictions on what code can be added. Requires code editor tier. |
| **Live Preview Sync** | Real-time preview updates | Code editor changes appear immediately in the preview panel. No manual rebuild or refresh required. Sub-second feedback loop for code tweaks. |

## Publishing & Hosting

| Feature | Description | Details |
|---------|-------------|---------|
| **One-Click Publish** | Instant production deployment | Click Publish to deploy your app to Hostinger's production infrastructure. No build configuration, no CI/CD pipeline, no deployment scripts. App is live immediately on a public URL. Republishing after changes is equally instant. |
| **Temporary Domain** | Free auto-generated URL | Every project gets a *.horizons.hostinger.com subdomain automatically. Immediately shareable after publishing. No configuration required. Available on all plans. |
| **Custom Domains** | Connect your own domain | Add a custom domain name to your published project. Auto-SSL certificate provisioning. Standard DNS configuration required for external domains. Free domain included for 1 year on Starter, Hobbyist, and Hustler plans. Explorer requires purchasing a domain separately. |
| **Hostinger CDN** | Global content delivery | Content served via Hostinger's in-house CDN across 4 continents (not Cloudflare/Fastly — Hostinger operates their own edge network). Static assets cached at edge locations for low-latency delivery worldwide. |
| **99.9% Uptime SLA** | Contractual availability guarantee | Hostinger guarantees 99.9% uptime for all published projects. This is a hosting-grade SLA that no standalone AI builder (Bolt, Lovable, v0) explicitly offers for user-generated apps. |
| **Auto SSL** | Automatic HTTPS | TLS certificates auto-provisioned and renewed for all published projects, including custom domains. No manual certificate management. Included on all plans. |
| **Built-in Firewall** | Server-level protection | Hostinger's firewall protects all published apps at the infrastructure level. No WAF configuration required by the user. |
| **DDoS Protection** | Distributed denial-of-service mitigation | Infrastructure-level DDoS protection included on all plans. No additional configuration or third-party service (like Cloudflare) needed. |
| **Malware Scanning** | Automated threat detection | Hostinger scans published apps for malware automatically. Server-level scanning — not application-level code analysis. |
| **Cloud Hosting Upgrade** | Scalability path | When a published app outgrows shared hosting (traffic spikes, performance needs), it can be upgraded to Hostinger's cloud hosting tier for up to 20x more speed, storage, and compute capacity. This provides a growth path without platform migration. |
| **Business Email** | Professional email included | Email mailboxes included with plans — Explorer: 1 mailbox per website for 1 year; Starter: 2 mailboxes; Hobbyist/Hustler: 5 mailboxes. Professional email on your custom domain without separate email hosting. |

## SEO & Discoverability

| Feature | Description | Details |
|---------|-------------|---------|
| **Auto sitemap.xml** | Search engine sitemap | Automatically generated when publishing to a custom domain. Provides search engines with a structured map of all pages on the site. No manual configuration required. |
| **Auto robots.txt** | Crawler directives | Automatically generated with sensible defaults for search engine crawling. Controls which pages search engines can and cannot index. |
| **Auto llms.txt** | AI discoverability file | Unique feature — automatically generates an llms.txt file that helps AI tools (ChatGPT, Claude, Gemini, Perplexity, Meta AI) understand and reference site content. This is a Generative Engine Optimization (GEO) feature that no other AI builder provides by default. |
| **SEO Metadata** | Title, description, OG tags | AI generates proper meta titles, descriptions, and Open Graph tags for social sharing. Included automatically with every published project. |
| **JavaScript-Rendered SEO** | SSR/pre-rendering for crawlers | Projects are optimized for indexing by search engines that support JavaScript-rendered content. Important because AI-generated apps are typically JavaScript-heavy single-page applications. |

## Backend & Data (Integrated Backend)

| Feature | Description | Details |
|---------|-------------|---------|
| **Integrated Backend** | Managed data persistence | Hostinger's proprietary backend service for user accounts, data storage, and dynamic content. No external BaaS setup required. The AI generates connection code automatically. Specific database engine not publicly documented. Appears to be a managed abstraction layer — users never configure databases directly. |
| **User Accounts & Auth** | Login and registration | AI generates complete authentication flows — signup, login, session management, protected routes. Powered by Integrated Backend. Implementation details (JWT, sessions, OAuth) not documented. |
| **Data Storage** | Persistent app data | Store and retrieve user-generated data (invoices, expenses, form submissions, etc.). AI generates CRUD operations automatically. Data persists across user sessions. |
| **Supabase Integration** | Optional advanced BaaS | For more advanced data needs, Horizons can integrate with external Supabase instances. AI generates Supabase client code, table schemas, and queries. This is optional — the Integrated Backend handles simpler use cases. |

## Integrations

| Feature | Description | Details |
|---------|-------------|---------|
| **Stripe** | Payment processing | AI generates Stripe integration code for one-time payments, subscriptions, and checkout flows. User provides Stripe API keys. Server-side payment handling via generated backend code. Available on Starter and above (e-commerce features). |
| **Google AdSense** | Ad monetization | AI integrates Google AdSense ad units into published apps. Generate passive revenue from app traffic. User provides AdSense account credentials. |
| **Custom API Integration** | Connect any external service | Describe the API you want to connect (e.g., "Add weather data from OpenWeatherMap API"), and the AI generates the integration code — API calls, authentication headers, data parsing, and UI display. Requires user to provide API keys/credentials. |
| **Hostinger Ecosystem** | Domain, email, hosting | Seamless integration with Hostinger's existing services: domain registration, email hosting, DNS management, cloud hosting upgrades. Single billing, single account, single dashboard for everything. |

## Collaboration & Management

| Feature | Description | Details |
|---------|-------------|---------|
| **Project Collaboration** | Team access to projects | Invite team members to work on projects together. Available on Starter ($13.99/mo) and above. Not available on Explorer. Details on permission levels (viewer, editor, admin) not documented. |
| **Built-in Analytics** | Visitor tracking | Track visitor traffic, page views, and engagement metrics for published projects. Built into the platform — no Google Analytics setup required. Available on Starter and above. |
| **Version History** | Project state timeline | Every project state is saved. Restore any previous version with one click. No git knowledge required — works like document version history. Available on all plans. Number of retained versions not documented. |
| **Project Duplication** | Clone projects as templates | Copy an existing project to use as a starting template for a new one. Useful for agencies building similar apps for multiple clients. Available on Hobbyist ($39.99/mo) and above. |

## Supported Output Types

| Output Type | Status | Details |
|-------------|--------|--------|
| **Websites** | ✅ Full support | Landing pages, portfolios, business sites, blogs. Core use case. |
| **Web Applications** | ✅ Full support | Interactive tools, dashboards, CRUD apps, calculators, trackers, quizzes. Primary differentiation from traditional website builders. |
| **E-Commerce** | ✅ Supported | Subscription products, digital goods, physical products via Stripe. Available on Starter+. |
| **Progressive Web Apps** | ❓ Not documented | No evidence of PWA manifest generation or service worker support. |
| **Native Mobile Apps** | ❌ Not supported | Explicitly stated: "designed for web-based app development, not native mobile apps." |
| **Desktop Apps** | ❌ Not supported | Web-only output. |
| **APIs/Backends** | ❌ Not standalone | Backend is generated as part of web app, not as a standalone API service. |

---

**Sources:**
- Hostinger Horizons homepage: https://www.hostinger.com/horizons
- Hostinger Horizons AI App Builder: https://www.hostinger.com/horizons/ai-app-builder
- Hostinger Horizons Features: https://www.hostinger.com/horizons/features
- Hostinger Horizons Features — Integrations: https://www.hostinger.com/horizons/features/integrations
- Hostinger Horizons Features — Advanced AI Models: https://www.hostinger.com/horizons/features/advanced-ai-models
- Hostinger Horizons Features — Code Editing: https://www.hostinger.com/horizons/features/code-editing
- Hostinger Horizons Features — No-Code Platform: https://www.hostinger.com/horizons/features/no-code-platform
- Hostinger Horizons Features — Design from Image: https://www.hostinger.com/horizons/features/design-from-image
- Hostinger Horizons Features — Built-in SEO: https://www.hostinger.com/horizons/features/built-in-seo
- Hostinger Horizons Features — Launch with One Click: https://www.hostinger.com/horizons/features/launch-with-one-click
- Hostinger Horizons Pricing (browser-rendered): https://www.hostinger.com/horizons/pricing
- Hostinger Horizons Blog Launch: https://www.hostinger.com/blog/hostinger-horizons-launch
- Tutorial — Create Invoice Generator: https://www.hostinger.com/tutorials/create-invoice-generator
- Tutorial — Create Personal Finance Web App: https://www.hostinger.com/tutorials/create-personal-finance-web-app
- Hostinger Financial Results 2025: https://www.hostinger.com/blog/financial-results-2025
- Horizons One Year Anniversary: https://www.hostinger.com/blog/horizons-turns-one
- Integrated Backend Launch: https://www.hostinger.com/blog/horizons-integrated-backend
- LLM Strategy: https://www.hostinger.com/blog/balancing-horizons-llms
- Product Updates 2026: https://www.hostinger.com/blog/product-updates-2026
