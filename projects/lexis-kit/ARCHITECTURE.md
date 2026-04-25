# Lexis - Architecture

---

## Overview

Lexis is a vocabulary learning app focused on academic English as it appears in high-stakes exams (TOEFL, IELTS) and university entrance tests. It presents a word, an example sentence in context, and asks the learner to translate it into Portuguese.

The app is a single-package React SPA. There is no backend. All data lives in a local JSON file (`data/vocabulary.json`) bundled with the app. Progress and spaced-repetition state are persisted in `localStorage`.

---

## Project Structure

```
lexis/
├── package.json
├── tsconfig.json
├── vite.config.ts
├── index.html
├── README.md
├── .gitignore
├── .env.example
├── data/
│   └── vocabulary.json          # 200-word dataset (provided)
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── index.css                # CSS variables from DESIGN-SPECS.md
│   ├── types.ts                 # all shared TypeScript types
│   ├── lib/
│   │   ├── storage.ts           # localStorage wrapper (typed)
│   │   ├── srs.ts               # spaced-repetition scheduling logic
│   │   └── format.ts            # date/number formatters
│   ├── hooks/
│   │   ├── useDeck.ts           # current session queue management
│   │   ├── useProgress.ts       # overall stats and streak
│   │   └── useSettings.ts       # user preferences
│   ├── components/
│   │   ├── FlashCard.tsx        # word card with flip animation
│   │   ├── TranslationInput.tsx # answer input with feedback state
│   │   ├── ProgressBar.tsx      # session progress strip
│   │   ├── LevelBadge.tsx       # B1 / B2 / C1 pill
│   │   ├── DomainBadge.tsx      # domain label pill
│   │   ├── ResultFeedback.tsx   # correct / incorrect overlay
│   │   └── Header.tsx
│   └── pages/
│       ├── Home.tsx             # dashboard with stats and "start session" CTA
│       ├── Session.tsx          # active study session
│       ├── Review.tsx           # end-of-session summary
│       ├── Browse.tsx           # full word list with filters
│       ├── WordDetail.tsx       # single word deep-dive
│       └── Settings.tsx
└── test/
    ├── srs.test.ts
    └── format.test.ts
```

---

## Stack Decisions

### Why React + Vite (no backend)
- Vocabulary dataset is static — no server needed
- All SRS state fits in localStorage (<50KB for 500 words)
- Zero deployment friction: `vite build` → static host

### Why no external component library
- Design system is fully specified in `DESIGN-SPECS.md`
- No Tailwind, no shadcn, no MUI — plain CSS with token variables

### Why Vitest
- Same config as Vite; zero friction
- SRS logic is pure and easy to unit-test

---

## Data Model

### vocabulary.json schema

```ts
interface VocabEntry {
  id: number;
  word: string;
  pos: string;                 // "verb" | "noun" | "adjective" | "adverb" | ...
  definition_en: string;       // short English definition
  translation_pt: string;      // Portuguese (Mozambique) translation
  example_en: string;          // sentence in English
  domain: DomainId;
  level: "B1" | "B2" | "C1";
  word_family?: string[];      // related forms (optional)
  moz_example?: string;        // Mozambique-contextualised example (optional)
  note_pt?: string;            // didactic note in Portuguese (optional)
}

type DomainId =
  | "research"
  | "argumentation"
  | "society"
  | "economy"
  | "science_tech"
  | "education"
  | "health"
  | "environment";
```

### SRS Card State (`src/lib/srs.ts`)

```ts
interface CardState {
  id: number;             // matches VocabEntry.id
  interval: number;       // days until next review
  easeFactor: number;     // SM-2 ease (starts at 2.5)
  nextReview: string;     // ISO date
  repetitions: number;    // successful reviews in a row
  lastResult: "correct" | "incorrect" | null;
}
```

### Session State

```ts
interface SessionConfig {
  mode: "new" | "review" | "mixed";
  levelFilter: ("B1" | "B2" | "C1")[];
  domainFilter: DomainId[];
  cardCount: number;       // 5 | 10 | 20
}

interface SessionResult {
  date: string;
  totalCards: number;
  correct: number;
  durationMs: number;
  levelBreakdown: Record<string, number>;
}
```

### localStorage Keys

| Key | Contents |
|-----|----------|
| `lexis:cards` | `Record<number, CardState>` — SRS state per word |
| `lexis:sessions` | `SessionResult[]` — history, max 200 entries |
| `lexis:settings` | `UserSettings` — preferences |

### UserSettings

```ts
interface UserSettings {
  targetLevel: "B1" | "B2" | "C1" | "all";
  defaultDomain: DomainId | "all";
  defaultCardCount: 5 | 10 | 20;
  showDefinitionFirst: boolean;    // show EN definition before asking for translation
  showWordFamily: boolean;         // show related forms after answering
  theme: "auto" | "light" | "dark";
}
```

---

## SRS Algorithm (`src/lib/srs.ts`)

Simplified SM-2:

1. **New card** → `interval = 1`, `easeFactor = 2.5`, `repetitions = 0`
2. **Correct answer:**
   - `repetitions += 1`
   - If `repetitions === 1`: `interval = 1`
   - If `repetitions === 2`: `interval = 6`
   - Else: `interval = round(interval × easeFactor)`
   - `easeFactor = max(1.3, easeFactor + 0.1)`
   - `nextReview = today + interval days`
3. **Incorrect answer:**
   - `repetitions = 0`
   - `interval = 1`
   - `easeFactor = max(1.3, easeFactor - 0.2)`
   - `nextReview = today`

Session queue is built by `useDeck`:
- **Review mode**: cards where `nextReview <= today`, sorted by overdue days desc
- **New mode**: cards with no `CardState` yet, in vocabulary order
- **Mixed mode**: due reviews first, then fill to `cardCount` with new cards

---

## Routing

Use `react-router-dom` v6+:

| Path | Page |
|------|------|
| `/` | Home |
| `/session` | Session |
| `/review` | Review (end-of-session summary) |
| `/browse` | Browse |
| `/browse/:id` | WordDetail |
| `/settings` | Settings |

---

## Answer Checking (`src/lib/srs.ts`)

The user types a Portuguese translation. Checking is lenient:

1. Normalise both strings: lowercase, strip accents, trim whitespace
2. Split `translation_pt` on `,` and `/` to get accepted variants
3. **Match** if the user's answer equals any variant (after normalisation)
4. Optionally: Levenshtein distance ≤ 1 for near-miss detection (surface "almost correct" feedback without counting as wrong)

Export: `checkAnswer(input: string, entry: VocabEntry): "correct" | "near" | "incorrect"`

---

## Testing

- `test/srs.test.ts` — scheduling math: new card, correct streak, incorrect reset, ease floor
- `test/format.test.ts` — date formatting, percentage formatting

Run: `pnpm test`
