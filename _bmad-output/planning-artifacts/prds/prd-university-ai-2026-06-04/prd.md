---
title: Enterprise AI Lab
status: final
created: 2026-06-04
updated: 2026-06-04
---

# PRD: Enterprise AI Lab (`university-ai`)

**Contents:** [0. Purpose](#0) · [1. Vision](#1) · [2. Target User](#2) · [3. Glossary](#3) · [4. Features](#4) · [5. NFRs](#5) · [6. Non-Goals](#6) · [7. MVP Scope](#7) · [8. Success Metrics](#8) · [9. Open Questions](#9) · [10. Assumptions](#10)

---

## 0. Document Purpose

This PRD is the authoritative requirements contract for the **Enterprise AI Lab** platform — a personal, open-source AI experimentation platform structured as a realistic enterprise application. It is written for the platform operator (a single senior/principal engineer), and for downstream workflow owners (architect, implementation agents) who will design and build from it.

The document uses Glossary-anchored vocabulary throughout. Features are grouped with Functional Requirements nested and globally numbered (FR-1 through FR-N). Assumptions are tagged inline and indexed in §10.

The **Product Brief** (`_bmad-output/planning-artifacts/product-brief.md`) is the upstream input — this PRD inherits its decisions without duplicating rationale. Architecture decisions deferred from this PRD are listed in §9 Open Questions and in the `.decision-log.md`.

---

## 1. Vision

Enterprise AI Lab is a platform built by a senior engineer, for senior engineers — a place to rigorously explore, compare, and demonstrate emerging AI capabilities within a realistic enterprise context. Where most AI learning resources optimize for "getting started," this platform is optimized for "going deep": every experiment is fully instrumented, evaluated, and comparable to every other.

The platform is organized around **GenOS** (Generative AI Operating System) — a shared core layer providing LLM abstraction, observability, tracing, and evaluation as infrastructure. Domain Services (starting with a university-context RAG service) are built on top of GenOS, never beside it. This mirrors how mature enterprise platforms separate platform engineering from application engineering — and that separation is the core architectural bet of the project.

The university domain is the business context, not the product. It provides realistic personas (Student, Advisor, Faculty, Admin), workflows (advising, enrollment, knowledge lookup), and data shapes that make experiments meaningful without requiring production-grade domain engineering. Every AI capability explored in this lab is grounded in a real workflow, observable through Langfuse, and evaluated against a consistent baseline.

The defining characteristic of the platform is **compound learning**: because all Experiments share the same domain data, evaluation baselines, and observability layer, every Experiment Run is comparable to every other — insights accumulate rather than reset.

---

## 2. Target User

### 2.1 Jobs To Be Done

- **Explore** emerging AI capabilities (RAG, agents, multi-agent, MCP, HITL, evals) in a realistic environment, not a notebook
- **Compare** approaches rigorously — variant A vs. variant B with shared eval baselines and traces
- **Accumulate** learning across Experiments — each one building on the last rather than starting from scratch, with cross-Experiment Eval Score history as evidence
- **Demonstrate** capability depth to peers, hiring committees, and engineering forums via a published GitHub repository
- **Practice** enterprise-grade AI engineering: instrumented, evaluated, reproducible, extensible

### 2.2 Non-Users (v1)

- End users expecting a finished product (this is an experimentation platform, not a SaaS product)
- Teams requiring multi-tenancy, role-based access control, or user management
- Anyone evaluating this as a production university management system, CRM, or ERP

### 2.3 Key User Journeys

*Single-operator platform; lighter UJ format used — single operator, no multi-role flows.*

- **UJ-1. Engineer runs a RAG variant and compares it to baseline.**
  Engineer adjusts chunk size and re-runs the RAG Q&A experiment. The Ingestion Pipeline re-indexes the Knowledge Base with new parameters. GenOS traces the full query → retrieval → generation → evaluation pipeline. Langfuse shows the new Experiment Run alongside the prior baseline: faithfulness 0.84 vs 0.79, latency 1.2s vs 0.9s. Engineer records the trade-off and adjusts configuration for the next run.

- **UJ-2. Engineer onboards the platform from the GitHub repository.**
  Engineer clones the repo, creates a Langfuse Cloud account, adds the Langfuse API key to `.env`, configures the LLM provider key, and runs the seed script to generate synthetic university data. Runs the RAG experiment end-to-end. Opens Langfuse and sees the first Trace with Eval Scores. Total time: under 30 minutes.

- **UJ-3. Engineer adds a new AI capability experiment.**
  Engineer builds a new Domain Service (e.g., an agentic advising workflow) using GenOS libraries. Imports `genos.trace`, `genos.llm`, `genos.eval`. Runs the experiment. Langfuse automatically captures Traces and Eval Scores alongside all prior Experiments. No changes to GenOS internals required.

---

## 3. Glossary

- **GenOS** — Generative AI Operating System. The shared platform core layer providing LLM abstraction, observability, tracing, evaluation, and configuration management as infrastructure. All Domain Services are built on GenOS, never beside it.
- **Experiment** — A discrete, runnable implementation of an AI capability (e.g., RAG with a specific configuration). Each Experiment produces Traces and Eval Scores that are stored and comparable to other Experiments.
- **Experiment Run** — A single execution of an Experiment. Produces one or more Traces and a set of Eval Scores.
- **GenOS LLM Client** — The provider-agnostic LLM interface exposed by GenOS. All LLM calls in the platform go through this client. Swapping providers requires configuration change only.
- **GenOS Trace** — A structured record of an Experiment Run's execution: inputs, outputs, intermediate steps, latency, token usage, and cost. Emitted as OpenTelemetry signals, stored in Langfuse.
- **Eval Score** — A measurable quality signal produced at the end of an Experiment Run. Examples: retrieval relevance, answer faithfulness, answer relevance. Stored in Langfuse alongside the Trace.
- **Langfuse** — The selected observability, tracing, and evaluation backend. Used via Langfuse Cloud for MVP. Provides the Experiment Dashboard UI.
- **Experiment Dashboard** — The Langfuse UI. Provides Experiment Run history, Eval Score trends, and Trace inspection. Not a custom-built surface.
- **Knowledge Base** — The corpus of university-domain documents ingested and indexed for RAG Experiments. MVP contains Synthetic Data: course catalogs, policy documents, and FAQs.
- **Ingestion Pipeline** — The process of loading, chunking, embedding, and indexing source documents into the Vector Store to produce a Knowledge Base.
- **Vector Store** — The database storing document embeddings for semantic retrieval. [ASSUMPTION A1: specific tool deferred to architecture phase.]
- **University Persona** — A simulated identity used to provide realistic context for Experiments. Personas (Student, Advisor, Faculty, Admin) are data/context constructs, not authenticated users.
- **Domain Service** — An application service implementing AI capability Experiments within the university domain context. Domain Services are built on GenOS.
- **Synthetic Data** — AI-generated university-domain content used as the MVP Knowledge Base (course catalogs, policy documents, FAQs).
- **Adapter** — A pluggable implementation of a GenOS interface (e.g., an Ingestion Adapter for a specific data source, or a tracing adapter for a specific backend). Adapters enable provider-switching without changing Domain Service code.
- **Baseline Run** — An Experiment Run designated as the reference point for metric comparison. Subsequent Experiment Runs display Eval Score deltas against the Baseline Run.

---

## 4. Features

### 4.1 GenOS Core — LLM Abstraction Client

**Description:** The GenOS LLM Client provides a single, provider-agnostic interface for all LLM calls across the platform. Domain Services import and call the GenOS LLM Client; they never import provider SDKs (Anthropic, OpenAI) directly. Switching LLM providers requires changing configuration, not code. The client handles API key management, retry logic, and automatic emission of a GenOS Trace for every call. Realizes UJ-1, UJ-3.

**Functional Requirements:**

#### FR-1: Provider-agnostic LLM interface

The GenOS LLM Client exposes a unified `complete()` interface that Domain Services call to invoke any configured LLM provider.

**Consequences (testable):**
- A Domain Service calling `genos.llm.complete()` produces a valid LLM response when the configured provider is Anthropic.
- The same Domain Service code, with only provider configuration changed, produces a valid response when the configured provider is OpenAI.
- No Domain Service file contains a direct import of `anthropic` or `openai` SDK.

**Out of Scope:** Fine-tuning, model training, streaming (v1).

#### FR-2: Automatic trace emission on every LLM call

Every invocation of the GenOS LLM Client automatically emits a GenOS Trace containing: provider, model, prompt tokens, completion tokens, latency (ms), estimated cost, and run metadata.

**Consequences (testable):**
- Zero LLM calls occur in any Experiment Run without a corresponding GenOS Trace record in Langfuse.
- Token usage and latency are present on every Trace record.

#### FR-3: Configuration-driven provider selection

The active LLM provider, model name, and API credentials are set via environment configuration. No code change is required to switch providers.

**Consequences (testable):**
- Setting `GENOS_LLM_PROVIDER=anthropic` and `GENOS_LLM_MODEL=claude-sonnet-4-6` routes all calls to Anthropic Claude.
- Setting `GENOS_LLM_PROVIDER=openai` routes all calls to OpenAI without code changes.

---

### 4.2 GenOS Core — Observability & Tracing Layer

**Description:** The GenOS Tracing Layer instruments every meaningful event in an Experiment Run — LLM calls, retrieval steps, eval scoring, pipeline transitions. It emits OpenTelemetry-compatible signals routed to Langfuse Cloud. Domain Services use the GenOS tracing interface (`genos.trace`) exclusively; they never import Langfuse SDK directly. This ensures observability backend portability. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-4: OpenTelemetry-compatible trace emission

The GenOS Tracing Layer emits spans and events as OpenTelemetry signals. The OTLP endpoint is configurable.

**Consequences (testable):**
- All Experiment Run events appear as linked spans in Langfuse trace view.
- Changing `GENOS_OTLP_ENDPOINT` reroutes traces without code changes.

#### FR-5: GenOS trace interface for Domain Services

Domain Services instrument their steps by calling `genos.trace.span()` and `genos.trace.event()`. No Langfuse SDK imports appear in Domain Service code.

**Consequences (testable):**
- A Domain Service with full tracing contains zero `langfuse` imports.
- Removing the Langfuse adapter and substituting a stub does not break Domain Service code.

#### FR-6: Experiment Run context propagation

Each Experiment Run is assigned a unique run ID that is propagated through all spans, enabling end-to-end Trace reconstruction.

**Consequences (testable):**
- Filtering Langfuse traces by run ID returns all spans for that run and only that run.

---

### 4.3 GenOS Core — Evaluation Framework

**Description:** The GenOS Eval Framework provides a consistent interface for defining, running, and storing Eval Scores for any Experiment. Built on Langfuse Evals for MVP — chosen because it is co-located with the tracing backend, eliminating a separate eval tool, and supports LLM-as-judge scoring natively. [ASSUMPTION A3: Langfuse Evals is sufficient for MVP; RAGAS or DeepEval remain options if Langfuse's scoring proves inadequate.] Every Experiment Run produces Eval Scores stored alongside its Trace, enabling cross-run and cross-Experiment comparison. Realizes UJ-1, UJ-3.

**Functional Requirements:**

#### FR-7: Pluggable eval scorer interface

GenOS exposes an `EvalScorer` interface. Domain Services register scorers and invoke them at the end of each Experiment Run. Scorer implementations are swappable without modifying GenOS internals.

**Consequences (testable):**
- A Domain Service can register a custom scorer implementing `EvalScorer` without modifying any GenOS file.
- MVP ships with at least one Langfuse-backed `EvalScorer` implementation.

#### FR-8: Eval Score storage linked to Experiment Run

Every Eval Score is stored in Langfuse linked to its parent Experiment Run Trace.

**Consequences (testable):**
- Selecting an Experiment Run in Langfuse shows all associated Eval Scores.
- Eval Score history for a named metric (e.g., `answer_faithfulness`) is retrievable across multiple Experiment Runs via the Langfuse API.

#### FR-9: Baseline Run designation and delta computation — Realizes UJ-1

The platform stores a tag identifying an Experiment Run as a Baseline Run. The Eval Framework computes and logs the delta between any Experiment Run's Eval Scores and the designated Baseline Run's scores for the same metrics.

**Consequences (testable):**
- Calling `genos.eval.set_baseline(run_id)` stores the baseline designation in Langfuse.
- Running `genos.eval.compare_to_baseline(run_id)` returns a dict of `{metric_name: delta}` for each shared metric between the two runs.
- The delta is computable from stored Eval Score records without relying on any specific Langfuse UI feature.

---

### 4.4 GenOS Core — Configuration Management

**Description:** Centralized, environment-driven configuration for all GenOS components and Domain Services. No hardcoded credentials or provider-specific values anywhere in the codebase. Realizes UJ-2.

**Functional Requirements:**

#### FR-10: Environment-based configuration

All secrets (API keys), provider selections, endpoint URLs, and feature flags are loaded from environment variables or a `.env` file at startup. No defaults contain real credentials.

**Consequences (testable):**
- The platform starts and runs correctly with only a populated `.env` file — no code edits required.
- The repository contains no hardcoded API keys or credentials (verified by `git grep` on commit).

#### FR-11: Configuration schema validation at startup

GenOS validates required configuration keys at startup and fails fast with a clear error message if any required key is missing.

**Consequences (testable):**
- Starting the platform with a missing required key produces a human-readable error naming the missing key within 2 seconds of startup.
- The platform does not silently fail or produce misleading errors on misconfiguration.

---

### 4.5 RAG Experiment Service — Knowledge Base Ingestion

**Description:** The Ingestion Pipeline loads Synthetic Data university-domain documents (course catalogs, policy documents, FAQs), chunks them, generates embeddings via the GenOS LLM Client, and indexes them into the Vector Store. The pipeline is designed as a pluggable Adapter pattern — adding a new data source (e.g., public university open data) requires implementing a new Ingestion Adapter, not modifying the pipeline. Realizes UJ-1, UJ-2.

**Functional Requirements:**

#### FR-12: Synthetic university Knowledge Base generation

A seed script generates a Synthetic Data Knowledge Base containing: at least 5 course catalog entries (each with name, course code, credits, description, and prerequisites fields), at least 3 policy documents (each with title, policy ID, effective date, and body fields), and at least 10 FAQs (each with question and answer fields).

**Consequences (testable):**
- Running the seed script produces all document types with all required fields populated.
- The Ingestion Pipeline processes the seed output without errors.

#### FR-13: Configurable chunking strategy — Realizes UJ-1

The Ingestion Pipeline supports configurable chunk size (in tokens) and chunk overlap (in tokens). Changing these parameters and re-running ingestion produces a new Knowledge Base version.

**Consequences (testable):**
- Setting `INGEST_CHUNK_SIZE=500` produces chunks where 90% of chunks are between 450 and 550 tokens.
- Setting `INGEST_CHUNK_SIZE=200` produces chunks where 90% of chunks are between 180 and 220 tokens.
- Two Ingestion Pipeline runs with different chunk sizes produce distinct, independently retrievable Knowledge Base versions.

#### FR-14: Pluggable Ingestion Adapter interface

The Ingestion Pipeline exposes an `IngestionAdapter` interface. The Synthetic Data loader is one implementation. Future Adapters (filesystem, URL, API) implement the same interface.

**Consequences (testable):**
- A new `IngestionAdapter` implementation can be registered and used without modifying any Ingestion Pipeline core file.
- [ASSUMPTION A2: v1 ships one Adapter (Synthetic Data loader). Real-data Adapters are v2+.]

#### FR-15: Ingestion run traced end-to-end

Each Ingestion Pipeline run emits GenOS Traces covering: document count, chunk count, embedding call count, Vector Store write count, and total duration.

**Consequences (testable):**
- After each Ingestion Pipeline run, Langfuse contains a Trace with all five metrics present.
- The Trace captures token usage from all embedding calls.

---

### 4.6 RAG Experiment Service — Q&A Service

**Description:** The RAG Q&A Service answers natural language questions from the Student Persona context using the indexed Knowledge Base. It implements a standard retrieve-then-generate pipeline: embed the query, retrieve top-K chunks, construct an augmented prompt, call the GenOS LLM Client, return the answer. The full pipeline — query through answer — is traced as a single Experiment Run. Realizes UJ-1, UJ-2.

**Functional Requirements:**

#### FR-16: Retrieve-then-generate pipeline

Given a natural language query, the Q&A Service retrieves the top-K relevant chunks from the Vector Store and passes them as context to the GenOS LLM Client to generate an answer.

**Consequences (testable):**
- A query about course registration policy returns an answer grounded in Knowledge Base content.
- The answer Trace includes the specific chunk IDs used as context, visible in Langfuse.

#### FR-17: Configurable retrieval parameters — Realizes UJ-1

Top-K chunk count and similarity threshold are configurable per Experiment Run via environment or runtime configuration.

**Consequences (testable):**
- Setting `RAG_TOP_K=3` results in exactly 3 chunks passed to the LLM call context.
- Setting `RAG_TOP_K=10` results in exactly 10 chunks passed to the LLM call context.
- Two Experiment Runs with different top-K values produce independently retrievable Traces in Langfuse.

#### FR-18: Full pipeline trace per query

Each Q&A query produces a single Experiment Run Trace with linked child spans for: query embedding, Vector Store retrieval, prompt construction, LLM call, and response.

**Consequences (testable):**
- Each Trace in Langfuse has exactly five child spans corresponding to the five pipeline stages.
- No pipeline stage executes without a corresponding span (verified by comparing span count to pipeline step count in tests).

---

### 4.7 RAG Experiment Service — Evaluation Suite

**Description:** After each Q&A Experiment Run, the Evaluation Suite scores the result on three metrics using Langfuse Evals: retrieval relevance (are the retrieved chunks on-topic?), answer faithfulness (does the answer stay grounded in retrieved context without hallucinating?), and answer relevance (does the answer address the question?). Scores are stored in Langfuse linked to the Experiment Run. Realizes UJ-1.

**Functional Requirements:**

#### FR-19: Retrieval relevance scoring

Each Q&A Experiment Run is scored on retrieval relevance using LLM-as-judge via Langfuse Evals.

**Consequences (testable):**
- Every Q&A Experiment Run has a `retrieval_relevance` Eval Score in Langfuse in the range [0.0, 1.0].
- A query with no relevant chunks in the Knowledge Base scores below 0.3 on `retrieval_relevance`.

#### FR-20: Answer faithfulness scoring

Each Q&A Experiment Run is scored on answer faithfulness using LLM-as-judge via Langfuse Evals.

**Consequences (testable):**
- Every Q&A Experiment Run has an `answer_faithfulness` Eval Score in Langfuse in the range [0.0, 1.0].
- An answer that directly contradicts retrieved chunk content scores below 0.3 on `answer_faithfulness`.

#### FR-21: Answer relevance scoring

Each Q&A Experiment Run is scored on answer relevance using LLM-as-judge via Langfuse Evals.

**Consequences (testable):**
- Every Q&A Experiment Run has an `answer_relevance` Eval Score in Langfuse in the range [0.0, 1.0].
- An answer that ignores the question and returns unrelated content scores below 0.3 on `answer_relevance`.

#### FR-22: Cross-run Eval Score retrieval

All three Eval Scores for any Experiment Run are retrievable programmatically via the Langfuse API, enabling cross-run comparison outside the Langfuse UI.

**Consequences (testable):**
- A script calling the Langfuse API returns `retrieval_relevance`, `answer_faithfulness`, and `answer_relevance` scores for a given run ID.
- Scores for at least two Experiment Runs can be fetched and compared in a single script.

---

## 5. Cross-Cutting NFRs

These apply to all GenOS components and Domain Services.

**Observability:** Every LLM call, retrieval step, and eval scoring event produces a GenOS Trace. Zero blind spots in any Experiment Run. (FR-2, FR-15, FR-18.)

**LLM Provider Portability:** No Domain Service or GenOS component imports a provider SDK directly. All LLM calls route through the GenOS LLM Client. Switching providers requires configuration change only. (FR-1, FR-3.)

**Observability Backend Portability:** No Domain Service imports Langfuse SDK directly. All tracing routes through the GenOS Tracing Layer via OpenTelemetry. Switching backends requires configuration change only. (FR-4, FR-5.)

**Extensibility:** GenOS interfaces (LLM Client, Tracing Layer, Eval Framework, Ingestion Adapter) are designed for extension without modification. Adding a new Experiment or Adapter requires no changes to GenOS internals. (FR-7, FR-14.)

**Experiment Parity:** Every Experiment operates on the same domain data (the Knowledge Base), the same evaluation metric names, and the same observability infrastructure. An Experiment Run that bypasses GenOS tracing or uses a private data source violates platform coherence and is not a valid Experiment. (FR-6, FR-8, FR-15, FR-18.)

**Data Persistence:** Langfuse Cloud is the system of record for all Traces and Eval Scores. The platform itself does not maintain a separate persistent store for Experiment Run history. Loss of Langfuse Cloud connectivity does not corrupt local data. Experiment Runs executed offline will not have Traces or Eval Scores until connectivity is restored. [ASSUMPTION A4: Langfuse Cloud availability is sufficient for a single-operator workflow.]

**Reproducibility:** Given the same configuration and seed data, an Experiment Run produces the same Traces and Eval Scores. Non-determinism from LLM responses is acceptable; pipeline structure and data routing are not.

**No Auth:** Authentication and authorization are explicitly out of scope. The platform is single-operator. University Personas are simulation constructs, not authenticated users.

**Python Primary:** All GenOS libraries and the RAG Domain Service are implemented in Python. Java is permitted for future Domain Services. GenOS must remain Python-first; no Java dependency in GenOS core.

**AI-component gate:** Any feature whose correct behavior does not require an AI component (LLM call, embedding, retrieval, or eval) is out of scope. CRUD endpoints, admin UIs, and reporting dashboards that do not involve an AI step belong to a different project.

---

## 6. Non-Goals (Explicit)

- **Not a university management system.** The university domain provides context, not product scope. Any feature that would be correct without an AI component is out of scope.
- **Not a SaaS product.** No multi-tenancy, billing, user management, or production deployment targets.
- **Not a polished end-user product.** The operator is the only user. UX quality of operator-facing surfaces is secondary to engineering quality.
- **Not a framework or library for others to use directly.** GenOS is designed for this platform; it is not an NPM/PyPI package in v1.
- **Not production-scalable in MVP.** Infrastructure scaling is explicitly out of scope.
- **Not a full-coverage evaluation harness.** MVP covers three RAG metrics. Comprehensive eval suites are post-MVP.

---

## 7. MVP Scope

### 7.1 In Scope

| Component | Section | Key FRs |
|-----------|---------|---------|
| GenOS LLM Abstraction Client | §4.1 | FR-1, FR-2, FR-3 |
| GenOS Observability & Tracing Layer | §4.2 | FR-4, FR-5, FR-6 |
| GenOS Evaluation Framework | §4.3 | FR-7, FR-8, FR-9 |
| GenOS Configuration Management | §4.4 | FR-10, FR-11 |
| Synthetic Knowledge Base seed script | §4.5 | FR-12 |
| Ingestion Pipeline (one Adapter: Synthetic Data) | §4.5 | FR-13, FR-14, FR-15 |
| RAG Q&A Service | §4.6 | FR-16, FR-17, FR-18 |
| RAG Evaluation Suite | §4.7 | FR-19, FR-20, FR-21, FR-22 |
| README, architecture overview, setup guide | §5 SM-5 | — |

### 7.2 Out of Scope for MVP

- Agentic Systems — v2 [NOTE FOR PM: high priority post-MVP, directly enabled by GenOS core]
- Human-in-the-Loop workflows — v2
- Multi-Agent Collaboration — v3
- MCP (Model Context Protocol) — v3
- Workflow Automation — v3
- Security & Guardrails — v4 (hardening phase)
- Java Domain Services — post-MVP
- Real-data Ingestion Adapters (MIT OpenCourseWare, etc.) — post-MVP
- Self-hosted Langfuse — post-MVP (Langfuse Cloud used for MVP)
- Custom-built Experiment Dashboard UI — not planned (Langfuse covers this)
- Streaming LLM responses — post-MVP
- Authentication / authorization — explicitly out of scope

---

## 8. Success Metrics

**Primary**

- **SM-1: GenOS reusability** — When the second Experiment service (Tier 2, post-MVP) is added, a diff of GenOS core files shows zero modifications. Target: 0 GenOS internal file changes. Validates FR-7, FR-14.
- **SM-2: Experiment comparability** — Two RAG Experiment Runs with different configurations produce side-by-side Eval Scores for all three metrics retrievable via the Langfuse API. Target: 100% of Experiment Runs have all three Eval Scores. Validates FR-19, FR-20, FR-21, FR-22.
- **SM-3: Observability completeness** — Zero LLM calls occur in any Experiment Run without a corresponding Trace in Langfuse. Target: 0 untraced calls. Validates FR-2, FR-18.
- **SM-4: Compound learning evidence** — After at least two RAG Experiment Runs, the operator can retrieve a time-series of Eval Scores per metric via the Langfuse API and identify a trend. Target: Eval Score history is programmatically queryable across all Experiment Runs without manual data assembly.
- **SM-5: Open-source readiness** — Repository contains README, architecture doc, and setup guide at v1 release. A senior engineer completes UJ-2 in under 30 minutes on a clean machine.

**Secondary**

- **SM-6: Configuration safety** — `git grep` on the repository finds zero hardcoded API keys or credentials. Validates FR-10.

**Counter-metrics (do not optimize)**

- **SM-C1: Eval score inflation** — Eval Scores should reflect real retrieval and generation quality, not be tuned to look good. A `answer_faithfulness` score above 0.95 on all runs is a signal to check scorer calibration, not a success indicator.
- **SM-C2: GenOS complexity** — GenOS should remain lean. If the core abstraction layer exceeds ~500 LOC before the second Experiment is added, it is over-engineered. Simplicity is a feature.

---

## 9. Open Questions

1. **Vector Store selection** — pgvector, Chroma, or Qdrant for MVP? Impacts GenOS Ingestion Adapter and retrieval implementation. [Architecture phase to resolve.]
2. **LLM orchestration framework** — Raw SDK calls via GenOS LLM Client, or a thin framework (LangChain, LlamaIndex, DSPy) under the abstraction? [Architecture phase to resolve; raw SDK is the simplest MVP path.]
3. **Monorepo vs. polyrepo** — Single repository for GenOS + all Domain Services, or separate repos? [Architecture phase to resolve.]
4. **Java service boundaries** — When Java Domain Services are introduced, do they consume GenOS via a Python subprocess, REST API, or a ported Java GenOS library? [Architecture phase to resolve.]
5. **Synthetic data generation approach** — Use an LLM to generate Synthetic Data, or hand-author it? [Implementation phase to decide; LLM generation is faster but requires a generation script.]

---

## 10. Assumptions Index

- **[ASSUMPTION A1]** Vector Store tool selection (pgvector, Chroma, Qdrant) is deferred to architecture phase. GenOS Ingestion Adapter interface is designed to be store-agnostic. *§3, §4.5*
- **[ASSUMPTION A2]** MVP ships one Ingestion Adapter (Synthetic Data loader). Real-data Adapters are v2+. *FR-14*
- **[ASSUMPTION A3]** Langfuse Evals (LLM-as-judge) is sufficient for MVP scoring. RAGAS and DeepEval remain options if Langfuse's scoring proves inadequate. Scorer calibration validated in first Experiment Run. *§4.3, FR-19–FR-21*
- **[ASSUMPTION A4]** Langfuse Cloud availability is sufficient for a single-operator development workflow. Offline experiment execution is out of scope. *§5 Data Persistence NFR*
- **[ASSUMPTION A5]** Synthetic university data generated by LLM is sufficiently realistic to produce meaningful RAG Eval Scores. If Eval Scores are uniformly high regardless of configuration, the Knowledge Base is the first thing to improve. *FR-12*
- **[ASSUMPTION A6]** LLM-as-judge Eval Scores are consistent enough across runs to be meaningful for comparison. Scorer consistency is validated by running the same query twice and checking score variance. *FR-19–FR-21*

---

*PRD authored by John (BMad PM Agent) — 2026-06-04. Input: Product Brief v1.0 (Mary, BMad Analyst). Status: final.*
