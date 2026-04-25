# Metical - Agentic Build Instructions

## What This Is

Metical is a currency converter for Mozambican users. It fetches exchange rates from the Banco de Moçambique API, caches them, and lets users convert between MZN and ~22 other currencies. Two-part monorepo: Express backend (proxy + cache + XML parser) and React frontend (SPA).

This document is the instruction set for Claude Code (or opencode / codex) to build Metical autonomously, part by part, following `PLAN.md`.

---

## CRITICAL: Read Before Writing Any Code

Read all of the following in full before starting any implementation:

1. `ARCHITECTURE.md` - technical reference: stack, project structure, data model, API, BCM integration
2. `PLAN.md` - build order with per-part success criteria and checkpoints
3. `DESIGN.md` - design tokens, colors, typography, component patterns
4. `DESIGN-SPECS.md` - per-view specs with references to HTML mockups
5. `design-mocks/*.html` - the authoritative visual specification for every view

Do not write any code before completing this reading pass.

## Skills

The `skills/` folder contains structured knowledge that should be consulted **before implementing** related features. Read the relevant `SKILL.md` in full when:

- Implementing BCM data integration → `skills/bcm-data/SKILL.md`
- Formatting numbers, money, or dates → `skills/mzn-formatting/SKILL.md`
- Following the spec-kit pattern → `skills/spec-kit-pattern/SKILL.md`
- Reusing the visual design tokens → `skills/metical-design-system/SKILL.md`

The "hard-and-fast rules" at the top of each `SKILL.md` are absolute - do not break them silently. If a rule conflicts with a project requirement, stop and ask.

---

## Design Mocks Are Mandatory

There are **8 HTML mockups** in `design-mocks/`. These are the authoritative visual specification for every user-facing surface. Every page built in React must match its corresponding mockup.

### Procedure for Each View

1. Open the relevant mockup in `design-mocks/`.
2. Open the `<style>` block at the top of the mockup to see token definitions (each mockup is self-contained).
3. Cross-reference against `DESIGN.md` and `DESIGN-SPECS.md`.
4. If `DESIGN.md`/`DESIGN-SPECS.md` conflict with a mockup, **the spec files are authoritative**. Mockups are canonical for layout and behavior; the spec files are canonical for tokens.
5. Implement. No extra cards. No omitted sections. No invented layouts.

### Additional Rules

- CSS variable names must match `DESIGN-SPECS.md` exactly. Do not invent new variables without adding them there first.
- No external UI component libraries (no shadcn, no MUI, no Chakra). Custom components only.
- No Tailwind. Plain CSS with the token system.
- All UI text in **Portuguese (Moçambique)**. No English in user-facing strings.
- No emojis in UI except currency flag emojis in `FlagBadge`.

---

## Package Manager

Use `pnpm` for all commands. Not npm. Not yarn. Not bun.

- `pnpm install`
- `pnpm add <pkg>` (for latest)
- `pnpm --filter @metical/<pkg> <command>` (for workspace-specific commands)
- `pnpm dev` at root runs backend + frontend concurrently
- `pnpm test` at root runs all tests

## Dependencies

Check for latest npm version when adding a dependency. Do not pin to memory - use `pnpm add <pkg>` (no version) to get latest.

Allowed dependencies:

- **Backend:** `express`, `cors`, `fast-xml-parser`, `vitest`, `@types/express`
- **Frontend:** `react`, `react-dom`, `react-router-dom`, `@tanstack/react-query`, `vitest`, `@testing-library/react`, `@vitejs/plugin-react`, `vite`

If you need something not on this list, **stop and ask**. Do not install it silently.

---

## TypeScript Rules

- Strict mode on everywhere. No `any`.
- Shared types live in `packages/shared/src/types.ts`. Import via `@metical/shared`.
- Explicit return types on exported functions.
- No `as` casts except for narrow, justified cases (e.g., DOM queries). Prefer proper type guards.

---

## Testing Rules

- Every Part in `PLAN.md` that creates new logic has tests listed.
- Tests run with Vitest.
- Backend tests use the `sample-data/*.xml` fixtures - never mock parsing logic.
- Frontend tests use `@testing-library/react`.
- Do not skip or `.only` tests before marking a part complete.

---

## Style Rules

- No comments that restate what the code does. Only comments that explain **why**.
- Function components only. No class components.
- Hooks for state; lift state only when needed.
- File naming: `PascalCase.tsx` for components, `camelCase.ts` for everything else.
- One default export per component file.
- Never commit `console.log` in production code paths.

---

## Budget Awareness

This project is being built during a workshop with a tight token budget. To stay efficient:

- Do not regenerate a file that already exists - edit it.
- Do not refactor unrelated code "while you're at it".
- Do not add features not in `PLAN.md`. If you think something is missing, stop and ask.
- Keep `/clear` usage in mind between parts to reduce context.

---

## Checkpoint Protocol

At the end of each Part in `PLAN.md`:

1. Run all the tests listed in Success Criteria.
2. Confirm all success criteria are met.
3. Summarize in 3-5 bullets what was built and what the next part will do.
4. **Stop.** Wait for the user to say "avança" or "go ahead".

Do not proceed past a Checkpoint unilaterally, even if confident.

---

## When In Doubt

- If a spec is ambiguous → ask the user, don't guess.
- If a mockup and a spec disagree → follow `DESIGN.md` / `DESIGN-SPECS.md`, flag the inconsistency in your response.
- If a dependency seems needed that isn't on the allow-list → ask.
- If a test fails and the fix is unclear → show the failure and ask for direction.

The goal is a correct, complete, well-tested app - not speed. A pause to ask is always better than an incorrect assumption.
