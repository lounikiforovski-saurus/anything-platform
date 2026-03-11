# 🚀 Hostopia AI Builder: Team Onboarding & Research Guide

Welcome to the **Perpetual SKU Factory** (AI Anything Builder) research repository. 

This directory contains over 60 exhaustive teardowns of competitor architectures, bare-metal sandboxing blue prints, and deep-think strategic analyses. **Do not attempt to read every file.** 

Use this guide to navigate the research based on your role and current sprint objectives.

---

## 🛑 1. REQUIRED READING (Everyone)
*Before writing any code or designing any architecture, everyone on the team must read these three documents to understand our "North Star."*

1. **`AI Architecture Blueprint for Perpetual SKU Factory.md`** 
   * *What it is:* The definitive, ruthless evaluation of our path forward, resulting from the Deep Think analysis. It highlights the fatal flaws of standard K8s/Docker and SQLite for our use case and defines the target architecture (WebContainers + Firecracker, Appwrite, AST Diffing).
2. **`MASTER_ARCHITECTURAL_CATALOG.md`** 
   * *What it is:* An objective, 8-dimension matrix comparing our plan against every major competitor. It outlines the "What," "Why," and "Trade-offs" for every major engineering decision.
3. **`PLAN.md`** 
   * *What it is:* Our original foundational plan, detailing the pre-built apps, the generic architecture, and the overall scope of the MVP. *(Note: Read this with the context that some architectural components have been superseded by the AI Blueprint).*

---

## 🛠️ 2. ROLE-SPECIFIC READING TRACKS

Depending on your engineering domain, consume the following deep-dive reports.

### ⚙️ Infrastructure & Backend Team
*Focus: How we securely execute untrusted AI code and manage multi-tenant state on bare metal.*
* **`Bare-Metal AI App Builder Blueprint.md`** - The overarching guide to owning the metal.
* **`SQLite Performance Limits Analysis.txt`** - Why SQLite WAL mode fails under 24/7 AI polling and multi-tenant scaling.
* **`Hostinger Horizons Backend Investigation.txt`** - How a competitor successfully packaged hosting + AI.
* **`Fly.io Architecture Teardown.md`** & **`Runloop Bare-Metal Sandbox Teardown.md`** - How to use Firecracker MicroVMs and snapshotting for instant, secure execution.

### 🧠 AI & LLM Orchestration Team
*Focus: How we handle context windows, prompt routing, and safe code modification.*
* **`AI Routing and Caching Strategy.txt`** - How to protect gross margins by routing prompts through an AI Gateway and leveraging semantic caching.
* **`AI Code Modification via AST Diffing.txt`** - How to prevent "lazy coding" by forcing the LLM to output AST-validated Unified Diffs instead of regex replacements.
* **`Low-Code AI Workflow Diffing Analysis.txt`** - How enterprise tools (Appsmith/ToolJet) safely diff complex visual graphs.
* **`Crafting App Engine Code Generation System Prompt.txt`** - The meta-prompt engineering required to enforce our strict Fastify/React stack.

### 🔒 Security, Compliance & Auth Team
*Focus: How we secure the platform, prevent prompt injection, and pass Telco audits.*
* **`Architectural Specification & Deep Threat Model.txt`** - Threat modeling for our `iframe` embedding strategy and cross-origin JWTs.
* **`Reverse-Engineering AI Authentication Generation.txt`** - How to force the AI to write secure, RBAC-compliant auth wrappers.
* **`AI Builder Security Audit Report.md`** - A teardown of how competitors handle security boundaries.
* **`Agentic Protocol Architecture Teardown.md`** - Deep dive into securing the Model Context Protocol (MCP) against "Confused Deputy" attacks.

### 🎨 Frontend & UX Team
*Focus: How we mask backend complexity and handle accessibility.*
* **`AI Builder UX_ Telemetry & Failures.txt`** - How to gracefully show "Thinking" states and handle backend crashes without exposing raw stack traces to non-technical SMBs.
* **`UX Friction Teardown_ Competitor Analysis.md`** - Where competitors lose users (e.g., manual DNS setup, complicated Stripe integrations).
* **`Competitor Accessibility & WCAG Audit.md`** - Why AI statistically fails at WCAG 2.2 AA and how we can enforce DOM ordering natively.

### 📈 Product & Go-To-Market (GTM)
*Focus: How we package, price, and sell this through the Telco channel.*
* **`Website Builder Channel Strategy Analysis.md`** - The definitive playbook for Telco B2B2C distribution, 1-click provisioning, and white-labeling.
* **`Competitor Cost-to-Serve Teardown.md`** - How to structure our pricing/compute paywall to avoid negative margins.
* **`AI Builder Churn & Scalability Analysis.md`** - Why users churn from AI builders (The "Logic Wall") and how we prevent it.

---

## 📚 3. REFERENCE LIBRARY (Read as Needed)
If you are working on a specific competitor parity feature, refer to these raw teardowns:
* **Consumer App Builders:** `boltnew.md`, `lovabledevnew.md`, `v0vercel.md`, `Base44 phases 1-6.md`
* **Enterprise Platforms:** `AppSmith Agentic Architecture Teardown.md`, `Reverse-Engineering ToolJet_s Agent Architecture.md`, `NocoBase Agentic UX Extraction.md`
* **Developer Agents:** `replit.md`, `openhands.md`, `Cursor` related files.

*(Note: Older variants, duplicate extractions, and superseded architectural plans have been moved to the `reference/` sub-directory to keep this root folder clean for NotebookLM uploads.)*

---
*End of Guide. If you encounter missing intelligence during development, refer to `FUTURE_RESEARCH_PROMPTS.md` for pre-written deep-research queries.*