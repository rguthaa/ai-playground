---
name: University AI
status: final
sources:
  - _bmad-output/planning-artifacts/prds/prd-university-ai-v2-2026-06-04/prd.md
  - _bmad-output/planning-artifacts/product-brief.md
updated: 2026-06-04
---

# University AI — Experience Design

## 1. Foundation

**Form Factor:** Single-surface web application. The Conversational UI occupies the full browser viewport on desktop, tablet, and mobile. No separate mobile app in MVP 1; the web interface is fully responsive.

**Web Stack:** TBD. No framework constraints are imposed by this document. The experience must be achievable with any modern component library that supports dynamic DOM updates (real-time streaming or chunked responses), accessible focus management, and CSS custom properties for theming.

**Visual Reference:** All color tokens (`{colors.*}`), typographic scales (`{typography.*}`), spacing units (`{spacing.*}`), radius values (`{radii.*}`), and shadow definitions (`{shadows.*}`) are defined in **DESIGN.md**. This document references those tokens by name. Implementors must resolve tokens against DESIGN.md before shipping. Where DESIGN.md is silent, use the brand voice and structural decisions herein as authority.

**Scope — MVP 1:**
- One active Persona Context: **Student**
- No authentication. No login wall.
- No history persistence across sessions. Browser tab = one session.
- RAG-powered answers drawn from the university's **Knowledge Base**.
- Persona Switcher rendered but shows only active personas; Admin, Advisor, and Faculty personas are hidden until MVP 3+.

---

## 2. Information Architecture

The Conversational UI is a single page. There are no sub-routes in MVP 1. All surfaces exist within this one view.

| Surface | Location | Purpose | MVP 1 State |
|---|---|---|---|
| **Top Nav Bar** | Fixed top, full width | Houses the product logo (left-aligned) and the Persona Switcher (right-aligned). Provides global identity anchoring. | Active. Student persona chip visible. |
| **Persona Switcher** | Top nav bar, right side | UI selector for active Persona Context. Displays the current persona as a chip. Expands to show available personas. | Visible. Only Student persona shown. Admin/Advisor/Faculty hidden. |
| **Chat Thread** | Main content area, scrollable | Ordered list of message bubbles — student queries and AI responses in chronological order. Empty on session start. | Active. |
| **Empty State** | Chat thread area, session start | Shown when no messages exist. Contains the prompt copy: "Ask me anything about your academics." Disappears once the first message is sent. | Active on every new session. |
| **Typing Indicator** | Chat thread, AI bubble position | Three animated dots in the AI bubble position. Shown while the Experiment Run is in progress. Replaced by the AI response bubble on completion. | Active during loading. |
| **AI Response Bubble** | Chat thread | Displays the AI's answer, formatted text (markdown support preferred). Aligned left, styled with `{colors.surface.ai}` background. | Active after response. |
| **Student Message Bubble** | Chat thread | Displays the student's submitted query. Aligned right, styled with `{colors.surface.student}` background. | Active after submit. |
| **Inline Error Bubble** | Chat thread, AI bubble position | Appears in place of an AI response when the LLM call fails. Muted red surface, crimson border, retry button, error message. | Active on error. |
| **Input Area** | Fixed bottom, full width | Text input field + send button. Accepts student queries. Disabled or visually locked while a response is loading. | Active. |

---

## 3. Voice and Tone

University AI speaks like a knowledgeable academic advisor who genuinely wants the student to succeed — warm, direct, and clear. Never robotic, never casual to the point of imprecision. Responses feel like they come from someone who has read the syllabi, checked the catalog, and remembers the student's history.

**Core voice attributes:**
- **Warm** — addresses the student as an individual, not a ticket number
- **Direct** — leads with the answer, then provides supporting detail
- **Professional** — maintains appropriate register for an academic context
- **Honest** — declines gracefully rather than guessing outside the Knowledge Base scope

**Microcopy Rules**

| Context | Do | Don't |
|---|---|---|
| Empty state prompt | "Ask me anything about your academics." | "Hello! I'm your AI assistant. How can I help you today?" |
| Out-of-domain decline | "That's outside what I have information on. I can help with your courses, degree requirements, and academic policies." | "I don't know the answer to that." or attempting a guess |
| Inline error message | "I wasn't able to get a response. Please try again." | "Error 500" or "Something went wrong" or silence |
| Retry button label | "Try again" | "Retry request" or "Reload" |
| Loading / typing indicator | (No text — animated dots only) | "Loading…" or "Thinking…" as text copy |
| AI referencing student context | "Based on your current courses…" or "Looking at your degree requirements…" | "According to records…" or "The system shows…" |
| Persona chip label | "Student" | "User" or "Guest" or "Anonymous" |
| Input placeholder | "Ask a question…" | "Type here" or "Enter your message" |
| Send button aria-label | "Send message" | "Submit" or "Go" |
| Session boundary notice (if shown) | "Your conversation history isn't saved between sessions." | "Session expired. Please refresh." |

Brand voice specifics (color palette for tone, illustration style, iconography) defer to DESIGN.md. This document owns copy register and interaction language only.

---

## 4. Component Patterns

All components are described in terms of behavior and state. Visual styling (color, type scale, spacing) resolves to DESIGN.md tokens.

| Component | Trigger | Behavior | Styling Tokens | Notes |
|---|---|---|---|---|
| **chat-bubble-ai** | AI response received | Appended to bottom of chat thread. Aligned left. Supports markdown rendering (bold, lists, inline code). Animates in (fade + slight upward translate). | Background: `{colors.surface.ai}`, text: `{colors.text.primary}`, border-radius: `{radii.bubble}`, padding: `{spacing.bubble}`, font: `{typography.body}` | Must be announced to screen readers as a live region update. |
| **chat-bubble-student** | Student submits a message | Appended immediately to bottom of chat thread (optimistic). Aligned right. Plain text only. | Background: `{colors.surface.student}`, text: `{colors.text.onStudent}`, border-radius: `{radii.bubble}`, padding: `{spacing.bubble}`, font: `{typography.body}` | Input area clears immediately on submit. |
| **typing-indicator** | Experiment Run begins | Inserted at AI bubble position in chat thread. Three dots animate in sequence (pulse/bounce). Occupies same space as a chat-bubble-ai would. Removed and replaced when response arrives or error fires. | Background: `{colors.surface.ai}`, dot color: `{colors.accent.primary}`, animation duration: 1.2s loop | Do not render text alongside dots. Aria-live region should read "University AI is responding" once on insert. |
| **chat-bubble-error** | LLM call fails | Inserted at AI bubble position replacing typing-indicator. Muted red surface. Crimson border. Contains error message copy and a "Try again" button. Must never be absent on a failed call. | Background: `{colors.surface.errorMuted}`, border: 1.5px solid `{colors.border.error}`, text: `{colors.text.error}`, button: `{colors.interactive.errorAction}`, border-radius: `{radii.bubble}` | Retry button re-submits the most recent student message. Focus moves to retry button on render. |
| **input-area** | Always visible | Fixed to bottom of viewport. Contains a text field and a send icon button. Enter key submits. Send button submits. Disabled (visually and functionally) while typing-indicator is active. Re-enabled and focused after response or error renders. | Background: `{colors.surface.input}`, border: `{colors.border.input}`, focus ring: `{colors.focus.ring}`, font: `{typography.body}`, placeholder color: `{colors.text.placeholder}` | Placeholder: "Ask a question…". Max height expands for multi-line input up to 4 lines, then scrolls internally. |
| **persona-chip** | Always visible (top nav) | Displays the active Persona Context label ("Student"). Clickable to open Persona Switcher dropdown. In MVP 1, dropdown shows only "Student" as active; other personas are not rendered. | Background: `{colors.surface.chip}`, text: `{colors.text.chip}`, border-radius: `{radii.chip}`, font: `{typography.label}` | If only one persona is active and no others are hidden-but-available, the chip may be non-interactive in MVP 1. Confirm with engineering. |
| **empty-state** | Session start, no messages | Centered in chat thread area. Displays the prompt: "Ask me anything about your academics." Disappears (unmounts) when the first student message is submitted. | Text: `{colors.text.secondary}`, font: `{typography.heading.sm}`, alignment: center, vertical position: middle of chat area | Not a button. Not interactive. Purely informational. |

---

## 5. State Patterns

States are mutually exclusive per chat thread session. Transitions are sequential.

| State | Trigger | Thread Contents | Input Area | Accessibility Signal |
|---|---|---|---|---|
| **Empty Thread** | Session start (new tab, page refresh) | Empty state component centered in thread. No message bubbles. | Enabled. Focused by default. | Page title reads "University AI". Input receives focus on load. |
| **Loading / Typing** | Student submits a message | Student bubble appended. Typing indicator at AI bubble position. | Disabled. No visual affordance to submit. | Aria-live region announces "University AI is responding." Typing indicator has role="status". |
| **Response Rendered** | Experiment Run completes successfully | Typing indicator replaced by AI response bubble. Thread scrolls to bottom. | Re-enabled. Focus returns to input field. | Aria-live region (role="log") appends new AI bubble content. Screen reader reads the new bubble. |
| **Error** | LLM call fails | Typing indicator replaced by inline error bubble. Error copy and retry button visible. | Re-enabled. Focus moves to retry button on error bubble render. | Aria-live region (role="alert") announces the error message. Retry button is keyboard-focusable. |
| **Out-of-Domain Decline** | AI determines query is outside Knowledge Base scope | AI responds with a graceful decline bubble (treated as a normal response bubble, not an error). Thread scrolls to bottom. Input re-enabled. | Re-enabled. Focus returns to input. | No error signal. Treated as a successful response structurally. |
| **Session Start** | User opens or refreshes the page | Always begins as Empty Thread. No history loaded. Any previous thread is discarded. | Enabled. Input focused. | Focus management: input field receives focus automatically. |
| **Mid-Session Error Recovery** | Student clicks "Try again" in error bubble | Error bubble removed. Typing indicator reinserted. Same student message re-submitted to Experiment Run. | Disabled during retry. | Aria-live announces "University AI is responding" again. Same flow as Loading / Typing. |

---

## 6. Interaction Primitives

### Input Submission

- **Enter key:** Submits the current message. If the input is empty or whitespace-only, Enter does nothing (no empty message sent).
- **Shift + Enter:** Inserts a newline within the input field. Does not submit.
- **Send button:** Tap/click submits the message. Identical behavior to Enter key. Button is visually a filled icon button using the send/arrow icon from DESIGN.md iconography.
- **Disabled state during loading:** Both Enter key submission and send button click are suppressed while the typing indicator is active. The input field remains visible and text can be typed (pre-staged), but submission is blocked until the current Experiment Run completes.

### Retry

- The "Try again" button in the error bubble re-submits the last student message verbatim.
- The error bubble is removed from the thread.
- A new typing indicator is inserted at the AI position.
- The Experiment Run restarts.
- If this retry also fails, a new error bubble is rendered. There is no limit on retry attempts in MVP 1.

### Scroll Behavior

- The chat thread is a vertically scrolling container.
- On each new message (student bubble, AI bubble, error bubble, typing indicator insertion), the thread auto-scrolls to the bottom to keep the latest content in view.
- If the user has manually scrolled up to read history, auto-scroll is suppressed until the user scrolls back to the bottom or submits a new message.
- Smooth scroll preferred (`scroll-behavior: smooth`) where supported, with instant fallback.

### Focus Management

- On page load: focus is placed on the input field.
- On message submit: focus remains in the input field; field value clears immediately.
- On response render: focus returns to the input field if it was lost.
- On error render: focus moves to the "Try again" button within the error bubble.
- On retry click: focus moves to the input field after the typing indicator appears.
- On Persona Switcher open: focus moves into the dropdown. On close: focus returns to the persona chip.

---

## 7. Accessibility Floor

University AI targets **WCAG 2.2 Level AA** compliance at launch.

### Keyboard Navigation

- All interactive elements are reachable via Tab in logical document order: Persona Switcher chip → (dropdown items if open) → chat thread (focusable bubbles for screen reader reading) → input field → send button.
- No keyboard traps. Escape closes any open dropdown and returns focus to the triggering element.
- Enter and Space activate buttons and the persona chip.
- Arrow keys navigate within an open Persona Switcher dropdown.

### Screen Reader Announcements

- The chat thread container is marked as `role="log"` and `aria-live="polite"`. Every new bubble (AI response, student message) is announced as it is appended.
- The typing indicator is marked `role="status"` with a single aria-label: "University AI is responding." It is announced once on insertion; subsequent animation frames are silent.
- The inline error bubble is marked `role="alert"` and `aria-live="assertive"` to interrupt and announce the error message immediately.
- The retry button within the error bubble has `aria-label="Try again"`.
- The send button has `aria-label="Send message"`.
- The input field has `aria-label="Message input"` or is associated with a visible label via `aria-labelledby`.

### Color Contrast

- All text on `{colors.surface.ai}`, `{colors.surface.student}`, `{colors.surface.errorMuted}`, and `{colors.surface.input}` must meet a minimum 4.5:1 contrast ratio against their respective text colors as defined in DESIGN.md.
- Error text on `{colors.surface.errorMuted}` must meet 4.5:1 against `{colors.text.error}`.
- The crimson border on the error bubble is decorative reinforcement; it does not carry meaning independently.

### Motion

- The typing indicator animation and bubble fade-in respect `prefers-reduced-motion`. When reduced motion is preferred, dots are static and bubbles appear without transition.

### Target Size

- All interactive elements (send button, retry button, persona chip) meet a minimum 44×44px touch target (WCAG 2.2 2.5.8).

---

## 8. Key Flows

### Primary Flow — Maya's Registration Eve

**Setting:** Sunday evening. Registration opens Monday morning. Maya, a CS junior, has questions about her degree completion and course availability. She opens University AI in her browser.

1. **Session start.** Maya opens the University AI URL in a new browser tab. The page loads. The top nav bar displays the University AI logo on the left. The Persona Switcher chip on the right reads "Student." The chat thread is empty. Centered in the thread: *"Ask me anything about your academics."* The input field at the bottom receives focus automatically.

2. **First query.** Maya types: *"What courses do I still need to complete my CS degree?"* She presses Enter. Her message appears immediately as a student bubble on the right. The empty state disappears. The typing indicator (three animated dots) appears in the AI bubble position on the left. The input field is disabled.

3. **Response.** Three seconds pass. The typing indicator is replaced by an AI response bubble. The response draws from Maya's Student Profile (her academic history, completed courses) and the Knowledge Base (CS degree requirements). The thread scrolls to the bottom. The input field re-enables and receives focus. The aria-live region announces the response.

4. **Follow-up query.** Maya reads her answer, then types: *"Is CIS 401 offered next semester?"* She presses Enter. The same sequence: student bubble, typing indicator, AI response. The answer comes from the Knowledge Base (course schedule data). Thread scrolls to bottom.

5. **Climax.** Under two minutes after opening the tab, Maya has her registration questions answered — without navigating a university portal maze, without hunting through PDFs, without calling an advisor on a Sunday evening. She closes the tab.

6. **Session end.** The browser tab closes. No state is persisted. If Maya opens University AI again in a new tab, she sees the empty thread and the prompt, ready for a fresh session.

---

### Failure Path — LLM Error Mid-Conversation

**Setting:** Same session. After Maya's first successful response, she types her second question and submits.

1. **Student bubble appears.** Maya's query appends to the thread. Typing indicator appears at AI position. Input is disabled.

2. **LLM call fails.** The Experiment Run does not complete successfully. The typing indicator is removed. An inline error bubble appears at the AI position: muted red surface, crimson border. Copy reads: *"I wasn't able to get a response. Please try again."* A "Try again" button is visible within the bubble. Focus moves to the "Try again" button. The aria-live `role="alert"` region announces the error message.

3. **Maya retries.** She clicks (or presses Enter on) "Try again." The error bubble is removed. The typing indicator reappears. The same question is re-submitted to the Experiment Run. The input remains disabled.

4. **Retry succeeds.** The typing indicator is replaced by a normal AI response bubble. Focus returns to the input field. The conversation continues.

5. **Retry fails again.** If the second attempt also fails, a new error bubble renders. Maya can retry again. There is no automatic circuit-breaker in MVP 1. Each retry is a fresh Experiment Run.

**Non-negotiable constraint:** At no point in either scenario is the chat thread blank or silently stalled. Every Experiment Run resolves to either a response bubble or an error bubble. Silence is never an acceptable outcome.
