# Project Context: University AI

**Project:** university-ai
**Updated:** 2026-06-04
**Status:** Pre-implementation — no code exists yet. All rules derived from planning artifacts.

---

## Product Summary

University AI is a conversational AI assistant for a university — one pane of glass for
students, admins, advisors, and faculty. Built on **GenOS** (Generative AI Operating System),
a shared platform core providing LLM abstraction, observability, tracing, and evaluation.
Every user interaction is also an AI experiment — traced, evaluated, and comparable.

**Active MVP:** MVP 1 — Student Q&A via RAG (Student persona only)

---

## Technology Stack

| Concern | Decision | Status |
|---------|----------|--------|
| Language | Python (GenOS + all MVP 1 Domain Services) | ✅ Confirmed |
| Frontend | Web UI (framework TBD) | 🔄 Architecture phase |
| Observability | Langfuse Cloud | ✅ Confirmed |
| Tracing protocol | OpenTelemetry (OTLP → Langfuse) | ✅ Confirmed |
| LLM providers | Anthropic + OpenAI (config-driven) | ✅ Confirmed |
| Embedding model | Config-driven via GENOS_EMBEDDING_MODEL | ✅ Confirmed |
| Vector Store | pgvector / Chroma / Qdrant (TBD) | 🔄 Architecture phase |
| Student Profile DB | TBD (relational / document / vector) | 🔄 Architecture phase |
| LLM orchestration | Raw SDK vs LangChain/LlamaIndex/DSPy (TBD) | 🔄 Architecture phase |
| Frontend framework | React / Vue / other (TBD) | 🔄 Architecture phase |
| Repo structure | Monorepo vs polyrepo (TBD) | 🔄 Architecture phase |
| Auth | None in MVP 1 — Persona Switcher only | ✅ Confirmed |
| Java services | Not in MVP 1 | ✅ Confirmed |

---

## GenOS Architecture Rules — NON-NEGOTIABLE

These rules apply to every file in every Domain Service. Violations break platform coherence.

1. **No direct provider SDK imports** — Domain Services NEVER import `anthropic` or `openai`.
   All LLM calls use `genos.llm.complete()` exclusively.

2. **No direct Langfuse SDK imports** — Domain Services NEVER import `langfuse`.
   All tracing uses `genos.trace.span()` and `genos.trace.event()` via OpenTelemetry.

3. **Zero untraced LLM calls** — Every LLM call, retrieval step, and eval scoring event
   MUST emit a GenOS Trace. An interaction without a Trace is not a valid Experiment Run.

4. **Experiment Run ID propagation** — Every Experiment Run gets a unique run ID propagated
   through ALL spans. Filtering by run ID must return exactly that run's spans.

5. **Extension without modification** — New Domain Services and Adapters require ZERO changes
   to GenOS internals. If you're editing GenOS to add a new experiment, stop — wrong approach.

6. **Configuration-driven everything** — Provider, model, credentials, endpoints, prompt
   templates loaded from env / `.env`. Zero hardcoded values. `git grep` for API keys = fail.

7. **Fail fast on missing config** — Missing required key → named human-readable error within
   2 seconds. Silent failures and misleading errors are bugs.

8. **AI-component gate** — Any feature whose correct behavior requires NO LLM call, embedding,
   retrieval, or eval is OUT OF SCOPE. CRUD without AI belongs to a different product.

9. **GenOS LOC cap** — GenOS core MUST NOT exceed ~500 LOC before MVP 2.
   If you're tempted to add abstractions, add an Adapter instead.

10. **Reproducibility** — Same config + seed data = same pipeline structure and routing.
    LLM response non-determinism is acceptable. Pipeline behavior non-determinism is not.

---

## Exact Glossary — Use These Terms Verbatim

Introducing a synonym anywhere in code, docs, or comments is a discipline violation.

| Term | Meaning |
|------|---------|
| **GenOS** | The shared platform core (LLM abstraction, tracing, evals, config) |
| **University AI** | The user-facing conversational assistant product built on GenOS |
| **Conversational UI** | The single-pane-of-glass web interface for all personas |
| **Persona Switcher** | UI selector for active Persona Context (no auth in MVP 1) |
| **Persona Context** | Active user identity + profile data injected into AI interactions |
| **Student Profile** | Long-term memory: academic history, courses, degree requirements, app state, conversation history |
| **Domain Service** | App service implementing AI capability Experiments on top of GenOS |
| **Experiment** | A discrete, runnable AI capability implementation (e.g. RAG with specific config) |
| **Experiment Run** | Single execution of an Experiment — produces Traces and Eval Scores |
| **GenOS Trace** | Structured record: inputs, outputs, steps, latency, tokens, cost — stored in Langfuse |
| **Eval Score** | Measurable quality signal per Experiment Run — stored in Langfuse |
| **Knowledge Base** | Corpus of university-domain documents indexed for RAG |
| **Ingestion Pipeline** | Load → chunk → embed → index process |
| **Vector Store** | DB storing document embeddings for semantic retrieval |
| **Synthetic Data** | AI-generated university-domain content for MVP 1 Knowledge Base |
| **Adapter** | Pluggable implementation of a GenOS interface |
| **Baseline Run** | Experiment Run designated as reference for Eval Score comparison |
| **Assistant Mode** | AI interaction triggered by user message (MVP 1 only) |
| **Autonomous Mode** | AI interaction triggered by system event (MVP 3+, NOT MVP 1) |

---

## MVP 1 Scope — What Agents Must Know

**Student persona is the ONLY active persona in MVP 1.**
Admin, Advisor, and Faculty are visible in the UI but disabled ("Coming soon").

### In Scope for MVP 1
- Conversational UI — Student persona, web, desktop + mobile responsive
- GenOS: LLM Client, Tracing Layer, Eval Framework, Config Management
- Student Profile Service (storage, retrieval, RAG injection, conversation history)
- Knowledge Base Ingestion Pipeline (Synthetic Data adapter only)
- RAG Q&A Service (profile-aware retrieve-then-generate)
- RAG Evaluation Suite (retrieval relevance, answer faithfulness, answer relevance)

### Explicitly Out of Scope for MVP 1
- Admin, Advisor, Faculty as active personas → MVP 3+
- Document submission and processing → MVP 3
- Agentic / autonomous workflows → MVP 3
- Human-in-the-Loop escalation → MVP 3
- Degree progress / gap detection → MVP 2
- Long-term memory beyond Student Profile → MVP 2
- Real authentication / authorization → MVP 3
- Java Domain Services → post-MVP
- Real-data ingestion adapters → MVP 2+
- Multi-agent coordination, MCP → MVP 4
- Security and guardrails → MVP 5
- Streaming LLM responses → post-MVP 1
- LLM fine-tuning or model training → never (out of product scope)

---

## Critical Implementation Rules

### Tracing Requirements
- Every Q&A query Trace MUST have exactly 5 child spans:
  `query_embedding → vector_retrieval → prompt_construction → llm_call → response`
- Every Ingestion Pipeline run Trace MUST contain:
  `document_count, chunk_count, embedding_call_count, vector_store_write_count, total_duration`
- Every LLM call Trace MUST contain:
  `provider, model, prompt_tokens, completion_tokens, latency_ms, estimated_cost, run_id`

### Eval Score Requirements
- Three metrics scored per Q&A Experiment Run — ALL three required, no partial:
  `retrieval_relevance`, `answer_faithfulness`, `answer_relevance` — each in [0.0, 1.0]
- Scores MUST be retrievable via Langfuse API by run ID (not just visible in UI)
- Out-of-domain query declines MUST be tagged: `response_type: out_of_domain`

### Retrieval Requirements
- `RAG_TOP_K` config MUST be honored exactly — `RAG_TOP_K=3` → exactly 3 chunks to LLM
- Chunk size tolerance: ±20% is acceptable (natural language sentence boundaries)
- Two ingestion runs with different chunk sizes MUST produce independently retrievable KB versions

### Student Profile Requirements
- Profile MUST be loaded automatically at session start for Student Persona Context
- Profile injection into RAG MUST produce persona-specific answers — generic non-personalized responses are a failure
- Conversation history stored per turn: `session_id, timestamp, speaker, message_text, experiment_run_id`
- History MUST be scoped to active Persona Context — cross-persona leakage is a bug

### Session Boundary (Conversational UI)
- Browser tab = one session. Page refresh = new session, empty chat thread.
- Failed LLM calls MUST render an error message in the chat thread — never a blank response or silent hang.

---

## Anti-Patterns — Never Do These

1. `import anthropic` or `import openai` in any Domain Service or UI code
2. `import langfuse` in any Domain Service (only allowed inside the GenOS Langfuse adapter)
3. Hardcoded API keys, model names, or endpoint URLs anywhere in the codebase
4. LLM calls that don't emit a GenOS Trace
5. Generic student responses that ignore the Student Profile
6. Answering out-of-domain questions instead of declining gracefully
7. Modifying GenOS internals to add a new experiment (use Adapter pattern instead)
8. Silent startup failures on missing config (must fail fast with named error)
9. Eval scores tuned to look good (calibration issue if all scores > 0.95)
10. Features with no AI component (LLM call, embedding, retrieval, or eval)

---

## Open Architecture Decisions — Do Not Assume

These are NOT yet decided — wait for architecture phase output before implementing:

- **Vector Store:** pgvector vs Chroma vs Qdrant
- **Student Profile DB:** relational vs document vs vector store
- **LLM orchestration:** raw SDK vs LangChain vs LlamaIndex vs DSPy
- **Frontend framework:** React vs Vue vs other
- **Repo structure:** monorepo vs polyrepo
- **Experiment registry:** formal named registry vs Langfuse metadata tagging

---

## Product Roadmap Context (for architectural extensibility)

Design GenOS and Domain Service interfaces with MVP 2-5 extensibility in mind.
Do NOT implement MVP 2-5 features in MVP 1 — they will be scoped in future PRDs.

| MVP | Theme | Key New AI Capability |
|-----|-------|-----------------------|
| MVP 1 | Student Q&A | RAG, embeddings, evals, observability |
| MVP 2 | Know Me | Long-term memory, degree reasoning, personalization |
| MVP 3 | Do Things For Me | Agentic workflows, HITL, document intelligence, auth |
| MVP 4 | Everyone's Assistant | Multi-agent orchestration, MCP, LLM-as-judge grading |
| MVP 5 | Hardening | Guardrails, prompt injection defense, adversarial evals |

---

## Key Planning Artifacts

| Artifact | Path |
|----------|------|
| PRD v2 (final, MVP 1) | `_bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/prd.md` |
| Decision Log | `_bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/.decision-log.md` |
| Product Brief | `_bmad-output/planning-artifacts/product-brief.md` |
| Brainstorming Session | `_bmad-output/brainstorming/brainstorming-session-2026-06-04-1700.md` |
