# Design System: Metical

## 1. Visual Theme & Atmosphere

Metical is a practical, trust-first currency converter designed for Mozambican users who check exchange rates daily - traders, freelancers, importers, students abroad, and families handling remittances. The design philosophy borrows from M-Pesa's clean mobile-first aesthetic and the formal authority of the Banco de Moçambique, blending them into an interface that feels both **official** (you trust the rates) and **effortless** (you convert in two taps).

The foundation is a warm off-white canvas (`#fafaf7`) - chosen over pure white to evoke paper, documentation, printed receipts. The singular brand accent is **Savanna Gold** (`#d4a017`), a warm ochre that references the yellow band of the Mozambican flag and the weathered tone of metical banknotes. A secondary **Dusk Teal** (`#0f766e`) signals positive change (appreciation) and active states, while **Earth Red** (`#c13515`) marks depreciation and errors.

Typography uses **Inter** for UI and **JetBrains Mono** for every numeric value. This is deliberate: mono signals data integrity, keeps digits aligned for scanning, and mirrors how financial institutions display numbers. Headings use tight negative letter-spacing (-0.02em) for a confident, composed voice.

Cards sit on the canvas with a two-layer shadow stack (subtle ring + soft ambient blur), 16px radius, and generous internal padding. The layout breathes - you're meant to consult rates calmly, not scramble through a cluttered dashboard.

**Key Characteristics:**

- Warm off-white canvas (`#fafaf7`) - paper-like, not clinical
- Savanna Gold (`#d4a017`) as singular brand accent
- Dusk Teal (`#0f766e`) positive; Earth Red (`#c13515`) negative
- Inter for UI; JetBrains Mono for **all** numeric values without exception
- Two-layer card shadows: inset ring + soft ambient blur
- 16px card radius, 8px button radius, 999px pill radius
- Circular currency flag badges (32px) on every rate row
- Tight heading tracking (-0.02em)
- Near-black text (`#1c1917`) - warm stone, never pure black

## 2. Color Palette & Roles

### Primary Brand

- **Savanna Gold** (`#d4a017`): `--brand`, primary CTA, brand accent, active indicators
- **Gold Deep** (`#a67c00`): `--brand-dark`, pressed state
- **Gold Light** (`#fef3c7`): `--brand-light`, hover tint, selected row background
- **Gold Muted** (`#fdebb0`): `--brand-muted`, sparkline fills

### Semantic - Rate Movement

- **Dusk Teal** (`#0f766e`): `--positive`, MT appreciation, active states
- **Teal Light** (`#ccfbf1`): `--positive-bg`, positive badge background
- **Earth Red** (`#c13515`): `--negative`, depreciation, errors
- **Red Light** (`#fee4d6`): `--negative-bg`, negative badge background
- **Neutral Stone** (`#78716c`): `--neutral`, unchanged, inactive

### Text Scale

- **Stone Near-Black** (`#1c1917`): `--text`, primary text
- **Stone 700** (`#44403c`): `--text-strong`, emphasized body
- **Stone 500** (`#78716c`): `--text-secondary`, labels, descriptions
- **Stone 400** (`#a8a29e`): `--text-muted`, timestamps, metadata
- **Stone 300** (`#d6d3d1`): `--text-disabled`

### Surface & Borders

- **Paper** (`#fafaf7`): `--bg-page`, page background
- **Card** (`#ffffff`): `--bg-card`, card surfaces
- **Subtle** (`#f5f5f4`): `--bg-subtle`, input fields, secondary surfaces
- **Hover** (`#f5f5f4`): `--bg-hover`, row hover
- **Border** (`#e7e5e4`): `--border`, card borders, dividers
- **Border Strong** (`#d6d3d1`): `--border-strong`, input borders
- **Border Focus** (`#d4a017`): `--border-focus`, focused input

### Shadows

- **Card Shadow**: `0 0 0 1px rgba(28, 25, 23, 0.04), 0 2px 8px rgba(28, 25, 23, 0.06)`
- **Hover Shadow**: `0 0 0 1px rgba(28, 25, 23, 0.06), 0 4px 16px rgba(28, 25, 23, 0.08)`
- **Focus Ring**: `0 0 0 3px rgba(212, 160, 23, 0.2)`

## 3. Typography Rules

### Font Family

- **UI**: `Inter, -apple-system, system-ui, Roboto, Helvetica Neue, sans-serif`
- **Numeric**: `JetBrains Mono, Menlo, Consolas, monospace`
- **Weights used**: 400, 500, 600, 700

### Hierarchy

| Role | Font | Size | Weight | Line Height | Letter Spacing |
|------|------|------|--------|-------------|----------------|
| Page Title | Inter | 28px | 700 | 1.2 | -0.02em |
| Section Heading | Inter | 20px | 600 | 1.3 | -0.01em |
| Card Title | Inter | 16px | 600 | 1.4 | normal |
| Body | Inter | 15px | 400 | 1.5 | normal |
| Small / Caption | Inter | 13px | 400 | 1.4 | normal |
| Label | Inter | 12px | 500 | 1.4 | 0.02em (uppercase) |
| Big Number (hero) | JetBrains Mono | 40px | 600 | 1.1 | -0.01em |
| Rate Number | JetBrains Mono | 18px | 500 | 1.2 | normal |
| Small Rate | JetBrains Mono | 14px | 500 | 1.3 | normal |
| Code / Metadata | JetBrains Mono | 12px | 400 | 1.3 | normal |

### Principles

- **Every number is mono.** Rates, amounts, percentages, timestamps, dates. No exceptions. This is the single most important typographic rule in this design.
- **Mono for confidence:** when a user sees `64.54 MZN`, the fixed-width digits signal "this is precise data, verified."
- **Weight 500 is the floor for numbers.** Never 400 for a rate - it looks flimsy.
- **Negative tracking on headings** (-0.02em on H1, -0.01em on H2) tightens the voice.

## 4. Component Stylings

### Buttons

**Primary (Gold)**
- Background: `#d4a017`
- Text: `#1c1917` (dark on gold for readability)
- Padding: `10px 20px`
- Radius: 8px
- Font: Inter 15px weight 600
- Hover: background `#a67c00`, shadow hover lift
- Active: scale(0.98)
- Disabled: background `#d6d3d1`, text `#a8a29e`

**Secondary (Ghost)**
- Background: transparent
- Text: `#1c1917`
- Border: 1px solid `#d6d3d1`
- Padding: `10px 20px`
- Radius: 8px
- Hover: background `#f5f5f4`

**Icon Button**
- Size: 36x36
- Background: transparent
- Radius: 8px
- Hover: background `#f5f5f4`

### Conversion Input (the hero component)

This is the centerpiece of the home screen. Two side-by-side panels:

- Background: `#ffffff`
- Border: 1px solid `#e7e5e4`
- Radius: 16px
- Padding: 20px
- Each panel has: currency selector (pill with flag + code), amount input (big mono number)
- Amount input: JetBrains Mono 40px weight 600, no border, background transparent
- Currency pill: 32px flag circle + 3-letter code + chevron
- Swap button centered between panels: circular 40x40, gold background, white swap icon

### Rate Row (table)

- Height: 64px
- Padding: `12px 16px`
- Border-bottom: 1px solid `#e7e5e4`
- Hover: background `#f5f5f4`
- Layout: [Flag 32px] [Code + Name stack] [Rate mono right-aligned] [Change badge]

### Change Badge

- Positive: background `#ccfbf1`, text `#0f766e`, arrow up icon
- Negative: background `#fee4d6`, text `#c13515`, arrow down icon
- Neutral: background `#f5f5f4`, text `#78716c`, dash icon
- Padding: `2px 8px`
- Radius: 999px (pill)
- Font: JetBrains Mono 12px weight 500

### Cards

- Background: `#ffffff`
- Border: 1px solid `#e7e5e4`
- Radius: 16px
- Padding: 20px
- Shadow: card shadow (see section 2)

### Sparkline / Chart

- Line: `#d4a017`, 2px width
- Fill: `#fdebb0` gradient to transparent
- Grid: `#e7e5e4`, 1px
- Axis labels: Inter 11px weight 400, color `#78716c`
- Data point labels: JetBrains Mono 11px

### Flag Badges

- Circular, 32px diameter
- Implemented as emoji or SVG flags, centered
- Fallback: gray circle with 3-letter code in mono

## 5. Layout Principles

### Spacing System

Base unit: 4px. Scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64.

### Grid & Container

- Desktop container max-width: **960px** (not full-width - the app is a focused tool, not a dashboard)
- Centered with 24px horizontal padding
- Vertical rhythm: 24px between major sections, 16px between cards in a group

### Header

- Sticky top, 64px tall
- White background, bottom border 1px `#e7e5e4`
- Contents: logo left, nav center (Home / Rates / History), settings icon right

### Whitespace Philosophy

Generous. The app is consulted, not worked-in. Users want to **glance** at a rate, convert, and leave. Padding around the hero conversion widget is especially generous (40px+ vertical).

### Border Radius Scale

- 4px - subtle (small inputs, chips)
- 8px - buttons, form inputs
- 16px - cards, conversion widget
- 999px - pills, badges, currency selectors
- 50% - circular elements (flags, icon buttons)

## 6. Depth & Elevation

| Level | Treatment | Use |
|-------|-----------|-----|
| Flat | No shadow | Page background, body text |
| Card | `0 0 0 1px rgba(28,25,23,0.04), 0 2px 8px rgba(28,25,23,0.06)` | All cards |
| Hover | `0 0 0 1px rgba(28,25,23,0.06), 0 4px 16px rgba(28,25,23,0.08)` | Card hover, button hover |
| Focus | `0 0 0 3px rgba(212,160,23,0.2)` | Focused input, selected item |

Shadows are **warm and low** - they suggest paper on a table, not glass on glass. Never use pure black in shadow color; always `rgba(28, 25, 23, ...)` (the warm stone).

## 7. Do's and Don'ts

### Do
- Use JetBrains Mono for **every** number - rates, amounts, percentages, timestamps
- Use `#1c1917` (warm stone) for text, never `#000000`
- Apply Savanna Gold (`#d4a017`) only for primary CTA and brand accents
- Use the two-layer card shadow on every elevated surface
- Use 16px radius on cards and 999px on badges/pills
- Give the hero conversion widget generous whitespace - it is the product
- Use teal for positive, red for negative - consistently, across the whole app
- Round rates to 2 decimal places for display; 4 decimals only in detail view

### Don't
- Don't use proportional fonts for numbers - ever
- Don't use pure black (`#000000`) anywhere
- Don't introduce a fourth brand color - gold/teal/red is the full palette
- Don't use heavy shadows (>0.1 opacity) - shadows are warm and subtle
- Don't use sharp corners (0-4px) on cards
- Don't put flags in anything other than circular badges
- Don't use Inter for rates even when space is tight - find room for mono

## 8. Responsive Behavior

### Breakpoints

| Name | Width | Key Changes |
|------|-------|-------------|
| Mobile | <640px | Single column, stacked conversion panels, bottom nav |
| Tablet | 640-960px | Conversion panels side-by-side, top nav |
| Desktop | >960px | Full layout at max-width 960px, centered |

### Collapsing Strategy

- Conversion widget: side-by-side → stacked with swap button between rows
- Rates table: full table → compact list (hide less-used columns)
- Header nav: horizontal → bottom tab bar on mobile
- Detail page chart: full-width → maintains aspect ratio

### Touch Targets

- Minimum 44x44px for all interactive elements
- Currency selector pills: 40px tall minimum
- Swap button: 48x48 on mobile (up from 40 on desktop)

## 9. Agent Prompt Guide

### Quick Color Reference

- Background: `#fafaf7` (warm paper)
- Card: `#ffffff`
- Text: `#1c1917` (warm stone)
- Secondary text: `#78716c`
- Brand: `#d4a017` (Savanna Gold)
- Positive: `#0f766e` (Dusk Teal)
- Negative: `#c13515` (Earth Red)
- Border: `#e7e5e4`

### Fonts

- UI: Inter (400, 500, 600, 700)
- Numbers: JetBrains Mono (400, 500, 600)

### Example Component Prompts

- "Create a conversion panel: white background, 16px radius, 20px padding, 1px border #e7e5e4. Top-left: currency pill (32px flag circle + 3-letter code in Inter 15 weight 600 + chevron icon). Below: amount input, JetBrains Mono 40px weight 600, no border, #1c1917 text."

- "Build a rate row: 64px tall, padding 12px 16px. Left: 32px circular flag. Middle: currency code in Inter 15 weight 600, currency name in Inter 13 weight 400 #78716c. Right: rate in JetBrains Mono 18 weight 500, below a change badge."

- "Design a change badge: pill (999px radius), padding 2px 8px, JetBrains Mono 12 weight 500. If positive: bg #ccfbf1, text #0f766e, arrow-up icon. If negative: bg #fee4d6, text #c13515, arrow-down icon."

- "Create a CTA button: bg #d4a017, text #1c1917, Inter 15 weight 600, padding 10px 20px, radius 8px. Hover: bg #a67c00."

### Iteration Guide

1. Start with paper canvas (#fafaf7)
2. Every number is JetBrains Mono, no exceptions
3. Gold is the only brand color; teal and red are semantic only
4. Generous whitespace around the conversion widget
5. Cards have 16px radius and the two-layer warm shadow
6. Flags are circular 32px badges, always
