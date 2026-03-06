# AI Anything Builder - MVP Plan

## Executive Overview

### Solution Overview

**Example design reference:**

![Home page design](assets/home-page-design.png)

The **AI Anything Builder** is a self-hosted platform — a **portal** that orchestrates standalone apps, shared services, and AI-powered creation tools.

The portal itself provides:
- **Home page** — hero section with prominent AI chat input ("Describe what you want to build...") supporting text, image upload, and voice. This is the primary entry point for all creation. Below: "Explore Apps" catalogue of pre-built apps with category filter chips + rich preview cards. Below that: "My Apps" showing the user's creations with status, context menu (Rename, Duplicate, Delete), and a "Create New App" card
- **Left sidebar navigation** — Home, Search, New Chat, My Apps, Templates
- **Identity & SSO** — centralised auth across all apps, org/employee management
- **Site Engine** — block-based visual website editor with AI chat, image generation, vision bootstrap, and publishing. Not a user-facing app — it's the invisible engine that powers every site
- **App Engine** — create custom applications through AI chat. A multi-stage pipeline analyses requirements, generates React + Vite code, validates it, and iterates until the build succeeds
- **Connector ecosystem** (Identity, AI, Mail, PDF, Storage) — shared capabilities that any app can plug into

The platform ships with **3 pre-built apps**, each a standalone service with its own database:

| App | What It Does |
|-----|-------------|
| **Appointment Scheduler** | Availability management, public booking page, automated email reminders |
| **Invoicing** | Invoice creation, payment tracking, automated late payment chasing with escalating reminders |
| **Proposals & Contracts** | Section-based document editor, e-signatures, recipient tracking, version history |

Users can also create **custom apps** via the App Engine, and **sites** via the Site Engine. Both appear as first-class items in "My Apps".

**The Site Engine is an optional wrapper.** When a user creates or uses an app (Scheduler, custom app, etc.), it runs standalone — no site needed. But when the user wants static content around the app (hero section, about text, testimonials, footer), the Site Engine wraps it: the app is embedded as an `app` block within a site's block tree, and the user can build content around it. A user building a salon website can say "add a booking widget" and the AI embeds their Scheduler directly into the site. If no matching app exists, the AI creates a custom app on-the-fly and embeds it.

The AI chat decides what to create based on the user's request:

```
User request
  ├── Pure functionality ("booking system", "invoice tracker")
  │   └── Create/assign standalone app — no site, no wrapper
  ├── Pure content ("website for my salon", "portfolio page")
  │   └── Create site via Site Engine — no app embeds
  ├── Content + functionality ("salon website with online booking")
  │   └── Create site + embed app(s) inside it
  └── Add content to existing app ("add a landing page to my scheduler")
      └── Create site wrapping the existing app
```

### Tech Stack

| Layer | Technology |
|-------|-----------|
| **Language** | Vanilla JavaScript (ES modules) |
| **Backend** | Fastify 5 |
| **Databases** | MariaDB 10.6 (portal + pre-built apps), SQLite (custom apps) |
| **Query Builder** | Knex.js |
| **Frontend** | React 19, Vite |
| **State Management** | Zustand |
| **CSS** | Tailwind CSS 4 |
| **AI** | Claude API (primary), OpenAI (fallback), Flux 2 Pro (image generation) |
| **Email** | Nodemailer + MJML templates |
| **PDF** | @react-pdf/renderer |
| **Testing** | Vitest (unit), Playwright (E2E) |
| **Infrastructure** | Kubernetes (existing cluster, Helm charts), Docker, Nginx Ingress |
| **Local Dev** | docker-compose (databases), npm workspaces (monorepo) |

### Architecture

- **Monorepo** with npm workspaces: shared packages, portal, connector services, and apps
- **Microservices**: 9+ containers (1 portal with Site Engine + App Engine + 1 identity service + 3 connector services + 3 pre-built apps + N custom apps + N sites)
- **Path-based routing**: Single domain, Nginx/Ingress routes to each service by path (`/apps/scheduler/`, `/apps/custom/<id>/`, `/sites/`, etc.)
- **Shared auth**: Identity service issues JWTs, all services validate with shared secret. SSO across all apps (same-origin, localStorage)
- **Connector SDK** (`@ab/connectors`): Thin HTTP clients that any app imports to use mail, PDF, AI, storage, and auth without direct dependencies
- **Multi-tenancy**: Pre-built apps are shared instances with data isolated by `user_id`/`org_id` from JWT
- **App Foundation** (`@ab/app-foundation`): Server + client factory that every app is built on, ensuring consistency

### Team & Timeline

| | Detail |
|---|--------|
| **Team** | 4 developers (2 backend, 2 frontend), each managing 3 AI coding agents |
| **Effective capacity** | 4 devs x 3 agents = 12 parallel work streams |
| **Duration** | 10 weeks (5 sprints) |
| **Sprint 0** (Week 1) | Foundation: monorepo, shared packages, portal, identity service, AI service, Nginx routing |
| **Sprint 1** (Weeks 2-3) | Site Engine + Scheduler + Mail connector |
| **Sprint 2** (Weeks 4-5) | Invoicing + Proposals + PDF connector |
| **Sprint 3** (Weeks 6-7) | Custom app system + app embedding in sites + publishing pipeline + polish |
| **Sprint 4** (Weeks 8-9) | Integration testing, E2E, production Helm charts, onboarding |

### Deliverables

**MVP delivers:**
1. Portal with home page (hero AI input, Explore Apps catalogue, My Apps grid), left sidebar navigation, org/employee management, app registry
2. **Site Engine** (portal capability) — block-based visual editor with AI chat, image generation, vision bootstrap, publishing, app embedding
3. **App Engine** (portal capability) — AI-powered custom app creation with multi-stage pipeline
4. Centralised identity service (auth, SSO, employee provisioning)
5. **Scheduler** — public booking, availability management, reminders, calendar views
6. **Invoicing** — payment tracking, auto-chasing, PDF generation, recurring invoices
7. **Proposals** — e-signatures, version history, templates, engagement tracking
8. Connector services: AI (Claude + OpenAI), Mail, PDF, Storage
9. K8s deployment (Helm charts for existing cluster)
10. Playwright E2E test suite, Lighthouse accessibility score >90

**Explicitly out of scope:**
- Cross-origin SSO, asymmetric JWT (JWKS)
- Social login (Google, Microsoft), MFA
- App marketplace
- Custom domain publishing
- Real-time features beyond AI chat
- Horizontal scaling / read replicas

### Key Risks

| Risk | Mitigation |
|------|-----------|
| Multi-service complexity (10+ containers) | docker-compose.dev.yml for local dev; K8s handles production |
| AI quality may not match tuned prototypes | Study prototype prompts closely; test with real APIs early; provider-agnostic design allows switching |
| App Engine (custom app system) is ambitious | Core product differentiator — prioritise early, test with real AI APIs from Sprint 1 |
| Multi-tenant data leaks | Every query filtered by `user_id` in Fastify preHandler; integration tests verify isolation |
| 12 parallel agents = merge risk | Each service is a separate directory; `@ab/shared` changes serialised; API contracts frozen per sprint |

### Reference Documents

| Document | Contents |
|----------|---------|
| [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) | Site Engine: block schema, AI integration patterns, editor architecture, publishing pipeline |
| [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) | App Engine: multi-stage AI pipeline, validation loop, app template, conversation management |
| [REFERENCE-AI-MIGRATION.md](REFERENCE-AI-MIGRATION.md) | Claude/OpenAI provider-agnostic design, model mapping, conversation continuity |
| [REFERENCE-APP-SCHEDULER.md](REFERENCE-APP-SCHEDULER.md) | Scheduler DB schema, slot calculation, booking API, reminder system |
| [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) | Invoice DB schema, status machine, chase reminders, PDF generation |
| [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) | Proposals DB schema, e-signature flow, version history, templates |

---

## Context

Building a new **AI Anything Builder** platform from scratch - a portal that connects standalone apps (pre-built and custom). Two existing prototypes serve as **reference only**:

1. **Site Builder prototype** (`ai-site-builder/master`) - Informs the portal's **Site Engine**: visual website editor with AI-powered block editing, vision-based site bootstrap, Flux image generation. See **[REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md)** for full pattern documentation.

2. **Anything Builder prototype** (`ai-site-builder/demo-build-me-anything-app`) - Informs the portal's **App Engine**: AI-powered app creation with multi-stage pipeline (analysis → planning → design → code generation → validation → refinement), conversational intent classification, progressive refinement with rollback. See **[REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md)** for full pattern documentation.

Prototype code is NOT refactored or ported - it is studied to replicate and improve upon these patterns in a clean new codebase. Prototypes used OpenAI; the new build uses **Claude (Anthropic) API as the primary AI provider** with OpenAI kept as a fallback. See **[REFERENCE-AI-MIGRATION.md](REFERENCE-AI-MIGRATION.md)** for the migration guide.

### Key Architectural Principle

The Anything Builder is a **portal** - not a monolith. Every app (pre-built or custom) is a **standalone codebase** with its own database, running in its own container. The portal orchestrates identity, connectivity, and discovery.

### Two Portal Creation Capabilities

The portal has two AI-powered creation tools. Neither is a user-facing app — they are invisible engines behind what users see in their dashboard.

**1. Site Engine** (portal capability, from Site Builder prototype)
- Block-based visual editor with AI chat, image generation, vision bootstrap
- JSON block tree rendered as HTML/CSS. Editor with inline editing, drag-and-drop
- Published to path slugs or custom domains
- User sees the **output** ("My Salon Website") in their My Apps — not "Site Builder"
- Architecturally: own backend service + own DB (sites, blocks) + editor frontend, but accessed **through the portal**, not as a standalone app in the app list

**2. App Engine** (portal capability, from Anything Builder prototype)
- Creates standalone applications via **multi-stage AI pipeline** (analysis → code gen → validation)
- Produces full React + Vite apps with their own DB, routes, logic
- Published as K8s pods
- User sees the **output** ("My Calculator") in their My Apps — not "App Engine"

### Site Engine as Optional App Wrapper

The Site Engine serves as an **optional composition layer** around apps. An app runs standalone by default. The Site Engine wraps it only when the user wants static content around the app.

**Three scenarios**:

1. **Standalone app** — User creates/uses an app (Scheduler, custom, etc.). It runs at its own path. No site, no wrapper. My Apps shows e.g. "My Booking System"
2. **Pure site** — User wants a website with static content (landing page, portfolio). Site Engine creates a block tree with text, images, containers. No `app` blocks. My Apps shows e.g. "Acme Salon Website"
3. **Site wrapping app(s)** — User wants content around an app. Site Engine provides the page structure, the app is embedded as an `app` block (iframe) within it. My Apps shows e.g. "Acme Salon Website" (which contains the booking functionality inside)

When a user editing a site requests a feature beyond static content (e.g. "add a booking widget", "I need a calculator"):

1. **Existing app** — The user already has a pre-built or custom app. The AI embeds it as an `app` block
2. **New app on-the-fly** — No matching app exists. The AI triggers the App Engine to create one, then embeds it

Apps are embedded via **iframe** (`app` block kind in the block schema). Each embedded app runs at its own path and is isolated from the site's DOM. The site provides the layout/positioning (wrapper styles), the app fills the inner space.

```
Site "Acme Salon Website" (block tree)
  ├── container (hero section)
  │   ├── text (heading)
  │   └── image (hero image)
  ├── container (content)
  │   └── text (paragraphs)
  ├── app (embedded Scheduler booking widget)   ← iframe to /apps/scheduler/public/book/:slug
  ├── app (embedded custom calculator)          ← iframe to /apps/custom/calc-123/
  └── container (footer)
```

A single published site can contain multiple embedded apps, each running independently.

### Home Page Layout

The portal home page has three sections, top to bottom:

**1. Hero + AI Input** (top, most prominent)
- Large hero area with tagline ("Create. Iterate. Launch." / "Turn ideas into fully functional apps.")
- Prominent AI chat input bar: "Describe what you want to build..."
- Input supports multimodal: text, image upload (vision bootstrap), voice
- Submitting starts a New Chat session where the AI classifies intent and builds the appropriate output (site, app, or site wrapping app)
- This is the **primary entry point** for all creation — not buried in a sub-page

**2. Explore Apps** (middle section)
- Category filter chips: All, Appointment Scheduler, Smart Invoice, Proposals & Sign, etc.
- Rich preview cards in a responsive grid (3 columns desktop, 2 tablet, 1 mobile)
- Each card shows: preview image, app name, short description
- Clicking a card opens a detail view with **Use** (activate as-is, add to My Apps) or **Fork** (clone as custom app starting point, extend via AI chat)

| Card | Description | Actions |
|------|------------|---------|
| Appointment Scheduler | Accept bookings and sync calendars automatically | **Use** / **Fork** |
| Smart Invoice | Automated invoicing system | **Use** / **Fork** |
| Proposals & Sign | Draft, send, and e-sign professional documents instantly | **Use** / **Fork** |

**3. My Apps** (bottom section)
- First card is always **"Create New App"** (+ icon) — opens the App Engine chat
- Remaining cards show user's creations and activated apps:

| Item Type | Example | Shows | Context Menu |
|-----------|---------|-------|-------------|
| **Site** | "Acme Salon Website" | Preview image + "Published"/"Draft" status | Rename, Duplicate, Delete |
| **Activated pre-built app** | "My Booking System" | Preview image + "Published"/"Unpublished" status | Rename, Duplicate, Delete |
| **Custom app** | "Mortgage Calculator" | Preview image + "Published"/"Unpublished" status | Rename, Duplicate, Delete |

**Left Sidebar Navigation:**

| Icon | Label | Destination |
|------|-------|------------|
| Home | Home | Home page (hero + explore + my apps) |
| Search | Search | Search across all apps, sites, and templates |
| New Chat | New Chat | Opens a fresh AI chat session (same as hero input) |
| My Apps | My Apps | Filtered view showing only the user's apps |
| Templates | Templates | Browse app templates and starting points |

**Top-right:** Usage counter (e.g. "5/5" apps on current plan) + user avatar (profile, settings, logout)

**Footer:** Help, Catalog, Pricing, Settings + social links

The user never sees "Site Engine" or "App Engine" as items — they see their creations. Pre-built apps move from "Explore Apps" to "My Apps" when activated. Forking creates a custom app seeded with the pre-built app's functionality, which the user can then modify via AI chat.

### The 3 Pre-built Apps (MVP)
1. **Appointment Scheduler** - multi-tenant, shared instance, own DB
2. **Invoicing & Late Payment Chasing** - multi-tenant, shared instance, own DB
3. **Proposals & Contracts** - multi-tenant, shared instance, own DB

### Custom Apps
- Created via the portal's App Engine (standalone) or triggered from within a site editing session (then embedded)
- Private to the creating user
- Lightweight DB (SQLite)
- Run as standalone containers

### Key Requirements
- On-prem K8s deployment (existing cluster with Helm charts and monitoring)
- Centralized OAuth (business owners create employee accounts, SSO across all apps)
- Secure, Accessible (WCAG AA), Mobile responsive
- Publish to path slugs (`platform.domain.com/apps/scheduler/`)

---

## Architecture Overview

```
                           NGINX REVERSE PROXY
                      (path-based routing, single domain)
                                |
        ┌───────────────────────┼──────────────────────┐
        |                       |                       |
   /                      /apps/scheduler/        /apps/custom/<id>/
   /api/                  /apps/invoicing/
   /sites/                /apps/proposals/
        |                       |                       |
   ┌────v──────────┐      ┌────v─────┐           ┌─────v─────┐
   │    PORTAL     │      │SCHEDULER │           │ CUSTOM    │
   │               │      │          │           │ APP N     │
   │  Dashboard    │      │ MariaDB  │           │           │
   │  App Engine   │      │ (appts,  │           │ SQLite    │
   │  Site Engine ─┼──┐   │  slots)  │           │           │
   │               │  │   └──────────┘           └───────────┘
   │  MariaDB      │  │
   │  (apps,       │  │   MariaDB (sites, blocks)
   │   config)     │  └──>  Site Engine DB
   └────┬──────────┘
        │
   ┌────v────┐          ┌─────────┐
   │IDENTITY │          │   AI    │
   │ SERVICE │          │ SERVICE │  (stateless, shared)
   │         │          │         │
   │ MariaDB │          │ Claude  │
   │ (realms,│          │ Flux    │
   │  users, │          └─────────┘
   │  tokens)│
   └─────────┘
```

**All services share one domain via Nginx path routing** - this eliminates cross-origin SSO complexity. JWT tokens work across all apps via shared signing keys managed by the Identity service.

**Key separation**: The Portal orchestrates everything (registry, lifecycle, discovery, Site Engine, App Engine). The **Site Engine** is a portal capability — its own service but accessed through the portal, not as a user-facing app. The **Identity connector** is a standalone service that manages all user authentication across every context - platform users AND app end-users. Apps built with the Anything Builder have their own branded auth; their end-users never see or know about the portal.

---

## Two Stacks

### Stack 1: Anything Builder (Portal + Site Engine + App Engine)

The portal is the orchestrator - it has its own stack optimized for app registry, connector configuration, app lifecycle, and the two creation engines (Site Engine + App Engine). It does NOT manage users directly - that's the Identity connector's job. The Site Engine is a separate service (own backend + DB) but is architecturally part of the portal — accessed through the portal dashboard, not as a standalone app. The App Engine runs within the portal process (no separate service).

| Layer | Choice | Why |
|-------|--------|-----|
| **Language** | Vanilla JavaScript (ES modules) | Highest AI training coverage of any JS variant — no TS compilation step, no type gymnastics, AI generates correct code on first pass more reliably |
| **Backend** | **Fastify 5** | JSON Schema validation built-in (AI generates schemas accurately), plugin architecture maps cleanly to microservice boundaries, fastest Node framework |
| **Database** | **MariaDB 10.6** | Battle-tested relational model for user/org/app data, excellent AI coverage for query generation, strong multi-tenant isolation primitives (row-level filtering, indexed scoping) |
| **Query Builder** | **Knex.js** | Works natively with vanilla JS (no ORM decorators), migrations built-in, composable query chains AI can reason about without abstraction layers |
| **Frontend** | **React 19** + **Vite** | React has the largest AI training corpus of any UI library — AI generates idiomatic components with high accuracy. Vite: instant HMR critical for AI edit → preview loop in App Engine |
| **State** | **Zustand** | 1KB, minimal API surface AI can learn completely, persistence middleware for offline/undo, no boilerplate (unlike Redux) |
| **CSS** | **Tailwind CSS 4** | Utility classes AI generates with near-perfect accuracy (no custom CSS naming decisions), design constraints prevent AI from producing inconsistent styling |
| **Testing** | **Vitest** + **Playwright** | Vite-native (shared config), Playwright for cross-browser E2E — both well-represented in AI training data |
| **Routing** | **Nginx** | Proven reverse proxy for path-based microservice routing, single domain eliminates CORS/cookie complexity |
| **Containers** | **Docker** + **Kubernetes** (Helm charts, existing cluster) | On-prem infrastructure with production-grade orchestration, health checks, auto-restart, horizontal scaling |

### Stack 2: App Foundation Template

All apps (pre-built AND custom) are built on the **same foundation template**. Pre-built apps are just the template with more pre-built code. This ensures consistency and connector compatibility.

| Layer | Choice | Why |
|-------|--------|-----|
| **Language** | Vanilla JavaScript (ES modules) | Same as portal — one language everywhere means AI agents context-switch less, generate more consistent code |
| **Backend** | **Fastify 5** | Same as portal — AI agents reuse identical patterns (plugin registration, route schemas, hooks) across all services |
| **Database (pre-built)** | **MariaDB 10.6** | Multi-tenant shared apps need robust relational DB with strong indexing for `user_id`/`org_id` scoped queries |
| **Database (custom)** | **SQLite via better-sqlite3** | Single-file, JSON functions, zero config — perfect for AI-generated apps (no DB provisioning step, synchronous API AI can use without async pitfalls) |
| **Query Builder** | **Knex.js** | One query pattern everywhere — AI-generated app code uses the same Knex API as portal and pre-built apps |
| **Frontend** | **React 19** + **Vite** | Same as portal — AI generates components using identical patterns. Vite HMR enables live preview during AI generation |
| **State** | **Zustand** | Same as portal — minimal API means AI-generated state code is correct without complex setup |
| **CSS** | **Tailwind CSS 4** | Same as portal — AI generates utility classes consistently, no risk of style conflicts between embedded apps and sites |
| **Connector SDK** | **`@ab/connectors`** | Standard interface to call connectors (identity, mail, PDF, AI, storage) — AI-generated apps get capabilities without dependency management |

---

## Connectors

Connectors are **shared, verified modules** that any app can plug into. Instead of each app implementing its own mailer, PDF generator, etc., apps call connectors through a standard SDK (`@ab/connectors`).

### Connector Architecture

```
App (any) ──> @ab/connectors SDK ──> Connector Service (HTTP)
```

Each connector is either:
- A **standalone service** (own container, own API) for heavy/stateful work
- A **shared library** (npm package) for lightweight/stateless utilities

### Connector Inventory

| Connector | Type | Container | Port | Description |
|-----------|------|-----------|------|-------------|
| **Identity** | Service | `identity-service` | 4008 | User management, auth, JWT issuance, realms. Own MariaDB. Apps call via `@ab/connectors/identity` |
| **AI** | Service | `ai-service` | 4005 | Claude (Anthropic) chat/vision + Flux image gen. Provider-agnostic, OpenAI kept as fallback. Apps call via `@ab/connectors/ai` |
| **Mail** | Service | `mail-service` | 4006 | Nodemailer + MJML templates. Apps send emails via `@ab/connectors/mail` |
| **PDF** | Service | `pdf-service` | 4007 | @react-pdf/renderer. Apps request PDFs via `@ab/connectors/pdf` |
| **Storage** | Service | `portal` (built-in) | 4000 | File uploads, asset management. Apps store/retrieve files via `@ab/connectors/storage` |

### How Apps Use Connectors

```javascript
// In any app's service code:
import { identity, mail, pdf, ai, storage } from '@ab/connectors';

// Create a user in the app's realm (end-user who never sees the portal)
await identity.createUser({
  realm: 'scheduler-org7',
  email: 'client@example.com',
  password: 'their-password',
  displayName: 'Jane Doe',
  role: 'client'
});

// Authenticate an app end-user
const { accessToken, refreshToken } = await identity.login({
  realm: 'scheduler-org7',
  email: 'client@example.com',
  password: 'their-password'
});

// Validate a token (works for both platform users and app end-users)
const user = await identity.verify(token);
// -> { userId, email, realm, role, orgId, ... }

// Send an email
await mail.send({
  to: 'client@example.com',
  template: 'booking-confirmation',
  data: { clientName: 'Jane', date: '2026-03-15', time: '10:00 AM' }
});

// Generate a PDF
const pdfBuffer = await pdf.generate({
  template: 'invoice',
  data: { invoiceNumber: 'INV-001', lineItems: [...], total: 500 }
});

// AI structured output (provider-agnostic - uses Claude or OpenAI behind the scenes)
const result = await ai.json({
  system: 'You are modifying a scheduling app config...',
  prompt: 'Add a cancellation policy',
  schema: { type: 'object', properties: { policy: { type: 'string' } }, required: ['policy'] },
});

// AI code generation (with extended thinking on Claude)
const code = await ai.code({
  system: codeGenPrompt,
  prompt: 'Build a booking calendar component',
  thinking: true,
});

// Upload a file
const url = await storage.upload({ file: buffer, filename: 'logo.png', appId: 'scheduler' });
```

### `@ab/connectors` SDK (shared package)

Each connector client is a thin HTTP wrapper that:
1. Reads the connector service URL from environment (e.g., `MAIL_SERVICE_URL`)
2. Attaches the app's identity (JWT or service token)
3. Makes the HTTP call to the connector service
4. Returns the result or throws a standardized error

This means apps never need to `npm install nodemailer` or `@react-pdf/renderer` - they just use `@ab/connectors/mail` and `@ab/connectors/pdf`.

---

## Service Inventory

### Portal + Site Engine + App Engine

| Service | Container | Port | Database | Description |
|---------|-----------|------|----------|-------------|
| **Portal** | `portal` | 4000 | `ab_portal` (MariaDB) | Dashboard, app registry, app lifecycle, Storage connector |
| **Site Engine** | `site-engine` | 4001 | `ab_sites` (MariaDB) | Block-based site editor, AI chat, publishing. Accessed through portal, not user-facing as a standalone app |
| **App Engine** | `portal` (built-in) | 4000 | `ab_portal` (MariaDB) | Custom app creation via multi-stage AI pipeline. Runs within the portal process, uses App Foundation template |

### Connector Services

| Service | Container | Port | Database | Description |
|---------|-----------|------|----------|-------------|
| **Identity Connector** | `identity-service` | 4008 | `ab_identity` (MariaDB) | Realms, users, auth, JWT issuance, password management |
| **AI Connector** | `ai-service` | 4005 | None (stateless) | Claude (primary) + OpenAI (fallback) + Flux proxy. See [REFERENCE-AI-MIGRATION.md](REFERENCE-AI-MIGRATION.md) |
| **Mail Connector** | `mail-service` | 4006 | None (stateless) | Nodemailer + MJML, email queue |
| **PDF Connector** | `pdf-service` | 4007 | None (stateless) | @react-pdf/renderer |

### Pre-built Apps (all built on App Foundation Template)

| App | Container | Port | Database | Description |
|-----|-----------|------|----------|-------------|
| **Scheduler** | `scheduler` | 4002 | `ab_scheduler` (MariaDB) | Appointment booking, multi-tenant |
| **Invoicing** | `invoicing` | 4003 | `ab_invoicing` (MariaDB) | Invoice management, payment chasing, multi-tenant |
| **Proposals** | `proposals` | 4004 | `ab_proposals` (MariaDB) | Document builder, e-signature, multi-tenant |
| **Custom App N** | `custom-<id>` | 5000+ | SQLite file | User-created apps from template |

---

## Monorepo Structure

```
anything-builder/
  package.json                         # npm workspaces root
  docker-compose.dev.yml               # Dev: databases + services locally
  helm/                                # Helm charts for K8s deployment
  nginx/
    nginx.conf                         # Path-based reverse proxy

  # ─── SHARED PACKAGES ───────────────────────────────────────
  packages/
    shared/                            # @ab/shared - low-level utils for ALL code
      jwt.js                           # JWT decode/extract (no verify - that's Identity's job)
      http.js                          # Request/response helpers
      helpers.js                       # parseJson, stringifyJson, isSet, etc.
      db-mariadb.js                    # MariaDB pool factory
      db-sqlite.js                     # SQLite connection factory
      migrate.js                       # Migration runner
      package.json

    connectors/                        # @ab/connectors - SDK for apps to call connectors
      identity.js                      # Identity connector client (auth, users, realms)
      mail.js                          # Mail connector client (send emails via mail-service)
      pdf.js                           # PDF connector client (generate PDFs via pdf-service)
      ai.js                            # AI connector client (chat, vision, image via ai-service)
      storage.js                       # Storage connector client (upload/retrieve files)
      index.js                         # Re-exports all connectors
      package.json

    app-foundation/                    # @ab/app-foundation - base for ALL apps (pre-built + custom)
      server/
        src/
          create-app.js               # Factory: creates a Fastify app with standard plugins
          plugins/
            auth.js                    # Auth via @ab/connectors/identity (JWT validation)
            db-mariadb.js              # MariaDB plugin (for pre-built multi-tenant apps)
            db-sqlite.js              # SQLite plugin (for custom apps)
            health.js                  # Standard /health endpoint
          middleware/
            tenant.js                  # Multi-tenancy: extracts userId/orgId, scopes queries
            error-handler.js           # Standard error responses
      client/
        src/
          create-client-app.js         # Factory: creates React app with standard providers
          providers/
            AuthProvider.jsx           # Reads JWT from localStorage, provides user context
            ConnectorsProvider.jsx     # Provides connector clients to app components
          components/
            AppShell.jsx               # Standard layout shell (nav, content area)
          hooks/
            useAuth.js                 # Auth hook (user, logout, isAuthenticated)
            useConnector.js            # Hook to call any connector
      package.json

  # ─── PORTAL (Stack 1: Anything Builder) ────────────────────
  portal/
    server/
      src/
        index.js                       # Fastify entry
        config.js
        routes/
          apps.routes.js               # App registry, entitlements
          sites.routes.js              # Site CRUD (delegates to Site Engine)
          custom-apps.routes.js        # Custom app lifecycle (clone, spawn, stop)
          storage.routes.js            # File upload/download (Storage connector backend)
        services/
          app-registry.service.js
          custom-app-manager.service.js
          storage.service.js
      migrations/
      package.json
    client/
      src/
        main.jsx
        pages/
          Login.jsx                    # Delegates to Identity connector
          Register.jsx                 # Delegates to Identity connector
          Home.jsx                     # Hero + AI input, Explore Apps, My Apps
          MyApps.jsx                   # Filtered view: only user's apps
          Templates.jsx                # Browse app templates
          Search.jsx                   # Search across apps, sites, templates
          OrgSettings.jsx              # Employee management (via Identity connector)
          ChatSession.jsx              # AI creation chat (New Chat / hero input)
        components/
          HeroInput.jsx                # AI chat input bar (text, image upload, voice)
          ExploreAppsGrid.jsx          # Category filter chips + rich preview cards
          MyAppsGrid.jsx               # User's apps + "Create New App" card
          AppCard.jsx                  # Rich card: preview image, name, description, status
          AppCardContextMenu.jsx       # Rename, Duplicate, Delete actions
          Sidebar.jsx                  # Left nav: Home, Search, New Chat, My Apps, Templates
          UsageCounter.jsx             # Plan usage display (e.g. "5/5")
          Footer.jsx                   # Help, Catalog, Pricing, Settings, social links
        stores/
          auth.store.js                # Calls @ab/connectors/identity
          myApps.store.js              # User's apps: sites + activated apps + custom apps
          explore.store.js             # Explore Apps: pre-built app catalogue + categories
          chat.store.js                # AI chat session state
        api/
          apps.js
          sites.js
          chat.js
      vite.config.js
      package.json
    Dockerfile

  # ─── SITE ENGINE (portal capability, own service) ──────────
  site-engine/
    server/
      src/
        index.js                       # Fastify entry
        routes/
          sites.routes.js              # Site CRUD
          ai.routes.js                 # AI edits (via @ab/connectors/ai)
          publish.routes.js            # Publishing pipeline
        services/
          sites.service.js             # Site management, normalization
          publish.service.js           # HTML/CSS generation from block tree
        models/
          site.model.js               # Site/block validation
      migrations/
      package.json
    client/
      src/
        editor/
          Renderer.jsx                 # Editor renderer with overlays
          ChatPanel.jsx                # AI chat interface (via @ab/connectors/ai)
          HoverMenu.jsx
          Toolbar.jsx
        renderer/
          Renderer.jsx                 # Pure block renderer
        stores/
          editor.store.js
      vite.config.js
      package.json
    Dockerfile

  # ─── CONNECTOR SERVICES ────────────────────────────────────
  connectors/
    identity-service/                  # Identity Connector backend (stateful, own DB)
      src/
        index.js
        routes/
          auth.routes.js               # POST /identity/login, /identity/register, /identity/refresh
          users.routes.js              # User CRUD within a realm
          realms.routes.js             # Realm management (create, configure, branding)
        services/
          auth.service.js              # Login, register, token issuance, password hashing (argon2)
          users.service.js             # User CRUD, role management
          realms.service.js            # Realm provisioning + config
          tokens.service.js            # JWT sign/verify, refresh token rotation
      migrations/                      # Knex migrations for ab_identity DB
      package.json
      Dockerfile

    ai-service/                        # AI Connector backend (stateless)
      src/
        index.js
        routes/
          chat.routes.js               # POST /ai/chat
          vision.routes.js             # POST /ai/vision
          image.routes.js              # POST /ai/image
        services/
          openai.service.js            # Study prototype ai.js for prompt patterns
          flux.service.js              # Study prototype images.js for Flux patterns
      package.json
      Dockerfile

    mail-service/                      # Mail Connector backend (stateless)
      src/
        index.js
        routes/
          send.routes.js               # POST /mail/send
          templates.routes.js          # GET /mail/templates (list MJML templates)
        services/
          mailer.service.js            # Nodemailer transport
        templates/                     # MJML email templates
          booking-confirmation.mjml
          invoice-sent.mjml
          payment-reminder.mjml
          proposal-review.mjml
      package.json
      Dockerfile

    pdf-service/                       # PDF Connector backend (stateless)
      src/
        index.js
        routes/
          generate.routes.js           # POST /pdf/generate
        templates/                     # React PDF templates
          invoice.jsx
          proposal.jsx
      package.json
      Dockerfile

  # ─── APPS (Stack 2: all built on App Foundation) ───────────
  apps/
    scheduler/                         # Appointment Scheduler (multi-tenant)
      server/
        src/
          index.js                     # Uses @ab/app-foundation create-app
          routes/
            availability.routes.js
            bookings.routes.js
            clients.routes.js
            public.routes.js           # Public booking (no auth)
          services/
            availability.service.js
            bookings.service.js        # Uses @ab/connectors/mail for confirmations
            reminders.service.js       # Uses @ab/connectors/mail for reminders
        migrations/
        package.json
      client/
        src/
          main.jsx                     # Uses @ab/app-foundation create-client-app
          pages/
            Calendar.jsx
            Availability.jsx
            BookingDetails.jsx
            PublicBooking.jsx           # Public booking form (no auth)
          stores/
            scheduler.store.js
        vite.config.js
        package.json
      Dockerfile

    invoicing/                         # Same foundation pattern
      server/
        src/
          index.js                     # Uses @ab/app-foundation create-app
          routes/ ...                  # Invoice CRUD, payments, clients
          services/ ...                # Uses @ab/connectors/mail + /pdf
        migrations/
      client/ ...
      Dockerfile

    proposals/                         # Same foundation pattern
      server/
        src/
          index.js                     # Uses @ab/app-foundation create-app
          routes/ ...                  # Proposal CRUD, sharing, signing
          services/ ...                # Uses @ab/connectors/mail + /pdf
        migrations/
      client/ ...
      Dockerfile

  # ─── CUSTOM APP TEMPLATE (cloned for new custom apps) ──────
  templates/
    custom-app/                        # Starting point for AI-generated apps
      server/
        src/
          index.js                     # Uses @ab/app-foundation create-app (SQLite mode)
          routes/
            data.routes.js             # Generic collection CRUD
      client/
        src/
          main.jsx                     # Uses @ab/app-foundation create-client-app
          App.jsx                      # Empty - AI populates this
      data/                            # SQLite DB dir (Docker volume)
      ai-manifest.json                 # Describes what AI can generate/modify
      Dockerfile
```

---

## Identity Connector (Auth & User Management)

### The Problem It Solves

Auth and user management must be **decoupled from the Portal**. Two reasons:

1. **App end-users should not know the Anything Builder exists.** When a business owner builds a Scheduler, their clients who book appointments see a branded login - not "Anything Builder."
2. **Every app may need its own user pool.** The Scheduler needs client accounts. The Invoicing app needs client accounts. Custom apps may need their own users. These are all separate from platform users.

### Two User Populations

| Population | Description | Where They Log In | Knows About Portal? |
|------------|-------------|-------------------|---------------------|
| **Platform users** | Business owners, admins, employees who build/manage apps | Portal login (`/auth/login`) | Yes - they use the portal dashboard |
| **App end-users** | Clients/customers who use a specific app (book appointments, view invoices, sign proposals) | App-specific login (`/apps/scheduler/login`) | **No** - they only see the app |

### Realms

The Identity service organizes users into **realms** - isolated user pools with independent credentials, roles, and branding.

| Realm | Created By | Users | Example |
|-------|-----------|-------|---------|
| `platform` | Seeded at startup | Business owners, admins, employees | The people who use the Anything Builder portal |
| `scheduler-org7` | Auto-created when org 7 enables Scheduler | Clients who book appointments with org 7 | "Jane's Salon" client portal |
| `invoicing-org7` | Auto-created when org 7 enables Invoicing | Clients who view/pay invoices from org 7 | Clients viewing their invoice |
| `custom-abc123` | Auto-created when custom app is spawned | Users of that custom app | Whatever the custom app needs |

Realms are isolated: a user in `scheduler-org7` cannot access `invoicing-org7` or `platform`. Each realm has its own branding config (logo, colors, app name) so login pages look like they belong to the business, not the platform.

### How It Works

```
Portal / App ──> @ab/connectors/identity ──> Identity Service (HTTP, port 4008)
                                                    │
                                              ┌─────v─────┐
                                              │ ab_identity│
                                              │  (MariaDB) │
                                              │            │
                                              │ realms     │
                                              │ users      │
                                              │ refresh_   │
                                              │   tokens   │
                                              └────────────┘
```

**JWT payload** (issued by Identity service):
```json
{
  "userId": 42,
  "email": "user@example.com",
  "realm": "platform",
  "orgId": 7,
  "role": "owner",
  "apps": ["scheduler", "invoicing", "custom-abc123"],
  "iat": 1700000000,
  "exp": 1700043200
}
```

For app end-users, the JWT is simpler:
```json
{
  "userId": 198,
  "email": "client@example.com",
  "realm": "scheduler-org7",
  "role": "client",
  "iat": 1700000000,
  "exp": 1700043200
}
```

### SSO Flow (Platform Users)

1. User navigates to `platform.domain.com/auth/login`
2. Portal frontend calls `@ab/connectors/identity.login({ realm: 'platform', email, password })`
3. Identity service returns JWT, stored in localStorage
4. User clicks "Scheduler" card in My Apps -> navigates to `/apps/scheduler/`
5. Scheduler frontend reads JWT from localStorage (same origin = shared)
6. Scheduler backend calls `@ab/connectors/identity.verify(token)` to validate
7. No re-login needed - the JWT's `realm: 'platform'` + `apps` claim grants access

### App End-User Flow (White-label)

1. Client navigates to `platform.domain.com/apps/scheduler/login` (or a vanity URL)
2. Scheduler shows a branded login page (logo, colors from realm config - no portal branding)
3. Scheduler frontend calls `@ab/connectors/identity.login({ realm: 'scheduler-org7', email, password })`
4. Identity service returns JWT scoped to that realm
5. Client uses the Scheduler - never sees the portal, never knows it exists

### Employee Provisioning

1. Business owner (platform user, `role: 'owner'`) calls Portal API to add employee
2. Portal calls `@ab/connectors/identity.createUser({ realm: 'platform', email, password, role: 'member', orgId })`
3. Portal grants app access in its own `user_apps` table
4. Employee logs in at portal, JWT contains `apps: [...]` claim
5. Apps validate the JWT - no per-app user registration needed

### Token Validation (`@ab/connectors/identity`)

Every service uses the Identity connector client to validate tokens:
```javascript
// In @ab/app-foundation auth plugin
import { identity } from '@ab/connectors';

// Fastify preHandler hook
async function authenticate(request, reply) {
  const token = request.headers.authorization?.replace('Bearer ', '');
  const user = await identity.verify(token);
  // user = { userId, email, realm, role, orgId, apps }
  request.user = user;
}
```

The Identity service exposes a `GET /identity/jwks` endpoint (JSON Web Key Set) so services can also validate JWTs locally without a network call per request (cache the public key). MVP uses shared `JWT_SECRET` for simplicity; asymmetric keys via JWKS is a future enhancement.

---

## Portal Database Schema

```sql
-- Portal DB is about app registry, site registry, and connectivity.
-- Users, orgs, and auth live in the Identity service.
-- Site content (blocks, etc.) lives in the Site Engine DB (ab_sites).

CREATE TABLE apps (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  slug VARCHAR(100) NOT NULL UNIQUE,
  name VARCHAR(255) NOT NULL,
  description TEXT,                        -- short description shown on Explore Apps cards
  type ENUM('prebuilt', 'custom') NOT NULL,
  category VARCHAR(100) DEFAULT '',        -- filter category for Explore Apps (e.g. "scheduling", "invoicing")
  icon_url VARCHAR(1024) DEFAULT '',
  preview_image_url VARCHAR(1024) DEFAULT '', -- rich preview image shown on Explore Apps cards
  api_base_url VARCHAR(1024) NOT NULL,     -- internal service URL
  frontend_url VARCHAR(1024) NOT NULL,     -- browser path
  health_check_path VARCHAR(255) DEFAULT '/health',
  status ENUM('active', 'building', 'maintenance', 'disabled') DEFAULT 'active',
  is_forkable BOOLEAN DEFAULT FALSE,       -- can users fork this as a custom app starting point?
  created_by_user_id BIGINT UNSIGNED NULL, -- references Identity service user ID; NULL for prebuilt
  config_json LONGTEXT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tracks which pre-built/custom apps a user has activated ("My Apps")
CREATE TABLE user_apps (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,        -- references Identity service user ID (platform realm)
  app_id BIGINT UNSIGNED NOT NULL REFERENCES apps(id),
  display_name VARCHAR(255) NULL,          -- user's custom name ("My Booking System"), NULL = use app default
  preview_image_url VARCHAR(1024) DEFAULT '', -- user-specific preview image (auto-generated screenshot or custom)
  publish_status ENUM('published', 'unpublished', 'building') DEFAULT 'unpublished',
  granted_by BIGINT UNSIGNED NULL,
  granted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY (user_id, app_id)
);

-- Dashboard items: sites the user has created (content in Site Engine DB)
-- The portal tracks site ownership; the Site Engine stores block content
CREATE TABLE user_sites (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,        -- references Identity service user ID (platform realm)
  site_engine_id BIGINT UNSIGNED NOT NULL,  -- FK to site-engine's sites table
  name VARCHAR(255) NOT NULL,              -- display name ("Acme Salon Website")
  icon_url VARCHAR(1024) DEFAULT '',
  preview_image_url VARCHAR(1024) DEFAULT '', -- auto-generated screenshot for My Apps card
  status ENUM('draft', 'published', 'disabled') DEFAULT 'draft',
  published_url VARCHAR(1024) NULL,        -- path slug when published
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_user (user_id)
);
```

## Identity Service Database Schema

```sql
-- Identity DB manages ALL users across ALL realms.

CREATE TABLE realms (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  slug VARCHAR(255) NOT NULL UNIQUE,       -- e.g. 'platform', 'scheduler-org7'
  name VARCHAR(255) NOT NULL,              -- display name, e.g. "Jane's Salon"
  owner_org_id BIGINT UNSIGNED NULL,       -- NULL for 'platform' realm
  branding_json LONGTEXT NULL,             -- { logo, primaryColor, appName } for white-label
  auth_config_json LONGTEXT NULL,          -- { allowRegistration, requireEmailVerification, ... }
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  realm_id BIGINT UNSIGNED NOT NULL REFERENCES realms(id),
  email VARCHAR(255) NOT NULL,
  password_hash VARCHAR(255) NOT NULL,     -- argon2
  display_name VARCHAR(255) DEFAULT '',
  role VARCHAR(50) DEFAULT 'member',       -- flexible per realm: 'owner', 'admin', 'member', 'client'
  org_id BIGINT UNSIGNED NULL,             -- for platform realm: which org the user belongs to
  status ENUM('active', 'invited', 'disabled') DEFAULT 'active',
  last_login_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE KEY uk_realm_email (realm_id, email)  -- email unique within a realm, not globally
);

CREATE TABLE refresh_tokens (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL REFERENCES users(id),
  token_hash VARCHAR(255) NOT NULL UNIQUE,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user (user_id),
  INDEX idx_expires (expires_at)
);

CREATE TABLE organizations (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL UNIQUE,
  owner_user_id BIGINT UNSIGNED NOT NULL REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Key design**: Emails are unique **within a realm**, not globally. The same person can have `jane@example.com` as a platform user AND as a client in `scheduler-org7` - these are separate accounts with separate passwords and roles.

Pre-built apps are seeded at startup:
```javascript
const prebuiltApps = [
  { slug: 'scheduler', name: 'Appointment Scheduler', type: 'prebuilt',
    description: 'Accept bookings and sync calendars automatically.',
    category: 'scheduling', isForkable: true,
    previewImageUrl: '/assets/apps/scheduler-preview.jpg',
    apiBaseUrl: 'http://scheduler:4002', frontendUrl: '/apps/scheduler/' },
  { slug: 'invoicing', name: 'Smart Invoice', type: 'prebuilt',
    description: 'Automated invoicing system with late payment chasing.',
    category: 'finance', isForkable: true,
    previewImageUrl: '/assets/apps/invoicing-preview.jpg',
    apiBaseUrl: 'http://invoicing:4003', frontendUrl: '/apps/invoicing/' },
  { slug: 'proposals', name: 'Proposals & Sign', type: 'prebuilt',
    description: 'Draft, send, and e-sign professional documents instantly.',
    category: 'documents', isForkable: true,
    previewImageUrl: '/assets/apps/proposals-preview.jpg',
    apiBaseUrl: 'http://proposals:4004', frontendUrl: '/apps/proposals/' },
];
// Note: Site Engine is NOT in the app registry - it's a portal capability,
// not a user-facing app. Sites are tracked separately (see site-engine DB).
```

---

## Pre-built App Multi-tenancy

Pre-built apps are **shared instances** - all users use the same service. Multi-tenancy is at the data layer:

- Every table has a `user_id` column (from JWT)
- Every query filters by `user_id`
- Organization scoping via `org_id` where needed (e.g., all employees see same appointments)
- No per-user app instances - one Scheduler service serves all Scheduler users

Example (Scheduler DB):
```sql
CREATE TABLE appointments (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT UNSIGNED NOT NULL,       -- tenant from JWT
  org_id BIGINT UNSIGNED NULL,            -- org scoping
  date DATE NOT NULL,
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  status ENUM('scheduled', 'cancelled', 'completed') DEFAULT 'scheduled',
  client_name VARCHAR(255) NOT NULL,
  client_email VARCHAR(255),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user_date (user_id, date)
);
```

---

## Custom App System

> **Prototype reference**: The Anything Builder prototype (`demo-build-me-anything-app`) is a complete working implementation of this system. See [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) for the full AI pipeline, validation loop, and app template patterns.

### The Empty Template

A minimal app built on `@ab/app-foundation` at `templates/custom-app/`. When a user creates a custom app:

1. Portal clones the template to `/data/custom-apps/<id>/`
2. `npm install` (with warmed npm cache for speed)
3. Portal spawns a **Vite dev server** as child process on an allocated port
4. AI generates code via multi-stage pipeline (analysis → plan → design → code → validate → refine)
5. Progressive refinement: AI validation + build validation, up to 5 iterations
6. Git snapshot before generation, rollback on total failure
7. Vite HMR delivers changes to the preview iframe in real time
8. Owner continues refining the app via chat — the Vite dev server stays running throughout

**Publishing** (making the app available at a path slug):
1. `npm run build` produces a production bundle
2. Build Docker image from the custom app
3. Deploy as a K8s pod (Helm chart, same cluster as all other services)
4. Nginx Ingress routes `platform.domain.com/apps/custom/<id>/` to the pod

The Vite dev server is the **persistent development environment**, not a temporary step. It runs whenever the owner is in the builder interface, providing instant preview of AI edits. Publishing (Docker build → K8s deploy) is a separate action that creates the production deployment.

### AI Generation Pipeline (from prototype)

The prototype proves a multi-stage approach:

1. **Conversational analysis**: Classify user intent (build/modify/fix), detect scope
2. **Clarification**: Ask max 3 conversational questions if requirements are ambiguous
3. **Planning**: Generate brief overview (2-3 sentences, no code)
4. **Design**: Generate design system (colors, typography, page structure, tech decisions)
5. **Code generation**: Full app code using Claude (with extended thinking for complex generation) and `filepath:` markers
6. **Validation**: AI code review (8-point checklist) + build validation (`npm run build`)
7. **Refinement**: Feed errors back to AI for auto-fix (max 5 iterations)
8. **Summary**: User-facing message (2-3 sentences for new, 1-2 for modification)

### Why SQLite (better-sqlite3)

- Single file (`app.db`) - trivially backed up as a Docker volume
- JSON functions (`json_extract`, `json_each`) built-in
- Synchronous API (no callback complexity)
- Zero config, no separate container needed
- Perfect for single-user private apps
- **Proven in prototype**: Document DB via Vite plugin (`/api/db` endpoint) with auto-created collections, seed.json support, and filter operators ($eq, $ne, $gt, $gte, $lt, $lte, $contains, $in)

### Custom App Template Structure
```
templates/custom-app/
  server/src/
    index.js               # Uses @ab/app-foundation create-app (SQLite mode)
    routes/data.routes.js   # Generic collection CRUD (mirrors prototype vite-db-plugin.js)
  client/src/
    main.jsx                # Uses @ab/app-foundation create-client-app
    App.jsx                 # Empty - AI populates
    db.js                   # Frontend DB client (fetch wrapper for /api/db)
  seed.json                 # Optional: initial data for collections (idempotent seeding)
  data/app.db               # SQLite file
  ai-manifest.json          # Tells AI what can be generated
  Dockerfile
```

Custom apps automatically get access to all connectors (mail, PDF, AI, storage) through `@ab/connectors` - no extra integration needed.

### App Embedding in Sites

When the Site Engine AI detects that a user's request requires dynamic functionality beyond static blocks, it embeds an app using the `app` block kind:

**Detection flow** (in Site Engine AI routes):
1. User makes a request in the site editor chat (e.g. "add a booking section", "I need a calculator")
2. AI analysis classifies the intent and determines if it requires an app vs. static content
3. If app needed → check user's existing apps (pre-built + custom) for a match
4. If match found → create `app` block with the existing app's URL
5. If no match → trigger the portal's App Engine to create a custom app
6. Once the custom app is ready → create `app` block referencing the new app
7. The `app` block renders as an iframe in both the editor and published site

**What the AI needs to know**:
- The user's available apps (from Portal app registry via `GET /api/apps`)
- Each app's embeddable views (pre-built apps expose known public URLs; custom apps expose their root)
- How to construct the `app` block schema (appId, appSlug, appUrl, height)

**Pre-built app embeddable views**:
| App | Embeddable URL | Use Case |
|-----|---------------|----------|
| Scheduler | `/apps/scheduler/public/book/:slug` | Booking widget on a business site |
| Invoicing | `/apps/invoicing/view/:token` | Invoice payment page (less common) |
| Proposals | `/apps/proposals/view/:token` | Proposal review page (less common) |

Custom apps embed their root URL (`/apps/custom/<id>/`), which renders the full app within the iframe.

---

## Nginx / Ingress Path Routing

```nginx
server {
    listen 80;

    # Portal frontend + API
    location / { proxy_pass http://portal-client:3000; }
    location /api/ { proxy_pass http://portal:4000; }

    # Site Engine (portal capability, not a user-facing app)
    location /sites/ { proxy_pass http://site-engine-client:3001/; }
    location /sites/api/ { proxy_pass http://site-engine:4001/api/; }

    # Identity service (auth for all contexts)
    location /identity/ { proxy_pass http://identity-service:4008/identity/; }

    # Scheduler
    location /apps/scheduler/ { proxy_pass http://scheduler-client:3002/; }
    location /apps/scheduler/api/ { proxy_pass http://scheduler:4002/api/; }

    # Invoicing
    location /apps/invoicing/ { proxy_pass http://invoicing-client:3003/; }
    location /apps/invoicing/api/ { proxy_pass http://invoicing:4003/api/; }

    # Proposals
    location /apps/proposals/ { proxy_pass http://proposals-client:3004/; }
    location /apps/proposals/api/ { proxy_pass http://proposals:4004/api/; }

    # AI Service (internal, not browser-accessible)
    # Apps call http://ai-service:4005 directly

    # Custom apps (dynamic port mapping)
    location ~ ^/apps/custom/([^/]+)/(.*) {
        proxy_pass http://127.0.0.1:$custom_app_port/$2;
    }
}
```

---

## Sprint Plan (10 weeks, 5 sprints)

**Team**: BD1, BD2 (backend devs) + FD1, FD2 (frontend devs), each managing 3 AI agents

---

### Sprint 0 - Foundation (Week 1)

**Goal**: Monorepo, shared packages, `@ab/app-foundation`, connectors SDK, Portal running, Identity + AI connector services, Nginx routing. Every agent can work independently after this.

| Dev | Agent 1 | Agent 2 | Agent 3 |
|-----|---------|---------|---------|
| **BD1** | Monorepo scaffold + `@ab/shared` (jwt.js, http.js, helpers.js, db-mariadb.js, db-sqlite.js) + `@ab/connectors` SDK (identity, mail, pdf, ai, storage client modules) | `@ab/app-foundation` server (create-app factory, auth plugin via `@ab/connectors/identity`, db plugins, tenant middleware, health endpoint, error handler) | Portal DB schema + Knex migrations (apps, user_apps, user_sites) + seed script for pre-built apps |
| **BD2** | **Identity connector service**: Fastify + own MariaDB (`ab_identity`). Realms, users, orgs tables. Auth routes (register, login, refresh, verify) + user CRUD + realm management. Uses argon2 for passwords, JWT for tokens | Portal Fastify server + app registry routes (list apps, get app, grant/revoke access) + Storage connector routes. Portal delegates all auth to Identity service | AI connector service: provider-agnostic with Claude as primary, OpenAI as fallback. Claude Messages API for chat/vision + Flux image gen. Provider router + `@anthropic-ai/sdk`. Study [REFERENCE-AI-MIGRATION.md](REFERENCE-AI-MIGRATION.md) for connector design |
| **FD1** | `@ab/app-foundation` client (create-client-app factory, AuthProvider via `@ab/connectors/identity`, ConnectorsProvider, AppShell, useAuth hook) | Portal Login + Register pages (delegate to Identity connector) + Portal Home page (Hero + AI input, Explore Apps grid, My Apps grid) | Vite + React 19 + Tailwind + Zustand scaffold for Portal client |
| **FD2** | Shared UI component library (Button, Input, Card, Modal, Toast, Spinner - accessible, Tailwind-based) | Portal layout shell (left sidebar nav: Home, Search, New Chat, My Apps, Templates + top-right usage counter + user avatar + footer) + org settings page (user management via Identity connector) | docker-compose.dev.yml (all MariaDB instances incl. `ab_identity` for local dev) + Helm chart scaffolding + Nginx/Ingress config + API client pattern |

**Prototype reference docs to study:**
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "AI Integration Patterns" - block schema prompting, chat completions, vision bootstrap
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "Image Generation" - Flux 2 Pro API, media placeholder detection
- [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) § "Code Generation" - AI generation patterns, conversation continuity
- [REFERENCE-AI-MIGRATION.md](REFERENCE-AI-MIGRATION.md) - Claude API migration guide, provider-agnostic connector design

**Sync point**: End of day 3 - Nginx routes to Portal, Identity service running, platform realm seeded, auth flow works (register + login via Identity connector). End of week - Portal home page shows hero input + Explore Apps with seeded pre-built apps + empty My Apps with "Create New App" card. Left sidebar navigation works. `@ab/app-foundation` usable.

---

### Sprint 1 - Site Engine + Scheduler + Mail Connector + App Engine Scaffold (Weeks 2-3)

**Goal**: Site Engine and Scheduler running. Mail connector service. Both integrated with Identity connector and other connectors. App Engine: custom app template functional, AI pipeline tested end-to-end with a simple app.

| Dev | Agent 1 | Agent 2 | Agent 3 |
|-----|---------|---------|---------|
| **BD1** | Site Engine server: site CRUD, JSON block tree normalization, ensureIds. Study prototype `services/sites.js` for patterns | Site Engine: AI routes using `@ab/connectors/ai` (block schema prompting, vision bootstrap, image gen). Study prototype `api/ai.js` for prompt patterns | **Mail connector service** (Nodemailer + MJML templates: booking-confirmation, invoice-sent, payment-reminder, proposal-review) |
| **BD2** | Scheduler DB schema + Knex migrations + Scheduler CRUD service on `@ab/app-foundation` (availability, bookings with conflict detection, clients) | Scheduler API routes (availability CRUD, booking CRUD, public booking without auth) - uses `@ab/connectors/mail` for confirmations | **App Engine scaffold**: Custom app template on `@ab/app-foundation` (SQLite mode, generic collection CRUD, `ai-manifest.json`) + clone/spawn pipeline in Portal (clone template, npm install, start Vite dev server, register in app registry). Study [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) § "App Template" |
| **FD1** | Site Engine editor client: Renderer, ChatPanel (uses `@ab/connectors/ai`), HoverMenu, Toolbar. Study prototype `editor/` | Site Engine client: pure block renderer (JSON blocks -> React elements, visual parity). Study prototype `renderer/Renderer.jsx` | Scheduler admin UI: calendar view, appointment list, booking details, status management |
| **FD2** | Scheduler admin UI: availability configuration (set hours, recurring patterns, buffer time) | Scheduler public booking page: date picker, available slot display, booking form, confirmation (no auth, mobile-first) | Site Engine + Scheduler: API client layers using `@ab/connectors` pattern |

**Reference docs to study:**
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "Site Data Model" - normalize(), ensureIds() pipeline
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "AI Integration Patterns" - getBlockSchemaHint(), buildBlock(), mergePreserveContent()
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "Editor Architecture" - wrapper/overlay, EditableTag, drag-and-drop
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "State Management" - history/undo, debounced save
- [REFERENCE-APP-SCHEDULER.md](REFERENCE-APP-SCHEDULER.md) § "Database Schema" - all 7 tables, indexes, foreign keys
- [REFERENCE-APP-SCHEDULER.md](REFERENCE-APP-SCHEDULER.md) § "Slot Calculation Logic" - availability rules + overrides - conflicts - buffer
- [REFERENCE-APP-SCHEDULER.md](REFERENCE-APP-SCHEDULER.md) § "Connector Usage" - Mail connector templates, reminder scheduler cron
- [REFERENCE-APP-SCHEDULER.md](REFERENCE-APP-SCHEDULER.md) § "Key Business Rules" - conflict detection, cancellation, timezone handling

**Sync point**: End of sprint - Site Engine works (create/edit/save sites). Scheduler works as standalone service. Scheduler sends booking confirmations via Mail connector.

---

### Sprint 2 - Invoicing + Proposals + PDF Connector (Weeks 4-5)

**Goal**: Invoicing and Proposals apps running on `@ab/app-foundation`. PDF connector service. AI chat in Site Engine. All apps use connectors for mail/PDF.

| Dev | Agent 1 | Agent 2 | Agent 3 |
|-----|---------|---------|---------|
| **BD1** | Invoicing app on `@ab/app-foundation`: DB + Knex migrations (invoices, line_items, clients, payments, reminders) + CRUD service with status state machine (draft->sent->viewed->paid/overdue) | Invoicing: payment tracking, auto-numbering, chase reminder scheduler (uses `@ab/connectors/mail` for escalating emails) | **PDF connector service** (React PDF templates: invoice, proposal) + `@ab/connectors/pdf` client |
| **BD2** | Proposals app on `@ab/app-foundation`: DB + Knex migrations (proposals, sections, recipients) + document CRUD with sections + template system | Proposals: share links (token-based, no auth), recipient tracking, simple e-signature (name + timestamp + IP). Uses `@ab/connectors/mail` for notifications | Proposals: version history (snapshot on each edit), locked-after-signing. Uses `@ab/connectors/pdf` for PDF export |
| **FD1** | Invoicing admin UI: invoice list with filters, invoice editor with line items, auto-calculated totals | Invoicing: client management, payment recording, aging report, PDF download (via `@ab/connectors/pdf`) | AI chat panel for Site Engine (SSE streaming, Zustand, uses `@ab/connectors/ai`) |
| **FD2** | Proposals editor UI: section-based editing, template picker, document preview, share/send interface | Proposals: public view (token-based, clean layout, accept/sign), version history | Invoicing + Proposals: public views (invoice via token link, print-friendly CSS) |

**Reference docs to study:**
- [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) § "Database Schema" - all 8 tables, status state machine
- [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) § "Auto-numbering Logic" - configurable prefix, sequential counter
- [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) § "Total Recalculation" - line item changes trigger recalc, tax per line
- [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) § "Chase Reminder Scheduler" - escalating reminders, schedule config, Mail connector
- [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) § "Connector Usage" - Mail templates (invoice-sent, payment-reminder, payment-received), PDF template
- [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) § "Database Schema" - all 9 tables, section types, recipient roles
- [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) § "E-Signature Flow" - typed name + consent + audit trail, lock-after-signing
- [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) § "Version History Logic" - immutable snapshots, when snapshots are created
- [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) § "Variable Interpolation" - merge fields, built-in variables
- [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) § "Connector Usage" - Mail templates (proposal-review, signed, accepted, declined), PDF template

**Sync point**: End of sprint - all 3 pre-built apps have full CRUD. All use connectors for mail/PDF. Invoices can be sent/viewed. Proposals can be shared/signed. Site Engine AI chat working.

---

### Sprint 3 - Custom Apps + App Embedding + Publishing + Polish (Weeks 6-7)

**Goal**: Custom app template on `@ab/app-foundation` works. App embedding in sites (`app` block kind). Publishing pipeline. Inline editing. Security hardened.

| Dev | Agent 1 | Agent 2 | Agent 3 |
|-----|---------|---------|---------|
| **BD1** | Custom app template on `@ab/app-foundation` (SQLite mode, generic collection CRUD, `ai-manifest.json` - automatically gets all connectors) | Custom app spawn pipeline in Portal (clone template, AI generates code via `@ab/connectors/ai`, npm install, start process, register in app registry) | Publishing pipeline for Site Engine (generate HTML+CSS from block tree, `app` blocks → iframes in published output, study prototype renderStaticAssets for patterns) |
| **BD2** | Rate limiting (`@fastify/rate-limit` on all services + connector services), CSP headers, CORS, input validation, XSS protection | Storage connector: file upload + asset management for all apps (logos, images via `@ab/connectors/storage`, centralized in Portal) | Inline content update API for Site Engine (`PATCH` with debounce-safe partial updates, normalize/ensureIds on write) |
| **FD1** | Custom app creation flow in Portal (describe app in chat -> AI generates -> show progress -> redirect to running app) | Inline text editing for Site Engine (contentEditable, blur-commit, debounced save, study prototype EditableTag/EditableHtml) | `app` block rendering in Site Engine editor (iframe with wrapper/inner overlay, interactive preview, height config) |
| **FD2** | Undo/redo for Site Engine (Zustand store, snapshot history, localStorage persistence, Ctrl+Z/Y) | Mobile responsive audit + fixes across Portal + all app UIs | Publish flow UI for Site Engine (publish button, preview, published URL, version history) |

**Reference docs to study:**
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "App Block (Embedded Apps)" - `app` block kind, iframe rendering, cross-over detection
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "Editor Architecture" - EditableTag, EditableHtml, wrapper/overlay system
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "State Management" - history/undo architecture, debounced save
- [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) § "Publishing Pipeline" - renderStaticAssets, version management
- [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) § "AI Pipeline Architecture" - multi-stage pipeline for custom app generation
- [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) § "App Template" - document DB, Vite plugin, seed.json pattern

**Sync point**: End of sprint - custom app creation works E2E. App embedding in sites works (existing apps + on-the-fly creation). Published sites accessible. Inline editing saves.

---

### Sprint 4 - Integration, E2E, Production (Weeks 8-9, ~8 days)

**Goal**: Full integration testing. Playwright E2E. Production Helm charts. Onboarding.

| Dev | Agent 1 | Agent 2 | Agent 3 |
|-----|---------|---------|---------|
| **BD1** | Integration tests: full flows across Portal + each app (auth -> grant access -> use app -> verify data isolation) | Production Helm charts (all services + DBs + Ingress, multi-stage Docker builds, health checks, liveness/readiness probes) | Database optimization (indexes, N+1 audit, connection pool tuning across all services) |
| **BD2** | API documentation (OpenAPI from Fastify schemas per service) | Seed data + demo mode (realistic sample data for all apps, demo org with employees) | Performance testing (load test multi-tenant isolation, concurrent bookings, AI timeout handling) |
| **FD1** | Playwright E2E: Portal auth + dashboard + navigate to each app + core flows | Playwright E2E: Scheduler booking flow, Invoicing send/view/pay, Proposals send/sign | Accessibility audit (Lighthouse >90, WCAG AA) + bundle optimization per app |
| **FD2** | Visual polish (dark theme, consistent design across all apps, empty states, error pages, rich preview cards for Explore Apps) | Onboarding flow (first-time guided tour: hero input highlights → explore apps → my apps, app setup wizard per app type) | Settings + account management (profile, password change, org management, app access control, usage/plan display) |

---

## Dependency Graph

```
WEEK 1 (Sprint 0) - Foundation layer
  BD1-A1 @ab/shared + @ab/connectors ──> ALL services depend on this
  BD1-A2 @ab/app-foundation ───────────> ALL apps built on this
  BD1-A3 Portal DB ────────────────────> Portal API
  BD2-A1 Identity connector service ──> ALL apps + Portal need auth
  BD2-A3 AI connector service ─────────> Site Engine AI, Custom App creation
  FD1-A1 @ab/app-foundation client ───> ALL frontend apps use this
  FD2-A1 UI component lib ────────────> ALL frontend apps + Site Engine use these

WEEKS 2-3 (Sprint 1) - Site Engine + Scheduler + Mail connector
  BD1-A3 Mail connector ──────────────> Scheduler confirmations, later Invoicing/Proposals
  BD1-A1/A2 Site Engine BE ───┐
  FD1-A1/A2 Site Engine FE ───┤──> Independent from Scheduler
  BD1-A3 + BD2 Scheduler BE ─┐
  FD1-A3 + FD2 Scheduler FE ─┤──> Independent from Site Engine

WEEKS 4-5 (Sprint 2) - Invoicing + Proposals + PDF connector
  BD1-A3 PDF connector ──────────────> Invoicing PDF, Proposals PDF
  BD1 Invoicing BE ──────> FD1 Invoicing FE (can use mock data initially)
  BD2 Proposals BE ──────> FD2 Proposals FE (can use mock data initially)

WEEKS 6-7 (Sprint 3) - Custom Apps + App Embedding + Publishing
  Custom App Template ──dep──> @ab/app-foundation (S0), AI connector (S0)
  App Embedding (app block) ──dep──> Site Engine (S1), App Registry (S0)
  On-the-fly app creation ──dep──> Custom App Template (S3), Site Engine AI (S1)
  Publishing ──dep──> Site Engine (S1)
  Inline Editing ──dep──> Site Engine FE (S1)

WEEKS 8-9 (Sprint 4) - All streams converge
  Integration tests ──dep──> ALL services + connectors running
```

**Critical path**: `@ab/shared` + `@ab/app-foundation` -> Identity Service -> Portal -> App Engine + Site Engine -> AI Chat -> Publishing -> Integration

**NOT on critical path** (can slip): PDF connector, analytics, onboarding, visual polish

---

## Reference Documents

Both prototypes are **not ported** - they are studied for proven patterns. The 3 pre-built apps have no prototypes - their reference docs define the complete specification. Full analysis is in separate reference documents:

### [REFERENCE-SITE-ENGINE.md](REFERENCE-SITE-ENGINE.md) - Site Engine Prototype (master branch)

Key patterns to replicate in the Site Engine:

| Pattern | Description | Applies To |
|---------|-------------|------------|
| Block schema (wrapper/inner CSS split) | Every block has separate wrapper (spacing/positioning) and inner (visual) style objects | Site Engine |
| `getBlockSchemaHint()` | Detailed JSON schema instruction for AI to produce correct block structure | AI Service + Site Engine |
| `buildBlock()` / `mergePreserveContent()` | AI edit flow with safe merging that preserves user's in-flight edits | AI Service + Site Engine |
| `buildSiteFromImage()` | Vision-based bootstrap: image → complete block tree → media placeholder fill | AI Service + Site Engine |
| `normalize()` / `ensureIds()` | Validation pipeline run on every save | Site Engine |
| `renderStaticAssets()` | JSON block tree → static HTML/CSS publishing pipeline | Site Engine |
| Editor renderer with overlays | Wrapper/inner rendering, hover menu, selection box, drag-and-drop | Site Engine client |
| `EditableTag` / `EditableHtml` | contentEditable inline editing with blur-commit | Site Engine client |
| History/undo (snapshot-based) | Max 100 snapshots in localStorage, Ctrl+Z/Y | Site Engine client |
| Debounced save (400ms) | Prevents excessive API calls during rapid editing | Site Engine client |
| Flux 2 Pro image generation | Polling API (3s interval, 60s timeout), photorealistic constraints | AI Service |

### [REFERENCE-APP-ENGINE.md](REFERENCE-APP-ENGINE.md) - App Engine Prototype (demo branch)

Key patterns to replicate:

| Pattern | Description | Applies To |
|---------|-------------|------------|
| Multi-stage AI pipeline | analysis → plan → design → code → validate → refine | App Engine |
| 8-type conversational analysis | answer, question, request, mixed, topic_change, off_topic, reverse_engineering, clarification_question | AI Service |
| Boundary enforcement | Off-topic rejection, reverse engineering detection, scope limitation | AI Service |
| Clarification system | Max 3 questions, pending/answered/superseded states, conversational style | AI Service |
| Progressive refinement (max 5 iterations) | AI validation + build validation loop with auto-fix | App Engine |
| Git-based rollback | Snapshot before generation, reset on total failure | App Engine |
| Design system extraction | Extract colors/typography/patterns from existing code before modifications | AI Service |
| Composable pipeline steps | Different step sets for build vs modify vs fix | App Engine |
| Document DB via Vite plugin | SQLite + JSON blobs, auto-created collections, seed.json support | Custom App Template |
| AI decision logging | Every AI decision stored in DB for debugging | AI Service |
| Conversation continuity | Explicit message history + prompt caching (replaces OpenAI's `previous_response_id`) | AI Service |
| WebSocket room-based updates | Socket.IO rooms per project, typed events (ai_message, build_status, etc.) | Portal + All Apps |
| Vite dev server per project | Dynamic port allocation, HMR for instant preview updates | App Engine |

### [REFERENCE-APP-SCHEDULER.md](REFERENCE-APP-SCHEDULER.md) - Appointment Scheduler

Complete specification for the Scheduler app (no prototype). Key sections:

| Section | Description | Sprint |
|---------|-------------|--------|
| Database schema (7 tables) | services, availability_rules, availability_overrides, booking_settings, appointments, clients, reminders | Sprint 1 |
| Slot calculation logic | Computed from rules + overrides - existing appointments - buffer time | Sprint 1 |
| Public booking API | Rate-limited, no auth. Slot availability + booking creation + cancellation tokens | Sprint 1 |
| Reminder scheduler | Cron-like job, configurable timing (24h, 1h before), uses Mail connector | Sprint 1 |
| Business rules | Conflict detection, cancellation deadline, min notice, timezone handling, auto-status transitions | Sprint 1 |

### [REFERENCE-APP-INVOICING.md](REFERENCE-APP-INVOICING.md) - Invoicing & Late Payment Chasing

Complete specification for the Invoicing app (no prototype). Key sections:

| Section | Description | Sprint |
|---------|-------------|--------|
| Database schema (8 tables) | clients, invoice_settings, invoices, line_items, payments, chase_reminders, recurring_invoices | Sprint 2 |
| Status state machine | draft → sent → viewed → paid/overdue/cancelled/write_off, with transition rules | Sprint 2 |
| Auto-numbering | Configurable prefix + sequential counter, never reuses numbers | Sprint 2 |
| Total recalculation | Triggered on any line item change, handles tax per line, partial payments | Sprint 2 |
| Chase reminder scheduler | Escalating reminders (gentle → firm → urgent → final), configurable schedule, uses Mail connector | Sprint 2 |
| Recurring invoices | Template-based auto-generation on schedule (weekly/monthly/quarterly/yearly) | Sprint 2 |
| PDF generation | Professional invoice PDF via PDF connector | Sprint 2 |

### [REFERENCE-APP-PROPOSALS.md](REFERENCE-APP-PROPOSALS.md) - Proposals & Contracts

Complete specification for the Proposals app (no prototype). Key sections:

| Section | Description | Sprint |
|---------|-------------|--------|
| Database schema (9 tables) | template_categories, templates, proposals, sections, pricing_items, recipients, versions, activity_log, proposal_settings | Sprint 2 |
| Section-based editor | Rich text sections, pricing tables, signature sections, drag-to-reorder | Sprint 2 |
| E-signature flow | Typed name + consent + timestamp + IP + user agent. Audit trail. Lock after all signers complete | Sprint 2 |
| Version history | Immutable snapshots on significant edits. Compare and restore | Sprint 2 |
| Variable interpolation | `{{client_name}}`, `{{date}}`, etc. Resolved at render time | Sprint 2 |
| Recipient tracking | Per-recipient view tokens, engagement tracking (view count, time), independent sign/decline | Sprint 2 |
| Template system | Reusable templates with categories, save-as-template from existing proposals | Sprint 2 |

### Everything is Built New
- `@ab/shared` package (low-level utils)
- `@ab/connectors` SDK + connector services (Identity, AI, Mail, PDF, Storage)
- `@ab/app-foundation` (server + client factories for all apps)
- Identity service (realms, users, auth - standalone, white-label)
- Portal (dashboard, app registry, lifecycle - delegates all auth to Identity)
- Site Engine (portal capability - informed by Site Builder prototype patterns)
- App Engine (portal capability - informed by Anything Builder prototype pipeline)
- Scheduler, Invoicing, Proposals apps (all on app-foundation)
- Custom app template + spawn system
- Nginx reverse proxy

---

## Verification Plan

### Per-Sprint
- **S0**: Nginx routes to Portal. Identity service running with `platform` realm. Auth flow works E2E (register + login via Identity connector). Home page shows hero input + Explore Apps + My Apps. AI service responds.
- **S1**: Site Engine works (type in hero input → AI creates site → edit blocks → save). Scheduler works (activate from Explore Apps → appears in My Apps). Scheduler accepts public bookings. Both validate JWTs via Identity connector.
- **S2**: All 3 pre-built apps have full CRUD. Invoices send/view. Proposals share/sign. AI chat edits site blocks.
- **S3**: Custom app creation works. Published sites accessible. Inline editing saves. Mobile responsive.
- **S4**: Playwright E2E passes. Multi-tenant isolation verified. Helm charts deploy to K8s. Lighthouse >90.

### Critical E2E Flows
1. Register on Portal -> Login -> See Home page (hero, explore, my apps) -> Type in hero input "build a website for my salon" -> AI chat creates site -> Edit blocks -> Publish -> Visit published URL
2. Owner creates employee -> Employee logs in -> Accesses Scheduler -> Creates availability -> Public user books appointment -> Confirmation email
3. Create invoice -> Add line items -> Send -> Client views via token link -> Record payment -> Chase reminder sent for overdue
4. Create proposal -> Edit sections -> Send for review -> Client views -> Client signs -> PDF download
5. Click "Create New App" card in My Apps -> AI chat session -> Describe app -> AI builds it -> App appears in My Apps grid with "Published" status -> Click to open -> App works
6. In site editor chat, ask "add a booking widget" -> AI embeds Scheduler `app` block -> Booking form visible in iframe -> Works on published site
7. In site editor chat, ask "add a mortgage calculator" -> AI creates custom app on-the-fly -> Embeds as `app` block -> Calculator works in iframe
8. User has standalone Scheduler in My Apps -> types "add a landing page to my booking system" in hero input -> AI creates site wrapping the Scheduler -> Site appears in My Apps
9. Browse Explore Apps -> filter by category -> click "Appointment Scheduler" -> Use -> App activates and appears in My Apps as "Appointment Scheduler" -> Rename via context menu to "My Salon Bookings"
10. Browse Explore Apps -> click "Smart Invoice" -> Fork -> Custom app created from invoice template -> Opens in AI chat for customization -> Modified app appears in My Apps

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Multi-service complexity (7+ containers) | Hard to run locally | Single `npm run dev:all` script + docker-compose.dev.yml for DBs only. K8s deployment via Helm charts |
| Shared JWT_SECRET across services | Security concern | Identity service signs, all services verify with shared secret. Future: asymmetric keys via JWKS endpoint |
| Identity service is a single point of failure | Auth down = everything down | K8s liveness/readiness probes + auto-restart. Multiple replicas behind K8s Service. Token validation can be cached locally |
| App Engine spawn system is complex | Core feature at risk | Start AI pipeline integration in Sprint 1 (AI connector). Scaffold template + spawn in Sprint 2 alongside pre-built apps. Full E2E in Sprint 3 |
| Replicating AI interaction quality | New code may not match prototype's tuned prompts | Study prototype prompts closely, adapt for Claude API patterns (json_schema, extended thinking), test with real Claude/Flux APIs early. See [REFERENCE-AI-MIGRATION.md](REFERENCE-AI-MIGRATION.md) |
| 12 agents across 7+ codebases | Merge conflicts | Each service is a separate directory. `@ab/shared` changes go through one agent at a time. API contracts frozen at sprint starts |
| Multi-tenant data leaks | Security breach | Every service enforces `user_id` filtering in a Fastify preHandler hook. Integration tests explicitly verify isolation |
| Nginx config complexity grows | Routing errors | Template-based config generation. Health check all routes in CI |

