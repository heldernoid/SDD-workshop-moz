# Metical - UI Design Specification

## Authoritative Mockup Set

There are **8 HTML mockups** in `design-mocks/`. All are authoritative specifications. The build agent must read the relevant mockup before implementing each view.

| File | View |
|---|---|
| `00_home.html` | Home page with hero conversion widget |
| `01_rates_table.html` | Full rates table (all BCM currencies) |
| `02_rate_detail.html` | Single currency detail with monthly chart |
| `03_history.html` | User's conversion history |
| `04_settings.html` | App settings and preferences |
| `05_error_offline.html` | Offline/API-unavailable error state |
| `06_empty_state.html` | First-time user, empty history |
| `07_mobile.html` | Mobile layout (home view) |

All mockups link to each other - click any nav item or currency row to navigate. This lets reviewers verify the full flow before implementation begins.

---

## Design Philosophy

Metical serves Mozambican users who **consult** exchange rates rather than work inside them. The experience prioritizes:

- **Glanceability** - a user opens the app, sees today's MZN/USD rate in under 1 second, and leaves
- **Trust** - the source (BCM) is always visible, the timestamp is always shown, numbers are in mono
- **Calm** - generous whitespace, no animations beyond subtle transitions, no push notifications

The app is **not** a trading platform. It does not show charts by default. It does not nudge users to refresh. It behaves like a good reference document: accurate, timestamped, and quiet.

See `DESIGN.md` for the complete design system (tokens, typography, colors, components).

---

## View-by-View Specification

### View 1: Home (`00_home.html`)

**Purpose:** primary entry point. User sees today's reference rate and converts a specific amount.

**Layout (top to bottom):**

1. **Header** - 64px sticky. Logo left, nav center (Home / Rates / History), settings icon right.
2. **Hero conversion widget** - centered, max-width 720px. Two panels side-by-side with a circular swap button between them.
   - Left panel: source currency (default MZN), amount input
   - Right panel: target currency (default USD), computed amount (read-only)
   - Below both panels: small text showing the exchange rate used and timestamp, e.g. `1 USD = 64.54 MZN · atualizado 24/04/2026 15:30`
3. **Quick rates strip** - horizontal row of 4 cards (USD, ZAR, EUR, GBP) showing current sell rate and daily change. Clicking a card jumps to the corresponding detail view.
4. **Footer** - BCM attribution, last update timestamp.

**Behaviors:**

- Typing in the left amount instantly updates the right amount (debounced 150ms)
- Clicking the swap button reverses source ↔ target currencies
- Currency selector pills open a dropdown with all 22 currencies from the daily feed
- Default MZN amount on first load: `1000`

**Data source:** `GET /api/rates/daily` → latest daily snapshot

---

### View 2: Rates Table (`01_rates_table.html`)

**Purpose:** see all available currencies at once, sorted and filterable.

**Layout:**

1. Header (same as Home)
2. Page title: "Taxas do Banco de Moçambique" + subtitle with date
3. Search/filter bar: free-text input + sort dropdown (code / name / rate / change)
4. Table of all 22 currencies. Each row:
   - Circular flag badge (32px)
   - Currency code (Inter 15 weight 600) and name (Inter 13 weight 400 secondary)
   - Buy rate and sell rate (JetBrains Mono 18 weight 500), right-aligned
   - 5-day change badge (% from weekly feed)
5. Footer with update timestamp and source attribution

**Behaviors:**

- Clicking any row navigates to the detail view (`02_rate_detail.html`)
- Search filters in real-time by code or name
- Sort persists per-session

**Data source:** `GET /api/rates/daily` + `GET /api/rates/weekly` (merged)

---

### View 3: Rate Detail (`02_rate_detail.html`)

**Purpose:** deep-dive on a single currency, see history and convert any amount.

**Layout:**

1. Header
2. Breadcrumb: `Taxas / USD - Dólar Americano`
3. **Hero stats card** - current buy/sell rates prominent, last update, 5-day change %
4. **Chart card** - line chart showing last 24 days of sell rate (from monthly feed)
5. **Mini converter** - simple amount input that converts using the current sell rate
6. **Data table** - last 7 daily snapshots with date, buy, sell

**Behaviors:**

- Chart tooltip on hover: date and exact rate in mono
- Time range selector: 7d / 24d (default 24d)

**Data source:** `GET /api/rates/monthly` filtered by currency

---

### View 4: History (`03_history.html`)

**Purpose:** user's past conversions for reference (e.g., "what rate did I use last week?").

**Layout:**

1. Header
2. Page title + total count (e.g., "12 conversões guardadas")
3. Clear-all button (secondary, right-aligned)
4. List of entries, newest first. Each entry:
   - Timestamp
   - Amount → converted amount (mono)
   - Rate used (mono, smaller)
   - Delete icon button

**Behaviors:**

- Conversions are stored in `localStorage` - no backend account needed
- Saved automatically when user actively converts on the Home view (not on every keystroke - only after 2s of input inactivity or on explicit "Save" click)
- Max 100 entries; oldest trimmed

---

### View 5: Settings (`04_settings.html`)

**Purpose:** customize defaults and appearance.

**Settings:**

- **Moeda favorita** - default target currency on Home (dropdown of all 22)
- **Casas decimais** - 2 / 4 (affects display only)
- **Tema** - Auto / Claro / Escuro
- **Limpar histórico** - destructive button, requires confirmation
- **Fonte de dados** - read-only display: "Banco de Moçambique · atualizado a cada 24h"
- **Sobre** - app version, link to source (GitHub)

**Persistence:** `localStorage`.

---

### View 6: Error - Offline (`05_error_offline.html`)

**Purpose:** graceful degradation when BCM API is unreachable and cache is stale/missing.

**Layout:**

- Full-page card centered
- Icon: offline/cloud-slash, subdued color
- Heading: "Sem ligação à API do BCM"
- Body: explain that cached data may be shown if available, with cache timestamp
- Two buttons: "Tentar novamente" (primary gold) and "Usar última taxa guardada" (secondary) if cache exists

**Behaviors:**

- Appears when server returns 503 or network fails and no cache is available
- If a cache exists (even if >24h old), show it with a prominent "Dados de [date]" banner at the top of Home instead of blocking the user

---

### View 7: Empty State (`06_empty_state.html`)

**Purpose:** onboarding message on History view when no conversions have been saved yet.

**Layout:**

- Illustration (simple SVG of a receipt or notebook)
- Heading: "Ainda não tens histórico"
- Body: "As tuas conversões vão aparecer aqui automaticamente."
- CTA: "Fazer primeira conversão" → goes to Home

---

### View 8: Mobile (`07_mobile.html`)

**Purpose:** show how the home view collapses on mobile (width <640px).

**Key changes vs desktop:**

- Conversion panels stack vertically; swap button sits between them as a row
- Top header collapses: logo + settings icon only
- Bottom tab bar appears: Home / Rates / History
- Touch targets increase to 48px minimum
- Quick rates strip becomes horizontally scrollable

---

## Consistency Evaluation - Required Before Implementation

Because mockups were built in a single pass, they should be consistent, but **always perform a consistency check before implementing a view**:

1. Open the mockup for the view you are implementing.
2. Open the `<style>` block at the top of the mockup to see the token definitions (each mockup is self-contained - the same token block is duplicated across all eight files).
3. Confirm: colors, font sizes, radii, and spacing in the mockup match `DESIGN.md`.
4. If conflict: `DESIGN.md` is authoritative. Apply the corrected version.

---

## CSS Variables

All CSS variables defined in the `:root` block at the top of every mockup must be used in the React implementation too (`src/index.css`). No hardcoded hex colors in components. No inventing new variables without updating `DESIGN.md`.

```css
:root {
  /* Brand */
  --brand: #d4a017;
  --brand-dark: #a67c00;
  --brand-light: #fef3c7;
  --brand-muted: #fdebb0;

  /* Semantic */
  --positive: #0f766e;
  --positive-bg: #ccfbf1;
  --negative: #c13515;
  --negative-bg: #fee4d6;
  --neutral: #78716c;

  /* Text */
  --text: #1c1917;
  --text-strong: #44403c;
  --text-secondary: #78716c;
  --text-muted: #a8a29e;
  --text-disabled: #d6d3d1;

  /* Surface */
  --bg-page: #fafaf7;
  --bg-card: #ffffff;
  --bg-subtle: #f5f5f4;
  --bg-hover: #f5f5f4;

  /* Borders */
  --border: #e7e5e4;
  --border-strong: #d6d3d1;
  --border-focus: #d4a017;

  /* Shadows */
  --shadow-card: 0 0 0 1px rgba(28,25,23,0.04), 0 2px 8px rgba(28,25,23,0.06);
  --shadow-hover: 0 0 0 1px rgba(28,25,23,0.06), 0 4px 16px rgba(28,25,23,0.08);
  --ring-focus: 0 0 0 3px rgba(212,160,23,0.2);

  /* Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;
  --radius-pill: 999px;

  /* Fonts */
  --font-ui: Inter, -apple-system, system-ui, Roboto, sans-serif;
  --font-mono: "JetBrains Mono", Menlo, Consolas, monospace;
}
```

## Localization

All user-facing strings are in **Portuguese (Moçambique)**. Currency names follow BCM terminology (see `sample-data/daily.xml`):

- "Dólar" (not "Dólar dos EUA")
- "Rand" (not "Rand sul-africano")
- "Euro"
- "Libra"
- "Metical" (for MZN)

Dates formatted as `DD/MM/YYYY`, times as `HH:MM`. Decimals use `.` as separator (matching BCM data format); consider thousand separator ` ` (space) for amounts ≥ 10000: `1 000 000.00`.
