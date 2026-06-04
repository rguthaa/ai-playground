---
stepsCompleted: [1, 2]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/prd.md
  - _bmad-output/planning-artifacts/architecture.md
  - _bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-university-ai-2026-06-04/EXPERIENCE.md
  - _bmad-output/project-context.md
---

# University AI - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for University AI, decomposing the requirements from the PRD, UX Design, and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

FR-1: The Conversational UI renders a persistent chat thread showing messages chronologically. Messages persist for the lifetime of the browser tab session. Refreshing the page starts a new empty session. A failed LLM call renders an error message in the thread, never a blank or silent response.
FR-2: The UI provides a Persona Switcher. In MVP 1 Student is the only active persona. Admin/Advisor/Faculty are visible but disabled with a "Coming soon" indicator.
FR-3: When an AI response contains structured university data (course info, application status), the UI surfaces that data in a scannable structured format. Application status responses include: current stage, blocking reason (if any), and next required action as distinct fields.
FR-4: The Conversational UI is accessible via a modern web browser on desktop and mobile. Renders correctly at 1440px and 390px viewport widths. Chat input and send button accessible without horizontal scrolling on mobile.
FR-5: The GenOS LLM Client exposes a unified `genos.llm.complete()` interface callable by all Domain Services. No Domain Service file contains a direct import of `anthropic` or `openai` SDK. Same code produces valid responses with both Anthropic and OpenAI configured.
FR-6: Every GenOS LLM Client call emits a GenOS Trace: provider, model, prompt tokens, completion tokens, latency (ms), estimated cost, run metadata. Zero LLM calls occur without a corresponding Trace in Langfuse.
FR-7: Active LLM provider, model, and credentials are set via environment configuration. `GENOS_LLM_PROVIDER=anthropic` or `openai` routes all calls accordingly without code changes.
FR-7b: Embedding model used for query embedding and Knowledge Base ingestion is set via `GENOS_EMBEDDING_MODEL` environment variable. Two ingestion runs with different embedding models produce independently retrievable KB versions.
FR-7c: System prompts and RAG augmentation prompt templates are stored as configurable files, not hardcoded strings. Each Experiment Run Trace records the prompt template version used.
FR-8: GenOS Tracing Layer emits spans and events as OpenTelemetry signals. OTLP endpoint is configurable via `GENOS_OTLP_ENDPOINT`.
FR-9: Domain Services call `genos.trace.span()` and `genos.trace.event()` exclusively. No `langfuse` imports in any Domain Service. Swapping the Langfuse adapter for a stub does not break Domain Service code.
FR-10: Each Experiment Run is assigned a unique run ID propagated through all spans. Filtering Langfuse by run ID returns all spans for that run and only that run.
FR-11: GenOS exposes an `EvalScorer` interface. Domain Services register and invoke scorers at Experiment Run completion. Scorer implementations are swappable without modifying GenOS internals.
FR-12: Every Eval Score is stored in Langfuse linked to its parent Experiment Run Trace. Eval Score history for a named metric is retrievable across multiple runs via Langfuse API.
FR-13: The platform stores a Baseline Run tag. The Eval Framework computes and logs Eval Score deltas between any run and the designated Baseline Run via `genos.eval.set_baseline(run_id)` and `genos.eval.compare_to_baseline(run_id)`.
FR-14: All secrets, provider selections, endpoint URLs, and feature flags load from environment variables or `.env` at startup. `git grep` on the repository finds zero hardcoded API keys.
FR-15: GenOS validates required configuration keys at startup and fails fast with a human-readable error naming the missing key within 2 seconds of startup. No silent failures.
FR-16: The Student Profile Service stores a structured profile per Student persona and retrieves it at the start of each conversation session. Profile contains at minimum: student ID, name, enrolled courses, completed courses, declared major, application state.
FR-17: When the RAG Q&A Service processes a Student query, the active Student Profile is injected as context alongside retrieved Knowledge Base chunks. Two students with different completed courses receive different answers for the same question.
FR-18: The Student Profile Service stores conversation history across sessions. Each stored turn contains: session ID, timestamp, speaker (user or AI), message text, and experiment run ID for AI turns. History is scoped to the active Persona Context — cross-persona leakage is a bug.
FR-19: A seed script generates a Synthetic Data Knowledge Base containing: at least 10 course catalog entries, at least 5 policy documents, at least 20 FAQs, at least 3 application requirement documents.
FR-20: Chunk size (tokens) and chunk overlap (tokens) are configurable. Re-running ingestion with different parameters produces a new independently retrievable Knowledge Base version. ±20% tolerance on chunk sizes is acceptable.
FR-21: Ingestion Pipeline exposes an `IngestionAdapter` interface. Synthetic Data loader is one implementation. New adapters require no pipeline core changes.
FR-22: Each Ingestion Pipeline run emits GenOS Traces with: document count, chunk count, embedding call count, Vector Store write count, total duration. Token usage from all embedding calls is captured.
FR-23: Given a Student query and active Student Profile, the Q&A Service retrieves top-K relevant chunks and passes them with the profile context to the GenOS LLM Client for a personalized answer. Answer Trace includes the chunk IDs used as context.
FR-24: Top-K chunk count and similarity threshold are configurable per Experiment Run. `RAG_TOP_K=3` results in exactly 3 chunks passed to the LLM; `RAG_TOP_K=10` results in exactly 10.
FR-25: Each Q&A query produces a single Experiment Run Trace with five child spans: `query_embedding`, `vector_retrieval`, `prompt_construction`, `llm_call`, `response`. No pipeline stage executes without a corresponding span.
FR-26: Every Q&A Experiment Run is scored on `retrieval_relevance` [0.0–1.0] using LLM-as-judge via Langfuse Evals. A query with no relevant KB chunks scores below 0.3.
FR-27: Every Q&A Experiment Run is scored on `answer_faithfulness` [0.0–1.0]. An answer contradicting retrieved chunk content scores below 0.3.
FR-28: Every Q&A Experiment Run is scored on `answer_relevance` [0.0–1.0]. An answer that ignores the question scores below 0.3.
FR-29: All three Eval Scores for any Experiment Run are retrievable programmatically via the Langfuse API by run ID. Scores for two or more runs can be fetched and compared in a single script.
FR-30: When a user query is outside the university domain, the RAG Q&A Service declines and redirects to university-related topics. Out-of-domain declines are traced in Langfuse with `response_type: out_of_domain` tag.

### NonFunctional Requirements

NFR-1 (Observability): Every LLM call, retrieval step, and eval scoring event produces a GenOS Trace. Zero blind spots. Zero untraced LLM calls allowed.
NFR-2 (LLM Provider Portability): No Domain Service or UI layer imports a provider SDK directly. All calls route through GenOS LLM Client.
NFR-3 (Observability Backend Portability): No Domain Service imports Langfuse SDK directly. All tracing via OpenTelemetry.
NFR-4 (Extensibility): GenOS interfaces designed for extension without modification. New experiments, adapters, and Domain Services require no GenOS internal changes.
NFR-5 (Experiment Parity): Every Experiment Run uses the same KB, same eval metric names, and same observability infrastructure. An interaction that bypasses GenOS tracing is not a valid Experiment Run.
NFR-6 (Student Profile Privacy): Student Profile data is used only for the active Persona Context. Profile data from one Persona Context is never visible in another.
NFR-7 (Data Persistence): Langfuse Cloud is system of record for Traces and Eval Scores. Student Profiles persist in the Student Profile Service store.
NFR-8 (Reproducibility): Same configuration + seed data = same pipeline structure and data routing. Pipeline behavior non-determinism is not acceptable.
NFR-9 (No Auth MVP 1): Authentication and authorization are out of scope for MVP 1. Persona context is set via the Persona Switcher only.
NFR-10 (Python Primary): GenOS and all MVP 1 Domain Services implemented in Python.
NFR-11 (AI-Component Gate): Any feature whose correct behavior does not require an LLM call, embedding, retrieval, or eval is out of scope.
NFR-12 (Publishable Quality): Code, documentation, and architecture meet the standard of a senior engineer's open-source reference project. README, architecture overview, and local setup guide exist at v1. Cold-start under 30 minutes from clone.
NFR-13 (GenOS LOC Cap): GenOS core MUST NOT exceed ~500 LOC before MVP 2.
NFR-14 (Configuration Safety): Zero hardcoded API keys. `git grep` check passes.
NFR-15 (Fail-Fast Config): Missing required config key → named human-readable error within 2 seconds. No silent failures.
NFR-16 (WCAG 2.2 AA): Conversational UI targets WCAG 2.2 Level AA compliance. Enforced via Playwright e2e tests.

### Additional Requirements

- **Monorepo scaffold (first story):** uv workspace root + individual package inits for `genos`, `services/rag_qa`, `services/student_profile`, `services/ingestion`, `services/eval`, plus React/Vite frontend. Project initialization is the first implementation story (Architecture §Starter Template).
- **Docker Compose:** Single `docker compose up` starts all services + Postgres. Required for publishable quality cold-start NFR.
- **Single `.env`:** All services share one `.env` at repo root. `.env.example` committed to repo showing required keys.
- **Makefile:** Dev shortcuts — `make up`, `make down`, `make seed`, `make test`, `make lint`.
- **Database:** Single shared Postgres instance with two schemas: `genos_vectors` (pgvector embeddings + chunk metadata) and `student_profile` (student records + conversation history). Each service owns its Alembic migration env pointing to its own schema.
- **Import enforcement CI gate:** `grep` for `import anthropic`, `import openai`, `import langfuse` in `services/` fails the build. Only `genos/adapters/anthropic.py`, `genos/adapters/openai.py`, `genos/adapters/langfuse.py` may import their respective SDKs.
- **GenOS LOC CI gate:** CI check on `genos/` directory — fail if LOC exceeds 500.
- **CORS:** Every FastAPI service must include `CORSMiddleware` allowing requests from `http://localhost:5173` in dev.
- **Service ports:** UI 5173, RAG Q&A 8001, Student Profile 8002, Ingestion 8003, Eval 8004.
- **Alembic schema ownership:** Each service runs migrations against its own schema only.
- **GitHub Actions CI:** Lint + test + LOC cap + import guard workflows.
- **REST API conventions:** FastAPI, Pydantic validation, auto OpenAPI docs at `/docs`. Direct response body (no envelope). `run_id` is always UUID4. ISO 8601 timestamps everywhere.
- **LangChain scope:** Use only for `RecursiveCharacterTextSplitter` (ingestion) and `ChatPromptTemplate` (RAG). All LLM calls, embeddings, and tracing route through GenOS interfaces.
- **Seed scripts:** LLM-generated synthetic student profiles (Maya + variants) via `seed/generate_student_profiles.py`. Knowledge Base generation via `seed/generate_knowledge_base.py`.
- **README + architecture overview + setup guide:** Required at v1 for publishable quality NFR.

### UX Design Requirements

UX-DR1: Implement design tokens from DESIGN.md mapped to `tailwind.config.js` — 10 color tokens (primary, accent, accent-foreground, surface-base, surface-muted, text-default, text-muted, border, error-surface, error-border), full typography scale (Playfair Display + Inter, 7 size steps, 4 weight levels, 3 line heights), 4-step border radius scale, 7-step spacing scale, 3-level elevation (none/subtle/card/overlay).
UX-DR2: Implement NavBar component — full-width, 56px height, navy (`#1B2A4A`) background, sticky top, elevation.subtle shadow. Left: university wordmark in Playfair Display 22px bold white. Right: PersonaChip. Flex layout with space-between. Mobile: wordmark truncates to icon + short name at < 640px.
UX-DR3: Implement PersonaChip component — pill-shaped badge, white 1.5px outline border, transparent fill, white text, Inter 14px semibold, `padding: 4px 12px`, `border-radius: full`. Optional 16px icon left with 6px gap. Read-only in MVP 1 (no persona switching). Label: "Student".
UX-DR4: Implement ChatBubble-Student component — right-aligned, surface-muted (`#F8F9FB`) background, no border, border-radius 12px, padding 10px 16px, Inter 16px, line-height 1.5, max-width 72%. Optional timestamp below (right-aligned, 12px, text-muted). `margin-left: auto` for right alignment.
UX-DR5: Implement ChatBubble-AI component — left-aligned, surface-base (`#FFFFFF`) background, 1px solid border (`#E5E7EB`), elevation.subtle shadow, border-radius 12px, padding 10px 16px, Inter 16px, line-height 1.7, max-width 72%. Markdown rendering: bold, italic, numbered/bulleted lists, inline code, fenced code blocks. Fade + slight upward translate animation on insert. aria-live region announcement on insert.
UX-DR6: Implement ChatBubble-Error component — left-aligned, error-surface (`#FEF2F2`) background, 1px solid crimson (`#A63232`) border, border-radius 12px, padding 10px 16px, Inter 16px. Contains: (1) error message "I wasn't able to get a response. Please try again." (2) "Try again" inline text link — 14px semibold, crimson, underlined, margin-top 4px. role="alert" aria-live="assertive". Focus moves to retry button on render.
UX-DR7: Implement TypingIndicator component — identical container styling to ChatBubble-AI (white bg, 1px border, elevation.subtle, border-radius 12px, padding 10px 14px). Three 8px dots, border-radius full, text-muted color, 4px gap. Animation: sequential opacity pulse 0.6s ease-in-out infinite, stagger 0.2s per dot. role="status", aria-label="University AI is responding" (announced once). Respects prefers-reduced-motion (static dots when reduced motion preferred).
UX-DR8: Implement InputArea component — sticky bottom, white background, 1px top border, padding 8px 16px. Contains: (1) auto-expanding textarea (flex 1) — surface-muted bg, 1px border, border-radius 8px, padding 10px 14px, Inter 16px, placeholder "Ask a question…", focus: primary border + 2px focus ring at 25% navy opacity, min-height 1 line, max-height 160px then internal scroll, resize none. (2) SendButton (right). Mobile: `padding-bottom: max(8px, env(safe-area-inset-bottom))`. Gains elevation.subtle shadow on scroll past 48px.
UX-DR9: Implement SendButton component — 44×44px square, border-radius 8px, crimson fill, white send-arrow icon 20px filled. States: default (crimson), hover (`#8C2929`, 150ms transition), active (`#732222`), disabled (border-color fill, text-muted icon, cursor not-allowed). aria-label="Send message". Disabled when textarea is empty or while typing indicator is active.
UX-DR10: Implement EmptyState component — centered in chat thread area (vertical + horizontal). Text: "Ask me anything about your academics." text-muted color, Playfair Display heading size. Not interactive. Disappears (unmounts) when first student message is submitted.
UX-DR11: Implement responsive layout — three-zone layout (NavBar sticky top, ChatThread flex-1 scrollable, InputArea sticky bottom). Breakpoints: mobile < 640px (gutter reduces to 12px, bubble max-width 85%), tablet 640–1024px (unchanged), desktop > 1024px (chat thread max-width 760px centered with auto horizontal margins). Chat thread: surface-muted background, padding 24px 16px 16px, 8px gap same-sender bubbles, 16px gap on sender change.
UX-DR12: Implement chat thread scroll behavior — auto-scroll to bottom on every new bubble/indicator insertion. Suppress auto-scroll if user has manually scrolled up; resume on user scroll to bottom or new message submit. `scroll-behavior: smooth` with instant fallback.
UX-DR13: Implement state machine: Empty Thread → Loading/Typing → Response Rendered / Error. Input area disabled (functionally + visually) during Loading/Typing. Focus management: on load → input; on submit → input clears + stays focused; on response → input refocused; on error → retry button focused; on retry click → input after typing indicator appears.
UX-DR14: Keyboard interaction primitives — Enter key submits (no-op on empty/whitespace input), Shift+Enter inserts newline, send button click = Enter, submission suppressed during loading. "Try again" retry re-submits last message, removes error bubble, inserts new typing indicator.
UX-DR15: WCAG 2.2 AA compliance — all interactive elements reachable via Tab in logical order. No keyboard traps. Escape closes dropdowns. role="log" aria-live="polite" on chat thread. Minimum 4.5:1 contrast ratio for all text against surfaces. All interactive elements meet 44×44px touch target minimum (WCAG 2.2 2.5.8). Playwright e2e tests enforce compliance.
UX-DR16: Font loading — Inter via `font-display: swap` from Google Fonts. Playfair Display loads async with Georgia fallback. No layout shift due to fixed nav flex layout.
UX-DR17: Microcopy specification — empty state: "Ask me anything about your academics." | out-of-domain decline: "That's outside what I have information on. I can help with your courses, degree requirements, and academic policies." | error message: "I wasn't able to get a response. Please try again." | retry button: "Try again" | input placeholder: "Ask a question…" | send button aria-label: "Send message" | typing indicator: no text (dots only) | persona chip label: "Student".

### FR Coverage Map

{{requirements_coverage_map}}

## Epic List

{{epics_list}}
