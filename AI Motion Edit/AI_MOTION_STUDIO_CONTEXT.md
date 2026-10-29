# AI Motion Studio — Project Context

> **Status:** Planning only. This document is the source of truth for future IDE/Codex work, not permission to start implementation.

## 1. Vision

Build a **local-first, multi-agent AI motion-production studio** with one website that makes agent conversations, files, decisions, execution, and progress visible. The interface has three separate chat panels, inspired by a three-column task board:

1. **Storyboarder** — interviews the user about the story, shots, pacing, style, and motion; fills important gaps; produces a structured storyboard.
2. **Briefer** — reads approved storyboard context, asks only for missing production requirements (assets, format, timing, captions, delivery), and produces a structured editing brief.
3. **AI Editor** — receives the approved storyboard and brief, loads the user's existing Claude Code `SKILL.md` package and templates, creates a motion edit through its actual supported execution/rendering tools, and returns previews, outputs, and logs.

The user can speak with each agent independently. Agents should share project-level facts without merging their chat identities or repeating answered questions.

## 2. Non-negotiable cost policy

- **Zero mandatory spending:** free/open-source local stack, no required subscription, billing account, paid hosting, paid model API, or hidden fees.
- Default to **local models** (e.g., Ollama) for no metered API charges. Local operation still uses the user's hardware/electricity.
- External model providers are **optional bring-your-own-provider connections**. Offer a free-tier provider only when its current eligibility, quotas, and terms are verified. Free tiers may change and can have limits.
- **Never automatically call a paid API**, upgrade to paid service, or fall back to a billable model. If free quota is exhausted, stop and ask the user to choose a free/local alternative.
- A user's login to a provider's consumer chat website **does not imply API access or free API credits**. Use only officially supported API keys, OAuth, or SDK authentication. Never extract cookies or bypass usage restrictions.
- Claude API and Claude Code are **not assumed free**. Claude Code integration is conditional on supported authentication, permitted usage, and confirmed zero-charge operation. Do not misrepresent this as guaranteed.
- Maintain provider-specific usage visibility and configurable safeguards; a software budget limit alone does not guarantee prevention of provider-side charges.

## 3. Proposed architecture

```text
Next.js + Tailwind dashboard (local browser)
    | HTTP / WebSocket
FastAPI Python backend
    |
LangGraph workflow orchestration
    |-- Storyboarder agent --> configurable LLM provider
    |-- Briefer agent ------> configurable LLM provider
    |-- AI Editor agent ----> configurable LLM + controlled editing tools
    |                         |-- existing SKILL.md + templates
    |                         `-- local render/export process (TBD)
    |-- human approval gates
    `-- event/status logging
    |
SQLite (via SQLAlchemy; schema migrations via Alembic)
    |-- projects, agents, chats, approvals, task state, artifact metadata
    `-- LangGraph checkpoint persistence (compatible adapter TBD)
Local filesystem
    |-- storyboard and brief JSON files
    |-- media assets, templates, skills, previews, renders
    `-- optional FAISS index + local embedding model for semantic retrieval
```

**Technology responsibilities:**

| Tool | Purpose | Decision |
|---|---|---|
| Next.js + Tailwind | Three-panel UI, file viewer, status, approvals, preview | Planned |
| Python + FastAPI | Local backend, controlled tool execution, events | Planned |
| LangGraph | Agent routing, durable workflows, pause/resume, approvals | Planned |
| SQLite | Local persistent relational data | Selected; SQLite CLI 3.54.1 tested on Windows |
| SQLAlchemy | Python database access and models | Planned |
| Alembic | Database schema migrations | Planned |
| FAISS + local embeddings | Optional semantic retrieval of prior projects | Later phase; not required for MVP |
| Ollama | Local inference, default free-of-API-charge option | Planned; model capability/hardware TBD |
| Gemini | Optional user-provided free-tier connection where eligible | Conditional |
| Claude API / Claude Code | Optional model/agent execution if terms and costs allow | Conditional, not free by assumption |
| Existing SKILL.md + templates | Motion-editing instructions and reusable assets | User has package; contents not yet inspected |

## 4. Per-agent LLM configuration

- Each agent has its **own provider, model, credential reference, settings, and availability state**.
- Suggested initial defaults: Storyboarder = local Ollama; Briefer = local Ollama; AI Editor = local Ollama **only if capable of required code/tool workflow**.
- Users may optionally select Gemini free tier or other verified no-charge providers independently per agent.
- Do not hardwire the entire application to Claude. Provider abstraction should separate model reasoning from tool execution.
- Credentials must not be stored in plaintext in SQLite, sent to other users, or exposed in client logs. For local development prefer environment variables or OS credential storage; use an appropriate secure credential design if multi-user support is added.
- Agent chats are separate, but approved project facts and versioned artifacts can be shared.

## 5. Production workflow

1. User creates a project and chats with **Storyboarder**.
2. Storyboarder asks targeted clarifying questions and generates `storyboard.json` with a schema/version.
3. User reviews and approves or requests changes.
4. **Briefer** reads the approved storyboard, avoids duplicate questions, and generates `editing_brief.json` with a schema/version.
5. User reviews the brief and the consolidated edit plan.
6. **LangGraph pauses at a human approval checkpoint.** No editing execution before approval.
7. **AI Editor** receives exact approved artifact versions and loads the relevant skill/templates.
8. The execution adapter invokes verified local scripts/rendering dependencies, logs operations, and creates a preview.
9. User reviews, requests revisions, and explicitly approves final export.

Agent handoffs use validated structured files and stable IDs, not blind copy/paste of entire chat transcripts. On changed requirements, invalidate dependent approvals and recheck downstream artifacts.

## 6. Dashboard requirements

- Three visible, independent chat panels: Storyboarder, Briefer, AI Editor.
- Agent states: idle, thinking, waiting for input, waiting for approval, executing, completed, failed.
- Project-wide timeline/pipeline, artifact list with version and producer, approval queue, execution logs, and preview/export area.
- Per-agent provider/model selector and free-tier/availability indicators.
- User controls: approve, reject, request revision, pause/cancel where supported, and retry safely.
- WebSockets or equivalent events for live progress; recover state after restart.
- Clear distinction between **proposed**, **approved**, and **executed** actions.

## 7. Storage and reliability

- SQLite stores relational records and file references; large video files stay on the local filesystem.
- Enable SQLite WAL where appropriate, short transactions, busy timeouts, and database backups. SQLite allows concurrent readers but only one writer at a time.
- Use SQLAlchemy for application tables and Alembic for schema changes. LangGraph checkpoint storage may use a separate compatible persistence implementation.
- Add FAISS only when semantic retrieval becomes useful. FAISS is an index, not the primary source of truth; embeddings need a model.
- Track project IDs, agent run IDs, artifact versions, approval decisions, timestamps, and errors.
- Editing commands must run in restricted workspaces, with safe paths, explicit permissions, resource limits, and recoverable outputs.

## 8. Motion-editing caveat

A Claude Code `SKILL.md` file is **instructions, not itself a renderer**. We must inspect the user's skill package and templates to determine the actual supported tools, scripts, runtime, licenses, and output formats. Adobe After Effects is **not required by default**, but some local rendering/code execution dependency will be needed. Do not choose or install one until the skill is reviewed.

The Claude API alone does not automatically include the Claude Code CLI/SDK execution environment. Any integration must use a documented, authorized method.

## 9. Suggested phased implementation

1. **Design:** inspect skill package; finalize artifact schemas, agent contracts, approvals, and provider policy.
2. **Local foundation:** Next.js UI, FastAPI backend, SQLite, SQLAlchemy, Alembic, basic LangGraph state.
3. **Agent collaboration:** independent chats, project context, structured storyboard/brief handoffs, approval checkpoints.
4. **Editor execution:** integrate tested skills/templates and local rendering adapter, sandboxing, preview, logs.
5. **Enhancements:** provider switching, usage visibility, semantic memory via FAISS, resilience and UX improvements.

## 10. Open decisions (do not guess)

- Contents and dependencies of the existing motion-editing `SKILL.md` package.
- Which free local model(s) work reliably on the user's laptop and can use required tools.
- Whether any Claude Code authentication path meets the strict zero-cost policy.
- Exact rendering engine, export formats, and preview pipeline.
- Whether the initial application is single-user local-only or needs multiple user accounts.
- Artifact JSON schemas, revision behavior, and retention/backup policy.

## 11. Instructions for Codex / IDE agents

- **Read this file first. Planning only until explicitly told to implement.**
- Preserve strict zero-mandatory-cost design; disclose any potential paid dependency before proposing it.
- Prefer small, testable modules over giant hardcoded parsers or unnecessarily complex infrastructure.
- Use context-aware agents with explicit shared state and validated structured outputs.
- Do not silently execute editing, install dependencies, call external providers, or deploy anything.
- Ask concise questions when a decision is essential; otherwise record open decisions rather than inventing requirements.
- Keep project files, secrets, and database local by default.
- Treat the user's approval as a required checkpoint before any consequential editing or publishing action.
