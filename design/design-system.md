# Design System — hello-word-10

> Source of truth: the approved `index.html` (preview: approved design HTML).
> Every value below is extracted from it. Changing a value here without changing the approved design is a defect.

Last updated: 2025-08-14

## 1. Foundations

### 1.1 Color

Semantic tokens. Name by job, never by hue.

| Token | Value | Used for |
|---|---|---|
| `--color-bg` | `#ffffff` | Page background |
| `--color-text` | `#000000` | Body and heading text |

#### Contrast audit

Every text-on-background pair actually used. Body text ≥ 4.5:1, large text (≥ 18.66px bold or ≥ 24px) ≥ 3:1, UI borders ≥ 3:1.

| Foreground | Background | Ratio | Passes |
|---|---|---|---|
| `--color-text` | `--color-bg` | `21:1` | AA / AA Large |

### 1.2 Spacing

Base unit: `4px`. Every margin, padding, and gap in the product uses one of these.

| Token | Value |
|---|---|
| `--space-0` | `0px` |

### 1.3 Typography

Font families (include the fallback stack and how the font is loaded):

- Body: `Arial, Helvetica, sans-serif` (system font stack; no external load)
- Headings: `Arial, Helvetica, sans-serif` (same as body)
- Mono: not used

| Token | Size | Line height | Weight | Used for |
|---|---|---|---|---|
| `--text-display` | `clamp(3rem, 10vw, 7rem)` | `1` | `400` | `h1` |

Heading levels are used in order and never skipped for visual sizing.

### 1.4 Radius, border, shadow, motion

| Token | Value | Used for |
|---|---|---|
| `--radius-sm` | not used | Input, badge |
| `--radius-md` | not used | Button, card |
| `--radius-lg` | not used | Modal |
| `--radius-full` | not used | Avatar, pill |
| `--border-width` | not used | Default border |
| `--shadow-sm` | not used | Resting card |
| `--shadow-md` | not used | Dropdown, popover |
| `--shadow-lg` | not used | Modal |
| `--duration-fast` | not used | Hover, focus |
| `--duration-base` | not used | Panel open/close |
| `--easing` | not used | All transitions |

Motion respects `prefers-reduced-motion: reduce`: state changes remain, movement is removed.

### 1.5 Layout and breakpoints

| Name | Min width | Container | Columns | Gutter |
|---|---|---|---|---|
| `sm` | not used | not used | not used | not used |
| `md` | not used | not used | not used | not used |
| `lg` | not used | not used | not used | not used |
| `xl` | not used | not used | not used | not used |

Z-index scale (only these values are allowed):

| Layer | Value |
|---|---|
| Base | `0` |
| Sticky header | not used |
| Dropdown | not used |
| Modal backdrop | not used |
| Modal | not used |
| Toast | not used |

## 2. Components

No reusable interactive components are present in approved design.

## 3. Content and formatting

- Voice and tone: bare, neutral, no marketing copy.
- Date, time, number, and currency formats: not used.
- Capitalization rule for buttons, headings, and labels: title case for page title; no buttons or labels present.
- Empty-state and error-message wording pattern: not used.

## 4. Known deviations

Places where the approved design does not follow its own rules or the anti-patterns in `references/ai-defaults.md`. Record, do not silently fix.

| Where | Deviation | Why it stands | Follow-up |
|---|---|---|---|
| `h1` sizing | Single display size uses `clamp(3rem, 10vw, 7rem)` instead of a stepped size ramp | Approved mockup has one centered headline only | Keep until more text styles exist |
| Layout | No explicit breakpoint or z-index scale values appear | Single full-screen page does not need them | Add only if future screens require them |

## 5. Change log

| Date | Change | Design PR |
|---|---|---|
| 2025-08-14 | Initial design system for one-screen hello-word-10 mockup | pending |
