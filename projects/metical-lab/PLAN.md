# Metical - Build Plan

## How to Use This Document

Work through parts in strict order. Each part ends with a **Checkpoint**. At every Checkpoint:

1. Run all tests specified in the Success Criteria section.
2. Stop. Do not proceed to the next part until the user explicitly says "avança" or "go ahead".

Read `ARCHITECTURE.md`, `AGENTS.md`, `DESIGN.md`, and `DESIGN-SPECS.md` in full before writing any code.

---

## Baseline Rules

Non-negotiable. Check every rule before marking any part complete.

- All user-facing text in **Portuguese (Moçambique)**.
- No hardcoded colors in components - use CSS variables from `DESIGN-SPECS.md`.
- Every number rendered to the user goes through a formatter in `lib/format.ts` and uses JetBrains Mono font.
- Never commit raw BCM credentials (there are none, but don't invent any).
- `METICAL_USE_FIXTURES=true` is the default during development - the app works fully offline using `sample-data/*.xml`.
- No `any` types in TypeScript. Strict mode is on.
- No external UI libraries beyond what's listed in `ARCHITECTURE.md` (React Query, React Router, fast-xml-parser).

---

## Part 1: Project Scaffold and Shared Types

### Goal

Monorepo setup with three packages (`shared`, `server`, `web`) scaffolded and building cleanly.

### Tasks

1. Root `package.json`, `pnpm-workspace.yaml`, root `tsconfig.json` with project references
2. `packages/shared/src/types.ts` with all types from ARCHITECTURE.md (`Rate`, `DailySnapshot`, `HistoricalRate`, `CurrencyHistory`, `ApiError`)
3. `packages/shared/src/index.ts` - barrel export
4. `packages/server/package.json`, `tsconfig.json` - depends on `@metical/shared`, includes `express`, `fast-xml-parser`, `vitest`
5. `packages/web/package.json`, `tsconfig.json`, `vite.config.ts` - depends on `@metical/shared`, includes `react`, `react-dom`, `react-router-dom`, `@tanstack/react-query`, `vitest`
6. `.env.example` at root with `METICAL_USE_FIXTURES=true` and `PORT=3001`
7. Root `README.md` with setup instructions (install, dev, test, build)
8. `.gitignore` covering `node_modules`, `dist`, `.env`, `.turbo`

### Success Criteria

- `pnpm install` completes without errors
- `pnpm -r build` builds all three packages
- `pnpm -r typecheck` passes with zero errors
- Shared types importable from `server` and `web`

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 2: BCM XML Parser

### Goal

Pure function that turns BCM XML (daily, weekly, monthly) into typed JSON.

### Tasks

1. `packages/server/src/xml-parser.ts`:
   - `parseDaily(xml: string): DailySnapshot`
   - `parseHistorical(xml: string): CurrencyHistory[]` - works for weekly or monthly (same schema)
   - Normalize `YYYYMMDD` date format → ISO `YYYY-MM-DD`
   - Normalize `lastUpdate` → ISO 8601
   - Handle the `<n>` element for currency name (literally named `<n>` in BCM XML)
2. `packages/server/test/xml-parser.test.ts`:
   - Parse each `sample-data/*.xml` file
   - Assert the number of currencies is 22 for daily
   - Assert weekly returns 5 dates
   - Assert monthly returns 24 dates
   - Assert USD buy rate matches what's in the fixture

### Success Criteria

- `pnpm --filter @metical/server test` passes
- All 22 currencies from the daily fixture are parsed
- No type errors

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 3: Cache and BCM Client

### Goal

HTTP client that calls BCM with timeout, retry, and cache. In fixture mode, reads from `sample-data/`.

### Tasks

1. `packages/server/src/cache.ts`:
   - `class Cache<T>` with `get(key)`, `set(key, value, ttlMs)`, `getStale(key)` (returns even if expired)
   - In-memory Map-based; no external dependencies
2. `packages/server/src/fixtures.ts`:
   - `loadFixture(type: "daily" | "weekly" | "monthly"): string` - reads from `sample-data/`
3. `packages/server/src/bcm-client.ts`:
   - `fetchDaily()`, `fetchWeekly()`, `fetchMonthly()` - each returns parsed JSON
   - If `METICAL_USE_FIXTURES === "true"`, return from fixtures
   - Otherwise fetch from BCM with 10s timeout, 1 retry
   - Cache responses (daily: 1h, weekly: 1h, monthly: 6h)
   - On failure, fall back to stale cache if available, otherwise throw `BCM_UNAVAILABLE`
4. `packages/server/test/cache.test.ts`:
   - TTL expiry works
   - `getStale` returns expired data

### Success Criteria

- Tests pass
- Running in fixture mode, calling `fetchDaily()` three times results in one fixture read (cache hit)

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 4: Express Server and Routes

### Goal

Running server that exposes the JSON API.

### Tasks

1. `packages/server/src/routes.ts` - define handlers for:
   - `GET /api/rates/daily`
   - `GET /api/rates/weekly`
   - `GET /api/rates/monthly`
   - `GET /api/rates/currency/:code`
   - `GET /healthz`
2. `packages/server/src/index.ts` - create Express app, mount routes, start listening on `PORT` (default 3001)
3. Enable CORS for `http://localhost:5173` (Vite default)
4. Error middleware: catches thrown errors, returns `ApiError` JSON with correct HTTP status
5. `packages/server/test/routes.test.ts`:
   - Boot server in fixture mode
   - Hit each endpoint, assert 200 and correct shape

### Success Criteria

- `pnpm --filter @metical/server dev` starts server on 3001
- `curl http://localhost:3001/api/rates/daily` returns valid JSON with 22 currencies
- `curl http://localhost:3001/healthz` returns `{ status: "ok", ... }`
- Tests pass

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 5: Design Tokens and Layout Shell

### Goal

React app scaffolded with design tokens, fonts, and the header shell. No data yet.

### Tasks

1. `packages/web/index.html` - include Inter and JetBrains Mono from Google Fonts
2. `packages/web/src/index.css` - paste all CSS variables from `DESIGN-SPECS.md` §"CSS Variables". Set body font, background color, base text.
3. `packages/web/src/App.tsx` - React Router setup with placeholder routes
4. `packages/web/src/components/Header.tsx` - matches `design-mocks/00_home.html` header exactly
5. `packages/web/src/pages/Home.tsx` - empty page, just "Home" heading for now
6. Similar stubs for `RatesTable`, `RateDetail`, `History`, `Settings`

### Success Criteria

- `pnpm --filter @metical/web dev` starts Vite on 5173
- Navigating to `/`, `/rates`, `/history`, `/settings` shows the correct page
- Header appears on every page
- Page background is `#fafaf7`, text is `#1c1917`, fonts load correctly
- Visually match the header from mockups

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 6: API Client and Data Hooks

### Goal

Frontend can fetch data from the backend.

### Tasks

1. `packages/web/src/lib/api.ts`:
   - Typed client: `getDaily()`, `getWeekly()`, `getMonthly()`, `getCurrency(code)`
   - Base URL from `import.meta.env.VITE_API_URL` with default `http://localhost:3001`
2. `packages/web/src/lib/format.ts`:
   - `formatRate(n, decimals = 2)`
   - `formatAmount(n)` with space thousands separator
   - `formatDate(iso)` → `DD/MM/YYYY`
   - `formatTime(iso)` → `HH:MM`
3. `packages/web/src/hooks/useRates.ts`:
   - `useDaily()`, `useWeekly()`, `useMonthly()`, `useCurrency(code)` - wrap React Query
4. Wrap app in `QueryClientProvider` in `main.tsx`
5. `packages/web/test/format.test.ts` - unit tests for formatters

### Success Criteria

- On Home page, log the result of `useDaily()` to the console - it returns the 22 currencies
- Formatter tests pass

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 7: Conversion Widget (Home Page)

### Goal

The hero conversion widget works. Users can convert MZN ↔ any currency.

### Tasks

1. `packages/web/src/components/FlagBadge.tsx` - circular 32px with emoji flag for each currency (map currency code → flag emoji; fallback gray circle with code)
2. `packages/web/src/components/CurrencySelector.tsx` - pill with flag, code, chevron, opens dropdown with all available currencies
3. `packages/web/src/components/ConversionWidget.tsx`:
   - Two panels (source + target)
   - Swap button between them
   - Debounced input (150ms) recalculates target
   - Uses `sell` rate by default
   - Displays current rate and timestamp below panels
4. `packages/web/src/hooks/useConversion.ts` - pure logic: given amount + source + target + rates, compute target
5. Render on Home page, matching `design-mocks/00_home.html`
6. `packages/web/test/ConversionWidget.test.tsx` - typing an amount updates the other side

### Success Criteria

- User can type `1000` on the left (MZN), see the USD equivalent on the right
- Swap button reverses currencies
- Currency selector dropdown lists all 22 currencies with flags
- Layout matches the mockup
- Component test passes

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 8: Rates Table and Rate Detail

### Goal

Tabela completa de moedas e vista de detalhe com gráfico.

### Tasks

1. `packages/web/src/components/ChangeBadge.tsx` - pill showing `+1.2%` / `-0.8%` with color per `DESIGN.md`
2. `packages/web/src/components/RateRow.tsx` - matches `design-mocks/01_rates_table.html`
3. `packages/web/src/pages/RatesTable.tsx` - list all 22 currencies; search + sort; clicking a row navigates to `/rates/:code`
4. `packages/web/src/components/SparklineChart.tsx` - simple SVG line chart (no external library), shows 24 data points
5. `packages/web/src/pages/RateDetail.tsx` - matches `design-mocks/02_rate_detail.html`. Hero card with buy/sell, chart, mini-converter, recent history table.
6. Calculate 5-day change % from weekly data

### Success Criteria

- Clicking a quick-rate card on Home navigates to the correct detail view
- Detail view shows a sparkline with 24 data points
- Search on the table filters in real-time
- All numbers are in JetBrains Mono

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 9: History and Settings (localStorage)

### Goal

User can save conversions and customize preferences.

### Tasks

1. `packages/web/src/lib/storage.ts`:
   - Typed wrapper for `localStorage`
   - `getHistory()`, `addHistory(entry)`, `clearHistory()`
   - `getSettings()`, `updateSettings(partial)`
   - Max 100 history entries
2. Home page: after 2s of input inactivity with a non-zero amount, save the conversion to history
3. `packages/web/src/pages/History.tsx` - matches `design-mocks/03_history.html`. Lists entries newest first, delete per entry, clear all.
4. `packages/web/src/pages/Settings.tsx` - matches `design-mocks/04_settings.html`. Favorite currency, decimals, theme, clear history, source info, about.
5. Empty state for History - matches `design-mocks/06_empty_state.html`.
6. Settings take effect (decimals affect all displayed rates; favorite currency becomes default target on Home)

### Success Criteria

- Saving a conversion on Home appears in History after navigating there
- Clearing history empties the list and shows the empty state
- Changing decimals in Settings updates display everywhere immediately

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 10: Error States and Mobile

### Goal

Graceful handling when the API is down + responsive mobile layout.

### Tasks

1. Global error boundary on the app
2. `packages/web/src/pages/OfflineError.tsx` - matches `design-mocks/05_error_offline.html`. Shown when API returns 503 or network fails and no cache exists.
3. Stale-cache banner: if API returns `X-Metical-Cache: stale`, show a subtle banner at the top of Home: "Dados de [date] · a tentar actualizar"
4. Mobile styles: match `design-mocks/07_mobile.html`. Conversion panels stack, bottom tab bar below 640px, touch targets ≥ 48px.

### Success Criteria

- Stopping the backend mid-session shows the offline error state instead of a white screen
- Resizing to <640px collapses to mobile layout correctly
- Bottom tab bar appears on mobile

### Checkpoint

Append a section to `TEST_REPORT.md` (create the file if it does not exist) with: Part name, date/time, success criteria checked, what passed, what failed, and any deviations from the plan. Stop. Show the user the structure. Wait for "go ahead".

---

## Part 11: Polish and Final Pass

### Goal

Eliminate rough edges. Ship.

### Tasks

1. Loading states: skeleton placeholders on rate rows, chart, and conversion widget while data loads (200ms delay before showing to avoid flash)
2. Keyboard shortcuts: `/` focuses search on rates table, `Esc` closes dropdowns
3. Accessibility pass: all interactive elements have proper labels, focus rings visible, color contrast ≥ 4.5:1
4. Lighthouse audit: performance ≥ 90, accessibility ≥ 95
5. README.md at root: clear `pnpm install && pnpm dev` setup, screenshots
6. Final visual diff against all 8 mockups

### Success Criteria

- All 8 mockups match the implementation within 5% visual difference
- Lighthouse passes
- No console errors
- Keyboard navigation works throughout

### Checkpoint

Append a final summary to `TEST_REPORT.md` covering all parts. Done.

---

## Out of Scope

Explicitly **not** part of this build:

- User accounts / authentication
- Backend database (Postgres, SQLite, etc.)
- Deployment scripts
- E2E tests (Playwright, Cypress)
- PWA / offline-first / service worker
- Push notifications when rates change
- Multiple languages beyond Portuguese

If a participant wants to add these after the workshop, they can replicate this spec-kit pattern for the new feature.
