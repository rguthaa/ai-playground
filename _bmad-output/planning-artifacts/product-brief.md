# Product Brief: Enterprise AI Lab (university-ai)

**Version:** 1.0  
**Date:** 2026-06-04  
**Author:** Mary (BMad Business Analyst)  
**Status:** Ready for PM Handoff

---

## 1. Concept Statement

**Enterprise AI Lab** is a personal, open-source AI experimentation platform structured as a realistic enterprise application. It uses a university domain as its business context to provide authentic workflows, data shapes, and personas — while the true purpose is incremental exploration, evaluation, and demonstration of emerging AI capabilities.

The platform is architected around a **GenOS (Generative AI Operating System)** core: a set of shared libraries and services that provide cross-cutting concerns (observability, tracing, evaluations, LLM abstraction, guardrails) to all domain services built on top. This mirrors how mature enterprise platforms separate platform engineering from application engineering.

---

## 2. Problem Statement

Senior engineers exploring AI capabilities today face a common trap: they build isolated demos that work in isolation but cannot be compared, combined, or evolved. Observations from one experiment don't inform the next. There is no shared evaluation baseline, no consistent tracing, and no coherent data layer — making it impossible to rigorously compare approaches or demonstrate compound learning over time.

Existing public examples (notebooks, sample apps, quickstarts) are optimized for "getting started" not for "going deep." They are not structured for enterprise-grade practices and do not teach engineers how these capabilities behave at realistic scale or complexity.

---

## 3. Vision

Build an enterprise-grade AI experimentation platform that:

- Treats every AI capability as a **first-class, comparable experiment** — not a throwaway demo
- Provides a **GenOS core layer** so all experiments share observability, tracing, evals, and LLM abstraction from day one
- Uses a **university domain** as realistic business context, with authentic personas, workflows, and data
- Grows incrementally across **capability breadth** (new AI techniques), **depth** (deeper implementations of each), and **domain complexity** (richer university workflows)
- Is **publishable and demonstrable** — structured so others can learn from it on GitHub or in conference/forum demos

---

## 4. Target Users

### Primary User (Operator)
**Senior/Principal Software Engineer — AI capability researcher**
- Deep software engineering background, learning AI/ML engineering practices
- Wants to evaluate and compare AI approaches rigorously, not just run tutorials
- Will operate, extend, and demo the platform personally
- Values enterprise-grade practices: observability, testability, reproducibility

### Secondary Users (Personas within the system — not real users)
These are simulated identities that make the university domain realistic for experiments:

| Persona | Role in Experiments |
|---------|---------------------|
| Student | Knowledge seeker, RAG consumer, HITL subject |
| Academic Advisor | Agentic workflow participant, decision maker |
| Faculty | Content producer, knowledge base contributor |
| Administrator | Workflow orchestrator, system overseer |

### Audience (for publication/demos)
- Fellow senior engineers evaluating AI approaches
- Engineering teams considering similar platforms
- Conference / forum attendees seeking practical enterprise AI patterns

---

## 5. Core Value Proposition

> **Enterprise AI Lab gives experienced engineers a coherent, observable, and comparable platform for exploring AI capabilities — so every experiment builds on the last instead of starting from scratch.**

The three differentiating bets:

1. **GenOS Core**: Shared cross-cutting concerns (obs, tracing, evals, LLM abstraction) are not bolted on — they are the foundation every service is built on
2. **Domain authenticity**: University workflows provide realistic complexity without building a real product
3. **Compound learning**: Because all experiments share the same data, evaluation baselines, and observability layer, results are comparable and insights accumulate

---

## 6. Capability Areas

Listed in recommended exploration order, grouped by dependency:

### Tier 1 — Foundation (MVP)
These must exist before anything else is meaningful:

| Capability | Purpose |
|-----------|---------|
| **Observability & Tracing** | Every LLM call, agent step, and workflow transition is traced; GenOS core concern |
| **Evaluations (Evals)** | Every experiment produces measurable quality signals; GenOS core concern |
| **RAG** | First domain-facing capability; validates the GenOS data and retrieval layer |

### Tier 2 — Agentic Layer
| Capability | Purpose |
|-----------|---------|
| **Agentic Systems** | Single-agent task execution within university workflows |
| **Human-in-the-Loop** | Advisor approval flows, escalation patterns |
| **Knowledge Management** | Structured knowledge bases powering RAG and agents |

### Tier 3 — Advanced Patterns
| Capability | Purpose |
|-----------|---------|
| **Multi-Agent Collaboration** | Coordinated agent workflows (e.g., student advising pipeline) |
| **MCP (Model Context Protocol)** | Tool/context standardization across agents |
| **Workflow Automation** | Orchestrated multi-step university processes |

### Tier 4 — Hardening
| Capability | Purpose |
|-----------|---------|
| **Security & Guardrails** | Prompt injection defense, output validation, policy enforcement |

---

## 7. Architecture Principles

These are non-negotiable constraints derived from the vision:

1. **GenOS First**: No domain service is built without the GenOS shared layer beneath it. Observability, tracing, evals, and LLM abstraction are infrastructure, not features.
2. **LLM Agnostic**: All LLM calls go through a GenOS abstraction layer. Swapping providers (Anthropic, OpenAI, local models) requires configuration change, not code change.
3. **Polyglot, Python Primary**: Python for all GenAI/orchestration code. Java or other languages permitted for domain services where appropriate. GenOS libraries must have Python-first implementations.
4. **Open-Source Components**: Built on OSS stack. Frontier model APIs (Anthropic, OpenAI) are acceptable exceptions. No proprietary platforms that cannot be self-hosted.
5. **No Auth Overhead**: Single operator model. Multiple university personas are simulated via data/context, not authentication. Auth is explicitly out of scope for now.
6. **Experiment Parity**: Every experiment operates on the same domain data, evaluation baselines, and observability infrastructure — results must be comparable.
7. **Publishable Quality**: Code, documentation, and architecture must meet the standard of something you'd be proud to share on GitHub and reference in a senior engineering forum.

---

## 8. MVP Scope

### MVP Goal
Prove the GenOS core architecture is sound by delivering one end-to-end capability experiment that is fully observable, evaluated, and traceable — using university domain data.

### MVP Deliverables

**GenOS Core (shared layer):**
- LLM abstraction client (provider-agnostic, supports Anthropic + OpenAI at minimum)
- Structured logging and tracing library (every LLM call instrumented)
- Evaluation framework skeleton (metric definition, result storage, baseline comparison)
- Shared configuration management

**Domain Service — RAG (first experiment):**
- University knowledge base ingestion pipeline (course catalog, policy documents, FAQs)
- Retrieval-augmented Q&A service for Student persona
- Evaluation suite: retrieval quality, answer faithfulness, answer relevance
- Full trace: query → retrieval → generation → evaluation → logged result

**Observability Dashboard (lightweight):**
- Experiment run history
- Eval metric trends over time
- LLM call logs with latency, token usage, cost estimates

### Explicitly Out of MVP
- Multi-agent coordination
- MCP integration
- Human-in-the-Loop workflows
- Security/Guardrails
- Java services
- UI beyond a minimal experiment dashboard

---

## 9. Success Criteria

| Criterion | Measurement |
|-----------|-------------|
| GenOS core is reusable | Second experiment added without modifying GenOS internals |
| Experiments are comparable | RAG variant A vs. variant B produces side-by-side eval metrics |
| Platform is demonstrable | A senior engineer unfamiliar with the project can run it and understand findings within 30 minutes |
| Observability is complete | Zero LLM calls occur without a trace record |
| Open-source ready | README, architecture doc, and setup guide exist at v1 |

---

## 10. Open Questions for PM / Architecture Phase

These are intentionally deferred — they require deeper technical investigation:

| # | Question | Impact |
|---|----------|--------|
| 1 | Which tracing/observability backend? (LangSmith, Langfuse, OpenTelemetry + custom, Arize) | GenOS core design |
| 2 | Which vector store? (pgvector, Chroma, Qdrant, Weaviate) | RAG architecture |
| 3 | Which eval framework? (RAGAS, DeepEval, custom, Langfuse evals) | Eval layer design |
| 4 | Which LLM orchestration framework? (LangChain, LlamaIndex, raw SDK, DSPy) | GenOS abstraction |
| 5 | Monorepo or polyrepo structure? | Developer experience |
| 6 | Java services: same monorepo or separate? | Architecture boundary |
| 7 | University seed data: synthetic generation or public datasets? | RAG experiment quality |

---

## 11. Handoff Notes for PM Agent

- **Project name**: `university-ai` (repo), **platform name**: Enterprise AI Lab
- **The GenOS concept is central** — the PRD must treat it as the foundational architectural concern, not an afterthought
- **Capability tiers** (Section 6) are the natural epic/story structure — Tier 1 = MVP epic
- **University personas** (Section 4) are simulation constructs, not real users — the PRD should not model them as auth-bearing users
- **Technology decisions** (Section 10) are open — architecture phase should resolve these before implementation
- **Publishing intent** means documentation and project structure are first-class concerns from the start

---

*Produced by Mary, BMad Business Analyst — ready for handoff to PM (`bmad-agent-pm`) for PRD creation.*
