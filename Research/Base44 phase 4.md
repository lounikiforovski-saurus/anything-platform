# Base44 — Phase 4: GTM & Telco Partner Strategy

## Pricing Model

### Tier Structure
| Tier | Price | Credits/Month | Key Limits |
|------|-------|---------------|------------|
| **Free** | $0 | 50 credits | 1 app, limited features |
| **Basic** | $29/month | 500 credits | Multiple apps, custom domains |
| **Pro** | $49/month | 2,000 credits | Priority support, advanced features |
| **Enterprise** | Custom | Custom | SSO, dedicated support, SLA |

### Credit Economy (The Hidden Cost Structure)
- **Standard AI message:** ~1 credit
- **Discuss mode message:** 0.3 credits
- **AI Agent message:** 3 integration credits
- **Automation run:** 1 integration credit
- **Backend function execution:** credits consumed per invocation
- Integration credits are separate from AI build credits on some tiers

### Hidden Limits & Traps
1. **Credit burn rate is unpredictable** — a complex prompt might consume multiple credits; a simple one uses ~1. Users can't predict costs.
2. **AI Agent messages cost 3x** — WhatsApp bot integrations burn credits fast for customer-facing use
3. **Automation limits:** 3-minute max execution time, 5-minute minimum interval between runs, per-run credit cost
4. **Database row limits** — not publicly documented but likely exist on lower tiers
5. **No credit rollover** — unused credits expire monthly (standard SaaS trap)
6. **Custom integrations being deprecated** (March 1, 2026 cutoff) — users who built on custom integrations must migrate to Backend Functions
7. **Backend Functions** require "new infrastructure" migration — existing apps must update

### Revenue Model Analysis
- Base44 monetizes primarily through **AI credit consumption**, not hosting
- This aligns their revenue with user engagement (more prompts = more revenue)
- But creates a perverse incentive: the less efficient the AI is (more attempts needed), the more Base44 earns
- Enterprise tier likely includes volume credit discounts and SLA guarantees

---

## B2B2C Channel Readiness

### Partner Program
- Base44 has an **Enterprise tier** with custom pricing
- Documentation mentions:
  - SSO integration (SAML/OIDC implied)
  - Dedicated support
  - Custom SLA
  - Team/workspace management
- **However, there is NO documented Channel Partner program:**
  - No /partners page
  - No reseller program
  - No white-label capabilities mentioned
  - No multi-tenant billing architecture documented
  - No API for programmatic app creation or management

### White-Labeling Assessment
- **NOT white-label ready:**
  - All apps deploy to `*.base44-apps.com` subdomains (custom domains available but Base44 branding persists in the builder UI)
  - No way to reskin the builder UI
  - No way to replace Base44 branding with a partner's brand
  - No embeddable builder widget
  - Login/auth shows Base44 branding

### Multi-Tenant Billing
- **Workspace feature** exists for team collaboration
- Each workspace can have multiple apps
- But no documented:
  - Sub-billing (partner bills their customers)
  - Usage attribution per sub-tenant
  - API-level billing controls
  - Metered billing hooks

### API & Integration Handoff
- No public management API documented
- No programmatic app creation endpoint
- No webhook for app lifecycle events (created, published, deleted)
- Integrations are app-level, not platform-level

### Key Finding for HostPapa
**Base44 is NOT B2B2C ready.** They have zero channel partner infrastructure. No white-labeling, no multi-tenant billing, no management APIs. This is HostPapa's biggest opportunity — building a white-label AI builder that Telco partners can reskin and resell with their own branding, billing, and customer management. Base44 is a direct-to-consumer product trying to serve enterprise; HostPapa can build enterprise-first.

---

## Positioning & Persona

### Homepage Hero Copy
- **H1:** "Build Apps with AI" (or variant — their homepage copy evolves)
- **H2/Subhead:** "Describe your app and Base44 will build it for you. No coding required."
- **CTA:** "Start Building" / "Try Free"

### Target Persona
**Primary: Non-technical SMB owner / entrepreneur**
- Wants a custom internal tool or customer-facing app
- Cannot code or afford a developer
- Has a clear idea of what they want but can't execute
- Values speed over customization
- Willing to accept AI-generated code quality

**Secondary: Technical founder / solo developer**
- Uses Base44 for rapid prototyping
- May eventually outgrow the platform and eject code
- Values speed of iteration over code quality
- Uses Base44 to validate ideas before investing in proper development

### Competitive Positioning
- Base44 positions against: hiring a developer, using no-code tools (Bubble, Webflow), building from scratch
- They do NOT position against Lovable, Bolt.new, or Hostinger Horizons directly
- Key messaging: "AI builds it for you" (passive — you describe, it creates)
- Differentiation claim: full-stack apps with real databases (vs. competitors that only do frontend)

### Marketing Channels (Observed)
- SEO / content marketing (documentation doubles as content)
- Product Hunt presence
- Social media (Twitter/X, LinkedIn)
- No visible paid advertising campaigns
- Community/word-of-mouth driven
- YouTube tutorials and demos
