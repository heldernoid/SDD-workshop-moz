# Lexis - Build Plan

## How to Use This Document

Work through parts in strict order. Each part ends with a **Checkpoint**. At every Checkpoint:

1. Run all tests specified in the Success Criteria section.
2. Stop. Do not proceed to the next part until the user explicitly says "avança" or "go ahead".

Read `ARCHITECTURE.md`, `AGENTS.md`, `DESIGN.md`, and `DESIGN-SPECS.md` in full before writing any code.

---

## Baseline Rules

Non-negotiable. Check every rule before marking any part complete.

- All user-facing text in **Portuguese (Moçambique)**.
- No hardcoded hex values in components — use CSS variables from `DESIGN-SPECS.md`.
- The study word always renders in Lora serif. No exceptions.
- Every number rendered to the user goes through a formatter in `lib/format.ts` and uses JetBrains Mono font.
- No external UI libraries beyond what's listed in `ARCHITECTURE.md`.
- No `any` types in TypeScript. Strict mode is on.
- `pnpm` only. No npm, yarn, or bun.

---

## Part 1: Project Scaffold and Types

### Goal

Vite + React + TypeScript project scaffolded and building cleanly, with all shared types defined.

### Tasks

1. `package.json` with scripts: `dev`, `build`, `test`, `typecheck`
2. `vite.config.ts` — React plugin, `test.environment: "jsdom"`
3. `tsconfig.json` — strict mode, path alias `@/` → `src/`
4. `src/types.ts` — all types from `ARCHITECTURE.md`: `VocabEntry`, `DomainId`, `CardState`, `SessionConfig`, `SessionResult`, `UserSettings`
5. `src/main.tsx` + `src/App.tsx` — minimal router shell (just renders "Lexis" on `/`)
6. `index.html` — Google Fonts: Inter (400,500,600,700), Lora (600,700), JetBrains Mono (500)
7. `src/index.css` — all CSS variables from `DESIGN-SPECS.md §CSS Variables`, body font, background color
8. `.gitignore`, `.env.example`, `README.md` (brief setup instructions)

### Success Criteria

- `pnpm install` completes without errors
- `pnpm build` produces a `dist/` with no TypeScript errors
- `pnpm typecheck` passes with zero errors
- `pnpm dev` starts on port 5173 and renders "Lexis" at `/`
- Google Fonts load; body background is `#f8fafc`

### Checkpoint

Append a section to `TEST_REPORT.md` (create if it doesn't exist) with: Part name, date/time, success criteria checked, what passed, what failed, any deviations. Stop. Show the file tree. Wait for "avança".

---

## Part 2: Data Layer and Storage

### Goal

Vocabulary data is loadable and all localStorage operations are typed and tested.

### Tasks

1. Copy `data/vocabulary.json` into the project (provided — do not regenerate)
2. `src/lib/storage.ts`:
   - `getCards(): Record<number, CardState>` — returns `{}` if key missing
   - `setCards(cards: Record<number, CardState>): void`
   - `getSessions(): SessionResult[]` — returns `[]` if key missing, max 200 entries enforced on write
   - `addSession(result: SessionResult): void`
   - `clearSessions(): void`
   - `getSettings(): UserSettings` — returns defaults if key missing
   - `updateSettings(partial: Partial<UserSettings>): void`
   - `clearAllProgress(): void` — deletes `lexis:cards` and `lexis:sessions`
3. `src/lib/format.ts`:
   - `formatCount(n: number): string` — e.g. `"42"`
   - `formatPercent(correct: number, total: number): string` — e.g. `"80%"`
   - `formatDuration(ms: number): string` — e.g. `"3 min 24 seg"`
   - `formatDate(iso: string): string` — e.g. `"24/04/2026"`
4. `test/format.test.ts` — unit tests for all four formatters

### Success Criteria

- `pnpm test` passes
- `getSettings()` with an empty localStorage returns a fully-typed `UserSettings` with correct defaults
- `addSession` called 201 times results in `getSessions().length === 200` (oldest trimmed)

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 3: SRS Engine

### Goal

Spaced-repetition scheduling logic is implemented, tested, and ready for the session loop.

### Tasks

1. `src/lib/srs.ts`:
   - `initCard(id: number): CardState` — creates a new card with defaults
   - `scheduleCard(card: CardState, result: "correct" | "incorrect"): CardState` — applies SM-2, returns updated card
   - `isDue(card: CardState, today?: string): boolean` — true if `nextReview <= today`
   - `normalise(s: string): string` — lowercase, strip accents, trim
   - `checkAnswer(input: string, entry: VocabEntry): "correct" | "near" | "incorrect"` — lenient matching with Levenshtein ≤ 1 for "near"
   - `buildQueue(entries: VocabEntry[], cards: Record<number, CardState>, config: SessionConfig): VocabEntry[]` — returns ordered array of entries for the session
2. `test/srs.test.ts`:
   - New card → correct × 2 → intervals are 1, 6
   - Correct streak × 3 → third interval is `round(6 × 2.5) = 15`
   - Incorrect resets repetitions to 0, interval to 1
   - Ease floor is 1.3 (can't go below)
   - `checkAnswer`: exact match → correct; accent variant → correct; 1-char typo → near; totally wrong → incorrect
   - `buildQueue` in "review" mode only returns due cards
   - `buildQueue` in "new" mode only returns cards with no CardState
   - `buildQueue` respects `cardCount`

### Success Criteria

- `pnpm test` passes
- All 8 test cases above pass

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 4: Routing Shell and Header

### Goal

All routes exist, the header renders correctly, and navigation works.

### Tasks

1. Full React Router setup in `App.tsx` with all 6 routes (stub pages for now)
2. `src/components/Header.tsx` — matches `design-mocks/00_home.html` header exactly
3. `src/components/LevelBadge.tsx` — pill with correct colors per level
4. `src/components/DomainBadge.tsx` — neutral slate pill with domain `title_pt`
5. Stub pages: `Home.tsx`, `Session.tsx`, `Review.tsx`, `Browse.tsx`, `WordDetail.tsx`, `Settings.tsx` — each renders just its page name for now
6. Mobile bottom tab bar in `App.tsx` rendered below `<640px`

### Success Criteria

- Navigating to all 6 routes renders the correct stub page title
- Header appears on all pages
- LevelBadge renders B1 in sky blue, B2 in violet, C1 in orange-rust
- Below 640px, bottom tab bar appears; header nav links are hidden
- `pnpm typecheck` passes

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Show screenshots or describe layout. Wait for "avança".

---

## Part 5: Home Page

### Goal

Dashboard shows real stats and the session launcher is functional.

### Tasks

1. `src/hooks/useProgress.ts`:
   - Returns `{ learned: number, dueToday: number, streak: number }`
   - `learned`: cards with `repetitions >= 2`
   - `dueToday`: cards where `isDue(card, today)`
   - `streak`: consecutive days with at least one `SessionResult` (check `lexis:sessions`)
2. `src/hooks/useSettings.ts` — thin wrapper around `storage.getSettings()` / `storage.updateSettings()`
3. `Home.tsx` — full implementation matching `design-mocks/00_home.html`:
   - Stats row (3 cards)
   - Session launcher card with mode selector, level toggles, card count selector
   - "Começar" button — disabled with message if 0 cards match filters
   - Recent sessions list (last 3)
   - On "Começar" click: saves `SessionConfig` to sessionStorage and navigates to `/session`

### Success Criteria

- Stats row shows correct numbers based on seeded localStorage data (test manually)
- "Começar" is disabled when all levels are deselected
- Clicking "Começar" with valid config navigates to `/session`
- Layout matches the mockup at desktop and mobile widths
- Streak shows fire icon when ≥ 3

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 6: Session Page

### Goal

The core study loop works end-to-end.

### Tasks

1. `src/hooks/useDeck.ts`:
   - Reads `SessionConfig` from sessionStorage (set by Home)
   - Calls `buildQueue` to get the card list
   - Tracks current index, session results array, and elapsed time
   - Exposes: `current: VocabEntry | null`, `progress: {done, total}`, `submitAnswer(input: string): AnswerResult`, `advance(): void`, `finish(): SessionResult`
2. `src/components/FlashCard.tsx` — matches spec exactly (word in Lora, badges, flip, definition, example)
3. `src/components/TranslationInput.tsx` — input + "Verificar" button + feedback states
4. `src/components/ProgressBar.tsx` — animated fill
5. `src/components/ResultFeedback.tsx` — correct / near / incorrect display with correct translation
6. `Session.tsx` — full implementation matching `design-mocks/01_session.html`
   - On last card `advance()`: calls `finish()`, saves session via `storage.addSession()`, updates cards via `storage.setCards()`, navigates to `/review` with result in state

### Success Criteria

- Starting a 5-card session cycles through exactly 5 cards
- Correct answer advances the card's interval in localStorage
- Incorrect answer resets the card's interval
- Pressing Enter submits the answer; pressing Enter again advances
- Abandon dialog appears on Escape / close button
- Layout matches the mockup
- `pnpm typecheck` passes

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 7: Review Page

### Goal

End-of-session summary shows results and offers follow-up actions.

### Tasks

1. `Review.tsx` — reads `SessionResult` and card results from router state (passed by Session)
   - Hero score card: "X / Y", percentage, duration
   - Mini stats: Corretas / Incorrectas / Quase
   - Word breakdown list grouped by result
   - "Revelar" toggle on incorrect/near rows
   - "Nova sessão" → `/`
   - "Rever incorrectas" → navigates to `/session` with config set to the failed cards only

### Success Criteria

- All card results from the session are shown
- Percentages add up correctly
- "Revelar" toggles `example_en` visibility per row
- "Rever incorrectas" is disabled (or hidden) if there were zero incorrect answers
- Layout matches `design-mocks/02_review.html`

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 8: Browse and Word Detail

### Goal

Users can explore the full vocabulary list with filters and drill into any word.

### Tasks

1. `Browse.tsx` — full implementation matching `design-mocks/03_browse.html`:
   - Text search (filters `word` and `translation_pt`)
   - Level pill filters (multi-select, AND logic)
   - Domain dropdown
   - Sort: A–Z / Nível / Domínio / "A Rever" (due cards first)
   - Table with word, POS, translation, LevelBadge, DomainBadge, SRS status icon
   - Row count label
   - Click row → `/browse/:id`
2. `WordDetail.tsx` — full implementation matching `design-mocks/04_word_detail.html`

### Success Criteria

- Text search filters in real-time
- Selecting B2 + C1 shows only B2 and C1 words
- Selecting a domain filters correctly
- "A Rever" sort puts due cards at the top
- WordDetail shows all optional fields when present (`word_family`, `moz_example`, `note_pt`)
- SRS status card shows "nunca estudaste" for new words

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 9: Settings Page

### Goal

User preferences persist and take effect immediately.

### Tasks

1. `Settings.tsx` — full implementation matching `design-mocks/05_settings.html`
2. All setting changes call `updateSettings()` immediately (no explicit save button)
3. Theme toggle: adds `data-theme="dark"` to `<html>` — dark theme CSS variables are out of scope for this build (the toggle just sets the attribute, which can be styled later)
4. "Limpar todo o progresso" — shows a confirmation dialog before calling `clearAllProgress()`
5. "Exportar progresso" — serialises `lexis:cards` + `lexis:sessions` to a JSON file and triggers a browser download

### Success Criteria

- Changing `defaultCardCount` in Settings is reflected in the session launcher on Home (test by navigating back)
- Confirmation dialog appears before clearing progress
- Export downloads a valid JSON file containing both keys
- `pnpm typecheck` passes

### Checkpoint

Append to `TEST_REPORT.md`. Stop. Wait for "avança".

---

## Part 10: Polish and Final Pass

### Goal

Eliminate rough edges and verify against all mockups.

### Tasks

1. Loading state: if vocabulary JSON takes > 200ms to parse (unlikely, but add a guard), show a centered spinner
2. Empty states:
   - Home: if `dueToday === 0` and `learned === 0`, show a welcome message "Bem-vindo! Começa com uma sessão de novas palavras."
   - Browse: if filters return 0 results, show "Nenhuma palavra corresponde aos filtros seleccionados."
3. Accessibility pass: all interactive elements have `aria-label` or visible text labels; focus rings visible; color contrast ≥ 4.5:1
4. Keyboard shortcuts: `/` focuses the search input on Browse; `Enter` on Home's launcher card submits
5. Final visual diff against all 6 mockups — note any deviations
6. `README.md` — complete with: setup, `pnpm install && pnpm dev`, how to use a custom vocabulary JSON

### Success Criteria

- `pnpm test` passes (all existing tests)
- `pnpm build` produces no errors or warnings
- `pnpm typecheck` passes
- All 6 mockups match the implementation in structure and token usage
- No `console.log` or `console.error` in production code paths
- Keyboard navigation works throughout

### Checkpoint

Append a final summary to `TEST_REPORT.md` covering all parts. Done.

---

## Out of Scope

Explicitly **not** part of this build:

- Backend or API of any kind
- User accounts or authentication
- Cloud sync of progress
- Audio pronunciation
- Dark theme CSS variables (the toggle sets the attribute; styling is a future task)
- Image or illustration assets
- E2E tests (Playwright, Cypress)
- PWA / offline service worker
- Multiple interface languages

If the participant wants to add these after the workshop, they can replicate this spec-kit pattern for the new feature.
