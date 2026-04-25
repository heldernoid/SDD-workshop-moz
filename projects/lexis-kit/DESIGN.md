# Design System: Lexis

---

## 1. Visual Theme & Atmosphere

Lexis is a study companion for students preparing for TOEFL, IELTS, and university entrance exams. The design should feel like a well-made study card — calm, focused, and encouraging. It does not feel like a game (no confetti explosions, no streaks on fire). It feels like a serious tool that respects the user's time.

The visual foundation is a cool off-white (`#f8fafc`) — cleaner and more focused than warm tones. The accent is **Ink Blue** (`#1e40af`) — authoritative, academic, trustworthy. Correct answers use **Leaf Green** (`#15803d`); incorrect answers use **Warm Red** (`#b91c1c`). Near-miss answers use **Amber** (`#b45309`) — you were close, try again.

Typography uses **Inter** for all UI text and headings. The word being studied is displayed in **Lora** (a serif) — evoking a dictionary or textbook, making the word feel significant and worth memorising. Numbers (scores, counts, days) use **JetBrains Mono**.

Cards use a clean white surface with a single-shadow elevation system. No gradients. No decorative illustration. The word is the hero.

**Key Characteristics:**

- Cool off-white canvas (`#f8fafc`) — focused, not clinical
- Ink Blue (`#1e40af`) as brand accent — academic authority
- Leaf Green / Warm Red / Amber for answer feedback
- Inter for UI; **Lora** for the study word itself; JetBrains Mono for numbers
- Single-layer card shadow; 12px card radius
- Level badges (B1 / B2 / C1) as colored pills
- No animations except the card flip (200ms ease-out) and subtle feedback flash

---

## 2. Color Palette & Roles

### Primary Brand
- **Ink Blue** (`#1e40af`): `--brand`, primary CTA, active nav, focus ring
- **Blue Deep** (`#1e3a8a`): `--brand-dark`, pressed state
- **Blue Light** (`#dbeafe`): `--brand-light`, hover tint, selected row

### Feedback — Answer States
- **Leaf Green** (`#15803d`): `--correct`, correct answer text and border
- **Green Light** (`#dcfce7`): `--correct-bg`, correct answer background
- **Warm Red** (`#b91c1c`): `--incorrect`, wrong answer
- **Red Light** (`#fee2e2`): `--incorrect-bg`, wrong answer background
- **Amber** (`#b45309`): `--near`, near-miss answer
- **Amber Light** (`#fef3c7`): `--near-bg`, near-miss background

### Level Colors
- **B1** — `--level-b1`: `#0369a1` (sky blue), bg `#e0f2fe`
- **B2** — `--level-b2`: `#7c3aed` (violet), bg `#ede9fe`
- **C1** — `--level-c1`: `#9a3412` (orange-rust), bg `#ffedd5`

### Text Scale
- `--text`: `#0f172a` (slate-900, primary)
- `--text-strong`: `#1e293b` (slate-800)
- `--text-secondary`: `#475569` (slate-600)
- `--text-muted`: `#94a3b8` (slate-400)
- `--text-disabled`: `#cbd5e1` (slate-300)

### Surface & Borders
- `--bg-page`: `#f8fafc`
- `--bg-card`: `#ffffff`
- `--bg-subtle`: `#f1f5f9`
- `--bg-hover`: `#f1f5f9`
- `--border`: `#e2e8f0`
- `--border-strong`: `#cbd5e1`
- `--border-focus`: `#1e40af`

### Shadows
- **Card shadow**: `0 1px 3px rgba(15, 23, 42, 0.08), 0 1px 2px rgba(15, 23, 42, 0.04)`
- **Card hover**: `0 4px 12px rgba(15, 23, 42, 0.10)`
- **Focus ring**: `0 0 0 3px rgba(30, 64, 175, 0.2)`

---

## 3. Typography Rules

### Font Families
- **UI**: `Inter, -apple-system, system-ui, sans-serif`
- **Study word**: `Lora, Georgia, "Times New Roman", serif`
- **Numbers/stats**: `JetBrains Mono, Menlo, Consolas, monospace`

Load from Google Fonts: `Inter` (400, 500, 600, 700), `Lora` (600, 700), `JetBrains Mono` (500).

### Hierarchy

| Role | Font | Size | Weight | Notes |
|------|------|------|--------|-------|
| Study word (card hero) | Lora | 36px | 700 | Letter-spacing -0.01em |
| Card definition | Inter | 16px | 400 | line-height 1.6 |
| Page title | Inter | 24px | 700 | Letter-spacing -0.02em |
| Section heading | Inter | 18px | 600 | |
| Body | Inter | 15px | 400 | |
| Small / caption | Inter | 13px | 400 | |
| Label (uppercase) | Inter | 11px | 600 | Letter-spacing 0.06em |
| Stats number | JetBrains Mono | 32px | 600 | |
| Score / count | JetBrains Mono | 20px | 500 | |

### Principles
- The word under study is always in Lora — it signals "this is the thing you are learning."
- Every score, count, streak number, and percentage uses JetBrains Mono.
- Never render a raw number with `.toString()` in components — use `lib/format.ts`.

---

## 4. Component Patterns

### FlashCard
- White card, 12px radius, card shadow, max-width 640px, centered
- Front side: word in Lora 36/700, `pos` in small Inter uppercase label, `level` badge top-right, `domain` badge below pos
- Back side: `definition_en` in Inter 16/400, `example_en` in Inter 15/400 italic with left blue border, translation input
- Flip animation: CSS `rotateY(180deg)` on `.card-inner`, `transform-style: preserve-3d`, 200ms ease-out
- If `showDefinitionFirst = true`, definition is visible before the user answers

### TranslationInput
- Full-width input, 48px min-height, border `--border-strong`, 8px radius
- On focus: border becomes `--border-focus`, focus ring applied
- On submit (Enter or button): border + background change to `--correct-bg` / `--incorrect-bg` / `--near-bg`
- Feedback text below: "Correcto!" / "Incorrecta. Era: [translation_pt]" / "Quase! Verifica a ortografia."
- Disabled after first answer until user advances

### LevelBadge
- Pill, 999px radius, 6px 12px padding, Inter 11/600 uppercase
- Colors per level (see §2)

### DomainBadge
- Same shape as LevelBadge, neutral slate colors (`#475569` text, `#f1f5f9` bg)

### ProgressBar
- Full-width strip, 6px height, bg `--border`
- Fill uses `--brand`; animated width transition 300ms
- "X / Y" count label right-aligned above bar in JetBrains Mono 13/500

### Buttons
- **Primary**: bg `--brand`, white text, 8px radius, 44px height, Inter 15/600
  - Hover: bg `--brand-dark`
  - Focus: focus ring
- **Secondary**: bg transparent, border `--border-strong`, text `--text`, same sizing
  - Hover: bg `--bg-hover`
- **Ghost**: no border, text `--text-secondary`
  - Hover: bg `--bg-subtle`, text `--text`

### Navigation
Desktop: header 60px sticky, logo left, nav links center (Home / Browse / Settings), session progress right if active session.
Mobile: bottom tab bar with 4 icons (Home, Session, Browse, Settings).

---

## 5. Responsive Breakpoints

- `< 640px`: mobile — single column, bottom nav, card full-width
- `640px – 1024px`: tablet — centered card, top nav
- `> 1024px`: desktop — wider layout, sidebar stats possible

All touch targets ≥ 48px on mobile.
