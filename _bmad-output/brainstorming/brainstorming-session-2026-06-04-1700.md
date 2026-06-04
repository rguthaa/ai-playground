---
stepsCompleted: [1, 2, 3]
inputDocuments: []
session_topic: 'University AI - all flows and features for a conversational AI university assistant'
session_goals: 'Map all university persona flows, define MVP tiers, flesh out agentic workflows, produce scoping input for PRD update and UX design'
selected_approach: 'progressive-flow'
techniques_used: ['role-playing']
ideas_generated: [28]
context_file: ''
---

# Brainstorming Session Results

**Facilitator:** AILab
**Date:** 2026-06-04

---

## Session Overview

**Topic:** University AI — all flows and features for a conversational AI university assistant
**Goals:** Map all university persona flows, define MVP tiers, flesh out agentic workflows, produce scoping input for PRD update and UX design

---

## Technique Execution Results

### Phase 1 — Role Playing (Persona Immersion)

Explored four university personas through immersive role play. Each persona surfaced distinct interaction modes, AI capability requirements, and workflow patterns.

---

## Ideas Captured

**[Category #1]: Application Status Tracker**
*Concept:* Student asks "where is my application?" and the AI explains the current stage, what's blocking progress, and what Maya needs to do next — not just a status code.
*Novelty:* Transforms a dumb status lookup into a guided "what do I do next?" conversation.

**[Category #2]: Inline Document Submission**
*Concept:* Student uploads a document (transcript, ID, recommendation letter) directly in the chat. Agent parses, validates completeness, and updates application state — all in one turn.
*Novelty:* No portal switching, no form filling. The conversation IS the submission interface.

**[Category #3]: Graceful Human Escalation**
*Concept:* When the AI hits a wall (policy exception, emotional distress, complex edge case), it hands off to a human with the full conversation context pre-loaded — the human never starts cold.
*Novelty:* Escalation feels like a warm handoff, not an abandonment.

**[Category #4]: Profile-Aware Contextual Advisor**
*Concept:* Every response is personalized to the student's academic profile — completed courses, remaining requirements, schedule, standing. The AI knows Maya, not just the university catalog.
*Novelty:* Replaces the "go look at DegreeWorks and figure it out yourself" experience with a genuine personalized academic guide.

**[Category #5]: Degree Progress Navigator**
*Concept:* Student asks "am I on track to graduate?" and gets a visual + conversational breakdown — what's done, what's left, what's at risk, what to prioritize next semester.
*Novelty:* Proactive gap detection — AI flags risks Maya hasn't noticed yet.

**[Category #6]: Persistent Student Memory**
*Concept:* The AI maintains a long-term profile for each student — academic history, submitted documents, past conversations, preferences, goals. Every interaction builds on the last.
*Novelty:* Transforms the AI from a stateless Q&A bot into a genuine ongoing relationship — like a personal academic concierge who remembers.

**[Category #7]: Autonomous Admission Processing Agent**
*Concept:* An AI agent manages the end-to-end admission document workflow — answering applicant questions, reviewing submissions, validating completeness, and updating application state. Sandra only intervenes when the agent explicitly needs a human decision.
*Novelty:* Sandra goes from processing every application to approving the ones the agent has already verified.

**[Category #8]: Document Intelligence Validator**
*Concept:* Agent reads the transcript, extracts GPA, graduation date, institution name, and cross-validates against what the applicant declared in their form.
*Novelty:* Catches discrepancies automatically — "Applicant declared 3.8 GPA but transcript shows 3.2" flagged before Sandra sees the file.

**[Category #9]: Sandra's Exception Queue**
*Concept:* Sandra's view is a curated queue of decisions only she can make — edge cases, policy exceptions, discrepancy flags, appeals. Everything routine never reaches her.
*Novelty:* Flips the workflow from "Sandra reviews everything" to "Sandra decides what matters."

**[Category #10]: Proactive Applicant Nudging**
*Concept:* Agent proactively contacts applicants about missing items, deadlines, and next steps — without Sandra initiating. Full conversation logged and visible to Sandra at any time.
*Novelty:* Sandra's team stops sending follow-up emails. The agent is the follow-up machine.

**[Category #11]: Advisor Pre-Appointment Brief**
*Concept:* Before each advising appointment, AI generates a one-page brief for Dr. Raj — student's academic standing, recent AI conversations, flagged risks, open questions the AI couldn't resolve.
*Novelty:* Dr. Raj never starts an appointment cold. Every meeting is already 10 minutes ahead.

**[Category #12]: Advisor Intake Agent**
*Concept:* Before a student reaches an advisor, the AI conducts a structured intake — clarifies the question, collects relevant context, attaches the student's profile and degree audit — then packages it cleanly for Dr. Raj.
*Novelty:* Dr. Raj receives a structured brief instead of a vague email. He can respond substantively in one message instead of three clarification rounds.

**[Category #13]: Faculty Knowledge Ingestion**
*Concept:* Prof. Chen's course materials — syllabus, policies, FAQs, assignment details — are ingested into the Knowledge Base automatically. Student questions about her courses get answered without her involvement.
*Novelty:* Prof. Chen uploads once, students ask forever. She stops answering "what's on the exam?" at 11pm.

**[Category #14]: AI-Assisted Grading**
*Concept:* Prof. Chen uploads rubric + submissions. AI scores each one, generates feedback, and surfaces only the borderline cases for human review. She approves the batch rather than grading one by one.
*Novelty:* Grading 80 assignments becomes reviewing 15 edge cases.

**[Category #15]: Faculty Question Escalation**
*Concept:* When students ask questions the AI can't answer from course materials, it batches similar questions and surfaces them to Prof. Chen as a single decision. Her one answer updates the Knowledge Base and resolves all pending student questions simultaneously.
*Novelty:* Prof. Chen makes one decision, the AI fans it out to all students and remembers it for future questions.

**[Category #16]: Grade Transparency Explainer**
*Concept:* After grades are posted, the AI explains a student's specific score in plain language, referencing the exact rubric criteria.
*Novelty:* Grade disputes drop significantly because students understand their score before they complain.

**[Category #17]: Event-Driven Autonomous Agents**
*Concept:* Agents that wake up on state changes — application submitted, document uploaded, deadline approaching, grade posted — and take action without waiting for a human prompt.
*Novelty:* The university starts to feel proactive rather than reactive. Things happen to students, not just for students when they ask.

---

## Functional User Journeys

| UJ # | Journey | Persona | MVP |
|------|---------|---------|-----|
| UJ-1 | Enrollment info lookup | Student | MVP 1 |
| UJ-2 | Course inquiry | Student | MVP 1 |
| UJ-3 | Application status check | Student | MVP 1 |
| UJ-6 | Course recommendation | Student | MVP 2 |
| UJ-7 | Degree progress + gap detection | Student | MVP 2 |
| UJ-18 | Grade explanation | Student | MVP 2 |
| UJ-4 | Document submission mid-chat | Student | MVP 3 |
| UJ-8 | Applicant Q&A deflection | Admin | MVP 3 |
| UJ-9 | Document review + validation | Admin | MVP 3 |
| UJ-10 | Application completion marking | Admin | MVP 3 |
| UJ-11 | Sandra's exception queue | Admin | MVP 3 |
| UJ-12 | Proactive applicant nudge | Autonomous | MVP 3 |
| UJ-19 | Full admission workflow | Autonomous | MVP 3 |
| UJ-5 | Human escalation | Student | MVP 4 |
| UJ-13 | Pre-appointment brief | Advisor | MVP 4 |
| UJ-14 | Advisor intake agent | Student→Advisor | MVP 4 |
| UJ-15 | Course knowledge ingestion | Faculty | MVP 4 |
| UJ-16 | AI-assisted grading | Faculty | MVP 4 |
| UJ-17 | Student question escalation | Faculty | MVP 4 |
| UJ-20 | Deadline nudge (proactive) | Autonomous | MVP 5 |

---

## MVP Tiers

### MVP 1 — Hello World
**Theme:** Conversational Student Q&A — prove the full stack works
**Journeys:** UJ-1, UJ-2, UJ-3
**AI Features:** E-1 Basic RAG, E-2 Embedding Models, E-3 LLM Response Quality, E-4 Prompt Engineering, E-5 Retrieval Evaluation

### MVP 2 — Know Me
**Theme:** Personalization — AI knows who Maya is
**Journeys:** UJ-6, UJ-7, UJ-18
**AI Features:** E-6 Long-term Memory, E-7 Contextual RAG, E-8 Knowledge Graph vs Vector Store, E-9 Reasoning over structured data

### MVP 3 — Do Things For Me
**Theme:** First agentic workflow — admission process
**Journeys:** UJ-4, UJ-8, UJ-9, UJ-10, UJ-11, UJ-12, UJ-19
**AI Features:** E-10 Document Intelligence, E-11 Agent State Machine, E-12 Tool Use, E-13 Autonomous Agent Triggers, E-14 HITL Design, E-15 Agent Reliability

### MVP 4 — Everyone's Assistant
**Theme:** Expand to Advisor + Faculty personas
**Journeys:** UJ-5, UJ-13, UJ-14, UJ-15, UJ-16, UJ-17
**AI Features:** E-16 Multi-Agent Orchestration, E-17 MCP, E-18 Agent-to-Agent Communication, E-19 Context Window Management, E-20 LLM-as-Judge

### MVP 5 — Hardening
**Theme:** Security, guardrails, adversarial evals
**Journeys:** UJ-20
**AI Features:** E-21 Prompt Injection Defense, E-22 Output Validation, E-23 Hallucination Detection, E-24 Adversarial Evals

---

## AI Experiments Map (28 total)

### MVP 1 — RAG Foundation
- E-1: Basic RAG (chunk size, overlap, top-K)
- E-2: Embedding model comparison
- E-3: LLM response quality per provider
- E-4: Prompt engineering patterns
- E-5: Retrieval evaluation (faithfulness, relevance)

### MVP 2 — Memory & Personalization
- E-6: Long-term memory storage + retrieval
- E-7: Contextual RAG with student profile injection
- E-8: Knowledge graph vs vector store for degree data
- E-9: LLM reasoning over structured degree audit data

### MVP 3 — Agentic Systems
- E-10: Document intelligence (extract + validate)
- E-11: Agent state machine across multi-turn conversations
- E-12: Tool use / function calling reliability
- E-13: Event-driven autonomous agent triggers
- E-14: HITL escalation patterns
- E-15: Agent reliability and failure modes

### MVP 4 — Multi-Agent & Collaboration
- E-16: Multi-agent orchestration
- E-17: MCP (Model Context Protocol)
- E-18: Agent-to-agent communication
- E-19: Context window management in long conversations
- E-20: LLM-as-judge for grading accuracy

### MVP 5 — Hardening
- E-21: Prompt injection defense
- E-22: Output validation and guardrails
- E-23: Hallucination detection
- E-24: Adversarial evals / red-teaming

### Cross-Cutting (Every MVP)
- E-25: Observability completeness
- E-26: Evaluation framework consistency
- E-27: LLM provider comparison (Anthropic vs OpenAI)
- E-28: GenOS abstraction durability

---

## Key Architectural Decisions Surfaced

1. **Long-term memory** — Student profile persists across sessions. DB selection deferred to architecture phase.
2. **Two agent modes** — Assistant mode (human-initiated) + Autonomous mode (event-triggered).
3. **Single pane of glass UI** — One conversational interface, multiple personas, structured UI elements alongside chat.
4. **GenOS as backbone** — Every AI call traced, evaluated, observable regardless of which persona or flow.
5. **Pluggable persona context** — UI switches persona context (Student vs Admin vs Faculty) without auth.

---

*Session complete — ready for PRD update and UX design.*
