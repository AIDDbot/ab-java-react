---
name: Health Status Design System
colors:
  background: "#fbfcfd"
  surface: "#ffffff"
  surface-container: "#f4f6f8"
  surface-container-high: "#eceff2"
  on-surface: "#0f1115"
  on-surface-variant: "#5a626e"
  outline: "#d6dbe1"
  outline-variant: "#e7eaee"
  primary: "#2f6df6"
  on-primary: "#ffffff"
  primary-container: "#e2ecff"
  on-primary-container: "#0a307e"
  success: "#1f9d55"
  on-success: "#ffffff"
  success-container: "#e3f6ec"
  on-success-container: "#0c4a2b"
  danger: "#e02424"
  on-danger: "#ffffff"
  danger-container: "#fde4e4"
  on-danger-container: "#7a0f0f"
  focus-ring: "#2f6df6"
  dark-background: "#0d0f13"
  dark-surface: "#15181e"
  dark-surface-container: "#1c2027"
  dark-on-surface: "#e6e9ee"
  dark-on-surface-variant: "#9aa3b1"
  dark-outline: "#2c323b"
typography:
  display:
    fontFamily: Inter
    fontSize: 32px
    fontWeight: "700"
    lineHeight: 40px
    letterSpacing: -0.02em
  title:
    fontFamily: Inter
    fontSize: 20px
    fontWeight: "600"
    lineHeight: 28px
    letterSpacing: -0.01em
  body:
    fontFamily: Inter
    fontSize: 15px
    fontWeight: "400"
    lineHeight: 24px
    letterSpacing: 0em
  label:
    fontFamily: Inter
    fontSize: 13px
    fontWeight: "500"
    lineHeight: 16px
    letterSpacing: 0.04em
  mono:
    fontFamily: "JetBrains Mono"
    fontSize: 13px
    fontWeight: "400"
    lineHeight: 20px
    letterSpacing: 0em
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.625rem
  lg: 0.875rem
  full: 9999px
spacing:
  unit: 4px
  container-max: 640px
  gutter: 16px
  margin-mobile: 16px
  margin-desktop: 32px
elevation:
  level-0: none
  level-1: "0 1px 2px rgba(15, 17, 21, 0.06)"
  level-2: "0 4px 16px rgba(15, 17, 21, 0.08)"
components:
  page-shell:
    backgroundColor: "{colors.background}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body}"
    padding: "{spacing.margin-desktop}"
  status-card:
    backgroundColor: "{colors.surface}"
    rounded: "{rounded.lg}"
    elevation: "{elevation.level-2}"
    padding: 24px
  status-badge-up:
    backgroundColor: "{colors.success-container}"
    textColor: "{colors.on-success-container}"
    typography: "{typography.label}"
    rounded: "{rounded.full}"
    padding: 6px
  status-badge-down:
    backgroundColor: "{colors.danger-container}"
    textColor: "{colors.on-danger-container}"
    typography: "{typography.label}"
    rounded: "{rounded.full}"
    padding: 6px
  status-dot-up:
    backgroundColor: "{colors.success}"
  status-dot-down:
    backgroundColor: "{colors.danger}"
  detail-row:
    textColor: "{colors.on-surface-variant}"
    typography: "{typography.body}"
    padding: 12px
  detail-value:
    textColor: "{colors.on-surface}"
    typography: "{typography.mono}"
  refresh-button:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    rounded: "{rounded.DEFAULT}"
    padding: 10px
    height: 40px
  refresh-button-hover:
    backgroundColor: "{colors.on-primary-container}"
---

## Brand & Style

The Health Status surface speaks the language of modern developer status pages: calm, precise, and instantly legible. The audience is developers and operators who glance at this view to answer one question — "is the system healthy?" The aesthetic is **"Operational Clarity"**: a quiet neutral canvas where a single status card carries the signal, and color is reserved almost exclusively for health semantics. Nothing competes with the status itself.

The mood is trustworthy and unhurried — flat surfaces, soft shadows, and ample whitespace rather than gradients or ornamentation. The interface should feel like instrumentation, not marketing.

## Colors

The palette is a restrained neutral system with three semantic accents.

- **Primary (Blue):** Reserved for the single interactive action (Refresh) and focus rings. It signals "you can act here," never status.
- **Success (Green):** The `UP` state. Used for the status dot, badge, and any healthy indicator.
- **Danger (Red):** The `DOWN` state. Used for unhealthy badges, dots, and error messaging.
- **Neutrals (Slate/White):** Background, surfaces, text, and borders. They carry structure and let the semantic color dominate.

Status color must never be the only signal — pair it with text (`UP`/`DOWN`) and an icon/dot for accessibility. A dark theme is defined via the `dark-*` tokens and follows the same role mapping.

## Typography

A single typeface, **Inter**, covers display, title, body, and label roles for a cohesive, neutral UI voice. A monospace, **JetBrains Mono**, is used exclusively for machine values — timestamps (`checked_at`) and raw status payloads — to reinforce their technical, copy-paste nature and to align columns. Display text uses tight tracking to feel deliberate; labels use slightly loose tracking and are the only uppercased text.

## Layout & Spacing

The layout is single-column and centered, capped at `container-max` (640px) — this is a focused, glanceable surface, not a dense dashboard. Spacing uses a 4px base unit. The status card is the hero, vertically centered with generous margins (`32px` desktop, `16px` mobile). Below 640px the card goes full-width within mobile margins. Internal grouping is tight (12–16px between rows) while the card sits in wide negative space to draw the eye.

## Elevation & Depth

Depth is subtle and shadow-based (no glassmorphism). The page is `level-0`; the status card rests at `level-2` to lift it off the canvas as the single focal object. Interactive elements (Refresh button) use `level-1` at rest. Avoid heavy shadows — the goal is a gentle hierarchy, not drama.

## Shapes

Soft, modern rectangles. Cards use `rounded-lg`, buttons and inputs use the `DEFAULT` (8px) radius, and status badges/dots are fully `rounded-full` (pill and circle) to read as live indicators. No sharp corners.

## Components

### Status Card

The focal container (`status-card`). Holds a title ("System Health"), a prominent status badge with a colored dot, and a list of detail rows. Renders three explicit states:
- **Loading:** skeleton/placeholder rows, neutral, no semantic color, `aria-busy="true"`.
- **Healthy:** `status-badge-up` + `status-dot-up`, text `UP`.
- **Unhealthy/Error:** `status-badge-down` + `status-dot-down`, text `DOWN`, with `role="alert"` on the message.

### Status Badge & Dot

Pill badge plus a 8px circular dot. Color comes from `success`/`danger` containers; always accompanied by the literal `UP`/`DOWN` label so color is never the sole carrier of meaning. Contrast meets WCAG AA (container/on-container pairs are tuned for ≥4.5:1 on text).

### Detail Rows

Key/value rows for `database` status and `checked_at`. Keys use `body` in `on-surface-variant`; values use `mono` in `on-surface`. Timestamps render in the user's locale with the raw ISO value available via `title`/tooltip.

### Refresh Button

The only primary action. Default uses `primary`; **hover** deepens to `on-primary-container`; **focus** shows a 2px `focus-ring` offset outline (keyboard-visible); **active** nudges down 1px; **disabled/loading** drops to 50% opacity with a spinner and `aria-disabled`. Minimum hit target 40px height.

## Accessibility & Responsive

- **Contrast:** all text meets WCAG AA; semantic states pair color with text + shape.
- **Keyboard:** Refresh is focusable with a visible `focus-ring`; logical tab order.
- **Screen readers:** live status changes announced via `aria-live="polite"`; error state via `role="alert"`; loading via `aria-busy`.
- **Motion:** spinner/transitions respect `prefers-reduced-motion`.
- **Theme:** honors `prefers-color-scheme` using the `dark-*` token set.
- **Breakpoints:** single column throughout; card caps at 640px on desktop, full-width within 16px margins below 640px.
