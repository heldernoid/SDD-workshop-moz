# Lexis - Agentic Build Instructions

## What This Is

Lexis is a vocabulary learning SPA for academic English (TOEFL/IELTS). It is a single React + Vite + TypeScript package. There is no backend — vocabulary data is bundled as JSON and all state lives in `localStorage`.

This document is the instruction set for Claude Code to build Lexis autonomously, part by part, following `PLAN.md`.

---

## CRITICAL: Read Before Writing Any Code

Read all of the following in full before starting any implementation:

1. `ARCHITECTURE.md` — stack, project structure, data model, SRS algorithm, routing
2. `PLAN.md` — build order with per-part success criteria and checkpoints
3. `DESIGN.md` — design tokens, colors, typography, component patterns
4. `DESIGN-SPECS.md` — per-view specs, CSS variable definitions, mockup list
5. `design-mocks/*.html` — the authoritative visual specification for every view

Do not write any code before completing this reading pass.

---

## Design Mocks Are Mandatory

There are **6 HTML mockups** in `design-mocks/`. Each page built in React must match its corresponding mockup.

### Procedure for Each View

1. Open the relevant mockup in `design-mocks/`.
2. Cross-reference against `DESIGN.md` and `DESIGN-SPECS.md`.
3. If they conflict, **the spec files are authoritative** for tokens; mockups are authoritative for layout and behaviour.
4. Implement exactly what is specified. No extra cards, no omitted sections, no invented layouts.

### Hard Rules

- CSS variable names must match `DESIGN-SPECS.md` exactly.
- No external UI component libraries. No Tailwind. No shadcn. No MUI. Plain CSS + token variables.
- All UI text in **Portuguese (Moçambique)**.
- The study word must always use the `Lora` serif font. This is non-negotiable.
- Every number rendered to the user goes through `lib/format.ts`.

---

## Package Manager

Use `pnpm`. Not npm. Not yarn.

```bash
pnpm install
pnpm dev           # starts Vite on 5173
pnpm test          # runs Vitest
pnpm build         # production build
pnpm typecheck     # tsc --noEmit
```

## Dependencies

Allowed:

- `react`, `react-dom`, `react-router-dom`
- `vite`, `@vitejs/plugin-react`
- `vitest`, `@testing-library/react`, `jsdom`
- `typescript`

**Nothing else.** If you think you need an additional dependency, stop and ask.

No charting library — the SRS stats on the Review page are rendered with plain SVG if needed, or just numbers.

---

## TypeScript Rules

- Strict mode everywhere. No `any`.
- All types in `src/types.ts`. Import from there.
- Explicit return types on all exported functions.
- No `as` casts except justified DOM queries.

---

## Testing Rules

- `test/srs.test.ts` covers all SRS scheduling cases — use the real algorithm, no mocks.
- `test/format.test.ts` covers all formatter functions.
- Do not skip or `.only` tests before marking a part complete.

---

## SRS Rules

- Never change the SM-2 parameters (ease floor 1.3, starting ease 2.5, initial intervals 1/6).
- `checkAnswer` must handle accented characters correctly: strip accents before comparing.
- Levenshtein distance ≤ 1 = "near" result. Do not count near as incorrect.
- Session results are saved to `localStorage` **only when the session is fully completed** — not abandoned.

---

## Style Rules

- Function components only.
- Hooks for all state.
- File naming: `PascalCase.tsx` for components, `camelCase.ts` for everything else.
- One default export per component file.
- No `console.log` in production code paths.
- No comments that restate what the code does. Only comments that explain why.

---

## Checkpoint Protocol

At the end of each Part in `PLAN.md`:

1. Run all tests listed in the Part's Success Criteria.
2. Confirm all criteria are met.
3. Summarise in 3–5 bullets what was built and what the next part will do.
4. **Stop.** Wait for the user to say "avança" or "go ahead".

Do not proceed past a checkpoint unilaterally.

---

## When In Doubt

- Spec is ambiguous → ask, don't guess.
- Mockup and spec disagree → follow spec files, flag the inconsistency.
- Need a dependency not on the list → ask before installing.
- A test fails and the fix is unclear → show the failure, ask for direction.
