# Metical - Architecture

---

## Overview

Metical is a currency converter consisting of two parts:

1. **Backend (Node + Express)** - small proxy/cache server that fetches XML data from the Banco de Moçambique API, parses it, caches it, and exposes clean JSON endpoints to the frontend
2. **Frontend (React + Vite + TypeScript)** - single-page app that displays rates, handles conversions, and persists user preferences in `localStorage`

The backend exists for three reasons:

- **Avoid CORS** - the BCM API does not support CORS headers, so browser direct fetch fails
- **Cache** - the BCM data updates once a day (business days); we cache for 1 hour by default to reduce load
- **Parse XML → JSON** - the frontend works with clean JSON; XML parsing lives on the server

---

## Project Structure

```
metical/
├── package.json                  # pnpm workspaces root
├── pnpm-workspace.yaml
├── tsconfig.json                 # shared TS config
├── .gitignore
├── .env.example
├── README.md
├── packages/
│   ├── shared/                   # shared types between backend and frontend
│   │   ├── src/
│   │   │   ├── types.ts          # Rate, DailySnapshot, HistoricalRate, etc.
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── server/                   # Express backend
│   │   ├── src/
│   │   │   ├── index.ts          # server entry, routes setup
│   │   │   ├── routes.ts         # route definitions
│   │   │   ├── bcm-client.ts     # fetches from BCM, handles errors
│   │   │   ├── xml-parser.ts     # XML -> typed JSON
│   │   │   ├── cache.ts          # in-memory cache with TTL
│   │   │   └── fixtures.ts       # loads sample-data/*.xml for offline dev
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── test/
│   │       ├── xml-parser.test.ts
│   │       └── cache.test.ts
│   └── web/                      # React frontend
│       ├── src/
│       │   ├── main.tsx
│       │   ├── App.tsx
│       │   ├── index.css         # tokens from DESIGN-SPECS.md
│       │   ├── lib/
│       │   │   ├── api.ts        # typed client for backend
│       │   │   ├── storage.ts    # localStorage wrapper
│       │   │   └── format.ts     # number/date formatting
│       │   ├── hooks/
│       │   │   ├── useRates.ts
│       │   │   └── useConversion.ts
│       │   ├── components/
│       │   │   ├── ConversionWidget.tsx
│       │   │   ├── CurrencySelector.tsx
│       │   │   ├── RateRow.tsx
│       │   │   ├── ChangeBadge.tsx
│       │   │   ├── FlagBadge.tsx
│       │   │   ├── SparklineChart.tsx
│       │   │   └── Header.tsx
│       │   └── pages/
│       │       ├── Home.tsx
│       │       ├── RatesTable.tsx
│       │       ├── RateDetail.tsx
│       │       ├── History.tsx
│       │       └── Settings.tsx
│       ├── package.json
│       ├── tsconfig.json
│       ├── vite.config.ts
│       ├── index.html
│       └── test/
│           ├── format.test.ts
│           └── ConversionWidget.test.tsx
└── sample-data/
    ├── daily.xml
    ├── weekly.xml
    └── monthly.xml
```

---

## Stack Decisions

### Why React + Vite

- **Vite** - fastest dev experience; HMR in milliseconds; production build is small
- **React** - most participants will know it; hiring standard in Mozambique; plenty of examples
- **TypeScript** - essential for teaching good practices; catches errors the LLM introduces

### Why Express (not Next.js or Fastify)

- Smallest possible server - ~50 lines for our use case
- No framework magic to explain
- Participants can port the pattern to any language easily

### Why pnpm workspaces

- Shared `types` package keeps frontend and backend in sync
- Faster install than npm
- One command (`pnpm dev`) starts both apps

### Why no database

- Rates data is already cached by the BCM - our own cache is just a speed optimization
- User history lives in `localStorage` - no accounts, no privacy concerns
- Keeps the project under the token budget

---

## Data Model

### Shared types (`packages/shared/src/types.ts`)

```ts
export interface Rate {
  currency: string;      // ISO code, e.g. "USD"
  name: string;          // e.g. "Dolar"
  location: string;      // e.g. "Estados Unidos"
  buy: number;           // MZN required to buy 1 unit
  sell: number;          // MZN received when selling 1 unit
}

export interface DailySnapshot {
  baseCurrency: "MZN";
  date: string;          // ISO format: "2026-04-24"
  lastUpdate: string;    // ISO format: "2026-04-24T15:30:00Z"
  rates: Rate[];
}

export interface HistoricalRate {
  date: string;          // ISO format
  buy: number;
  sell: number;
}

export interface CurrencyHistory {
  currency: string;
  history: HistoricalRate[];  // sorted oldest → newest
}

export interface ApiError {
  error: string;
  code: "BCM_UNAVAILABLE" | "INVALID_RESPONSE" | "CACHE_MISS";
  cachedSnapshot?: DailySnapshot;  // present if code === "BCM_UNAVAILABLE" and cache exists
}
```

### Frontend localStorage keys

- `metical:history` - array of saved conversions (max 100)
- `metical:settings` - user preferences (favorite currency, decimals, theme)

---

## Backend API

All endpoints return JSON. Errors use HTTP status codes + a body matching `ApiError`.

### `GET /api/rates/daily`

Returns the latest daily snapshot.

Response: `DailySnapshot`

Cache: in-memory, TTL 1 hour. If BCM is unreachable and cache exists, returns cached data with header `X-Metical-Cache: stale`.

### `GET /api/rates/weekly`

Returns the last 5 days of rates.

Response: `{ baseCurrency: "MZN", rates: Array<{ date: string, rates: Rate[] }> }`

Cache: TTL 1 hour.

### `GET /api/rates/monthly`

Returns the last ~24 days of rates.

Response: same shape as weekly.

Cache: TTL 6 hours.

### `GET /api/rates/currency/:code`

Convenience endpoint. Returns historical rates for one currency across the monthly range.

Response: `CurrencyHistory`

### `GET /healthz`

Returns `{ status: "ok", bcmReachable: boolean, cacheAge: number }` for monitoring.

---

## BCM Integration

### Endpoints (upstream)

- `https://www.bancomoc.mz/bmapi/exchangerates/` - returns `QuoteCurrency` XML (daily snapshot)
- `https://www.bancomoc.mz/bmapi/exchangerates-weekly/` - returns `QuoteCurrencyByDate` XML (5 days)
- `https://www.bancomoc.mz/bmapi/exchangerates-monthly/` - returns `QuoteCurrencyByDate` XML (~24 days)

### XML shape (simplified, see `sample-data/` for full)

Daily:

```xml
<QuoteCurrency>
  <baseCurrency>MZN</baseCurrency>
  <date>20260424</date>
  <lastUpdate>20260424 15:30:00</lastUpdate>
  <rates>
    <QuoteRates>
      <buy>63.27</buy>
      <currency>USD</currency>
      <location>Estados Unidos</location>
      <n>Dolar</n>
      <sell>64.54</sell>
    </QuoteRates>
    <!-- ...22 more currencies... -->
  </rates>
  <type>daily</type>
</QuoteCurrency>
```

Weekly and monthly use `<QuoteCurrencyByDate>` with nested `<QuoteRatesByDate>` containing `<date>` + `<values>`.

### Parsing strategy

- Use `fast-xml-parser` (small, no dependencies, works in Node and browser)
- Normalize date format: BCM uses `YYYYMMDD` → we convert to `YYYY-MM-DD`
- Normalize `lastUpdate`: BCM uses `YYYYMMDD HH:MM:SS` → ISO 8601
- Handle the `<n>` element - this is the currency display name (note: in the XML it is literally named `<n>`, not `<name>`, despite seeming unusual)

### Error handling

- Timeout: 10 seconds on each BCM request
- Retries: 1 retry with 2s delay
- If BCM returns non-XML or invalid XML, return `INVALID_RESPONSE` and keep serving cache
- If network fails, return `BCM_UNAVAILABLE` and include cached snapshot if present
- Never expose raw BCM errors to the client

### Development mode

Set `METICAL_USE_FIXTURES=true` in `.env` to serve `sample-data/*.xml` instead of hitting BCM. This is the default for local dev. Makes the app work offline and during demos.

---

## Frontend

### Routing

Use `react-router-dom` v6+ with the following routes:

- `/` → Home
- `/rates` → RatesTable
- `/rates/:code` → RateDetail (e.g., `/rates/USD`)
- `/history` → History
- `/settings` → Settings

### Data fetching

Use `@tanstack/react-query` for all API calls. Stale time: 5 minutes (matches typical update cadence). Cache time: 1 hour.

### Number formatting

All numbers are rendered via `lib/format.ts`:

- `formatRate(n: number, decimals = 2)` → `"64.54"` or `"64.5400"`
- `formatAmount(n: number)` → `"1 000 000.00"` (space thousands separator)
- `formatDate(iso: string)` → `"24/04/2026"`
- `formatTime(iso: string)` → `"15:30"`

Never render a raw number with `.toString()` in components.

### State

- Server state: React Query (rates)
- Persistent client state: `localStorage` via `lib/storage.ts`
- Transient UI state: `useState` inside components

No global state manager needed. Keep it simple.

---

## Testing

### Backend

- Unit tests for `xml-parser.ts` using the real `sample-data/*.xml` files
- Unit tests for `cache.ts` (TTL behavior, invalidation)
- Integration test: one smoke test that boots the server against fixtures and hits `/api/rates/daily`

### Frontend

- Unit tests for `lib/format.ts`
- Component test for `ConversionWidget` - typing an amount updates the other side
- No E2E tests in this project (out of scope for workshop budget)

Run: `pnpm test` at root runs all tests.

---

## Deployment (out of scope for workshop)

For reference only:

- Backend: any Node host (Railway, Fly.io, a VPS). Docker image optional.
- Frontend: static hosting (Netlify, Vercel, Cloudflare Pages)
- Environment variable `VITE_API_URL` on frontend points to deployed backend
- BCM endpoints are called only from the backend, never from the browser

---

## Security & Privacy

- No user accounts, no login, no personal data collection
- `localStorage` is per-device; clearing browser data clears the app
- Backend does not log user requests or amounts - only aggregated count of BCM fetches
- BCM API is public; no keys or credentials needed
