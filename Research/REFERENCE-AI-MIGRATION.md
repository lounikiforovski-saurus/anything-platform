# AI Provider Migration: OpenAI → Claude API

> This document describes how to migrate all AI flows from OpenAI to Claude (Anthropic) API while keeping the `@ab/connectors/ai` abstraction provider-agnostic.

---

## Current OpenAI Usage (from prototypes)

| Flow | OpenAI Feature | Model | Output |
|------|---------------|-------|--------|
| Conversational analysis | Responses API | gpt-5.x | JSON (intent classification) |
| UI/UX planning decision | Responses API | gpt-5.x | JSON (needsPlanning, complexity) |
| Project overview | Responses API | gpt-5.x | Plain text (2-3 sentences) |
| UI/UX design plan | Responses API | gpt-5.x | JSON (colors, pages, tech decisions) |
| Code generation (new app) | Responses API | gpt-5.x-codex | Markdown with \`\`\`filepath: blocks |
| Code modification | Responses API | gpt-5.x-codex | Markdown with \`\`\`filepath: blocks |
| AI code validation | Responses API + JSON mode | gpt-5.x-codex | JSON (valid, issues[]) |
| Summary generation | Responses API | gpt-5.x | Plain text (1-3 sentences) |
| Question answering | Responses API | gpt-5.x | Plain text |
| Clarification check | Responses API | gpt-5.x | JSON (needsClarification, questions[]) |
| Design system extraction | Responses API | gpt-5.x | JSON (colors, typography, patterns) |
| Block schema editing | Chat Completions + JSON mode | gpt-4o | JSON (block tree) |
| Vision site bootstrap | Vision API | gpt-4o | JSON (complete block tree) |
| Image generation | Flux 2 Pro (BFL.ai) | flux-2-pro | Image URL (unchanged) |

---

## Claude API Mapping

### Model Selection

| OpenAI Model | Claude Equivalent | Rationale |
|-------------|-------------------|-----------|
| gpt-5.x (analysis, planning) | **claude-sonnet-4-6** | Fast, cost-effective for classification and short outputs |
| gpt-5.x-codex (code gen) | **claude-opus-4-6** | Best coding model, extended thinking for complex generation |
| gpt-5.x-codex (validation) | **claude-sonnet-4-6** | Validation is structured checklist, doesn't need Opus |
| gpt-4o (site builder chat) | **claude-sonnet-4-6** | Block editing is well-constrained, Sonnet is sufficient |
| gpt-4o (vision) | **claude-sonnet-4-6** | Claude vision works via image content blocks |

**Cost-aware defaults** (configurable via env):
- `AI_MODEL_FAST=claude-sonnet-4-6` — analysis, planning, validation, summaries, Q&A
- `AI_MODEL_CODE=claude-opus-4-6` — code generation, complex modifications
- `AI_MODEL_VISION=claude-sonnet-4-6` — image analysis for site bootstrap

### Feature Mapping

| OpenAI Feature | Claude Equivalent | Notes |
|---------------|-------------------|-------|
| `openai.responses.create()` | `anthropic.messages.create()` | Different request format |
| `previous_response_id` | Explicit message history array | Must manage conversation state ourselves |
| `text.format.type: 'json_object'` | `output_config.format.type: 'json_schema'` | Claude validates against schema at generation time |
| `instructions` parameter | `system` parameter (top-level) | Not a message role |
| Vision (image URL in content) | Image content block (`type: 'image'`) | Base64 or URL source |
| Streaming | SSE streaming (`stream: true`) | Same SSE pattern, different event types |
| N/A | Extended thinking (`thinking.type: 'adaptive'`) | New capability for code gen |
| N/A | Prompt caching (`cache_control: 'ephemeral'`) | Cache large system prompts for 5 min |

---

## Provider-Agnostic AI Connector Design

### `@ab/connectors/ai` SDK Interface

The connector SDK exposes provider-agnostic methods. The AI connector service translates to the active provider.

```javascript
// @ab/connectors/ai - what apps call

export const ai = {
  // Structured JSON output (analysis, validation, design extraction)
  async json({ system, prompt, schema, model, history }) → object,

  // Plain text output (summaries, Q&A, planning overview)
  async text({ system, prompt, model, history }) → string,

  // Code generation (returns raw text with ```filepath: markers)
  async code({ system, prompt, model, history, thinking }) → string,

  // Vision (image → structured output)
  async vision({ system, prompt, image, schema, model }) → object,

  // Streaming text (for chat UX)
  async stream({ system, prompt, model, history, onChunk }) → string,
};
```

### AI Connector Service (provider translation)

```javascript
// connectors/ai-service/src/providers/claude.js

import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

export async function jsonCompletion({ system, prompt, schema, model, history }) {
  const response = await client.messages.create({
    model: model || process.env.AI_MODEL_FAST,
    max_tokens: 4096,
    system,
    messages: buildMessages(history, prompt),
    output_config: {
      format: {
        type: 'json_schema',
        schema,
      },
    },
  });

  return JSON.parse(response.content[0].text);
}

export async function codeCompletion({ system, prompt, model, history, thinking }) {
  const response = await client.messages.create({
    model: model || process.env.AI_MODEL_CODE,
    max_tokens: 128000,   // Large output for full app generation
    system,
    messages: buildMessages(history, prompt),
    // Extended thinking for complex code generation
    ...(thinking ? { thinking: { type: 'adaptive' } } : {}),
  });

  // Extract text content (skip thinking blocks)
  return response.content
    .filter(block => block.type === 'text')
    .map(block => block.text)
    .join('');
}

export async function visionCompletion({ system, prompt, image, schema, model }) {
  const response = await client.messages.create({
    model: model || process.env.AI_MODEL_VISION,
    max_tokens: 16384,
    system,
    messages: [{
      role: 'user',
      content: [
        {
          type: 'image',
          source: image.startsWith('data:')
            ? { type: 'base64', media_type: 'image/jpeg', data: image.split(',')[1] }
            : { type: 'url', url: image },
        },
        { type: 'text', text: prompt },
      ],
    }],
    output_config: schema ? {
      format: { type: 'json_schema', schema },
    } : undefined,
  });

  return schema
    ? JSON.parse(response.content[0].text)
    : response.content[0].text;
}

// Convert app-level history to Claude message format
function buildMessages(history, currentPrompt) {
  const messages = [];

  if (history?.length) {
    for (const msg of history) {
      messages.push({
        role: msg.role === 'user' ? 'user' : 'assistant',
        content: msg.content,
      });
    }
  }

  messages.push({ role: 'user', content: currentPrompt });
  return messages;
}
```

### OpenAI Provider (kept intact)

```javascript
// connectors/ai-service/src/providers/openai.js

import OpenAI from 'openai';

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export async function jsonCompletion({ system, prompt, schema, model, history }) {
  const response = await client.responses.create({
    model: model || process.env.AI_MODEL_FAST,
    instructions: system,
    input: prompt,
    text: { format: { type: 'json_object' } },
    ...(history?.lastResponseId ? { previous_response_id: history.lastResponseId } : {}),
  });

  return {
    result: JSON.parse(response.output_text),
    responseId: response.id,  // For conversation continuity
  };
}

// ... similar pattern for other methods
```

### Provider Router

```javascript
// connectors/ai-service/src/providers/index.js

import * as claude from './claude.js';
import * as openai from './openai.js';

const providers = { claude, openai };

export function getProvider() {
  const name = process.env.AI_PROVIDER || 'claude';
  return providers[name];
}
```

---

## Flow-by-Flow Migration

### 1. Conversational Analysis

**Before (OpenAI):**
```javascript
const response = await openai.responses.create({
  model: 'gpt-5.x',
  instructions: analysisPrompt,
  input: userMessage,
});
const result = JSON.parse(response.output_text.match(/\{[\s\S]*\}/)[0]);
```

**After (Claude via connector):**
```javascript
const result = await ai.json({
  system: analysisPrompt,
  prompt: userMessage,
  schema: {
    type: 'object',
    properties: {
      messageType: { type: 'string', enum: ['answer','question','request','mixed','topic_change','off_topic','reverse_engineering','clarification_question'] },
      answeredQuestionId: { type: 'string' },
      requestType: { type: 'string', enum: ['build','modify','fix'] },
      suggestedAction: { type: 'string' },
    },
    required: ['messageType', 'suggestedAction'],
  },
  history: conversationHistory,
});
// No regex parsing needed - schema guarantees valid JSON
```

**Improvement**: Claude's `json_schema` output guarantees valid JSON matching the schema. No more regex extraction (`content.match(/\{[\s\S]*\}/)`) or fallback handling for malformed JSON.

### 2. Code Generation

**Before (OpenAI):**
```javascript
const response = await openai.responses.create({
  model: 'gpt-5.x-codex',
  instructions: codeGenPrompt,
  input: userRequest,
  previous_response_id: lastResponseId,
});
const code = response.output_text;
const responseId = response.id; // Store for continuity
```

**After (Claude via connector):**
```javascript
const code = await ai.code({
  system: codeGenPrompt,
  prompt: userRequest,
  history: conversationMessages, // Explicit history instead of previous_response_id
  thinking: true,               // Extended thinking for complex generation
});
```

**Key difference**: No `previous_response_id`. Conversation continuity is managed by passing the full message history. Use **prompt caching** on the system prompt (which contains the large technical stack rules) to reduce costs.

### 3. Vision Site Bootstrap

**Before (OpenAI):**
```javascript
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [{
    role: 'user',
    content: [
      { type: 'image_url', image_url: { url: imageUrl } },
      { type: 'text', text: 'Analyze this image and generate a block tree...' },
    ],
  }],
  response_format: { type: 'json_object' },
});
```

**After (Claude via connector):**
```javascript
const blockTree = await ai.vision({
  system: 'You are a web designer AI. Analyze the image and generate a block tree.',
  prompt: blockSchemaHint + '\n\nGenerate a complete block tree matching this image.',
  image: imageUrl,
  schema: blockTreeSchema,
});
// Returns parsed JSON directly, schema-validated
```

### 4. AI Code Validation

**Before (OpenAI):**
```javascript
const response = await openai.responses.create({
  model: 'gpt-5.x-codex',
  instructions: 'Return ONLY valid JSON',
  input: validationPrompt,
  text: { format: { type: 'json_object' } },
});
const result = JSON.parse(response.output_text);
```

**After (Claude via connector):**
```javascript
const result = await ai.json({
  system: 'You are a code validation expert.',
  prompt: validationPrompt,
  schema: {
    type: 'object',
    properties: {
      valid: { type: 'boolean' },
      issues: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            severity: { type: 'string', enum: ['critical', 'warning', 'suggestion'] },
            category: { type: 'string' },
            description: { type: 'string' },
            file: { type: 'string' },
          },
          required: ['severity', 'category', 'description'],
        },
      },
    },
    required: ['valid', 'issues'],
  },
});
// Guaranteed valid structure, no try/catch JSON parsing needed
```

---

## Conversation Continuity: `previous_response_id` → Explicit History

The biggest architectural change. OpenAI's Responses API chains responses implicitly. Claude requires explicit history.

### Trade-offs vs `previous_response_id`

| Aspect | OpenAI `previous_response_id` | Claude explicit history |
|--------|-------------------------------|------------------------|
| **Token cost per call** | Only new message sent | Full history re-sent (mitigated by prompt caching) |
| **AI sees full context** | Yes (server-stored chain) | Yes (you send it) |
| **Context control** | Opaque - can't prune or edit | Full control - prune, summarize, inject updated state |
| **Long conversations** | Unlimited (server stores everything) | Needs sliding window + summary strategy |
| **Provider portability** | Locked to OpenAI | Works with any provider |
| **Resilience** | Chain breaks if response lost | Self-contained per request |
| **Updated project state** | Can't inject mid-chain | Can inject latest code files each call |

**For the Anything Builder use case, explicit history is equally effective** because:
- Conversations are project-scoped (20-50 messages typical, not thousands)
- The heavy context is **current project source code** (rebuilt per request anyway), not chat history
- You actively *want* to inject updated project state after each code generation
- Prompt caching closes the cost gap on system prompts

### Strategy: Sliding Window + Summary

For sessions that grow beyond ~20 messages, compress old history to stay within context limits:

```javascript
// context.service.js

const RECENT_MESSAGES = 15;   // Keep last 15 verbatim, summarize the rest

async function buildConversationHistory(projectId) {
  const allMessages = await db('chat_messages')
    .where({ project_id: projectId })
    .orderBy('timestamp', 'asc');

  if (allMessages.length <= RECENT_MESSAGES) {
    // Short conversation: send everything
    return allMessages.map(msg => ({
      role: msg.type === 'USER' ? 'user' : 'assistant',
      content: msg.content,
    }));
  }

  // Long conversation: summary of old + recent verbatim
  const oldMessages = allMessages.slice(0, -RECENT_MESSAGES);
  const recentMessages = allMessages.slice(-RECENT_MESSAGES);

  // Get or generate summary of old conversation
  const summary = await getOrCreateConversationSummary(projectId, oldMessages);

  return [
    // Injected summary as first assistant message
    { role: 'assistant', content: `[Previous conversation summary]\n${summary}` },
    // Recent messages verbatim
    ...recentMessages.map(msg => ({
      role: msg.type === 'USER' ? 'user' : 'assistant',
      content: msg.content,
    })),
  ];
}

async function getOrCreateConversationSummary(projectId, messages) {
  const context = await db('conversation_contexts')
    .where({ project_id: projectId })
    .first();

  // Re-summarize if message count has grown since last summary
  if (context.summary_covers_count < messages.length) {
    const summary = await ai.text({
      system: 'Summarize this conversation history concisely. Focus on: what the user asked for, what was built, key design decisions made, and any unresolved items.',
      prompt: messages.map(m => `${m.type}: ${m.content}`).join('\n'),
      model: 'fast',
    });

    await db('conversation_contexts')
      .where({ project_id: projectId })
      .update({
        conversation_summary: summary,
        summary_covers_count: messages.length,
      });

    return summary;
  }

  return context.conversation_summary;
}
```

### Project Context Injection (advantage over `previous_response_id`)

Unlike OpenAI's frozen chain, explicit history lets us inject **current project state** on every call:

```javascript
async function buildAIMessages(projectId, history, userPrompt) {
  // 1. Conversation history (with sliding window)
  const conversationMessages = await buildConversationHistory(projectId);

  // 2. Current project source code (always fresh, not stale from chain)
  const projectFiles = await filesystem.getSourceFiles(projectId);
  const codeContext = projectFiles
    .map(f => `**${f.path}**\n\`\`\`\n${f.content}\n\`\`\``)
    .join('\n\n');

  // 3. Known requirements and design decisions (from context service)
  const context = await db('conversation_contexts')
    .where({ project_id: projectId })
    .first();

  // Inject project state as the latest context before the user's new message
  return [
    ...conversationMessages,
    {
      role: 'user',
      content: `[Current project code]\n${codeContext}\n\n[Known requirements]\n${JSON.stringify(context.known_requirements)}\n\n${userPrompt}`,
    },
  ];
}
```

This is actually **better** than `previous_response_id` for code generation - the AI always sees the latest file contents after any edits, not a potentially stale snapshot frozen in the conversation chain.

### Cost Optimization: Prompt Caching

Claude supports **prompt caching** - mark large, static content as cacheable to avoid re-tokenizing on every call:

```javascript
const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 4096,
  system: [
    {
      type: 'text',
      text: technicalStackPrompt,        // ~83 lines, rarely changes
      cache_control: { type: 'ephemeral' },  // Cached for 5 minutes
    },
    {
      type: 'text',
      text: blockSchemaHint,             // Block schema, static per request type
      cache_control: { type: 'ephemeral' },
    },
  ],
  messages: buildMessages(history, prompt),
});
```

Caching reduces input token costs by 90% for repeated system prompts within a 5-minute window. Combined with the sliding window strategy, total per-call cost is comparable to OpenAI's `previous_response_id` approach.

---

## Extended Thinking for Code Generation

Claude's extended thinking is a significant advantage for code generation - the model reasons through the problem before writing code.

```javascript
// For new app generation (complex, benefits from thinking)
const response = await client.messages.create({
  model: 'claude-opus-4-6',
  max_tokens: 128000,
  thinking: { type: 'adaptive' },  // Opus decides when to think
  system: codeGenSystemPrompt,
  messages: [{ role: 'user', content: userRequest }],
});

// Thinking blocks are not shown to user but improve output quality
const codeOutput = response.content
  .filter(b => b.type === 'text')
  .map(b => b.text)
  .join('');
```

**When to enable thinking:**
- New app generation (complex, many files) → `thinking: { type: 'adaptive' }`
- Code modifications → `thinking: { type: 'adaptive' }` (Opus decides if needed)
- Simple analysis/classification → no thinking (Sonnet, fast responses)

---

## Image Generation (Unchanged)

Flux 2 Pro (BFL.ai) is not an OpenAI service. It remains unchanged:

```javascript
// Still uses BFL API directly
// Not affected by OpenAI → Claude migration
const imageUrl = await flux.generateImage(prompt);
```

---

## Environment Variables

```env
# Provider selection
AI_PROVIDER=claude                        # 'claude' or 'openai'

# Claude (primary)
ANTHROPIC_API_KEY=sk-ant-...
AI_MODEL_FAST=claude-sonnet-4-6           # Analysis, planning, validation, summaries
AI_MODEL_CODE=claude-opus-4-6             # Code generation
AI_MODEL_VISION=claude-sonnet-4-6         # Image analysis

# OpenAI (kept as fallback)
OPENAI_API_KEY=sk-...
OPENAI_MODEL_FAST=gpt-5.1
OPENAI_MODEL_CODE=gpt-5.1-codex

# Image generation (unchanged)
BFL_API_KEY=...
```

---

## Migration Checklist

- [ ] Add `@anthropic-ai/sdk` to `connectors/ai-service/` dependencies
- [ ] Create `providers/claude.js` with all completion methods
- [ ] Create `providers/openai.js` preserving current patterns
- [ ] Create `providers/index.js` router (env-based provider selection)
- [ ] Update `@ab/connectors/ai` SDK interface to be provider-agnostic
- [ ] Replace `previous_response_id` with explicit message history in context service
- [ ] Add prompt caching for large system prompts (technical stack, block schema)
- [ ] Define JSON schemas for all structured outputs (analysis, validation, design)
- [ ] Enable extended thinking for code generation calls
- [ ] Update all prompts: `instructions` → `system` parameter
- [ ] Remove regex JSON extraction (`match(/\{[\s\S]*\}/)`) - use `json_schema` output
- [ ] Add retry logic with exponential backoff for Claude rate limits
- [ ] Test vision flow with Claude image content blocks
- [ ] Verify streaming works for chat UX (SSE event type differences)
- [ ] Update PLAN.md sprint tasks to reference Claude API patterns
