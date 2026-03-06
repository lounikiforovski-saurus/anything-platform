# opi-send Tech Stack vs AI Builder Blueprint — Comparison Report

## Overview

**opi-send** is a multi-tenant SaaS for transactional and marketing email, built with a modern stack optimized for AI-assisted development. This report compares its technology choices against the bare-metal AI app builder blueprint to evaluate what carries over, what doesn't, and what each approach optimizes for.

---

## Stack Comparison

| Layer | opi-send | Blueprint (AI Builder) | Verdict |
|-------|---------|----------------------|---------|
| **Framework** | Next.js 16 (App Router) | Next.js (for generated apps) | ✅ Same — Next.js is the right choice for both |
| **React** | React 19 + React Compiler | React 19 | ✅ Same |
| **Language** | TypeScript (strict) | TypeScript | ✅ Same |
| **Styling** | Tailwind v4 + shadcn/ui | Tailwind + shadcn/ui | ✅ Same — this is the optimal AI codegen stack |
| **Database** | Drizzle ORM + Postgres | Self-hosted Supabase (PostgREST) | ⚠️ Different — both use Postgres, different access layers |
| **Auth** | Supabase Auth (GoTrue) | Self-hosted Supabase Auth (GoTrue) | ✅ Same — opi-send uses managed, blueprint self-hosts |
| **ORM** | Drizzle (type-safe SQL) | PostgREST (auto-generated REST) + raw SQL | ⚠️ Different approaches to the same database |
| **Rich Text** | Tiptap (email editor) | N/A | N/A — domain-specific |
| **Email Rendering** | React Email | N/A | N/A — domain-specific |
| **Flow Builder** | React Flow (@xyflow) | N/A | N/A — domain-specific |
| **Data Fetching** | TanStack Query + Form | Generated per-app | ⚠️ Different — opi-send uses explicit caching, builder generates simpler fetch patterns |
| **Validation** | Zod | Zod (in generated code) | ✅ Same |
| **Billing** | Stripe | Stripe (for builder billing) | ✅ Same |
| **Linting** | Biome | Biome or ESLint | ✅ Same |
| **Testing** | Vitest | Vitest | ✅ Same |
| **Secrets** | 1Password (op inject) | K8s secrets / env vars | ⚠️ Different — both secure, different mechanisms |
| **Hosting** | Self-hosted (no Vercel) | k3s bare metal | ✅ Same philosophy — own the metal |
| **Container** | N/A (direct deploy) | Docker + k3s + Firecracker | Blueprint adds container orchestration layer |
| **Preview** | N/A (dev server) | WebContainers + Firecracker | Blueprint adds instant preview tech |
| **AI Gateway** | N/A | Custom AI Gateway | Blueprint adds LLM routing layer |

---

## Deep Analysis

### 1. Database Layer: Drizzle ORM vs PostgREST

**opi-send (Drizzle):**
- Type-safe SQL queries with explicit joins
- Schema defined in TypeScript (`src/db/schema.ts` — 1000+ lines)
- Migrations managed by drizzle-kit
- Full control over query optimization
- Developers write queries explicitly

**Blueprint (PostgREST via Supabase):**
- Auto-generated REST API from database schema
- No ORM — AI generates API calls or raw SQL
- Schema created dynamically by AI-generated SQL migrations
- Less control over query optimization
- AI generates the data access patterns

**Why each was chosen:**
- **opi-send uses Drizzle because** it's a developer-built SaaS where query performance and type safety matter. Drizzle gives explicit control over every SQL query, which is essential for a high-volume email platform (sending millions of emails requires optimized queries).
- **Blueprint uses PostgREST because** AI-generated apps need an automatic API layer. The AI generates a schema, PostgREST instantly exposes it as REST endpoints. No code needed for CRUD operations. This is the right choice for an AI builder where the user doesn't write SQL.

**Pros/Cons:**

| | Drizzle (opi-send) | PostgREST (Blueprint) |
|---|---|---|
| ✅ Pros | Type-safe, fast queries, full control, great DX | Zero-code API, instant from schema, AI-friendly |
| ❌ Cons | Requires developer to write queries, more code | Less control, harder to optimize, no type safety |
| 🎯 Best for | Developer-built SaaS | AI-generated apps |

### 2. Auth: Managed vs Self-Hosted Supabase

**opi-send:** Uses `@supabase/ssr` and `@supabase/supabase-js` — managed Supabase auth. Clean integration with Next.js middleware for SSR auth.

**Blueprint:** Same GoTrue auth engine, but self-hosted. Identical API, identical JWT tokens, just running on our infrastructure.

**Key insight:** opi-send validates that Supabase auth works well with Next.js App Router. The same patterns (server-side client creation, middleware-based session refresh) carry directly to the AI builder's generated apps. The AI should generate auth code that matches opi-send's patterns because they're proven to work.

### 3. UI Components: shadcn/ui in Both

Both use shadcn/ui (Radix primitives + Tailwind). This is the optimal stack for AI code generation because:
- **Components are copy-pasted, not imported** — no dependency lock-in, AI can modify them freely
- **Radix provides accessibility** — ARIA attributes, keyboard navigation built in
- **Tailwind provides styling** — utility classes are easier for AI to generate than CSS modules
- **v0 by Vercel** proved that constraining AI to shadcn/ui dramatically improves output quality

opi-send's component list (25+ Radix components) demonstrates the full breadth of what shadcn/ui offers. The AI builder should generate from the same component set.

### 4. Form Handling: TanStack Form + Zod

**opi-send:** Uses TanStack Form for complex multi-field forms with Zod validation. This is sophisticated — field-level validation, dependent fields, async validation.

**Blueprint:** Generated apps would use simpler form patterns. TanStack Form is overkill for AI-generated CRUD forms. The AI should generate native React form handling with Zod validation for simple cases, and TanStack Form only for complex forms.

**Key insight:** The AI builder should have two form generation modes:
1. Simple: native `<form>` + `useActionState` + Zod (for basic CRUD)
2. Complex: TanStack Form + Zod (for multi-step, dependent fields)

### 5. Hosting Philosophy: Both "Own the Metal"

**opi-send:** No Vercel. Self-hosted with 1Password for secrets management. This was an intentional choice — the developers chose to avoid vendor lock-in.

**Blueprint:** k3s on bare metal. Same philosophy, industrial scale.

**Key insight:** opi-send proves that Next.js 16 can be self-hosted without Vercel. The blueprint's k3s deployment should follow opi-send's patterns — `next build` → Docker container → k3s pod. The main difference is automation: opi-send deploys manually, the builder deploys automatically on "Publish."

---

## What opi-send Teaches the AI Builder

### 1. The AI Should Generate opi-send-Style Code
opi-send's stack is exactly what modern AI coding tools are trained on:
- Next.js App Router patterns (server components, server actions, route handlers)
- Supabase client initialization patterns
- shadcn/ui component usage
- Zod schema validation
- Tailwind utility classes

The AI builder should generate code that looks like opi-send code. This maximizes LLM accuracy because the training data is rich with these patterns.

### 2. Drizzle vs PostgREST Is a Spectrum
For the AI builder:
- **Simple apps:** PostgREST auto-generated API (zero code for CRUD)
- **Complex apps:** Generate Drizzle ORM code for explicit queries
- **Power users:** Let them drop into raw SQL via Supabase SQL editor

This gives users a progression path: start with no-code (PostgREST), graduate to type-safe ORM (Drizzle), eventually write raw SQL. All on the same Postgres database.

### 3. Domain-Specific Libraries Are the Value Add
opi-send's unique value comes from domain-specific libraries:
- Tiptap for email editing
- React Email for email rendering
- React Flow for automation builders
- Stripe for billing

The AI builder should have a "library recommendation engine" — when the AI detects the user is building an email tool, it should suggest/generate Tiptap + React Email code. When building a workflow tool, it should use React Flow. This domain awareness is how the AI builder goes from generic to genuinely useful.

### 4. TypeScript Strict Mode Is Non-Negotiable
opi-send uses TypeScript strict mode. The AI builder should generate TypeScript (not JavaScript) with strict mode enabled. This:
- Catches AI-generated bugs at compile time
- Enables the self-healing loop (type errors → auto-fix)
- Improves code quality without user effort
- Makes Drizzle/Zod type inference work properly

---

## Conclusion

opi-send's stack is the **ideal target for AI code generation.** It represents the modern best-practice stack that LLMs are most familiar with. The AI builder should generate apps that look structurally similar to opi-send — Next.js App Router, Supabase auth, shadcn/ui components, Tailwind styling, Zod validation, TypeScript strict mode.

The key differences between building a SaaS (opi-send) and building an AI builder are:
1. **Data access:** ORM for developer-built → PostgREST for AI-generated
2. **Deployment:** Manual for SaaS → Automated pipeline for builder
3. **Preview:** Dev server for SaaS → WebContainers for builder
4. **Scale:** Single-tenant SaaS → Multi-tenant platform

But the core technology choices are validated by opi-send's existence. It's proof that this stack works in production, self-hosted, without Vercel.
