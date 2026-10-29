# AI Motion Studio

> **Status:** Concept and planning stage. Features described here are planned, not yet implemented.

## What is AI Motion Studio?

AI Motion Studio is a planned **local-first, multi-agent AI motion-production workspace**. Its website provides one place to talk to three specialized AI assistants, coordinate their work, inspect what they produce, and approve actions before an editing agent attempts to generate motion graphics or video.

The goal is to make AI-assisted production **visible, organized, controllable, and reproducible**, instead of juggling separate chat windows and manually transferring instructions between them.

## What can the website do?

### 1. Three dedicated agent chats

- **Storyboarder:** Discusses the creative concept, scenes, shot order, visual style, timing, and transitions. Asks follow-up questions when essential information is missing. Produces a structured storyboard.
- **Briefer:** Reviews the storyboard and gathers production requirements such as aspect ratio, assets, duration, export format, captions, and constraints. Avoids repeating questions already answered. Produces a structured editing brief.
- **AI Editor:** Receives **approved versions** of the storyboard and brief, interprets the editing instructions, and uses the existing Claude Code `SKILL.md` package, templates, and compatible local execution tools to attempt motion-edit production.

Users can converse with each agent separately while all agents refer to the same project context where permitted.

### 2. Coordinated production workflow

1. Describe an idea to the Storyboarder.
2. Review and revise the storyboard.
3. Ask the Briefer to complete the production requirements using the storyboard.
4. Review both deliverables and resolve conflicting or missing details.
5. Explicitly approve the handoff to the AI Editor.
6. Let the editor execute supported local editing/rendering tasks.
7. Inspect logs, previews, and outputs; request revisions and approve final delivery.

The user can control which stage runs next. Agents should not silently begin costly, destructive, or production-changing operations.

### 3. Unified dashboard

The planned website will show:

- Three chat panels and project navigation
- Agent state: idle, thinking, waiting, running, failed, or complete
- Production pipeline and dependency status
- File versions and handoff history
- Approval requests and decision history
- Execution logs, errors, and recovery options
- Motion previews and render output when supported
- AI provider selection and usage visibility

### 4. Shared local project memory

The system keeps project details, chat history, approvals, artifact metadata, and workflow checkpoints on the user's laptop. Agents can retrieve relevant approved information instead of requiring the user to repeat everything. Large media assets and renders live in local folders, not inside the database.

## Why build it? Advantages

- **Less manual coordination:** Structured handoffs replace copying and pasting between AI chats.
- **Clearer responsibilities:** Each agent focuses on a defined production stage.
- **Better context continuity:** Shared project data reduces repetitive questions and inconsistent instructions.
- **Human control:** Editing and other consequential actions wait for explicit approval.
- **Visibility:** The dashboard exposes task status, artifacts, logs, and failures.
- **Reproducibility:** Versioned inputs and recorded decisions help explain how an output was produced.
- **Local ownership:** Project files and databases remain on the user's laptop by default.
- **Flexible models:** Each agent can use a different supported AI provider or local model.
- **Expandable design:** Additional agents, including a separately permissioned website security agent, may be added later.

## Technology direction (planned)

| Layer | Technology | Purpose |
| --- | --- | --- |
| Website | Next.js + Tailwind CSS | Three-panel interface and production dashboard |
| Local backend | Python + FastAPI | APIs, tools, files, and event streaming |
| Agent orchestration | LangGraph | Task coordination, state, checkpoints, approval gates |
| Application database | SQLite | Chats, projects, statuses, approvals, metadata |
| Database access | SQLAlchemy | Python database models and queries |
| Schema migrations | Alembic | Controlled database structure changes |
| Optional semantic retrieval | FAISS + local embedding model | Search past briefs, projects, and relevant memories |
| AI models | Ollama/local models; optional supported provider APIs | Reasoning, questioning, and planning |
| Editing execution | Existing Claude Code `SKILL.md`, templates, and required local tools | Code-based editing and rendering, subject to skill verification |
| Storage | Local project directories | Source media, JSON handoffs, previews, renders |

SQLite's command-line tool has been tested locally; this does **not** mean the application or its database schema has been built.

## Free-first and user-owned AI policy

The target is **no mandatory subscriptions, paid APIs, hosted databases, or cloud infrastructure**. Core orchestration, storage, and website development should run locally using free/open-source components.

Each agent should have its **own model configuration**, so a user may choose a supported local model or connect an eligible external provider through an official authentication method. Credentials must be protected and never committed to GitHub.

**Important:** A login to Claude, Grok, or another chatbot does not automatically include free API access. Claude API and Claude Code availability, authentication, and billing are separate considerations. Free-tier cloud quotas can change. The application must never silently fall back to a paid endpoint; users who require a hard zero-charge guarantee should use local models and block chargeable API calls.

## Limitations and trade-offs

1. **Not a finished product:** This README describes the intended system, not working features.
2. **AI can be wrong:** Storyboards, briefs, and editing code require review; structured validation reduces but does not eliminate errors.
3. **Skills are not renderers:** A `SKILL.md` describes a workflow. Actual video generation requires executable code and suitable rendering dependencies. The existing skill package must be inspected before claiming Adobe-free editing capability.
4. **Hardware matters:** Local models and rendering can consume significant RAM, GPU, CPU, disk space, and time.
5. **Offline limitations:** Local models can work offline, but cloud models require internet access and appropriate credentials.
6. **Concurrency limits:** SQLite is well suited to an initial local deployment, but heavy concurrent writes or distributed deployments may require another database.
7. **Memory is not automatic:** FAISS needs embeddings and retrieval logic; SQLite needs carefully designed records and context selection.
8. **Agent coordination can fail:** Handoffs, retries, and partial execution need idempotency, logging, and recovery mechanisms.
9. **Security boundaries:** Local execution tools should have restricted filesystem and command permissions. Untrusted briefs, assets, and skill instructions must not receive unrestricted access.
10. **Costs are not universally zero:** Free software does not remove electricity, hardware, internet, or optional third-party API costs.
11. **No guaranteed creative quality:** Output depends on available assets, templates, skill quality, chosen models, and rendering tools.

## Project principles

- Local-first, free-first, and modular
- One shared project record, with separate agent conversations
- Structured, versioned JSON handoffs
- Explicit approval before editing execution
- Clear task status and observable logs
- No secret or unapproved paid API usage
- No assumption that a Claude account can be used as an API credential
- Prefer simple components until more complexity is justified

## Current status and next steps

**Current:** Planning and architecture exploration. SQLite command-line installation has been verified on Windows. The existing motion-editing skill package has not yet been audited.

**Next:** Inspect `SKILL.md` and templates; confirm the rendering engine and dependencies; define storyboard and brief JSON schemas; prototype the three-chat dashboard; build a small LangGraph workflow with local persistence and approval checkpoints; test one end-to-end motion edit.

---

*This README is intended as a GitHub project overview for collaborators and future contributors. Implementation details and coding-agent instructions should live in separate architecture/context documents.*
