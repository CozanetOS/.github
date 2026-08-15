# Cozanet AI — Repository Architecture Standard

> This document defines the canonical architecture for the Cozanet AI repository (cozanet-chat).
> Every AI agent and developer must follow this standard.

## Architecture Overview

```
User Message
     ↓
API Layer (app/api/) — thin, no business logic
     ↓
AI Orchestration Layer (src/ai/)
  ├── processor.ts — main pipeline
  ├── system-prompts.ts — all prompts
  └── memory-service.ts — memory operations
     ↓
Capability Layer (src/lib/)
  ├── tools/ — Tool & Action Engine
  │   ├── registry.ts — single tool registry (ONE source of truth)
  │   ├── engine.ts — discovery + execution pipeline
  │   ├── types.ts — all tool types
  │   └── tools/ — individual tool implementations
  ├── research/ — Web Research Engine
  │   ├── orchestrator.ts — research pipeline
  │   ├── provider.ts — search provider (Tavily, swappable)
  │   └── (14 files total)
  └── memory.ts — Supabase-backed memory system
     ↓
External Providers
  ├── Groq (LLM inference, round-robin keys)
  ├── Google Gemini (LLM fallback)
  ├── Tavily (web search)
  └── Supabase (database + memory)
```

## Rules

1. **API routes are thin.** They parse the request, call a service, return the response. No business logic.
2. **Business logic lives in src/ai/ and src/lib/.** Not in app/api/.
3. **ONE tool registry.** src/lib/tools/registry.ts is the only registry. No duplicates.
4. **Tool discovery is rule-based.** Zero LLM calls for discovery. Each tool has a discover() method.
5. **Tool execution goes through the engine.** Never call tool implementations directly.
6. **tavily.ts is DEPRECATED.** Use src/lib/research/provider.ts (TavilyProvider) instead.
7. **UI uses custom CSS.** NOT Tailwind. See COZANET_CONSTITUTION.md.
8. **Environment variables are centralized.** Import from src/config/index.ts, not process.env directly.
9. **Adding a tool = 3 files max.** See docs/ADDING_A_TOOL.md.
10. **Build before deploy. Verify after deploy.** See COZANET_CONSTITUTION.md.

## Adding a New Capability

1. Create the tool file in src/lib/tools/tools/
2. Register it in src/lib/tools/engine.ts (initializeTools)
3. Add input extraction in src/ai/processor.ts (if needed)
4. Build → test → deploy → verify

See docs/ADDING_A_TOOL.md for the full guide.

## Repository Structure

```
app/
  api/
    chat/route.ts          — Main chat (thin, delegates to src/ai/)
    memory/route.ts         — Memory CRUD (thin)
    web-search/route.ts     — Search endpoint (uses research/provider.ts)
    research-debug/route.ts — Debug: research metrics
    tool-debug/route.ts     — Debug: tool engine status
  globals.css              — All UI styles (custom CSS)
  layout.tsx               — Root layout
  page.tsx                 — Chat UI

src/
  ai/                      — AI Orchestration Layer
    processor.ts            — Main pipeline
    system-prompts.ts       — System prompts
    memory-service.ts       — Memory operations
  config/
    index.ts                — Centralized env config
  lib/
    memory.ts               — Memory system (Supabase)
    tavily.ts               — DEPRECATED → research/provider.ts
    research/               — Web Research Engine (14 files)
    tools/                  — Tool & Action Engine (17 files)

docs/
  ARCHITECTURE.md           — Full architecture documentation
  AI_TOOLING.md             — Tool system guide
  ADDING_A_TOOL.md          — How to add a new capability

COZANET_CONSTITUTION.md     — Guard rules (read first!)
.env.example                — Environment variable template
```
