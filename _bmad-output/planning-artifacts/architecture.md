---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
lastStep: 8
status: 'complete'
completedAt: '2026-06-04'
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/prd.md
  - _bmad-output/planning-artifacts/product-brief.md
  - _bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/EXPERIENCE.md
  - _bmad-output/project-context.md
workflowType: 'architecture'
project_name: 'university-ai'
user_name: 'AILab'
date: '2026-06-04'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
30 FRs across 9 components. The core AI loop is: Student query → Student Profile injection → Vector retrieval (TOP_K configurable) → LLM generation → 3-metric eval scoring — all traced as a single Experiment Run with 5 child spans in Langfuse.

**Non-Functional Requirements:**
- Zero untraced LLM calls (every call produces a GenOS Trace)
- GenOS core ≤ ~500 LOC (simplicity constraint)
- No direct provider SDK imports in any Domain Service
- Configurable-everything: provider, model, embeddings, prompts, chunk size, TOP_K
- Fail-fast on missing config (named error within 2 seconds)
- Publishable quality: cold-start in <30 min from clone
- Reproducibility: same config + seed = same pipeline structure
- WCAG 2.2 AA for the Conversational UI

**Scale & Complexity:**
- Primary domain: Full-stack (Python backend + SPA frontend + AI/RAG pipeline)
- Complexity level: Medium-high
- Estimated architectural components: 8 (GenOS Core, Student Profile Service, Ingestion Pipeline, RAG Q&A Service, RAG Eval Suite, Conversational UI, Vector Store Adapter, Synthetic Data Generator)

### Technical Constraints & Dependencies

- Python for GenOS and all MVP 1 Domain Services
- Langfuse Cloud for observability backend (OTLP ingestion) — cloud service only, never imported in Domain Services
- Anthropic + OpenAI as LLM providers (config-driven via GenOS adapter)
- Embedding model config-driven via GENOS_EMBEDDING_MODEL
- LangChain acceptable for MVP 1 pragmatically: use for text splitting and prompt templates; route LLM calls through genos.llm.complete(). Do not over-engineer the boundary in the first cut.
- No auth in MVP 1 (Persona Switcher only)
- No streaming in MVP 1
- Java services deferred to post-MVP

### Cross-Cutting Concerns Identified

1. **Trace propagation** — Experiment Run ID must thread through all spans across all services
2. **Adapter pattern** — required at: LLM provider, Embedding provider, Vector Store, Ingestion source, Eval scorer
3. **Configuration management** — all config from env/.env, zero hardcoded values
4. **Error handling** — all failures must surface to chat UI as error bubbles, never silent
5. **Student Profile scoping** — persona isolation enforced at the service boundary; cross-persona leakage is a named bug
6. **Observability** — tracing is the primary product value for the Operator persona, not an afterthought
7. **Import enforcement** — no Domain Service may import `anthropic`, `openai`, or `langfuse`; CI lint gate required

## Starter Template Evaluation

### Primary Technology Domain

Full-stack: Python monorepo backend (GenOS + Domain Services) + React SPA frontend (Conversational UI)

### Backend — Python Monorepo with uv Workspaces

No traditional starter template — the backend is hand-rolled Python with a uv workspace monorepo structure.

**Initialization:**
```bash
uv init --no-package university-ai   # virtual workspace root
uv init --lib genos
uv init --lib services/rag_qa
uv init --lib services/student_profile
uv init --lib services/ingestion
uv init --lib services/eval
```

**Monorepo structure:**
```
university-ai/
  pyproject.toml          # workspace root (virtual, package=false)
  uv.lock                 # single shared lockfile
  genos/                  # GenOS core (~500 LOC cap)
  services/
    student_profile/
    rag_qa/
    ingestion/
    eval/
  frontend/
  seed/
  .github/ci.yml
```

Each service declares genos as a workspace dependency:
```toml
[tool.uv.sources]
genos = { workspace = true }
```

### Frontend — React + Vite + TypeScript

```bash
npm create vite@latest frontend -- --template react-ts
```

Additional packages added in first implementation story: `tailwindcss` (design token mapping from DESIGN.md), `playwright` (E2E + WCAG 2.2 AA assertions).

**Note:** Project initialization using these commands should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Single Postgres instance (pgvector + student_profile schemas)
- FastAPI for all Domain Service APIs
- HTTP inter-service communication
- Docker Compose for local dev orchestration

**Important Decisions (Shape Architecture):**
- LangChain scoped to text splitting + prompt templates only
- Zustand for frontend state management
- Tailwind CSS mapped to DESIGN.md tokens
- GitHub Actions for CI/CD

**Deferred Decisions (Post-MVP):**
- Streaming LLM responses
- Real authentication/authorization
- Per-service environment isolation

### Data Architecture

- **Database:** Single shared Postgres instance
  - Schema `genos_vectors` — pgvector embeddings and chunk metadata
  - Schema `student_profile` — student records, conversation history
- **Migrations:** Alembic (per-service migration scripts)
- **Seed data:** LLM-generated synthetic student profiles (Maya + variants) via seed script

### API & Communication Patterns

- **Style:** REST
- **Framework:** FastAPI (async, Pydantic validation, auto OpenAPI docs)
- **Inter-service:** HTTP — each Domain Service is an independent FastAPI process
- **Error handling:** FastAPI exception handlers → structured JSON errors → surfaced as chat error bubbles in UI
- **API docs:** Auto-generated via FastAPI at `/docs` (OpenAPI) — satisfies publishable quality requirement

### Frontend Architecture

- **State management:** Zustand — chat thread, loading state, active persona
- **API client:** Fetch API (native) — no additional HTTP dependency
- **Styling:** Tailwind CSS — design tokens from DESIGN.md mapped to `tailwind.config.js`
- **Component scope:** Single-page, three zones (Nav, Chat Thread, Input Area) matching EXPERIENCE.md layout

### Infrastructure & Deployment

- **Local dev:** Docker Compose — one command starts all services + Postgres
- **CI/CD:** GitHub Actions (free for public repos, fits open-source/publishable goal)
- **Environment config:** Single `.env` at repo root, shared across all services
- **Service ports (local dev):**
  - Conversational UI (Vite): 5173
  - RAG Q&A Service: 8001
  - Student Profile Service: 8002
  - Ingestion Pipeline: 8003
  - Eval Service: 8004

### GenOS-Specific Decisions

- **LangChain scope:** Text splitting (`RecursiveCharacterTextSplitter`) + prompt templates only. All LLM calls, embeddings, and tracing route through GenOS interfaces exclusively.
- **LOC enforcement:** CI check on `genos/` directory — fail if LOC exceeds 500
- **Import enforcement:** CI lint gate — `grep` for `import anthropic`, `import openai`, `import langfuse` in `services/` fails the build

## Implementation Patterns & Consistency Rules

### Naming Patterns

**Python Code:**
- Functions/variables: `snake_case` (`get_student_profile`, `run_id`)
- Files/modules: `snake_case` (`student_profile_service.py`)
- Classes: `PascalCase` (`StudentProfileService`, `RAGQueryRequest`)
- Constants: `UPPER_SNAKE` (`RAG_TOP_K`, `GENOS_LLM_PROVIDER`)

**Database:**
- Tables: `snake_case` plural (`student_profiles`, `conversation_turns`, `knowledge_chunks`)
- Columns: `snake_case` (`student_id`, `experiment_run_id`, `created_at`)
- Foreign keys: `{table_singular}_id` (`student_id`, `session_id`)

**REST API URLs:**
- `snake_case` plural nouns (`/api/rag/query`, `/api/student_profiles`, `/api/ingestion/run`)

**Frontend (React/TypeScript):**
- Components: `PascalCase` (`ChatBubble`, `PersonaChip`)
- Files: `PascalCase.tsx` for components, `snake_case.ts` for utilities
- Variables/functions: `camelCase` (`chatThread`, `sendMessage`)
- JSON fields from API consumed as-is (`snake_case` — no conversion layer)

### Format Patterns

**API Responses:**
- Direct response body + HTTP status codes (FastAPI default, no envelope wrapper)
- Success: `200` with Pydantic model as JSON body
- Not found: `404` with `{"detail": "human-readable message"}`
- Server error: `500` with `{"detail": "what failed"}`
- Never swallow exceptions silently — always return a structured `detail`

**Run IDs:**
- UUID4 everywhere: `str(uuid.uuid4())`
- Field name: always `run_id` (never `runId`, `experimentId`, `trace_id`)

**Dates/timestamps:**
- ISO 8601 strings in all API payloads (`"2026-06-04T17:00:00Z"`)
- Store as `TIMESTAMP WITH TIME ZONE` in Postgres

### Structure Patterns

**Per-service layout:**
```
services/rag_qa/
  src/
    rag_qa/
      __init__.py
      main.py        # FastAPI app
      pipeline.py    # RAG pipeline logic
      models.py      # Pydantic request/response models
  tests/
    test_pipeline.py
    test_api.py
  pyproject.toml
```

**Tests:** Co-located `tests/` directory per service — never top-level.

### GenOS Tracing Pattern

Every pipeline stage MUST use this exact pattern:

```python
with genos.trace.span("stage_name") as span:
    span.set_attribute("run_id", run_id)
    result = do_work()
    span.set_attribute("output_summary", ...)
return result
```

**Fixed span names — no variations permitted:**

| Stage | Span name |
|---|---|
| Query embedding | `query_embedding` |
| Vector retrieval | `vector_retrieval` |
| Prompt construction | `prompt_construction` |
| LLM call | `llm_call` |
| Response | `response` |

### Enforcement Guidelines

**All AI agents MUST:**
- Use exact span names above — Langfuse dashboards and eval assertions depend on them
- Never import `anthropic`, `openai`, or `langfuse` in any file under `services/`
- Always return a `detail` message on error — never a blank response or silent failure
- Use `run_id` (UUID4) as the field name for Experiment Run IDs everywhere
- Name DB tables in `snake_case` plural, columns in `snake_case`

**CI enforces:**
- `grep` for banned imports in `services/` — fails build on violation
- LOC count on `genos/` — fails build if > 500 lines

## Project Structure & Boundaries

### Complete Project Directory Structure

```
university-ai/
├── README.md
├── .env                          # single env file, all services
├── .env.example                  # committed, shows required keys
├── .gitignore
├── pyproject.toml                # uv workspace root (virtual, package=false)
├── uv.lock                       # single shared lockfile
├── docker-compose.yml            # starts all services + Postgres
├── Makefile                      # dev shortcuts (up, down, seed, test, lint)
│
├── .github/
│   └── workflows/
│       ├── ci.yml                # lint + test + LOC cap check
│       └── import-guard.yml      # banned import check
│
├── genos/                        # GenOS core — ~500 LOC cap enforced in CI
│   ├── pyproject.toml
│   └── src/
│       └── genos/
│           ├── __init__.py
│           ├── llm.py            # genos.llm.complete() — LLM abstraction
│           ├── embed.py          # genos.embed() — embedding abstraction
│           ├── trace.py          # genos.trace.span/event() — OTel wrapper
│           ├── eval.py           # EvalScorer interface + set_baseline/compare
│           ├── config.py         # env config loader, fail-fast validation
│           └── adapters/
│               ├── anthropic.py  # Anthropic LLM adapter (only file that imports anthropic)
│               ├── openai.py     # OpenAI LLM adapter (only file that imports openai)
│               └── langfuse.py   # Langfuse OTel adapter (only file that imports langfuse)
│
├── services/
│   ├── rag_qa/                   # RAG Q&A Service — FR-23–25 (port 8001)
│   │   ├── pyproject.toml
│   │   ├── src/
│   │   │   └── rag_qa/
│   │   │       ├── __init__.py
│   │   │       ├── main.py       # FastAPI app, POST /api/rag/query
│   │   │       ├── pipeline.py   # 5-span RAG pipeline
│   │   │       ├── prompt.py     # LangChain ChatPromptTemplate
│   │   │       └── models.py     # Pydantic request/response models
│   │   └── tests/
│   │       ├── test_pipeline.py
│   │       └── test_api.py
│   │
│   ├── student_profile/          # Student Profile Service — FR-16–18 (port 8002)
│   │   ├── pyproject.toml
│   │   ├── src/
│   │   │   └── student_profile/
│   │   │       ├── __init__.py
│   │   │       ├── main.py       # FastAPI app, GET/POST /api/student_profiles
│   │   │       ├── repository.py # DB read/write (SQLAlchemy)
│   │   │       ├── models.py     # Pydantic + SQLAlchemy models
│   │   │       └── migrations/   # Alembic migrations
│   │   └── tests/
│   │       ├── test_repository.py
│   │       └── test_api.py
│   │
│   ├── ingestion/                # Ingestion Pipeline — FR-19–22 (port 8003)
│   │   ├── pyproject.toml
│   │   ├── src/
│   │   │   └── ingestion/
│   │   │       ├── __init__.py
│   │   │       ├── main.py       # FastAPI app, POST /api/ingestion/run
│   │   │       ├── pipeline.py   # load→chunk→embed→index pipeline
│   │   │       ├── splitter.py   # LangChain RecursiveCharacterTextSplitter
│   │   │       ├── adapters/
│   │   │       │   └── synthetic.py  # SyntheticDataAdapter (FR-21)
│   │   │       └── models.py
│   │   └── tests/
│   │       ├── test_pipeline.py
│   │       └── test_adapters.py
│   │
│   └── eval/                     # RAG Eval Suite — FR-26–30 (port 8004)
│       ├── pyproject.toml
│       ├── src/
│       │   └── eval/
│       │       ├── __init__.py
│       │       ├── main.py       # FastAPI app, POST /api/eval/score
│       │       ├── scorers/
│       │       │   ├── retrieval_relevance.py
│       │       │   ├── answer_faithfulness.py
│       │       │   └── answer_relevance.py
│       │       └── models.py
│       └── tests/
│           ├── test_scorers.py
│           └── test_api.py
│
├── seed/                         # Synthetic data generator
│   ├── generate_knowledge_base.py  # LLM-generates course catalog, FAQs, policies
│   ├── generate_student_profiles.py # LLM-generates Maya + variants
│   └── data/
│       └── .gitkeep
│
└── frontend/                     # Conversational UI — FR-1–4 (port 5173)
    ├── package.json
    ├── vite.config.ts
    ├── tailwind.config.js        # DESIGN.md tokens mapped here
    ├── tsconfig.json
    ├── index.html
    └── src/
        ├── main.tsx
        ├── App.tsx
        ├── store/
        │   └── chat_store.ts     # Zustand store: thread, loading, persona
        ├── components/
        │   ├── NavBar.tsx
        │   ├── PersonaChip.tsx
        │   ├── ChatThread.tsx
        │   ├── ChatBubble.tsx    # variants: ai, student, error
        │   ├── TypingIndicator.tsx
        │   ├── InputArea.tsx
        │   └── EmptyState.tsx
        ├── api/
        │   └── rag_client.ts     # Fetch API calls to RAG Q&A Service
        └── tests/
            └── e2e/              # Playwright — WCAG 2.2 AA assertions
```

### Architectural Boundaries

**API Boundaries:**

| Service | Endpoint | Consumer |
|---|---|---|
| RAG Q&A | `POST /api/rag/query` | Frontend |
| Student Profile | `GET /api/student_profiles/{id}` | RAG Q&A Service |
| Student Profile | `POST /api/conversation_turns` | RAG Q&A Service |
| Ingestion | `POST /api/ingestion/run` | Operator (manual trigger) |
| Eval | `POST /api/eval/score` | RAG Q&A Service |

**Data Flow — RAG Query:**
```
Frontend → RAG Q&A (8001)
  → Student Profile (8002) [fetch profile]
  → Postgres/pgvector [retrieve chunks]
  → genos.llm.complete() → Anthropic/OpenAI
  → Eval (8004) [score run]
  → Langfuse Cloud [traces + scores]
  → Frontend [answer]
```

**GenOS Boundary — enforced in CI:**
- Only `genos/adapters/anthropic.py` may import `anthropic`
- Only `genos/adapters/openai.py` may import `openai`
- Only `genos/adapters/langfuse.py` may import `langfuse`
- All `services/**` import from `genos` only

### Future Extraction Path

Each service is designed for independent extraction to a separate repository when needed:

1. Each `services/*/pyproject.toml` is already a self-contained Python package
2. Services communicate via HTTP only — no cross-service direct Python imports
3. Extraction steps when ready:
   - Publish `genos` to PyPI (or private registry) with a version tag
   - Update extracted service's `pyproject.toml` to use versioned `genos` instead of `workspace = true`
   - Split Docker Compose into per-service compose files
   - Split `.env` into per-service env configs

**Implementation rule:** agents must never add direct Python imports between services. HTTP only across service boundaries.

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:** All technology choices are compatible — uv workspaces + FastAPI + SQLAlchemy + pgvector is a well-established Python stack. React + Vite + Zustand + Tailwind have no version conflicts. LangChain scoped to text splitting and prompt templates does not conflict with GenOS's OTel boundary.

**Pattern Consistency:** `snake_case` naming is consistent across Python code, DB schema, and REST URLs. UUID4 run IDs flow consistently through all service boundaries. The 5 fixed span names are defined once and referenced everywhere.

**Structure Alignment:** Every FR maps to a specific service directory. GenOS adapter boundary is structurally enforced (one file per provider). HTTP-only inter-service communication supports future extraction.

### Requirements Coverage Validation ✅

| FR Group | Coverage |
|---|---|
| FR-1–4 Conversational UI | ✅ `frontend/` — React, Zustand, Tailwind, Playwright |
| FR-5–7c GenOS LLM + Embedding + Config | ✅ `genos/llm.py`, `genos/embed.py`, `genos/config.py` |
| FR-8–10 Tracing | ✅ `genos/trace.py`, `genos/adapters/langfuse.py` |
| FR-11–13 Eval Framework | ✅ `genos/eval.py`, `services/eval/` |
| FR-14–15 Config Management | ✅ `genos/config.py`, single `.env` |
| FR-16–18 Student Profile | ✅ `services/student_profile/` |
| FR-19–22 Ingestion Pipeline | ✅ `services/ingestion/`, `seed/` |
| FR-23–25 RAG Q&A | ✅ `services/rag_qa/pipeline.py` — 5 spans |
| FR-26–30 Eval Suite | ✅ `services/eval/scorers/` — 3 metrics |

**NFR Coverage:**
- Zero untraced LLM calls ✅ — enforced via GenOS adapter + CI lint
- GenOS ≤500 LOC ✅ — CI LOC check
- Cold-start <30 min ✅ — `docker compose up`, single `.env.example`, seed scripts
- WCAG 2.2 AA ✅ — Playwright e2e tests
- Publishable quality ✅ — OpenAPI auto-docs, README, architecture doc
- Reproducibility ✅ — seed scripts + config-driven pipeline

### Gap Analysis Results

No critical gaps identified. Two minor implementation notes for agents:

1. **Alembic schema ownership** — Each service owns its own Alembic env pointing to its schema (`genos_vectors` or `student_profile`). Services must not run migrations against each other's schemas.
2. **CORS configuration** — FastAPI services must include `CORSMiddleware` allowing requests from `http://localhost:5173` in dev. Without this, frontend calls will fail with a confusing cross-origin error.

### Architecture Completeness Checklist

**Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

**Architectural Decisions**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

**Implementation Patterns**
- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

**Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status: READY FOR IMPLEMENTATION**
**Confidence Level: High**

**Key Strengths:**
- GenOS boundary is structurally enforced and CI-gated — agents cannot accidentally violate it
- Every FR traces to a specific file/directory
- HTTP-only inter-service design enables future extraction to polyrepo
- Single Postgres instance with schema separation keeps dev simple without coupling data models

**Areas for Future Enhancement:**
- Publish GenOS to PyPI when extracting services to separate repos (MVP 2+)
- Add RAGAS/DeepEval if Langfuse Eval scoring proves insufficient (MVP 2)
- Add streaming LLM responses post-MVP 1

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented
- Use exact span names (`query_embedding`, `vector_retrieval`, `prompt_construction`, `llm_call`, `response`) — no variations
- Never import `anthropic`, `openai`, or `langfuse` outside of `genos/adapters/`
- Never add direct Python imports between services — HTTP only
- Use `run_id` (UUID4) as the Experiment Run ID field name everywhere
- Add `CORSMiddleware` to every FastAPI service

**First Implementation Priority:**
```bash
# 1. Scaffold monorepo
uv init --no-package university-ai
uv init --lib genos
uv init --lib services/rag_qa
uv init --lib services/student_profile
uv init --lib services/ingestion
uv init --lib services/eval

# 2. Scaffold frontend
npm create vite@latest frontend -- --template react-ts

# 3. Create docker-compose.yml, .env.example, Makefile
# 4. Implement GenOS core (llm.py, embed.py, trace.py, eval.py, config.py, adapters/)
# 5. Run seed scripts, then implement services in order: student_profile → ingestion → rag_qa → eval
# 6. Implement frontend last against working backend
```
