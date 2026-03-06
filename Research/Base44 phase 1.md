# Base44 — Phase 1: Product & UX Friction Audit

## Exhaustive UI Matrix

### AI Chat Interface
- **Default Mode:** Prompt → instant AI action on codebase. No confirmation step.
- **Discuss Mode:** Sandboxed conversation for planning/brainstorming. 0.3 credits per message. No code changes applied. User must switch out of Discuss mode and re-prompt to apply.
- **Visual Edit Mode:** Click any element in the live preview to adjust visuals. Toolbar appears with: Edit Element, Delete Element, Select Parent. AI receives the clicked element context and applies CSS/layout changes.
- **AI Controls Panel:**
  - Design Guidelines: Free-text instructions that persist across all prompts (e.g., "always use Inter font, blue primary color")
  - File Freeze: Lock specific files/pages so AI cannot modify them
  - Tone/persona settings for agent behavior
- **Suggested Next Steps:** AI auto-suggests contextual actions below each response (e.g., "Add user authentication", "Create a dashboard page")
- **Auto Model Selection:** Base44 automatically picks between Anthropic Claude, OpenAI GPT, and Google Gemini based on prompt complexity. User has zero control over model selection.
- **Credits Display:** Per-message credit cost visible via "More Actions" (⋯) menu on each message.

### Dashboard
- **Overview Panel:** App name, visibility toggle (Private/Workspace/Public), "Require login" checkbox, app URL, invite system
- **Data Section:**
  - Data entity list (tables)
  - Per-entity: view all rows, search bar, add/edit/delete records manually
  - Field types: Text, Number, Boolean (Yes/No), Date/Time, File, Reference (foreign key), Object (JSON)
  - Import/export: CSV import, CSV/JSON export, full data backup download
  - Sample data: AI can generate sample data for testing
- **Security Section:**
  - Per-entity visibility: Public vs Restricted
  - CRUD access rules per entity (Create, Read, Update, Delete)
  - Rule types: No restrictions, Only creator, Only with matching field, Custom condition
  - Warning icons on public entities
- **Agents Section:**
  - Enable/disable AI agents toggle
  - Agent management: name, description, instructions, persona
  - Tool permissions: which data entities the agent can read/write
  - WhatsApp integration: connect agent to WhatsApp Business
  - Each agent message costs 3 integration credits
- **Automations Section:**
  - Create scheduled (recurring or one-time) or data-triggered automations
  - Recurring options: daily/weekly/monthly, specific time, end date, max runs
  - Data triggers: on record create, update, or delete for specific entities
  - Each automation run costs 1 integration credit
  - 3-minute max execution time per run
  - 5-minute minimum interval between runs
  - Run logs with success/failure status
- **Integrations Section:**
  - Pre-built catalog: Stripe, Google Maps, OpenAI, WhatsApp, SendGrid, custom webhooks
  - Custom integrations being deprecated (March 1, 2026 cutoff)
  - Backend functions: Node.js serverless functions, activated per-app
  - Secrets management: Dashboard → Secrets for API keys
- **Settings Section:**
  - App name, description
  - SEO: meta title, meta description, social preview image (og:image)
  - Custom domain attachment
  - App visibility and login requirements
  - PWA configuration
  - Danger zone: delete app

### App Editor / Code View
- **File tree:** Multi-file React project structure visible
- **Code editor:** Syntax-highlighted, editable code
- **Live preview:** Real-time rendering of the app with hot reload
- **Version history:** Undo/redo with granular prompt-level rollback
- **NPM packages:** Can add npm packages to the project (new infrastructure only)
- **Responsive preview:** Desktop/tablet/mobile toggle

### Design System
- Colors, fonts, spacing configurable via AI chat or code
- shadcn/ui components available via NPM
- Tailwind CSS for styling
- Reference images: can paste/upload images for AI to replicate
- Theme system with dark/light mode support

---

## JTBD Friction Map

### Flow 1: Connecting a Custom Domain

**Note:** Base44 docs for custom domain setup returned 404 — feature exists but documentation is incomplete or restructured.

**Estimated flow based on dashboard structure:**
1. Open app editor (1 click)
2. Click Dashboard (1 click)
3. Click Settings (1 click)
4. Scroll to Custom Domain section
5. Enter domain name (1 action)
6. Copy CNAME record value provided by Base44
7. **COGNITIVE LEAP #1:** User must leave Base44, go to their DNS provider (GoDaddy, Cloudflare, etc.), and create a CNAME record pointing to Base44's servers. Non-technical SMBs often don't know what a CNAME is or where their DNS is managed.
8. Return to Base44 and click Verify (1 click)
9. Wait for DNS propagation (can take minutes to hours)

**Total clicks in Base44:** ~5
**Cognitive leaps:** 1 major (DNS configuration at external provider)
**Friction rating:** MEDIUM-HIGH — the DNS step is the universal friction point for all platforms. Base44 doesn't solve this; they just tell you to go do it elsewhere.

### Flow 2: Setting Up a Database Table

1. Open app editor (1 click)
2. Click Dashboard (1 click)
3. Click Data (1 click)
4. Click "New Entity" or equivalent (1 click)
5. Enter entity name (1 action)
6. Add fields: click "Add Field", choose type, name it (2 clicks + typing per field)
7. For 5 fields: ~15 actions total

**OR (the magic path):**
1. Type in AI chat: "Create a products table with name, price, description, image, and stock count" (1 action)
2. AI creates the entity with all fields automatically (0 actions)
3. Done.

**Total clicks via AI:** 1 prompt
**Total clicks manually:** ~20
**Cognitive leaps:** 0 via AI path; 1 via manual path (understanding field types, especially Reference/Object)
**Friction rating:** LOW via AI path — this is where Base44 shines

---

## The "Aha!" Moment

**The "Aha!" moment is the first prompt result.**

When a user types their first prompt — something like "Build me a CRM for photographers with client galleries and booking" — and sees a fully functional, multi-page, styled application appear in the preview within 10-30 seconds, complete with working data tables, navigation, and responsive design.

The specific interaction chain:
1. User types a natural language description of what they want
2. The AI chat shows it's "building" with a progress indicator
3. The preview panel on the right renders a complete, clickable, styled application
4. The user realizes they can click through the app, see pages, see forms that actually save data

**The second "Aha!" is data persistence.** The user fills out a form in the preview, sees the data appear in the Data dashboard, and realizes this isn't a mockup — it's a real application with a real database.

**The third "Aha!" is one-click publish.** User clicks Publish and gets a live URL (appname.base44-apps.com) that they can share with anyone immediately. The transition from "I typed a sentence" to "I have a live website" is the core product magic.

**Key UX weakness:** The "Aha!" requires trust. Users must type a non-trivial prompt to see the magic. If they type something too simple ("make a website"), the result is underwhelming. Base44 mitigates this with example prompts and a prompt guide, but the initial prompt quality determines first impression quality.
