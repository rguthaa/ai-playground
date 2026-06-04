# University AI — Conversational University Assistant

> An AI-powered assistant for universities — built on a shared GenOS platform.  
> This repo demonstrates end-to-end AI product engineering using the **BMAD Method**.

---

## 1. The Problem Worth Solving

Universities are complex. The systems built to navigate them are not.

Five portals. Three support queues. One advisor who books out three weeks. A policy PDF last updated in 2019.

**University AI collapses all of that into a single conversation.**

Ask anything. Get a personalized, sourced answer in seconds — not a search result, not a ticket number.

| Who | What changes |
|-----|-------------|
| **Students** | One place for every question — courses, deadlines, policies, advising |
| **Advisors** | Walk into appointments already informed |
| **Admins** | Stop answering the same 50 questions by hand |
| **Faculty** | Never answer "what's on the syllabus?" at 11pm again |

Every answer runs through **GenOS** — a shared AI operating system that traces, evaluates, and logs every interaction. The university gets smarter. The engineering team gets a world-class AI lab.

**MVP 1:** A student asks a question. GenOS retrieves the right context, generates a grounded answer, and logs the full trace to Langfuse — observable end-to-end.

```mermaid
sequenceDiagram
    participant Maya as 👩‍🎓 Maya (Student)
    participant UI as Conversational UI
    participant RAG as RAG Q&A Service
    participant Profile as Student Profile Service
    participant PG as pgvector (Knowledge Base)
    participant LLM as GenOS → Anthropic / OpenAI
    participant Langfuse as Langfuse Cloud

    Maya->>UI: "What courses count toward my CS degree?"
    UI->>RAG: POST /api/rag/query

    RAG->>Profile: fetch Maya's academic history
    RAG->>PG: retrieve top-K relevant chunks
    RAG->>LLM: generate personalized answer
    LLM-->>Langfuse: trace with 5 spans + 3 eval scores

    RAG-->>UI: personalized answer + structured course info
    UI-->>Maya: "CS301 counts as a core requirement.\nYou still need CS201 as a prerequisite."
```

---

## 2. How is BMAD Used?

[BMAD (Build Me A Dream)](https://bmad-method.org) is an agentic AI development framework. Instead of one general-purpose AI, it orchestrates **specialized agents** — PM, Architect, UX Designer, Engineer, Test Architect — each driving their own phase with genuine domain expertise.

The workflow is structured and sequential. Each phase produces artifacts the next phase depends on. No blank-page paralysis. No requirements drift.

```mermaid
flowchart LR
    A([💡 Idea]) --> B

    subgraph B["Phase 1 — Analysis"]
        B1["📊 Brainstorming\nAgent"]
        B2["📋 Product Brief\nJohn · PM Agent"]
        B1 --> B2
    end

    B --> C

    subgraph C["Phase 2 — Planning"]
        C1["📄 PRD\nJohn · PM Agent"]
        C2["🎨 UX Design\nSally · UX Agent"]
        C1 --> C2
    end

    C --> D

    subgraph D["Phase 3 — Solutioning"]
        D1["🏗️ Architecture\nWinston · Architect Agent"]
        D2["📋 Epics & Stories\nMulti-agent review"]
        D1 --> D2
    end

    D --> E

    subgraph E["Phase 4 — Implementation"]
        E1["⚡ Sprint Planning"]
        E2["💻 Dev Stories\nAmelia · Dev Agent"]
        E3["🔍 Code Review"]
        E1 --> E2 --> E3
    end

    style A fill:#1B2A4A,color:#fff
    style B fill:#f0f4ff,stroke:#1B2A4A
    style C fill:#f0f4ff,stroke:#1B2A4A
    style D fill:#fff8f0,stroke:#A63232,stroke-width:2px
    style E fill:#f5f5f5,stroke:#aaa,stroke-dasharray:5 5
```

### Party Mode — Multi-Agent Review

Before finalizing the epic structure, four AI agents independently reviewed and critiqued the plan. They disagreed with each other. They caught real issues.

```mermaid
graph TD
    Plan["📋 Proposed Epic Structure"]

    Plan --> John["📋 John — PM\n'Epic 5 is an afterthought.\nWhat's the earliest a student\ncan get an answer?'"]
    Plan --> Winston["🏗️ Winston — Architect\n'Defer Langfuse adapter to Epic 4.\nAdd shared Alembic runner to Epic 1.'"]
    Plan --> Amelia["💻 Amelia — Engineer\n'Epic 1 is a merge-conflict factory.\nSplit scaffold from GenOS core.\ndocker-compose.yml needs\nCompose include: pattern.'"]
    Plan --> Murat["🧪 Murat — Test Architect\n'Write the eval harness before\nRAG code — not alongside it.\nATDD at the epic boundary.'"]

    John --> Revised["✏️ Revised Epic Plan"]
    Winston --> Revised
    Amelia --> Revised
    Murat --> Revised

    style Plan fill:#1B2A4A,color:#fff
    style Revised fill:#A63232,color:#fff
    style John fill:#e8f5e9,stroke:#2e7d32
    style Winston fill:#e3f2fd,stroke:#1565c0
    style Amelia fill:#f3e5f5,stroke:#6a1b9a
    style Murat fill:#fff8e1,stroke:#f57f17
```

---

## 3. What is Completed?

All planning artifacts are done. The project is **implementation-ready**.

| Artifact | Status | Description |
|---|---|---|
| [Product Brief](_bmad-output/planning-artifacts/product-brief.md) | ✅ | The *why* — product vision, personas, constraints |
| [PRD v2](_bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/prd.md) | ✅ | 30 functional requirements across 9 components, 5-MVP roadmap |
| [Architecture](_bmad-output/planning-artifacts/architecture.md) | ✅ | Full stack decisions, directory structure, CI enforcement rules |
| [UX Design System](_bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/DESIGN.md) | ✅ | Design tokens, 8 components specified to pixel precision |
| [UX Interaction Spec](_bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/EXPERIENCE.md) | ✅ | State machine, focus management, WCAG 2.2 AA floor, microcopy |
| [Epics & Requirements](_bmad-output/planning-artifacts/epics.md) | 🔄 | 30 FRs + 16 NFRs + 17 UX-DRs extracted; epic structure in review |
| [Project Context](_bmad-output/project-context.md) | ✅ | LLM-optimized context file loaded by dev agents at implementation time |

### System Architecture

```mermaid
graph TB
    subgraph UI["🖥️  Conversational UI — React + Vite + Tailwind (port 5173)"]
        Chat["Chat Thread"]
        Input["Input Area"]
        Persona["Persona Switcher"]
    end

    subgraph GenOS["⚙️  GenOS Core — shared AI platform (≤ 500 LOC enforced by CI)"]
        LLM["llm.py · genos.llm.complete()"]
        Embed["embed.py · genos.embed()"]
        Trace["trace.py · genos.trace.span()"]
        Eval["eval.py · EvalScorer interface"]
        Config["config.py · fail-fast config"]
        subgraph Adapters["Adapters — only files that may import provider SDKs"]
            A1["anthropic.py"]
            A2["openai.py"]
            A3["langfuse.py"]
        end
    end

    subgraph Services["🔧  Domain Services"]
        RAG["RAG Q&A\nport 8001"]
        Profile["Student Profile\nport 8002"]
        Ingest["Ingestion Pipeline\nport 8003"]
        EvalSvc["Eval Suite\nport 8004"]
    end

    subgraph Data["🗄️  Data"]
        PG[("PostgreSQL\npgvector · student_profile")]
        Seed["seed/\nSynthetic KB + Student profiles"]
    end

    subgraph Obs["📊  Observability"]
        Langfuse["Langfuse Cloud\nTraces · Eval Scores · Baselines"]
    end

    UI -->|POST /api/rag/query| RAG
    RAG --> Profile & PG
    RAG -->|genos.llm.complete| LLM
    RAG -->|POST /api/eval/score| EvalSvc
    Ingest -->|genos.embed| Embed
    Ingest --> PG
    Seed --> Ingest
    Profile --> PG
    LLM --> A1 & A2
    Trace --> A3 -->|OTLP| Langfuse
    EvalSvc --> Langfuse

    style GenOS fill:#1B2A4A,color:#fff
    style Adapters fill:#2d4a7a,color:#fff
    style Obs fill:#fff3e0,stroke:#A63232
    style UI fill:#e8f5e9,stroke:#2e7d32
```

### Key Design Decisions

| Concern | Decision | Why |
|---|---|---|
| LLM access | `genos.llm.complete()` only — no direct SDK imports in services | Switch providers via config, zero code change |
| Observability | OpenTelemetry → Langfuse Cloud — no `langfuse` imports in services | Swap backends without touching domain code |
| Eval scores | 3 metrics per Q&A run: `retrieval_relevance`, `answer_faithfulness`, `answer_relevance` | Every run is comparable; baselines are meaningful |
| GenOS size | CI gate: fails build if `genos/` exceeds 500 LOC | Simplicity is a feature; complexity goes in Adapters |
| Import safety | CI gate: `grep` for banned imports in `services/` fails build | Architectural rules enforced mechanically, not by convention |
| Local dev | `docker compose up` — one command, everything running | Cold-start under 30 minutes from clone |

---

## 4. What are the Next Steps?

```mermaid
flowchart TD
    A["✅ Planning Complete\n(PRD · Architecture · UX · Requirements)"]
    A --> B["1️⃣  Finalize Epic Structure\nIncorporate Party Mode feedback:\n• Split Epic 1 into scaffold + GenOS core\n• ATDD-first sequencing for Epic 4\n• Shared Alembic runner in Epic 1"]
    B --> C["2️⃣  Story Creation\nDecompose each epic into developer stories\nwith Given / When / Then acceptance criteria"]
    C --> D["3️⃣  Sprint Planning\nSequence stories · identify blockers\nProduce implementation sprint plan"]
    D --> E["4️⃣  Implementation\nAmelia (Dev Agent) executes stories\ntest-first · story by story"]
    E --> F["5️⃣  Code Review\nBMAD Code Review agent validates\neach story before moving to the next"]
    F --> G["🚀  MVP 1 — Student Q&A\nRunning · traced · evaluated"]

    style A fill:#2e7d32,color:#fff
    style G fill:#A63232,color:#fff
```

### MVP Roadmap Beyond MVP 1

| MVP | Theme | Key New Capability |
|---|---|---|
| **MVP 1** *(this repo)* | **Student Q&A** | RAG, embeddings, evals, observability |
| MVP 2 | Know Me | Long-term memory, degree reasoning, proactive flags |
| MVP 3 | Do Things For Me | Agentic workflows, document intelligence, real auth |
| MVP 4 | Everyone's Assistant | Multi-agent orchestration, MCP, all personas active |
| MVP 5 | Hardening | Guardrails, prompt injection defense, adversarial evals |

---

## Exploring This Repo

```bash
# Planning artifacts — start here
_bmad-output/planning-artifacts/

# The AI platform design
_bmad-output/planning-artifacts/architecture.md

# The product requirements
_bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/prd.md

# The UX design system
_bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/

# The BMAD workflow skills (how each phase was driven)
.claude/skills/
```

If you have [Claude Code](https://claude.ai/code) installed:

```bash
/bmad-help          # see where you are in the workflow
/bmad-dev-story     # start implementing the next story
/bmad-sprint-status # check current sprint state
```

---

*Built with the [BMAD Method](https://bmad-method.org) · June 2026*
