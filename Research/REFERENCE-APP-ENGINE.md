# App Engine - Prototype Reference

> Source: `ai-site-builder/demo-build-me-anything-app` branch
> Purpose: AI-powered app builder that generates complete React applications from chat
> Stack: NestJS (TypeScript), Prisma ORM (PostgreSQL), React + TypeScript frontend, Socket.IO, Vite, Tailwind, better-sqlite3 document DB for generated apps

This document captures every proven pattern from the Anything Builder prototype. The new implementation will be built from scratch but must replicate and improve upon these AI pipeline patterns.

---

## Architecture Summary

The Anything Builder lets users describe an app in natural language, then generates a complete React + Tailwind + SQLite application. A chat interface drives the entire workflow. The backend orchestrates a multi-stage AI pipeline (analysis → planning → design → code generation → validation → refinement) with automatic rollback on failure.

**Core flow**: User describes app → AI analyzes intent → Clarification if needed → Planning → Design system → Code generation → Validation → Build → Preview in iframe

---

## AI Pipeline Architecture

### High-Level Flow

```
User Message
     │
     ▼
┌─────────────────────┐
│  Conversational      │ ← OpenAI gpt-5.x
│  Analysis            │   Classifies message type (8 types)
│  (analysis.service)  │   Detects scope, security threats
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Decision Logger     │ ← Stores every AI decision in DB
│  (context.service)   │   For debugging and learning
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Message Router      │ ← Routes to handler by message type
│  (message-handler)   │   answer | question | request | mixed | ...
└─────────┬───────────┘
          │
    ┌─────┴──────┐
    │            │
    ▼            ▼
 Questions    Requests
 (answer)     (build/modify/fix)
    │            │
    │      ┌─────┴──────────────┐
    │      │ Clarification      │ ← Max 3 questions
    │      │ Check              │   Conversational, not technical
    │      └─────┬──────────────┘
    │            │
    │      ┌─────┴──────────────┐
    │      │ Planning Decision  │ ← Only for complex/new apps
    │      │ (shouldDoUIUX...)  │   Simple changes skip planning
    │      └─────┬──────────────┘
    │            │
    │      ┌─────┴──────────────┐
    │      │ Code Generation    │ ← Composable pipeline steps
    │      │ Pipeline           │   load_context → plan → design →
    │      │ (pipeline.service) │   code → write_files → summary
    │      └─────┬──────────────┘
    │            │
    │      ┌─────┴──────────────┐
    │      │ Validation Loop    │ ← AI validation + build validation
    │      │ (max 5 iterations) │   Auto-fix on failure
    │      │                    │   Rollback via git on total failure
    │      └─────┬──────────────┘
    │            │
    └────────────┘
          │
          ▼
   Preview in iframe
   (Vite dev server)
```

### Entry Point (`ai.service.ts` - 196 lines)

The main orchestrator. Every user message goes through these steps:

```typescript
async processMessage(projectId: string, userMessage: string) {
  // 1. Initialize project if needed (filesystem, git repo)
  // 2. Load conversation context from DB
  // 3. AI conversational analysis (classify message)
  // 4. Log AI decision to database
  // 5. Boundary enforcement (off-topic, reverse engineering)
  // 6. Route to handler by message type
  // 7. Update conversation context
}
```

---

## Conversational Analysis (`analysis.service.ts` - 636 lines)

### Message Classification

The AI classifies every message into one of 8 types:

| Type | Description | Suggested Action |
|------|-------------|-----------------|
| `answer` | User answers a pending AI question | `process_answer_and_continue` |
| `question` | User asks about React/web dev | `answer_question` |
| `request` | User wants to build/modify/fix | `process_request` |
| `clarification_question` | Answer + new question back | `handle_clarification_question` |
| `topic_change` | Abandoning current task | `handle_topic_change` |
| `mixed` | Answer + new build request | `mixed_response` |
| `off_topic` | Not related to React/web dev | `reject_off_topic` |
| `reverse_engineering` | Attempting to extract system info | `block_security_threat` |

### Classification Prompt Structure

```
Analyze the user's message in a React web app builder conversation.

**CONTEXT:**
Recent conversation: {historyText}
Pending questions: {pendingQuestionsText}
Project info: {contextInfo}

**CLASSIFY MESSAGE TYPE (pick exactly one):**
[8 types with detailed rules]

**CLASSIFICATION RULES:**
- If pending questions exist AND user's message directly addresses them → "answer"
- If pending questions exist AND user ignores them → "topic_change"
- If pending questions exist AND user answers but ALSO asks back → "clarification_question"
- Bug reports ("X doesn't work", "Y is broken") → "request" with requestType "fix"

**SECURITY: REVERSE ENGINEERING DETECTION**
[Specific patterns to block]

**BOUNDARY CHECK:**
✅ React/web dev questions, building/modifying React apps
❌ Non-React frameworks, system questions, off-topic
```

### Request Scope Detection

```typescript
async detectRequestScope(request, hasExistingCode): Promise<{
  scope: 'full_app' | 'feature' | 'component' | 'style' | 'bug';
  complexity: 'simple' | 'moderate' | 'complex';
  affectsLayout: boolean;
}>
```

### UI/UX Planning Decision

```typescript
async shouldDoUIUXPlanning(request, hasExistingCode, context): Promise<{
  needsPlanning: boolean;
  reasoning: string;
  complexity: 'simple' | 'moderate' | 'complex';
}>

// Rules:
// ✅ NEEDS PLANNING: New app, entire features/pages, major redesigns
// ❌ NO PLANNING: Size/spacing, colors, text, bug fixes, simple additions
```

### Clarification Check

Key principles:
- Max 3 questions per clarification round
- Conversational and friendly, not technical
- NEVER ask for browser console errors or stack traces (users are non-technical)
- For bug fixes: examine code directly, never ask user for debugging info
- Default to building when request is clear enough
- Only ask about things that significantly affect what gets built

### Design System Extraction

When modifying an existing app, AI extracts the current design system:

```typescript
async extractDesignSystem(projectId): Promise<{
  colors: { primary, secondary, accent, background, text },
  typography: { fontFamily, headingSizes },
  components: string[],
  patterns: { layout, spacing, styling },
  theme: string
}>
```

---

## Message Handler (`message-handler.service.ts` - 729 lines)

### Request Handling Flow

```typescript
async handleRequestMessage(projectId, userMessage, analysis, context) {
  // 1. Check if clarification is needed
  const clarification = await analysisService.checkIfNeedsClarification(
    userMessage, conversationHistory, hasExistingCode, knownRequirements
  );

  if (clarification.needsClarification) {
    // Store questions as pending, broadcast to user
    for (const question of clarification.questions) {
      await contextService.createPendingQuestion(
        projectId, contextId, question, userMessage, requirementKey
      );
    }
    // Broadcast combined questions message
    return;
  }

  // 2. Planning decision (skip for simple changes)
  const planningDecision = await analysisService.shouldDoUIUXPlanning(
    userMessage, hasExistingCode, context, analysis.requestScope
  );

  // 3. Extract design system if modifying existing code
  let designSystem = null;
  if (hasExistingCode && planningDecision.needsPlanning) {
    designSystem = await analysisService.extractDesignSystem(projectId);
  }

  // 4. Execute code generation pipeline
  await pipelineService.executeCodeGeneration(projectId, {
    userMessage,
    requestType: analysis.requestType,
    hasExistingCode,
    needsPlanning: planningDecision.needsPlanning,
    designSystem,
    knownRequirements: context.knownRequirements,
  });
}
```

### Answer Handling

```typescript
async handleAnswerMessage(projectId, userMessage, analysis, context) {
  // 1. Find the pending question being answered
  // 2. Store the answer
  // 3. Update known requirements
  // 4. Check if we have enough info now
  // 5. If yes → continue with the original request (code generation)
  // 6. If no → wait for more answers
}
```

### Topic Change Handling

```typescript
async handleTopicChange(projectId, userMessage, analysis) {
  // 1. Supersede all pending questions
  // 2. Clear current task description
  // 3. Process as new request
}
```

---

## Code Generation Pipeline (`pipeline.service.ts` - 757 lines)

### Composable Steps

The pipeline assembles steps based on the request:

```typescript
// For new app (full build):
steps = ['load_context', 'plan', 'design', 'code', 'write_files', 'summary'];

// For modification:
steps = ['load_context', 'code', 'write_files', 'summary'];

// For bug fix:
steps = ['load_context', 'code', 'write_files', 'summary'];
```

### Step Details

**`load_context`**: Load all current project source files

**`plan`**: Generate project overview (2-3 sentences, max 80 words, no code)

```typescript
// planning.prompts.ts - getProjectOverviewPrompt()
"Generate a brief, high-level overview of what you'll build.
 - 2-3 sentences describing the application
 - Maximum 80 words
 - No code, no technical implementation details
 - Focus on what the user will see and interact with"
```

**`design`**: Generate UI/UX design plan with 3 decisions:

```typescript
// planning.prompts.ts - getUIUXDesignPlanPrompt()
"Create a concise design plan with exactly 3 decisions:
 1. Visual Identity: Overall style, 2-3 hex colors, font suggestion
 2. UI Structure: List of pages/sections with brief descriptions
 3. Technical Decisions: Data model needs, special APIs, key dependencies"
```

**`code`**: Generate or modify application code

For initial generation:
```typescript
// generation.prompts.ts - getCompleteAppWithDesignSystemPrompt()
// Uses gpt-5.x-codex model
// Includes technical stack rules + design system + project overview
// Output: Complete file contents with ```filepath: markers
```

For modifications:
```typescript
// generation.prompts.ts - getModificationGenerationPrompt()
// Includes current code as context
// Rules: maintain design consistency, only output changed files
// Includes design system to prevent visual regression
```

**`write_files`**: Extract files from AI response, write to filesystem

```typescript
// File extraction regex: ```filepath:path/to/file
// Each file block starts with ```filepath: and ends with ```
// Content between is the complete file
```

**`summary`**: Generate user-facing summary
- New build: 2-3 sentences, max 70 words
- Modification: 1-2 sentences, max 50 words

### Progressive Refinement Loop

```typescript
async executeWithValidation(projectId, options) {
  // Take pre-generation snapshot (git commit)
  const snapshotCommit = await filesystem.createGitCommit(projectId, 'Pre-generation snapshot');

  for (let iteration = 1; iteration <= MAX_REFINEMENT_ITERATIONS; iteration++) {
    // 1. Run code generation pipeline
    await executeSteps(steps);

    // 2. AI validation (code review by gpt-5.x-codex)
    const aiValidation = await validationService.validateCodeWithAI(files, prompt);

    if (!aiValidation.valid) {
      // Feed critical issues back to AI for auto-fix
      // Continue to next iteration
      continue;
    }

    // 3. Build validation (npm run build)
    const buildValidation = await validationService.validateBuild(projectId);

    if (!buildValidation.success) {
      // Feed build errors back to AI for auto-fix
      // Categorize errors: missingDependencies, syntaxErrors, importErrors, typeErrors
      continue;
    }

    // Both passed → success!
    return;
  }

  // All iterations failed → rollback
  await filesystem.gitResetToCommit(projectId, snapshotCommit);
}
```

**Max refinement iterations**: 5

---

## Code Generation (`generation.service.ts` - 290 lines)

### OpenAI API Usage

Uses the **Responses API** (not chat completions):

```typescript
// All calls use: openai.responses.create()
// NOT: openai.chat.completions.create()

// Models are configured via environment variables (tested with gpt-5.1, source code references gpt-5.2)
// Key parameters:
{
  model: 'gpt-5.x',              // For analysis, planning, summaries (env-configured)
  // OR
  model: 'gpt-5.x-codex',        // For code generation and validation (env-configured)
  instructions: systemPrompt,
  input: userMessage,
  previous_response_id: lastResponseId,  // Conversation continuity
  text: { format: { type: 'json_object' } },  // When JSON needed
}
```

### Conversation Continuity

```typescript
// Each project stores lastResponseId in ConversationContext
// New API calls chain from previous responses
// This gives the AI full conversation memory without resending history

const lastResponseId = await contextService.getLastResponseId(projectId);

const response = await openai.responses.create({
  model: 'gpt-5.x-codex',
  instructions: systemPrompt,
  input: userMessage,
  ...(lastResponseId ? { previous_response_id: lastResponseId } : {}),
});

// Store for next call
await contextService.storeResponseId(projectId, response.id);
```

### File Extraction

```typescript
function extractFiles(aiResponse: string): ExtractedFile[] {
  // Regex: /```filepath:(.*?)\n([\s\S]*?)```/g
  // Returns: [{ path: 'src/App.jsx', content: '...' }, ...]
}
```

### Generation Functions

| Function | Model | Purpose |
|----------|-------|---------|
| `generateProjectOverview()` | gpt-5.x | Brief app description |
| `generateUIUXDesignPlan()` | gpt-5.x | Design decisions (colors, pages, tech) |
| `generateCompleteAppWithDesignSystem()` | gpt-5.x-codex | Full app code generation |
| `generateModifications()` | gpt-5.x-codex | Modify existing code |
| `generateSummary()` | gpt-5.x | User-facing completion message |
| `generateAnswer()` | gpt-5.x | Answer React/web dev questions |

---

## Validation System (`validation.service.ts` - 243 lines)

### Two-Stage Validation

**Stage 1: AI Code Review**

```typescript
// Uses gpt-5.x-codex with JSON mode
// 8-point validation checklist:
// 1. Import/Export Integrity (CRITICAL)
// 2. Function/Variable Reference Integrity (CRITICAL)
// 3. State Hook Integrity (CRITICAL)
// 4. Prop Contract Integrity (CRITICAL)
// 5. Router Integrity (CRITICAL, if react-router used)
// 6. Context/Hook Provider Integrity (CRITICAL, if contexts used)
// 7. Data Layer Integrity (CRITICAL, if db used)
// 8. Null Safety (WARNING)

// Only CRITICAL issues fail validation
// Warnings and suggestions are acceptable
```

**Stage 2: Build Validation**

```typescript
// Runs: npm run build (with CI=true, 60s timeout)
// Parses errors into categories:
// - missingDependencies
// - syntaxErrors
// - importErrors
// - typeErrors
// - other
```

### Error Categories for AI Feedback

When feeding errors back to AI for refinement, errors are categorized so the AI can prioritize:

```typescript
categorizeErrors(errors) → {
  missingDependencies: string[],  // "Cannot find module '...'"
  syntaxErrors: string[],          // "SyntaxError: ..."
  importErrors: string[],          // "Failed to resolve import '...'"
  typeErrors: string[],            // "TS...: ..."
  other: string[]
}
```

---

## Technical Stack Prompt (`technical-stack.prompt.ts` - 83 lines)

Defines the coding rules for generated apps:

```
CODING RULES:
- JavaScript (not TypeScript) with JSX
- Tailwind CSS only (standard utility classes)
- lucide-react for icons (import { IconName } from 'lucide-react')
- Never remove packages from template package.json
- App lives under src/ directory

BUILT-IN DATABASE API:
import { db } from '@/db';

// Operations:
db.insert(collection, data)     → Promise<document>
db.find(collection, filter?, options?)  → Promise<document[]>
db.findOne(collection, id)      → Promise<document|null>
db.update(collection, id, data) → Promise<document>
db.delete(collection, id)       → Promise<{deleted: boolean}>

// Filter operators: $eq, $ne, $gt, $gte, $lt, $lte, $contains, $in
// Options: { sort: { field: 1|-1 }, limit: number }

// Seeding: Create seed.json at project root
// Format: { "collection_name": [ { ...doc1 }, { ...doc2 } ] }
// Auto-seeded on first server start (idempotent - won't re-seed if data exists)

PROTECTED FILES (never modify):
- src/db.js
- src/main.jsx
- vite.config.js
- vite-db-plugin.js
- postcss.config.js
- tailwind.config.js
```

---

## App Template (`templates/default-react-app/`)

### File Structure

```
default-react-app/
  package.json          # React 18 + Vite 5 + Tailwind 3 + better-sqlite3 + lucide-react
  vite.config.js        # React plugin + dbPlugin()
  vite-db-plugin.js     # SQLite document DB served via Vite middleware (/api/db)
  postcss.config.js     # Tailwind + autoprefixer
  tailwind.config.js    # Default Tailwind config
  index.html            # Entry HTML
  src/
    main.jsx            # React DOM root
    App.jsx             # Placeholder app (Sparkles icon + "Welcome" message)
    index.css           # Tailwind directives (@tailwind base/components/utilities)
    db.js               # Frontend DB client (fetch wrapper for /api/db)
  public/
    .gitkeep
```

### Document Database Architecture

The database is a **Vite plugin** that serves SQLite via HTTP:

```
Frontend (React)
  │
  │ fetch('/api/db', { action: 'find', collection: 'todos', filter: {...} })
  │
  ▼
vite-db-plugin.js (Vite middleware)
  │
  │ better-sqlite3 operations
  │
  ▼
data.db (SQLite file)
```

**Key design decisions:**
- Collections are auto-created on first use (no schema required)
- Documents stored as JSON blobs with auto-generated UUIDs
- Auto timestamps: `created_at`, `updated_at`
- Seeding via `seed.json` (idempotent - only seeds empty collections)
- WAL mode enabled for concurrent access
- Collection names validated: `^[a-zA-Z_][a-zA-Z0-9_]*$`
- SQL injection prevented via parameterized queries

### Package Dependencies

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "lucide-react": "^0.300.0",
    "better-sqlite3": "^11.0.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0",
    "tailwindcss": "^3.3.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0"
  }
}
```

---

## Conversation Context (`context.service.ts` - 348 lines)

### Persistent State Per Project

```typescript
// Stored in ConversationContext table:
{
  isAwaitingClarification: boolean,
  currentTaskDescription: string,
  knownRequirements: JSON,        // Accumulated from user answers
  implementedFeatures: JSON[],    // What's been built so far
  designDecisions: JSON,          // Colors, fonts, layout choices
  architecturalPatterns: JSON,    // Patterns detected in code
  lastResponseId: string,         // OpenAI conversation continuity
  messageCount: number,
  clarificationCount: number,
  codeGenerationCount: number,
}
```

### Pending Questions System

```typescript
// PendingAIQuestion table:
{
  question: string,           // The clarification question
  originalRequest: string,    // What the user originally asked
  requirementKey: string,     // Which requirement this answers
  status: 'PENDING' | 'ANSWERED' | 'SUPERSEDED',
  userAnswer: string,         // User's response
}

// Topic changes supersede all pending questions
// Answered questions update knownRequirements
```

### AI Decision Logging

Every AI decision is logged:

```typescript
// AIDecision table:
{
  userMessage: string,
  decisionType: 'INTENT_ANALYSIS' | 'CLARIFICATION' | 'CODE_GENERATION' | ...,
  primaryIntent: string,
  requiresClarification: boolean,
  needsUIUXPlanning: boolean,
  needsCodeGeneration: boolean,
  confidence: number,
  reasoning: string,
  actionsPlanned: JSON,
  actionsCompleted: JSON,
  durationMs: number,
  tokenUsage: number,
}
```

---

## Project Filesystem (`filesystem.service.ts` - 585 lines)

### Key Patterns

1. **Template cloning**: Copy `templates/default-react-app/` to `user-projects/{projectId}/` (excluding `node_modules`)
2. **Git initialization**: Each project gets its own git repo for version control / rollback
3. **npm cache warming**: On module init, install template deps once to warm npm cache
4. **File tree generation**: Builds ASCII file tree for AI context (max depth 5, excludes `node_modules` and dotfiles)
5. **Source file collection**: Collects all `.jsx`, `.js`, `.tsx`, `.ts`, `.css` files under `src/` plus `seed.json` and `package.json`

### Git-Based Versioning

```typescript
// Before code generation: snapshot
const commitHash = await createGitCommit(projectId, 'Pre-generation snapshot');

// After successful generation: commit
await createGitCommit(projectId, 'AI: Added feature X');

// On total failure: rollback
await gitResetToCommit(projectId, commitHash);

// Version tracking stored in ProjectVersion table:
{
  versionNumber: number,
  description: string,
  triggeredBy: string,
  commitHash: string,
  changedFiles: JSON,
  previousVersionId: string,
  canRevert: boolean,
}
```

---

## Build Service (`build.service.ts` - 704 lines)

### Dev Server Management

```typescript
// Each project gets its own Vite dev server
// Dynamic port allocation starting from base port 5174

async startDevServer(projectId: string) {
  // 1. npm install (with --prefer-offline for speed)
  // 2. Check for port conflicts
  // 3. Spawn Vite dev server as child process
  // 4. Wait for "ready" message on stdout
  // 5. Store server reference in Map
  // 6. Return preview URL
}

async stopDevServer(projectId: string) {
  // Kill the child process
  // Remove from Map
}
```

### Hot Module Replacement

Vite's HMR handles file changes automatically. When AI writes files, the preview updates in real-time without full page reload.

---

## WebSocket Events

### Backend → Frontend Events

| Event | Payload | Purpose |
|-------|---------|---------|
| `connected` | `{ message, userId }` | Connection acknowledged |
| `chat_history` | `{ projectId, messages[] }` | Initial message load |
| `new_message` | `{ id, projectId, type, content, timestamp }` | New chat message |
| `ai_message` | `{ content, type }` | AI progress updates |
| `ai_complete` | `{}` | AI finished processing |
| `build_status` | `{ projectId, status, message }` | Build state change |
| `code_verification_initiated` | `{}` | Validation phase started |
| `user_typing` | `{ userId, projectId, isTyping }` | Typing indicator |

### `ai_message` Types

| Type | Purpose |
|------|---------|
| `plan` | AI's plan for what it will build |
| `info` | Informational status update |
| `progress` | Ephemeral progress (replaces previous) |
| `completion` | Final summary message |
| `error` | Error message |

### Frontend → Backend Events

| Event | Payload | Purpose |
|-------|---------|---------|
| `join_project` | `{ projectId }` | Join project room |
| `leave_project` | `{ projectId }` | Leave project room |
| `send_message` | `{ projectId, content }` | Send user message |
| `typing` | `{ projectId, isTyping }` | Typing indicator |

### Room-Based Architecture

- Each project has a Socket.IO room: `project:{projectId}`
- Users join on entering builder page, leave on exit
- All broadcasts scoped to project room
- JWT authentication on WebSocket connection

---

## Database Schema (Prisma/PostgreSQL)

### Core Tables

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `users` | User accounts | email, passwordHash |
| `projects` | User projects | name, status, hasGeneratedCode |
| `chat_messages` | Chat history | projectId, type (USER/AI/SYSTEM), content |
| `builds` | Build history | projectId, status, errorMessage |

### AI Orchestration Tables

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `conversation_contexts` | Persistent AI state per project | knownRequirements (JSON), designDecisions (JSON), lastResponseId |
| `ai_decisions` | Decision audit log | decisionType, confidence, reasoning, actionsPlanned |
| `pending_ai_questions` | Clarification tracking | question, originalRequest, status (PENDING/ANSWERED/SUPERSEDED) |
| `generated_artifacts` | What was generated | artifactType (COMPONENT/FEATURE/PAGE/...), filePaths, isActive |
| `project_versions` | Version history (git-backed) | commitHash, changedFiles, canRevert |
| `security_logs` | Boundary violation logging | eventType, severity, details |

### Enums

```
BuildStatus: IDLE | BUILDING | DEPLOYING | SUCCESS | ERROR
MessageType: USER | AI | SYSTEM
DecisionType: INTENT_ANALYSIS | CLARIFICATION | UIUX_PLANNING | CODE_GENERATION | QUESTION_ANSWER | EXPLANATION | BOUNDARY_ENFORCEMENT
QuestionStatus: PENDING | ANSWERED | SUPERSEDED
ArtifactType: COMPONENT | FEATURE | DESIGN_SYSTEM | PAGE | UTILITY | CONFIGURATION
SecurityEventType: BOUNDARY_VIOLATION | RATE_LIMIT_EXCEEDED | SUSPICIOUS_ACTIVITY | UNAUTHORIZED_ACCESS
SecuritySeverity: LOW | MEDIUM | HIGH | CRITICAL
```

---

## Frontend Architecture

### Builder Page Layout

```
┌──────────────────────────────────────────────────┐
│ Header                                            │
├──────────────────┬───────────────────────────────┤
│                  │                               │
│  Chat Panel      │  Preview Panel                │
│  (collapsible)   │  (iframe to Vite dev server)  │
│                  │                               │
│  - MessageList   │  - BuildingIndicator          │
│  - TypingIndicator│  - Verification overlay      │
│  - ChatInput     │  - Placeholder (no app yet)   │
│                  │                               │
├──────────────────┴───────────────────────────────┤
│ Expand FAB (when chat collapsed)                  │
└──────────────────────────────────────────────────┘
```

### Message Types in UI

- `user` - User's messages
- `ai` - AI responses (plans, summaries, answers)
- `system` - System messages (build status, errors)
- `progress` - Ephemeral progress updates (replaced by next progress)

### State Management (Redux Toolkit)

- `chatSlice`: messages[], isTyping
- `projectSlice`: currentProject, status
- `authSlice`: user, token

### WebSocket Service Pattern

Singleton service with event emitter pattern:
- `wsService.connect(token)` - Connect with JWT
- `wsService.joinProject(projectId)` - Join project room
- `wsService.on(event, callback)` - Listen for events
- `wsService.off(event, callback)` - Remove listener
- Deduplication: same callback can't register twice for same event

---

## Key Numbers

| Metric | Value |
|--------|-------|
| `ai.service.ts` | 196 lines |
| `pipeline.service.ts` | 757 lines |
| `message-handler.service.ts` | 729 lines |
| `analysis.service.ts` | 636 lines |
| `generation.service.ts` | 290 lines |
| `validation.service.ts` | 243 lines |
| `filesystem.service.ts` | 585 lines |
| `build.service.ts` | 704 lines |
| `context.service.ts` | 348 lines |
| Max clarification questions | 3 per round |
| Max refinement iterations | 5 |
| Build validation timeout | 60s |
| OpenAI client timeout | 5 minutes |
| Retry attempts | 3 (exponential backoff) |
| Dev server base port | 5174 |
| Summary max words (new) | 70 |
| Summary max words (modify) | 50 |

---

## What to Replicate in the New Build

### Must Have (proven patterns)
1. Multi-stage AI pipeline (analysis → plan → design → code → validate → refine)
2. 8-type conversational analysis (answer, question, request, mixed, topic_change, etc.)
3. Boundary enforcement (off-topic rejection, reverse engineering detection)
4. Clarification system (max 3 questions, pending/answered/superseded states)
5. Progressive refinement loop (AI validation + build validation, max 5 iterations)
6. Git-based rollback on failure
7. Design system extraction for modification consistency
8. Composable pipeline steps (different step sets for build vs modify vs fix)
9. AI decision logging for debugging
10. WebSocket room-based architecture for real-time updates
11. Vite dev server per project with HMR
12. Document DB via Vite plugin (SQLite + JSON blobs)
13. npm cache warming for fast installs
14. File extraction from AI output (`filepath:` markers)
15. Conversation continuity via `previous_response_id`

### Improvements for New Build
1. Use Fastify instead of NestJS (vanilla JS, as per plan)
2. Use Knex.js + MariaDB instead of Prisma + PostgreSQL
3. Use `@ab/connectors/ai` instead of direct OpenAI calls
4. Use `@ab/app-foundation` for the portal framework
5. Use Zustand instead of Redux Toolkit on frontend
6. Add SSE streaming for AI responses
7. Use the `@ab/connectors` SDK in generated apps too
8. The custom app template should use `@ab/app-foundation` (SQLite mode)
9. Consider WebSocket through `@ab/app-foundation` rather than custom Socket.IO setup
10. Unify Site Builder AI patterns with Anything Builder pipeline (single AI connector)
