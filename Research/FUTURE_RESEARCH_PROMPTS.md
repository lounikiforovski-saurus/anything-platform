# STRATEGIC IMPERATIVES & FUTURE RESEARCH NEEDS

Based on an impartial review of the current intelligence gathered, there are several highly specific areas where data is either missing, opaque, or requires deeper technical validation. The following are the five core research gaps, accompanied by exhaustive Gemini Deep Research prompts designed to extract the necessary intelligence.

---

### 1. The Reality of Hostinger's "Integrated Backend" (BaaS Architecture)
**The Gap:** Hostinger Horizons is the closest commercial parallel to the Telco/Hosting strategy, but their database architecture is highly obfuscated. They offer full-stack capabilities via an "Integrated Backend" without exposing connection strings. We must determine exactly how Hostinger provisions this data layer at scale to validate our own MariaDB/SQLite bifurcation strategy.

**Gemini Deep Research Prompt:**
> "Execute a deep technical investigation into the underlying architecture of 'Hostinger Horizons' and its 'Integrated Backend' service. Search technical documentation, engineering blogs, developer forums (Reddit/HackerNews), and architectural teardowns. Your objective is to determine exactly what database engine powers this backend. Are they spinning up shared SQLite files, utilizing a massive multi-tenant Postgres cluster with Row-Level Security, or using a proprietary NoSQL layer? Furthermore, investigate how they handle multi-tenancy (shared vs. isolated instances) and estimate the cost-to-serve for this specific component. Deliver a structured report detailing their provisioning methodology, limitations for end-users regarding data export, and how their approach compares technically and economically to a bifurcated MariaDB/SQLite architecture."

---

### 2. Code Mutability & AST Diffing in Proprietary Workflows
**The Gap:** We have deep intelligence on how consumer tools (Lovable/Cursor) utilize AST-diffing to modify React code safely. However, we lack data on how enterprise orchestration platforms (Appsmith/ToolJet/NocoBase) handle AI-driven modifications to *complex visual workflow nodes* and underlying JavaScript logic without breaking the visual graph.

**Gemini Deep Research Prompt:**
> "Investigate the code mutability and diffing engines utilized by enterprise low-code platforms, specifically Appsmith, ToolJet, and NocoBase, when their AI agents generate or modify complex background workflows. Search developer documentation, GitHub pull requests, and technical teardowns to determine how these platforms apply 'surgical diffs' to proprietary visual workflow nodes. Do they re-generate the entire workflow JSON file on every AI prompt, do they use JSON-patching algorithms, or do they possess proprietary Abstract Syntax Tree (AST) diffing algorithms specifically for visual node graphs? Provide a comparative analysis of their approaches to preventing AI 'lazy coding' or structural degradation when modifying existing complex logic."

---

### 3. Telemetry UX and the "Thinking" State for Non-Technical Users
**The Gap:** Developer tools (Replit, OpenHands) show raw terminal logs to build trust. Consumer tools (Bolt, v0) hide them behind "Thinking..." spinners, but users churn when these spinners hang indefinitely due to backend errors. We need an analysis of the optimal UX patterns for gracefully presenting complex backend failures.

**Gemini Deep Research Prompt:**
> "Conduct a UX research analysis focusing on how modern AI application builders (specifically Vercel v0, Bolt.new, Lovable.dev, and Base44) present backend telemetry and AI execution states to non-technical users. Search UX teardowns, user complaint threads (Reddit/Twitter), and product updates. Detail the specific UI patterns they use when an AI agent is executing a long-running background task (e.g., 'Thinking' spinners, progress bars, chronological steps). Most importantly, analyze how these platforms handle backend failures (e.g., a failing Edge Function or crashed dependency install). How do they mask raw stack traces while still providing enough actionable context to allow the AI to self-heal or the non-technical user to restart the process? Conclude with a set of best-practice UX recommendations for displaying failure states without inducing user churn."

---

### 4. Scalability Limits of SQLite in Multi-Tenant Agentic Environments
**The Gap:** Our Current Plan relies on SQLite for generated custom apps to achieve instant provisioning. However, the AI Blueprint notes that background AI agents operating 24/7 create unpredictable, concurrent read/write bursts. We need to stress-test the viability of SQLite's WAL mode in this specific context.

**Gemini Deep Research Prompt:**
> "Perform a rigorous technical analysis of the performance limits of SQLite—specifically the `better-sqlite3` library operating in WAL (Write-Ahead Logging) mode—when deployed in a multi-tenant Node.js/Fastify environment and subjected to concurrent read/write operations from 24/7 background AI agents. Search database engineering benchmarks, GitHub issues, and architectural case studies. Identify the exact breaking points or volume thresholds where SQLite file-locking becomes a catastrophic bottleneck for a generated application. Evaluate the impact of continuous agent polling and frequent schema mutations on SQLite's durability. Conclude by outlining the specific architectural triggers that would necessitate migrating a custom app off SQLite and onto a clustered database like PostgreSQL/MariaDB."

---

### 5. Identity and RBAC Mapping in Automated Code Generation
**The Gap:** Our Current Plan relies on a centralized `ab_identity` service. We know how Appsmith uses robust RBAC. But how do generative platforms like v0 or Bolt actually instruct the LLM to wire up *custom* Role-Based Access Control within the generated code so that it securely maps back to a centralized identity provider?

**Gemini Deep Research Prompt:**
> "Reverse-engineer the prompt engineering techniques and system instructions used by generative AI platforms like Lovable.dev and Vercel v0 to automatically generate secure authentication wrappers, protected routes, and Role-Based Access Control (RBAC) logic. Search through leaked system prompts, open-source prompt libraries, and developer guides for these platforms. How exactly do these platforms force the LLM to output client-side code that correctly interfaces with an external Backend-as-a-Service (like Supabase Auth or a custom Identity service) without hallucinating insecure fallback logic? Provide examples of the specific prompt constraints and architectural patterns they use to guarantee that the generated React/Next.js code respects the centralized platform's identity provider and correctly maps tenant/org permissions."
