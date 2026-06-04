---
title: University AI — Conversational University Assistant
status: final
version: 2.0
created: 2026-06-04
updated: 2026-06-04
supersedes: prd-university-ai-2026-06-04/prd.md
---

# PRD: University AI — Conversational University Assistant

**Contents:** [0. Purpose](#0) · [1. Vision](#1) · [2. Target Users](#2) · [3. Glossary](#3) · [4. Features — MVP 1](#4) · [5. Cross-Cutting NFRs](#5) · [6. Non-Goals](#6) · [7. MVP 1 Scope](#7) · [8. Product Roadmap](#8) · [9. Success Metrics](#9) · [10. Open Questions](#10) · [11. Assumptions](#11)

---

## 0. Document Purpose

This PRD is the authoritative requirements contract for **University AI** — a conversational AI assistant for a university, built on the **GenOS** (Generative AI Operating System) platform. It is written for the platform operator, UX designer, architect, and downstream implementation agents.

This is **version 2.0**, superseding the v1 PRD (`prd-university-ai-2026-06-04`). The primary change: University AI is now a real product with real user personas — not an operator-only experimentation interface. The GenOS platform backbone (LLM abstraction, observability, tracing, evals) is fully preserved and unchanged.

This PRD covers **MVP 1 in full detail**. MVP 2-5 are captured as a lightweight roadmap in §8 — sufficient for architectural extensibility decisions, not full FR detail.

**Source inputs:** Product Brief v1.0 · PRD v1.0 · Brainstorming Session 2026-06-04

---

## 1. Vision

University AI is a conversational assistant that makes the university experience smarter for everyone — students get instant, personalized answers; admins handle more applications with less effort; advisors walk into appointments already informed; faculty stop answering the same questions at 11pm.

The product is a **single conversational interface** — one pane of glass — that routes intelligently to different AI services depending on who is asking and what they need. Behind every interaction is **GenOS**: a shared platform core that ensures every AI call is traced, evaluated, and comparable. The university gets a smarter institution. The platform operator gets a world-class AI experimentation lab. Both are true at the same time.

University AI is built to be open-source, publishable, and demonstrable — a reference implementation of enterprise-grade AI engineering applied to a realistic domain. Every capability is an experiment. Every experiment is a lesson.

---

## 2. Target Users

### 2.1 Personas

*These are real product users, not simulation constructs. In MVP 1, Persona Context is switched via a UI selector — no authentication required. Real auth is introduced in MVP 3.*

| Persona | Name | Role | Primary Need |
|---------|------|------|-------------|
| **Student** | Maya | Sophomore CS student | Answers to course, enrollment, and application questions — fast and personalized |
| **Admin** | Sandra | Admissions coordinator | Application workflow automation — routine handled by AI, exceptions surfaced to her |
| **Advisor** | Dr. Raj | Academic advisor | Students arrive prepared; AI handles routine questions so he focuses on judgment calls |
| **Faculty** | Prof. Chen | CS professor | Course materials answer student questions; grading assistance for assignments |
| **Operator** | AILab | Platform builder | Extend, observe, and demonstrate AI capabilities through a real product |

### 2.2 Jobs To Be Done

**Student (Maya)**
- Get accurate answers to university questions without navigating 5 different portals
- Know exactly where her application or enrollment stands and what's next
- Get course recommendations that actually account for her specific degree progress
- Feel like the university knows who she is

**Admin (Sandra)**
- Stop spending time answering the same applicant questions repeatedly
- Know which applications need her attention without reviewing all of them
- Trust that documents are validated before they reach her queue

**Advisor (Dr. Raj)**
- Stop spending appointment time on questions the AI could answer
- Walk into every advising session already knowing the student's situation
- Have students arrive with specific, well-formed questions

**Faculty (Prof. Chen)**
- Stop answering "what's on the syllabus?" emails at 11pm
- Grade assignment batches without reviewing every submission individually
- Have her course policies update student answers automatically

### 2.3 Non-Users (MVP 1)

- Admin, Advisor, Faculty as active users — MVP 1 is Student-only (other personas in MVP 3-4)
- Anyone expecting a production-deployed, multi-tenant SaaS product
- Anyone requiring SSO, LDAP, or enterprise identity integration

### 2.4 Key User Journeys — MVP 1

*Student persona only for MVP 1. Lighter UJ format — single operator context, no multi-role flows in scope.*

- **UJ-1. Maya asks about enrollment.**
  Maya opens University AI, selects Student persona. Types: "When does enrollment open for next semester?" AI retrieves the answer from the Knowledge Base and responds with the correct date, registration window, and a note about her current enrollment eligibility. Takes under 10 seconds. Maya closes the app and registers on the correct date.

- **UJ-2. Maya asks about a course.**
  Maya types: "Tell me about CS301 — does it count toward my major?" AI retrieves course details (description, credits, prerequisites, instructor) from the Knowledge Base and cross-references Maya's degree requirements from her Student Profile. Response confirms CS301 counts as a core requirement and notes she hasn't taken the prerequisite CS201 yet.

- **UJ-3. Maya checks her application status.**
  Maya types: "What's the status of my application?" AI retrieves her application state, explains the current stage in plain language, and tells her what's needed next: "Your application is under review. We're still missing your official transcript — please upload it to continue."

---

## 3. Glossary

- **University AI** — The conversational AI assistant product. The user-facing product built on GenOS.
- **GenOS** — Generative AI Operating System. The shared platform core layer providing LLM abstraction, observability, tracing, evaluation, and configuration management. All Domain Services are built on GenOS.
- **Conversational UI** — The single-pane-of-glass web interface through which all personas interact with University AI. Combines a chat interface with structured UI elements (status cards, document upload, action buttons).
- **Persona Switcher** — A UI selector that sets the active Persona Context (Student, Admin, Advisor, Faculty). Does not require authentication in MVP 1.
- **Persona Context** — The active user identity and associated profile data injected into every AI interaction. Set by the Persona Switcher.
- **Student Profile** — The long-term memory record for a Student persona. Contains academic history, enrolled courses, degree requirements, application state, and conversation history. Persists across sessions.
- **Domain Service** — An application service implementing AI capability Experiments within the university domain. All Domain Services are built on GenOS.
- **Experiment** — A discrete, runnable AI capability implementation (e.g., RAG Q&A with a specific configuration). Every product interaction is also an Experiment.
- **Experiment Run** — A single execution of an Experiment. Produces Traces and Eval Scores.
- **GenOS Trace** — A structured record of an Experiment Run: inputs, outputs, intermediate steps, latency, token usage, cost. Emitted via OpenTelemetry, stored in Langfuse.
- **Eval Score** — A measurable quality signal for an Experiment Run. Examples: retrieval relevance, answer faithfulness, answer relevance.
- **Langfuse** — The observability, tracing, and evaluation backend. Used via Langfuse Cloud.
- **Knowledge Base** — The corpus of university-domain documents ingested and indexed for RAG. MVP 1 uses Synthetic Data.
- **Ingestion Pipeline** — The process of loading, chunking, embedding, and indexing source documents into the Vector Store.
- **Vector Store** — The database storing document embeddings for semantic retrieval. [ASSUMPTION A1: tool deferred to architecture phase.]
- **Synthetic Data** — AI-generated university-domain content: course catalog, policies, FAQs, application requirements.
- **Adapter** — A pluggable implementation of a GenOS interface. Enables provider/backend switching without Domain Service code changes.
- **Baseline Run** — An Experiment Run designated as the reference point for Eval Score comparison.
- **Assistant Mode** — AI interaction triggered by a user message. The AI responds to the human.
- **Autonomous Mode** — AI interaction triggered by a system event (document uploaded, deadline approaching). The AI acts without a human prompt. MVP 3+.

---

## 4. Features — MVP 1

### 4.1 Conversational UI

**Description:** A web-based single-pane-of-glass interface that is the entry point for all University AI interactions. Combines a persistent chat thread with structured UI elements (status cards, action buttons, document upload areas) that appear contextually. In MVP 1, only the Student Persona Context is active — the Persona Switcher is present but Admin/Advisor/Faculty are disabled. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-1: Chat interface with message history
The Conversational UI renders a persistent chat thread showing the conversation between the active persona and University AI. Messages are ordered chronologically and persist for the lifetime of the browser tab session. Refreshing the page or closing the tab ends the session; conversation history from prior sessions is retrievable via the Student Profile Service (FR-18), not the UI layer.

**Consequences (testable):**
- Sending a message appends it to the thread and triggers an AI response.
- The conversation thread is visible and scrollable for the duration of the tab session.
- Refreshing the page starts a new session with an empty thread.
- A failed LLM call renders an error message in the thread ("Something went wrong — please try again") rather than a blank response or silent hang.

#### FR-2: Persona Switcher
The UI provides a Persona Switcher that sets the active Persona Context. In MVP 1, Student is the only active persona. Admin, Advisor, and Faculty are visible but disabled with a "Coming soon" indicator.

**Consequences (testable):**
- Selecting Student persona sets Persona Context to Student and loads the Student Profile.
- Attempting to select Admin/Advisor/Faculty shows a disabled state — no navigation occurs.

#### FR-3: Structured data surfacing in responses
When an AI response contains structured university data, the UI surfaces that data in a scannable, structured format alongside the conversational text. For application status responses, the structured data must include: current stage, what is blocking progress (if anything), and the specific next required action with enough detail for Maya to act on it without asking a follow-up question.

**Consequences (testable):**
- A course inquiry response surfaces: course name, credits, prerequisites, and whether it fulfills a degree requirement — regardless of the specific UI rendering (card, table, or inline).
- An application status response surfaces: current stage, blocking reason (if any), and next required action as distinct data fields — not buried in prose.
- An application status response for a blocked application tells Maya *what* is missing and *what to do about it*, not only that the application is incomplete.

#### FR-4: Responsive web interface
The Conversational UI is accessible via a modern web browser on desktop and mobile viewport sizes.

**Consequences (testable):**
- UI renders correctly at 1440px (desktop) and 390px (mobile) viewport widths.
- Chat input and send button are accessible without horizontal scrolling on mobile.

---

### 4.2 GenOS Core — LLM Abstraction Client

**Description:** Provider-agnostic LLM interface. All LLM calls across the platform route through the GenOS LLM Client. No Domain Service or UI layer imports provider SDKs directly. Switching providers requires configuration change only. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-5: Provider-agnostic LLM interface
The GenOS LLM Client exposes a unified `complete()` interface callable by all Domain Services.

**Consequences (testable):**
- A Domain Service calling `genos.llm.complete()` produces a valid response with Anthropic configured.
- The same code produces a valid response with OpenAI configured — zero code changes.
- No Domain Service file contains a direct import of `anthropic` or `openai` SDK.

**Out of Scope:** Fine-tuning, model training, streaming (MVP 1).

#### FR-6: Automatic trace emission on every LLM call
Every GenOS LLM Client call emits a GenOS Trace: provider, model, prompt tokens, completion tokens, latency (ms), estimated cost, run metadata.

**Consequences (testable):**
- Zero LLM calls occur without a corresponding Trace record in Langfuse.
- Token usage and latency are present on every Trace.

#### FR-7: Configuration-driven provider and model selection
Active LLM provider, model, and credentials are set via environment configuration. No code change required to switch providers or models.

**Consequences (testable):**
- `GENOS_LLM_PROVIDER=anthropic` routes all calls to Anthropic.
- `GENOS_LLM_PROVIDER=openai` routes all calls to OpenAI without code changes.

#### FR-7b: Configuration-driven embedding model selection
The embedding model used for query embedding and Knowledge Base ingestion is set via environment configuration. No code change required to switch embedding models.

**Consequences (testable):**
- `GENOS_EMBEDDING_MODEL=text-embedding-3-small` and `GENOS_EMBEDDING_MODEL=text-embedding-ada-002` both produce valid embeddings without code changes.
- Two ingestion runs with different embedding models produce independently retrievable Knowledge Base versions traceable in Langfuse.

#### FR-7c: Configurable prompt templates
System prompts and RAG augmentation prompt templates are stored as configurable files, not hardcoded strings. Changing a prompt template does not require a code change.

**Consequences (testable):**
- The RAG Q&A system prompt is loaded from a configurable path at startup.
- Changing the system prompt file and restarting the service causes subsequent LLM calls to use the updated prompt — verified via Langfuse trace inspection.
- Each Experiment Run Trace records the prompt template version used.

---

### 4.3 GenOS Core — Observability & Tracing Layer

**Description:** Instruments every meaningful event in an Experiment Run. Emits OpenTelemetry-compatible signals to Langfuse Cloud via OTLP. Domain Services use `genos.trace` exclusively — no direct Langfuse SDK imports. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-8: OpenTelemetry-compatible trace emission
GenOS Tracing Layer emits spans and events as OpenTelemetry signals. OTLP endpoint is configurable.

**Consequences (testable):**
- All Experiment Run events appear as linked spans in Langfuse.
- Changing `GENOS_OTLP_ENDPOINT` reroutes traces without code changes.

#### FR-9: GenOS trace interface for Domain Services
Domain Services call `genos.trace.span()` and `genos.trace.event()` exclusively. No `langfuse` imports in any Domain Service.

**Consequences (testable):**
- A fully traced Domain Service contains zero `langfuse` imports.
- Swapping the Langfuse adapter for a stub does not break Domain Service code.

#### FR-10: Experiment Run context propagation
Each Experiment Run is assigned a unique run ID propagated through all spans.

**Consequences (testable):**
- Filtering Langfuse by run ID returns all spans for that run and only that run.

---

### 4.4 GenOS Core — Evaluation Framework

**Description:** Consistent interface for defining, running, and storing Eval Scores. Built on Langfuse Evals for MVP 1 — co-located with tracing, supports LLM-as-judge natively. [ASSUMPTION A3: Langfuse Evals sufficient for MVP; RAGAS/DeepEval available if needed.] Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-11: Pluggable eval scorer interface
GenOS exposes an `EvalScorer` interface. Domain Services register and invoke scorers at Experiment Run completion. Scorer implementations are swappable without modifying GenOS internals.

**Consequences (testable):**
- A custom `EvalScorer` implementation can be registered without modifying any GenOS file.
- MVP ships with at least one Langfuse-backed scorer implementation.

#### FR-12: Eval Score storage linked to Experiment Run
Every Eval Score is stored in Langfuse linked to its parent Experiment Run Trace.

**Consequences (testable):**
- Selecting an Experiment Run in Langfuse shows all associated Eval Scores.
- Eval Score history for a named metric is retrievable across multiple runs via Langfuse API.

#### FR-13: Baseline Run designation and delta computation — Realizes UJ-1
The platform stores a Baseline Run tag. The Eval Framework computes and logs Eval Score deltas between any run and the designated Baseline Run.

**Consequences (testable):**
- `genos.eval.set_baseline(run_id)` stores the baseline tag in Langfuse.
- `genos.eval.compare_to_baseline(run_id)` returns `{metric_name: delta}` for each shared metric.
- Delta is computable from stored Eval Score records, independent of Langfuse UI features.

---

### 4.5 GenOS Core — Configuration Management

**Description:** Environment-driven configuration for all GenOS components and Domain Services. No hardcoded credentials anywhere in the codebase. Realizes UJ-2 (onboarding).

**Functional Requirements:**

#### FR-14: Environment-based configuration
All secrets, provider selections, endpoint URLs, and feature flags load from environment variables or `.env` at startup.

**Consequences (testable):**
- Platform starts and runs correctly with only a populated `.env` — no code edits required.
- `git grep` on the repository finds zero hardcoded API keys.

#### FR-15: Startup configuration validation
GenOS validates required configuration keys at startup and fails fast with a human-readable error naming the missing key.

**Consequences (testable):**
- Missing required key produces a named error within 2 seconds of startup.
- No silent failures or misleading errors on misconfiguration.

---

### 4.6 Student Profile Service

**Description:** Stores and retrieves the Student Profile — the long-term memory record for each Student persona. Contains academic history, enrolled courses, degree requirements, application state, and conversation history. Persists across sessions. Profile data is injected into RAG queries to produce personalized responses. Realizes UJ-2, UJ-3.

**Functional Requirements:**

#### FR-16: Student Profile storage and retrieval
The Student Profile Service stores a structured profile per Student persona and retrieves it at the start of each conversation session.

**Consequences (testable):**
- Starting a new session with Student persona loads the existing profile automatically.
- Profile contains at minimum: student ID, name, enrolled courses, completed courses, declared major, application state.

#### FR-17: Profile injection into RAG queries — Realizes UJ-2
When the RAG Q&A Service processes a Student query, the active Student Profile is injected as context alongside retrieved Knowledge Base chunks.

**Consequences (testable):**
- A degree requirement query returns an answer referencing Maya's specific completed courses, not a generic answer.
- Two students with different completed courses receive different course recommendations for the same question.

#### FR-18: Conversation history persistence
The Student Profile Service stores conversation history for the active Student Persona Context across sessions. Each stored conversation turn contains: session ID, timestamp, speaker (user or AI), message text, and the Experiment Run ID for AI turns.

**Consequences (testable):**
- After ending a session and starting a new one, calling the Student Profile Service API for the active persona returns prior conversation turns with all five required fields present.
- The AI, when asked "what did we discuss last time?", retrieves and accurately summarizes the most recent prior session's conversation turns.
- Conversation history is scoped to the active Persona Context — switching Persona Context does not expose a different persona's history.

---

### 4.7 Knowledge Base — Ingestion Pipeline

**Description:** Loads Synthetic Data university documents (course catalog, policy documents, FAQs, application requirements), chunks them, generates embeddings, and indexes them into the Vector Store. Pluggable Adapter pattern for data sources. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-19: Synthetic university Knowledge Base generation
A seed script generates a Synthetic Data Knowledge Base containing: at least 10 course catalog entries (name, code, credits, description, prerequisites), at least 5 policy documents (title, policy ID, effective date, body), at least 20 FAQs (question, answer), and at least 3 application requirement documents (program, requirements list, deadlines).

**Consequences (testable):**
- Running the seed script produces all document types with all required fields populated.
- The Ingestion Pipeline processes the seed output without errors.

#### FR-20: Configurable chunking strategy — Realizes UJ-1
Chunk size (tokens) and chunk overlap (tokens) are configurable. Re-running ingestion with different parameters produces a new Knowledge Base version.

**Consequences (testable):**
- `INGEST_CHUNK_SIZE=500` produces chunks where the majority of chunks are between 400–600 tokens. (Natural-language sentence boundaries prevent exact token counts; a ±20% tolerance is acceptable.)
- Two ingestion runs with different chunk sizes produce independently retrievable Knowledge Base versions in the Vector Store.

#### FR-21: Pluggable Ingestion Adapter interface
Ingestion Pipeline exposes an `IngestionAdapter` interface. Synthetic Data loader is one implementation. Future adapters (filesystem, URL, API) implement the same interface without modifying pipeline core.

**Consequences (testable):**
- A new `IngestionAdapter` can be registered and used without modifying any pipeline core file.

#### FR-22: Ingestion run traced end-to-end
Each Ingestion Pipeline run emits GenOS Traces: document count, chunk count, embedding call count, Vector Store write count, total duration.

**Consequences (testable):**
- Langfuse contains a complete ingestion Trace with all five metrics after each run.
- The Trace captures token usage from all embedding calls.

---

### 4.8 RAG Q&A Service

**Description:** Answers Student natural language questions using the indexed Knowledge Base and active Student Profile. Implements retrieve-then-generate: embed query + profile context → retrieve top-K chunks → construct augmented prompt → generate answer. Full pipeline traced as one Experiment Run. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-23: Profile-aware retrieve-then-generate pipeline
Given a Student query and active Student Profile, the Q&A Service retrieves top-K relevant chunks and passes them with the profile context to the GenOS LLM Client to generate a personalized answer.

**Consequences (testable):**
- A query about course prerequisites returns an answer that references Maya's specific academic history.
- The answer Trace includes the chunk IDs used as context, visible in Langfuse.

#### FR-24: Configurable retrieval parameters — Realizes UJ-1
Top-K chunk count and similarity threshold are configurable per Experiment Run.

**Consequences (testable):**
- `RAG_TOP_K=3` results in exactly 3 chunks passed to the LLM call context.
- `RAG_TOP_K=10` results in exactly 10 chunks.

#### FR-25: Full pipeline trace per query
Each Q&A query produces a single Experiment Run Trace with child spans for: query embedding, Vector Store retrieval, prompt construction, LLM call, response.

**Consequences (testable):**
- Each Langfuse Trace has five child spans corresponding to the five pipeline stages.
- No pipeline stage executes without a corresponding span.

---

### 4.9 RAG Evaluation Suite

**Description:** Scores every Q&A Experiment Run on three metrics via Langfuse Evals: retrieval relevance, answer faithfulness, answer relevance. Scores stored in Langfuse linked to the run. Realizes UJ-1.

**Functional Requirements:**

#### FR-26: Retrieval relevance scoring
Every Q&A Experiment Run is scored on retrieval relevance using LLM-as-judge via Langfuse Evals.

**Consequences (testable):**
- Every run has a `retrieval_relevance` Eval Score in [0.0, 1.0] in Langfuse.
- A query with no relevant Knowledge Base chunks scores below 0.3.

#### FR-27: Answer faithfulness scoring
Every Q&A Experiment Run is scored on answer faithfulness using LLM-as-judge.

**Consequences (testable):**
- Every run has an `answer_faithfulness` Eval Score in [0.0, 1.0].
- An answer contradicting retrieved chunk content scores below 0.3.

#### FR-28: Answer relevance scoring
Every Q&A Experiment Run is scored on answer relevance using LLM-as-judge.

**Consequences (testable):**
- Every run has an `answer_relevance` Eval Score in [0.0, 1.0].
- An answer that ignores the question scores below 0.3.

#### FR-29: Cross-run Eval Score retrieval
All three Eval Scores for any Experiment Run are retrievable programmatically via the Langfuse API.

**Consequences (testable):**
- A script returns all three metric scores for a given run ID.
- Scores for two or more runs can be fetched and compared in a single script.

#### FR-30: Out-of-domain decline behavior
When a user query is outside the university domain (e.g., general knowledge questions, weather, personal advice), the RAG Q&A Service declines to answer and redirects the user to university-related topics.

**Consequences (testable):**
- A query of "What is the capital of France?" returns a polite decline response, not a fabricated or Knowledge Base-hallucinated answer.
- A query of "Help me write a cover letter" returns a redirect to university services, not a general response.
- Out-of-domain declines are traced in Langfuse with a distinct `response_type: out_of_domain` tag.

---

## 5. Cross-Cutting NFRs

**Observability:** Every LLM call, retrieval step, and eval scoring event produces a GenOS Trace. Zero blind spots. (FR-6, FR-22, FR-25.)

**LLM Provider Portability:** No Domain Service or UI layer imports a provider SDK directly. All calls route through GenOS LLM Client. (FR-5, FR-7.)

**Observability Backend Portability:** No Domain Service imports Langfuse SDK directly. All tracing via OpenTelemetry. (FR-8, FR-9.)

**Extensibility:** GenOS interfaces are designed for extension without modification. New experiments, adapters, and Domain Services require no GenOS internal changes. (FR-11, FR-21.)

**Experiment Parity:** Every Experiment Run uses the same Knowledge Base, the same eval metric names, and the same observability infrastructure. An interaction that bypasses GenOS tracing is not a valid Experiment Run. (FR-10, FR-12.)

**Student Profile Privacy:** Student Profile data is used only to personalize responses for the active Persona Context. Profile data from one Persona Context is never visible in another. [ASSUMPTION A4: single-operator usage makes this a design constraint, not an enforcement requirement in MVP 1.]

**Data Persistence:** Langfuse Cloud is the system of record for all Traces and Eval Scores. Student Profiles persist in the Student Profile Service store. [ASSUMPTION A5: DB selection for Student Profile store deferred to architecture phase.]

**Reproducibility:** Same configuration + seed data = same pipeline structure and data routing. LLM response non-determinism is acceptable; pipeline behavior is not.

**No Auth (MVP 1):** Authentication and authorization are out of scope for MVP 1. Persona context is set via the Persona Switcher, not login. Real auth introduced in MVP 3.

**Python Primary:** GenOS and all MVP 1 Domain Services implemented in Python. Java permitted for future Domain Services.

**AI-Component Gate:** Any feature whose correct behavior does not require an LLM call, embedding, retrieval, or eval is out of scope. CRUD without AI is a different product.

**Publishable Quality:** Code, documentation, and architecture meet the standard of a senior engineer's open-source reference project. README, architecture overview, and local setup guide exist at v1.

---

## 6. Non-Goals (Explicit)

- **Not a production SaaS product.** No multi-tenancy, billing, or production-grade deployment in MVP 1.
- **Not a university management system.** Domain provides context; CRUD features without AI are out of scope.
- **Not a complete university portal.** Covers specific AI-powered flows only — not the full range of student services.
- **Not a polished consumer product.** MVP 1 quality standard is "demonstrable to senior engineers," not "shippable to 10,000 students."
- **Not a general-purpose chatbot.** Responses are grounded in the Knowledge Base. The AI does not answer questions outside the university domain.
- **No real authentication in MVP 1.** Persona switching is a dev/demo convenience, not a security boundary.

---

## 7. MVP 1 Scope

### 7.1 In Scope

| Component | Section | Key FRs |
|-----------|---------|---------|
| Conversational UI (Student, web) | §4.1 | FR-1–4 |
| GenOS LLM Abstraction Client | §4.2 | FR-5–7c |
| GenOS Observability & Tracing | §4.3 | FR-8–10 |
| GenOS Evaluation Framework | §4.4 | FR-11–13 |
| GenOS Configuration Management | §4.5 | FR-14–15 |
| Student Profile Service | §4.6 | FR-16–18 |
| Knowledge Base Ingestion Pipeline | §4.7 | FR-19–22 |
| RAG Q&A Service | §4.8 | FR-23–25 |
| RAG Evaluation Suite | §4.9 | FR-26–30 |
| README, architecture overview, setup guide | §5 | — |

### 7.2 Out of Scope for MVP 1

- Admin, Advisor, Faculty as active personas — MVP 3+
- Document submission and processing — MVP 3
- Agentic workflows and autonomous agents — MVP 3
- Human-in-the-Loop escalation — MVP 3
- Degree progress and gap detection — MVP 2
- Long-term memory beyond Student Profile — MVP 2
- Real authentication / authorization — MVP 3
- Java Domain Services — post-MVP
- Real-data ingestion adapters — MVP 2+
- Multi-agent coordination — MVP 4
- MCP integration — MVP 4
- Security and guardrails — MVP 5

---

## 8. Product Roadmap

*Lightweight descriptions — sufficient for architectural extensibility decisions. Full FRs written per MVP before implementation.*

### MVP 2 — Know Me
**Theme:** Personalization — AI knows Maya's full academic context.
**Key additions:** Degree progress engine, gap detection, proactive risk flags, grade explanation, knowledge graph for degree requirements.
**New AI experiments:** Long-term memory patterns, contextual RAG with profile injection, LLM reasoning over structured degree data.

### MVP 3 — Do Things For Me
**Theme:** First agentic workflow — admission process end-to-end.
**Key additions:** Document submission mid-chat, document intelligence validator, admission workflow agent (autonomous), Sandra's exception queue (HITL), proactive applicant nudging, real authentication for Admin persona.
**New AI experiments:** Document intelligence, agent state machine, tool use/function calling, event-driven autonomous agents, HITL escalation patterns.

### MVP 4 — Everyone's Assistant
**Theme:** All personas active; multi-agent coordination.
**Key additions:** Advisor intake agent, pre-appointment brief, faculty knowledge ingestion UI, AI-assisted grading, student question escalation/batching, human escalation from student chat.
**New AI experiments:** Multi-agent orchestration, MCP (Model Context Protocol), agent-to-agent communication, LLM-as-judge for grading.

### MVP 5 — Hardening
**Theme:** Security, guardrails, adversarial robustness.
**Key additions:** Prompt injection defense, output validation, hallucination detection, adversarial evals, proactive deadline nudging (fully autonomous).
**New AI experiments:** Guardrails design, red-teaming, hallucination detection at scale.

---

## 9. Success Metrics

### User-Value Metrics (does the product work for users?)

**Primary**

- **SM-1: Answer accuracy** — Maya asks 10 representative university questions; at least 8 receive accurate, Knowledge Base-grounded answers. Validated by manual spot-check at MVP 1 release.
- **SM-2: Personalization correctness** — Maya's course inquiry returns a response referencing her specific academic profile (completed courses, degree requirements). Generic non-personalized responses are a failure. Validates FR-17, FR-23.
- **SM-3: Application status clarity** — Maya's application status query returns current stage + next required action in plain language. No status codes, no jargon. Validates UJ-3, FR-3.

### Platform Metrics (does the AI lab work?)

**Primary**

- **SM-4: Observability completeness** — Zero LLM calls in any Experiment Run without a corresponding Trace in Langfuse. Target: 0 untraced calls. Validates FR-6, FR-25.
- **SM-5: Experiment comparability** — Two RAG variants (different chunk size or top-K) produce side-by-side Eval Scores for all three metrics via Langfuse API. Target: 100% of runs scored. Validates FR-26–29.
- **SM-6: LLM provider comparison** — Running the same set of 10 test queries against Anthropic and OpenAI produces two independently retrievable sets of Eval Scores and Traces in Langfuse, comparable side-by-side. Validates FR-7, FR-29. [Forward metric: GenOS reusability validated when MVP 2 Domain Service is added with zero GenOS core modifications.]

### Secondary

- **SM-7: Onboarding time** — Senior engineer clones repo, configures `.env`, runs seed script, runs first Q&A, sees Langfuse trace — in under 30 minutes. Validates publishable quality NFR.
- **SM-8: Configuration safety** — `git grep` finds zero hardcoded API keys. Validates FR-14.

### Counter-Metrics (do not optimize)

- **SM-C1: Answer scope creep** — AI should not answer questions outside the university domain. Responses to off-topic questions ("what's the weather?") should decline gracefully, not fabricate answers.
- **SM-C2: Eval score inflation** — Eval Scores above 0.95 uniformly across all runs indicate scorer miscalibration, not product excellence. Check scorer first.
- **SM-C3: GenOS complexity** — GenOS core exceeding ~500 LOC before MVP 2 is over-engineered. Simplicity is a feature.

---

## 10. Open Questions

1. **Vector Store selection** — pgvector, Chroma, or Qdrant for MVP 1? Impacts Ingestion Adapter and retrieval implementation. [Architecture phase.]
2. **Student Profile store** — Relational DB, document store, or vector DB for long-term student memory? [Architecture phase.]
3. **LLM orchestration framework** — Raw SDK via GenOS LLM Client, or thin framework (LangChain, LlamaIndex, DSPy) under the abstraction? [Architecture phase; raw SDK is simplest MVP path.]
4. **Monorepo vs. polyrepo** — Single repository for GenOS + all Domain Services? [Architecture phase.]
5. **Conversational UI tech stack** — React, Vue, or simpler? Framework for structured response cards? [UX + architecture phase.]
6. **Synthetic data generation** — LLM-generated vs. hand-authored university documents? [Implementation phase; LLM generation faster but requires generation script.]
7. **Student Profile seed data** — How many synthetic student profiles for MVP 1? What degree progress scenarios to cover? [Implementation phase.]
8. **Experiment registry as first-class construct** — The brainstorming session defines 28 named AI experiments (E-1 through E-28). Should MVP 1 include a formal experiment registry (named configurations, versioned runs) or is Langfuse trace metadata sufficient for now? [Architecture phase — lean toward Langfuse metadata for MVP 1.]

---

## 11. Assumptions Index

- **[ASSUMPTION A1]** Vector Store tool (pgvector, Chroma, Qdrant) deferred to architecture phase. GenOS Ingestion Adapter is store-agnostic. *§3, §4.7*
- **[ASSUMPTION A2]** MVP 1 ships one Ingestion Adapter (Synthetic Data loader). Real-data adapters in MVP 2+. *FR-21*
- **[ASSUMPTION A3]** Langfuse Evals (LLM-as-judge) sufficient for MVP 1 scoring. RAGAS/DeepEval available if Langfuse proves inadequate. Scorer calibration validated in first Experiment Run. *§4.9*
- **[ASSUMPTION A4]** Single-operator usage in MVP 1 means Student Profile privacy is a design constraint, not an enforcement requirement. Real isolation required in MVP 3 when multiple real users are active. *§5*
- **[ASSUMPTION A5]** Student Profile store DB selection deferred to architecture phase. Service interface is store-agnostic. *§4.6*
- **[ASSUMPTION A6]** Persona Switcher (no auth) is acceptable for MVP 1 because the platform is single-operator. Real auth in MVP 3 when Admin/Faculty personas become active for real users. *§5, DL-016*
- **[ASSUMPTION A7]** Synthetic university data is sufficiently realistic for meaningful RAG Eval Scores. If scores are uniformly high regardless of query, the Knowledge Base is the first thing to improve. *FR-19*

---

*PRD v2.0 authored by John (BMad PM Agent) — 2026-06-04.*
*Inputs: Product Brief v1.0 · PRD v1.0 · Brainstorming Session 2026-06-04.*
*Status: final.*
