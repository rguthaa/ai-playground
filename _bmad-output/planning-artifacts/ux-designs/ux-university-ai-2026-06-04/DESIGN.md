---
name: University AI
status: final
updated: 2026-06-04
---

```yaml
tokens:
  colors:
    primary: "#1B2A4A"          # Navy — nav bar, headings, icon fills
    accent: "#A63232"           # Crimson — send button, active states, links
    accent-foreground: "#FFFFFF" # Text/icons placed on crimson surfaces
    surface-base: "#FFFFFF"     # Default page and card background
    surface-muted: "#F8F9FB"    # Chat area background, student bubble fill
    text-default: "#1F2937"     # Body copy, bubble text
    text-muted: "#6B7280"       # Timestamps, placeholder, secondary labels
    border: "#E5E7EB"           # Dividers, AI bubble outline, input border
    error-surface: "#FEF2F2"    # Error bubble background
    error-border: "#A63232"     # Error bubble border (reuses crimson)

  typography:
    font-heading: "'Playfair Display', Georgia, serif"
    font-body: "'Inter', system-ui, sans-serif"
    size-xs: "12px"
    size-sm: "14px"
    size-base: "16px"
    size-lg: "18px"
    size-xl: "22px"
    size-2xl: "28px"
    weight-regular: "400"
    weight-medium: "500"
    weight-semibold: "600"
    weight-bold: "700"
    line-height-tight: "1.25"
    line-height-base: "1.5"
    line-height-relaxed: "1.7"

  rounded:
    sm: "4px"
    md: "8px"
    lg: "12px"
    full: "9999px"  # pill shapes: persona chip, typing indicator dots

  spacing:
    xs: "4px"
    sm: "8px"
    md: "16px"
    lg: "24px"
    xl: "32px"
    2xl: "48px"
    chat-gutter: "16px"        # horizontal margin from viewport edge to bubble
    bubble-padding-x: "16px"
    bubble-padding-y: "10px"
    input-height: "56px"
    nav-height: "56px"

  elevation:
    none: "none"
    subtle: "0 1px 3px rgba(0,0,0,0.08)"
    card: "0 2px 8px rgba(0,0,0,0.10)"
    overlay: "0 8px 24px rgba(0,0,0,0.14)"

  components:
    nav-bar:
      background: "{colors.primary}"
      height: "{spacing.nav-height}"
      padding-x: "{spacing.lg}"
      logo-font: "{typography.font-heading}"
      logo-size: "{typography.size-xl}"
      logo-weight: "{typography.weight-bold}"
      logo-color: "{colors.accent-foreground}"
      border-bottom: "none"
      elevation: "{elevation.subtle}"
      layout: "flex; align-items: center; justify-content: space-between"

    persona-chip:
      background: "transparent"
      border: "1.5px solid {colors.accent-foreground}"
      color: "{colors.accent-foreground}"
      font: "{typography.font-body}"
      font-size: "{typography.size-sm}"
      font-weight: "{typography.weight-medium}"
      padding: "4px 12px"
      border-radius: "{rounded.full}"
      icon-size: "16px"
      icon-gap: "6px"

    chat-bubble-student:
      background: "{colors.surface-muted}"
      border: "none"
      border-radius: "{rounded.lg}"
      padding: "{spacing.bubble-padding-y} {spacing.bubble-padding-x}"
      font: "{typography.font-body}"
      font-size: "{typography.size-base}"
      color: "{colors.text-default}"
      max-width: "72%"
      align: "right"
      margin-bottom: "{spacing.sm}"

    chat-bubble-ai:
      background: "{colors.surface-base}"
      border: "1px solid {colors.border}"
      border-radius: "{rounded.lg}"
      padding: "{spacing.bubble-padding-y} {spacing.bubble-padding-x}"
      font: "{typography.font-body}"
      font-size: "{typography.size-base}"
      color: "{colors.text-default}"
      max-width: "72%"
      align: "left"
      margin-bottom: "{spacing.sm}"
      elevation: "{elevation.subtle}"

    chat-bubble-error:
      background: "{colors.error-surface}"
      border: "1px solid {colors.error-border}"
      border-radius: "{rounded.lg}"
      padding: "{spacing.bubble-padding-y} {spacing.bubble-padding-x}"
      font: "{typography.font-body}"
      font-size: "{typography.size-base}"
      color: "{colors.text-default}"
      max-width: "72%"
      align: "left"
      margin-bottom: "{spacing.sm}"
      retry-button:
        label: "Retry"
        font-size: "{typography.size-sm}"
        font-weight: "{typography.weight-semibold}"
        color: "{colors.accent}"
        background: "transparent"
        border: "none"
        margin-top: "{spacing.xs}"
        cursor: "pointer"
        text-decoration: "underline"

    typing-indicator:
      container-background: "{colors.surface-base}"
      container-border: "1px solid {colors.border}"
      container-border-radius: "{rounded.lg}"
      container-padding: "10px 14px"
      container-elevation: "{elevation.subtle}"
      container-align: "left"
      dot-size: "8px"
      dot-color: "{colors.text-muted}"
      dot-gap: "4px"
      dot-border-radius: "{rounded.full}"
      animation: "sequential opacity pulse, 0.6s ease-in-out, infinite, stagger 0.2s"

    send-button:
      background: "{colors.accent}"
      color: "{colors.accent-foreground}"
      border: "none"
      border-radius: "{rounded.md}"
      width: "44px"
      height: "44px"
      icon: "send-arrow (filled, 20px)"
      elevation: "none"
      hover-background: "#8C2929"   # crimson darkened ~10%
      active-background: "#732222"
      disabled-background: "{colors.border}"
      disabled-color: "{colors.text-muted}"
      transition: "background 150ms ease"

    input-area:
      background: "{colors.surface-base}"
      border-top: "1px solid {colors.border}"
      padding: "{spacing.sm} {spacing.chat-gutter}"
      height-min: "{spacing.input-height}"
      height-max: "160px"   # auto-expands up to this before scrolling
      layout: "flex; align-items: flex-end; gap: 8px"
      textarea:
        background: "{colors.surface-muted}"
        border: "1px solid {colors.border}"
        border-radius: "{rounded.md}"
        padding: "10px 14px"
        font: "{typography.font-body}"
        font-size: "{typography.size-base}"
        color: "{colors.text-default}"
        placeholder-color: "{colors.text-muted}"
        focus-border: "{colors.primary}"
        focus-outline: "2px solid rgba(27,42,74,0.25)"
        resize: "none"
        flex: "1"
```

---

## Brand & Style

University AI is a white-label conversational AI assistant for universities. The brand must recede — it is a platform any university can adopt and skin. The visual language is institutional and credible: a navy anchor conveys authority and tradition; crimson signals action and academic heritage; Playfair Display headers bring academic character without being decorative. The product premise: a student should feel they are talking to a knowledgeable, warm advisor — not a generic chatbot and not a cold university portal.

The design avoids the visual language of consumer AI products (neon gradients, animated blobs, oversized marketing copy). Instead it borrows from the lexicon of institutional design — clean whitespace, considered typography, muted surfaces — updated with the clarity and efficiency standards of modern web products. Every element earns its place.

---

## Colors

| Role | Token | Hex |
|---|---|---|
| Primary — nav, headings | `primary` | `#1B2A4A` |
| Accent — action, links | `accent` | `#A63232` |
| Accent foreground | `accent-foreground` | `#FFFFFF` |
| Base surface | `surface-base` | `#FFFFFF` |
| Muted surface | `surface-muted` | `#F8F9FB` |
| Body text | `text-default` | `#1F2937` |
| Muted / secondary text | `text-muted` | `#6B7280` |
| Border / divider | `border` | `#E5E7EB` |
| Error surface | `error-surface` | `#FEF2F2` |
| Error border | `error-border` | `#A63232` |

**Navy (`#1B2A4A`)** is reserved for structural chrome: the nav bar, icon fills, and any heading treatments that reinforce the institutional frame. It does not appear in the chat body or content surfaces.

**Crimson (`#A63232`)** is reserved for a single primary action at a time. In MVP 1 that is the send button. It also appears as the border and link on error bubbles to maintain a consistent "action required" signal.

**Surface muted (`#F8F9FB`)** grounds the student bubble and the textarea background, giving the chat thread a dual-tone rhythm that makes it immediately legible which side each message is on without requiring avatars.

**Error surface (`#FEF2F2`)** with a crimson border forms the exclusive visual treatment for AI errors. It signals urgency without being alarming — it reads as a note, not a modal interruption.

---

## Typography

| Role | Family | Size | Weight |
|---|---|---|---|
| Page / section headings | Playfair Display, serif | 22–28px | 700 |
| Nav logo wordmark | Playfair Display, serif | 22px | 700 |
| Chat bubble body | Inter, sans-serif | 16px | 400 |
| Timestamp / muted label | Inter, sans-serif | 12px | 400 |
| Retry link / chip label | Inter, sans-serif | 14px | 600 |
| Input placeholder | Inter, sans-serif | 16px | 400 |

**Playfair Display** is used sparingly and only for structural identity elements (the wordmark in the nav bar, and any empty-state heading). It is never used inside chat bubbles, buttons, or form controls. Its presence signals "this is a university product" without competing with legibility.

**Inter** handles all functional type. Its generous x-height and optical spacing make it well-suited for conversational content at 16px. Line-height of 1.5 is applied to bubble body text for comfortable reading across long AI responses. For muted labels (timestamps, chip text), 14px / 1.25 is sufficient.

**Font loading strategy:** Load Inter via `font-display: swap` from Google Fonts. Playfair Display loads async — the wordmark falls back to Georgia. No layout shift occurs because the wordmark width is fixed by the nav flex layout.

---

## Layout & Spacing

The viewport is divided into two horizontal bands:

```
┌─────────────────────────────────────┐
│  Nav Bar (56px, navy, sticky top)   │
├─────────────────────────────────────┤
│                                     │
│   Chat Thread (flex-1, scrollable)  │
│   — surface-muted background        │
│   — 16px horizontal gutter          │
│   — bubbles max-width 72%           │
│                                     │
├─────────────────────────────────────┤
│  Input Area (min 56px, sticky bot)  │
└─────────────────────────────────────┘
```

**Nav Bar** is 56px tall, sticky, navy. It contains: left-aligned university wordmark (Playfair Display, white, 22px) and right-aligned persona chip.

**Chat Thread** is the remaining viewport height, overflow-y scroll, with `padding: 24px 16px 16px`. Messages are stacked vertically with 8px gap between consecutive bubbles from the same sender, 16px gap when sender changes. Bubbles are constrained to 72% of the chat width to preserve the conversational indent pattern on all screen sizes.

**Input Area** is sticky to the bottom, white background, 1px top border. It contains the auto-expanding textarea and send button. The textarea grows from 1 line up to a maximum of 160px, then scrolls internally. On mobile, the input area lifts with the software keyboard (use `env(safe-area-inset-bottom)` for notched devices).

**Responsive breakpoints:**
- `< 640px` (mobile): chat gutter reduces to 12px; bubble max-width extends to 85%; nav wordmark truncates to icon + short name.
- `640px–1024px` (tablet): layout unchanged from base spec.
- `> 1024px` (desktop): chat thread content is centered at max-width 760px with auto horizontal margins, preserving a readable column width on wide monitors.

---

## Elevation & Depth

Elevation is used minimally. The interface is fundamentally flat with one level of shadow used to disambiguate interactive chrome from content.

| Layer | Token | Usage |
|---|---|---|
| Flat | `none` | Student bubble, error bubble, all surfaces |
| Subtle | `0 1px 3px rgba(0,0,0,0.08)` | Nav bar, AI bubble, typing indicator, input area on scroll |
| Card | `0 2px 8px rgba(0,0,0,0.10)` | Reserved — future modals or drawers |
| Overlay | `0 8px 24px rgba(0,0,0,0.14)` | Reserved — future modals |

The AI bubble receives `elevation.subtle` to give it a slight lift over the muted chat background, reinforcing its role as a delivered artifact distinct from the student's own text. The student bubble is flat (no shadow) because it is the user's own input — it recedes.

The input area gains `elevation.subtle` only when the chat thread has scrolled past a threshold (e.g., 48px) to indicate content continues beneath the sticky bar.

---

## Shapes

**Border radius** follows a three-step scale:

| Size | Value | Usage |
|---|---|---|
| `sm` | 4px | Retry button (inline text action — minimal rounding) |
| `md` | 8px | Send button, textarea, input area container |
| `lg` | 12px | Chat bubbles (all variants), typing indicator container |
| `full` | 9999px | Persona chip, typing indicator dots |

Chat bubbles use uniform `border-radius: 12px` on all four corners. There is no "bubble tail" or asymmetric corner treatment in MVP 1 — the alignment (left vs. right) plus background color provides sufficient sender distinction without complexity.

---

## Components

### Nav Bar

The nav bar spans the full viewport width at 56px height on a navy (`#1B2A4A`) background. Content is laid out as a flex row with `align-items: center; justify-content: space-between; padding: 0 24px`.

Left side: university wordmark in Playfair Display, 22px, bold, white. This is the only surface in the product where Playfair Display is used at full weight — it signals identity without taking visual attention away from the conversation.

Right side: persona chip (see below).

The nav bar carries `elevation.subtle` shadow so it reads above the chat thread during scroll without a visible border line that would create a harsh cut.

### Persona Chip

A pill-shaped badge in the nav bar right slot identifying the active AI persona (e.g., "Academic Advisor"). White outline (`1.5px solid #FFFFFF`), transparent fill, white text, 14px Inter semibold, padding `4px 12px`, `border-radius: full`. An optional 16px icon (e.g., a graduation cap outline) may appear left of the label with a 6px gap.

The chip is read-only in MVP 1 — no persona switching. It communicates context to the student ("you are talking to Academic Advisor mode") without requiring interaction.

### Chat Bubble — Student

Right-aligned bubble with `surface-muted` (`#F8F9FB`) background, no border, `border-radius: lg` (12px), `padding: 10px 16px`. Text is `text-default` (`#1F2937`), Inter 16px, line-height 1.5. Max-width 72% of the chat column. Floated to the right with `margin-left: auto`.

Timestamp (optional) appears below the bubble, right-aligned, Inter 12px, `text-muted`, `margin-top: 2px`.

### Chat Bubble — AI

Left-aligned bubble with `surface-base` (`#FFFFFF`) background, `1px solid border` (`#E5E7EB`) border, `elevation.subtle` shadow, `border-radius: lg`, `padding: 10px 16px`. Text is `text-default`, Inter 16px, line-height 1.7 (slightly more relaxed than student bubble to support longer structured responses). Max-width 72%.

The AI bubble supports markdown rendering: `**bold**`, `*italic*`, numbered and bulleted lists, `inline code`, and fenced code blocks. Code blocks receive a `surface-muted` inner background with a `border: 1px solid border` and `border-radius: md`.

Timestamp appears below the bubble, left-aligned, Inter 12px, `text-muted`.

### Chat Bubble — Error

Left-aligned bubble replacing the expected AI response when a request fails. Background `#FEF2F2` (`error-surface`), border `1px solid #A63232` (crimson), `border-radius: lg`, `padding: 10px 16px`. Text is `text-default`, Inter 16px.

Content structure:
1. Short error message text (e.g., "Something went wrong. Please try again.")
2. Below the message text: a "Retry" inline text link — Inter 14px semibold, crimson color (`#A63232`), no background, `text-decoration: underline`, `margin-top: 4px`.

No icon is used inside the error bubble — the crimson border already signals the state. Adding a warning icon would introduce redundancy and visual noise.

### Typing Indicator

Appears in the AI bubble position (left-aligned) while a response is being generated. The container matches `chat-bubble-ai` styling exactly: white background, `1px solid border`, `elevation.subtle`, `border-radius: lg`, `padding: 10px 14px`.

Inside: three dots, 8px diameter each, `border-radius: full`, color `text-muted` (`#6B7280`), spaced 4px apart. Animation: each dot fades between opacity 0.3 and 1.0 with a 0.6s ease-in-out cycle, staggered by 0.2s per dot, looping infinitely. The animation communicates "the AI is thinking" without text, which is appropriate for all language localizations.

The typing indicator is dismissed and replaced by the actual AI bubble as soon as the first token is received (or with a streaming approach, it morphs into the AI bubble in place).

### Send Button

Square button, 44×44px, `border-radius: md` (8px), crimson fill (`#A63232`), white send-arrow icon (20px, filled). Positioned at the right end of the input area, vertically centered with the textarea.

States:
- **Default:** crimson fill, white icon.
- **Hover:** `#8C2929` (crimson darkened ~10%), `transition: background 150ms ease`.
- **Active / pressed:** `#732222`.
- **Disabled** (empty textarea): `border` color (`#E5E7EB`) fill, `text-muted` icon. No border. Cursor `not-allowed`.

The send button is the only crimson fill element in the chat surface. This exclusivity is intentional — the eye finds it instantly as the primary action.

### Input Area

Sticky bar at the bottom of the viewport, white background, `1px solid border` top border, `padding: 8px 16px`. On scroll, gains `elevation.subtle` upward shadow.

Contains:
- **Textarea** (flex 1): `surface-muted` background, `1px solid border` border, `border-radius: md`, `padding: 10px 14px`, Inter 16px, `color: text-default`, placeholder `text-muted`. On focus: border color transitions to `primary` (`#1B2A4A`) with a 2px offset focus ring at 25% navy opacity (`rgba(27,42,74,0.25)`). Auto-expands vertically from 1 line (36px inner height) up to 160px; beyond that the textarea scrolls internally.
- **Send Button** (right, flex-shrink 0): see above.

On mobile, the input area bottom padding accounts for the safe area inset: `padding-bottom: max(8px, env(safe-area-inset-bottom))`.

---

## Do's and Don'ts

| # | Do | Don't |
|---|---|---|
| 1 | **White-label discipline:** Keep all university-specific identity in the nav wordmark and persona chip only. The chat body, bubbles, and input area must be identity-neutral so any university can adopt the shell without visual conflict. | Don't embed institution logos, mascots, or brand colors inside chat bubbles, the input area, or error states. Those areas belong to the content layer, not the identity layer. |
| 2 | **Color usage — crimson exclusivity:** Use crimson (`#A63232`) for one primary action at a time — the send button — and for error state borders and retry links. This single-accent discipline ensures crimson always reads as "action required" or "do this now." | Don't use crimson for decorative purposes, hover highlight backgrounds, or to color AI-generated text. Multiple simultaneous crimson elements destroy the signal value of the accent. |
| 3 | **Serif type usage — structural only:** Playfair Display is permitted in the nav wordmark and empty-state headings only. In those positions it signals institutional character without competing with legibility. | Don't use Playfair Display for chat bubble text, timestamps, buttons, labels, or error messages. Mixing serif into functional UI copy creates visual noise and slows reading in a fast-turn conversational context. |
| 4 | **Error state treatment — inline, contained:** Render errors as a chat bubble in the AI position using `error-surface` background and crimson border. Keep the retry action as an inline text link within the bubble. This treats errors as part of the conversation flow, not as system interruptions. | Don't use toast notifications, modal dialogs, or full-screen error screens for recoverable chat errors. Interrupting the conversation thread breaks the advisor-student interaction model and feels disproportionate to the failure. |
| 5 | **Chat bubble visual hierarchy — contrast by surface, not by color:** Distinguish student vs. AI bubbles using surface color (`surface-muted` vs. `surface-base` with border + shadow) and alignment (right vs. left). This creates an immediately readable thread without requiring avatars or labels. | Don't introduce additional background colors (e.g., a tinted AI bubble) or colored text inside bubbles to differentiate sides. Additional colors fragment the palette and undermine the white-label neutrality of the chat surface. |
| 6 | **Elevation — one level in the chat surface:** Apply `elevation.subtle` only to the AI bubble, typing indicator, and sticky chrome (nav bar, input area on scroll). Student bubbles and error bubbles remain flat. | Don't add drop shadows to student bubbles or increase shadow intensity on AI bubbles. Over-elevation makes the chat surface feel cluttered and reduces the signal value of shadows used on chrome. |
| 7 | **Responsive width — preserve the column:** On desktop, constrain the chat thread to `max-width: 760px` centered. Bubbles then respect the 72% max-width rule within that column, keeping line lengths readable. | Don't allow chat bubbles to span the full width of large monitor viewports. Full-width bubbles on a 1400px screen produce unreadable line lengths and destroy the conversational rhythm. |
