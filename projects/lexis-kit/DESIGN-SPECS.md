# Lexis - UI Design Specification

## Authoritative Mockup Set

There are **6 HTML mockups** in `design-mocks/`. All are authoritative visual specifications. Read the relevant mockup before implementing each view.

| File | View |
|------|------|
| `00_home.html` | Dashboard — stats, streak, "Start Session" CTA |
| `01_session.html` | Active study session with FlashCard |
| `02_review.html` | End-of-session summary |
| `03_browse.html` | Full word list with filters |
| `04_word_detail.html` | Single word deep-dive |
| `05_settings.html` | User preferences |

---

## CSS Variables

Paste these into `src/index.css`. All component styles must reference these variables — no hardcoded hex values in component files.

```css
:root {
  /* Brand */
  --brand:            #1e40af;
  --brand-dark:       #1e3a8a;
  --brand-light:      #dbeafe;

  /* Feedback */
  --correct:          #15803d;
  --correct-bg:       #dcfce7;
  --incorrect:        #b91c1c;
  --incorrect-bg:     #fee2e2;
  --near:             #b45309;
  --near-bg:          #fef3c7;

  /* Levels */
  --level-b1:         #0369a1;
  --level-b1-bg:      #e0f2fe;
  --level-b2:         #7c3aed;
  --level-b2-bg:      #ede9fe;
  --level-c1:         #9a3412;
  --level-c1-bg:      #ffedd5;

  /* Text */
  --text:             #0f172a;
  --text-strong:      #1e293b;
  --text-secondary:   #475569;
  --text-muted:       #94a3b8;
  --text-disabled:    #cbd5e1;

  /* Surfaces */
  --bg-page:          #f8fafc;
  --bg-card:          #ffffff;
  --bg-subtle:        #f1f5f9;
  --bg-hover:         #f1f5f9;
  --border:           #e2e8f0;
  --border-strong:    #cbd5e1;
  --border-focus:     #1e40af;

  /* Shadows */
  --shadow-card:      0 1px 3px rgba(15,23,42,0.08), 0 1px 2px rgba(15,23,42,0.04);
  --shadow-hover:     0 4px 12px rgba(15,23,42,0.10);
  --shadow-focus:     0 0 0 3px rgba(30,64,175,0.2);

  /* Radii */
  --radius-card:      12px;
  --radius-btn:       8px;
  --radius-pill:      999px;
  --radius-input:     8px;
}
```

---

## View-by-View Specification

### View 1: Home (`00_home.html`)

**Purpose:** show the learner where they stand and get them into a session in one tap.

**Layout (top to bottom):**

1. **Header** — 60px sticky. Logo + "Lexis" left. Nav links center: Home / Browse / Settings. (On mobile: bottom tab bar instead.)
2. **Stats row** — 3 cards side by side (max-width 800px, centered):
   - **Palavras aprendidas** — count of cards with `repetitions ≥ 2` (JetBrains Mono 32/600)
   - **Em revisão hoje** — count of cards due today (JetBrains Mono 32/600), highlighted if > 0
   - **Sequência** — consecutive days with at least one session completed (JetBrains Mono 32/600 + fire icon if ≥ 3)
3. **Session launcher card** — white card, centered, max-width 480px:
   - Heading: "Sessão de estudo"
   - Mode selector: 3 pills — "Novas", "Revisão", "Misto" (default: Misto)
   - Level filter: 3 toggles — B1, B2, C1 (default: all selected)
   - Card count selector: 5 / 10 / 20 (default: 10)
   - Large primary button: "Começar" — disabled if 0 cards match current filters, with message "Não há palavras para os filtros seleccionados."
4. **Recent sessions** — small section below launcher, last 3 sessions each showing date, score (e.g. 8/10), and duration in JetBrains Mono.

**Data source:** `localStorage` (card states, session history, settings)

---

### View 2: Session (`01_session.html`)

**Purpose:** the core study loop. One card at a time.

**Layout:**

1. Header — minimal: logo left, "X de Y" progress right in JetBrains Mono, close (×) icon to abandon session
2. **ProgressBar** — full-width strip below header, animated fill
3. **FlashCard** — centered, max-width 640px:
   - **Card front** (always visible):
     - Top right: LevelBadge + DomainBadge
     - Center: word in Lora 36/700
     - Below word: part of speech label in Inter 11/600 uppercase (e.g. "VERBO")
     - If `showDefinitionFirst = true`: definition appears here in Inter 16/400
   - **Card back** (revealed after answer):
     - Definition in Inter 16/400
     - Example sentence in Inter 15/400 italic, left border `--brand` 3px
     - If `moz_example` exists: second example line, label "Exemplo Moçambique" in small uppercase
     - If `word_family` exists and `showWordFamily = true`: "Família de palavras:" + pills for each form
     - If `note_pt` exists: small note box with amber left border
4. **TranslationInput** — below card:
   - Placeholder: "Escreve a tradução em português..."
   - Enter key or "Verificar" button submits
   - After submission: feedback state (correct / near / incorrect)
   - "Próxima →" button to advance (appears after answering)

**Behaviors:**
- Pressing Escape opens "Abandonar sessão?" confirmation dialog
- After answering, "Próxima →" is the only active action (Enter also advances)
- No skipping without answering

---

### View 3: Review (`02_review.html`)

**Purpose:** show the outcome of the session and guide the learner on what to do next.

**Layout:**

1. Header — full
2. **Hero result card** — centered, max-width 480px:
   - Big score: "8 / 10" in JetBrains Mono 48/600
   - Percentage: "80%" in JetBrains Mono 20/500 muted
   - Duration: "3 min 24 seg" in Inter 13 muted
   - Horizontal row of 3 mini stats: Corretas / Incorrectas / Quase
3. **Word breakdown** — list of all cards from the session:
   - Each row: word (Inter 15/600), result icon (✓ / ✗ / ~), correct translation
   - Rows grouped: Corretas first, then Incorrectas, then Quase
   - Incorrect and near rows have a "Revelar" toggle to show `example_en`
4. **CTA row**:
   - Primary: "Nova sessão" → back to Home
   - Secondary: "Rever incorrectas" → starts a new session with only the incorrect cards from this session

---

### View 4: Browse (`03_browse.html`)

**Purpose:** explore the full vocabulary list with filtering and search.

**Layout:**

1. Header
2. Title: "Vocabulário Académico" + subtitle "200 palavras · TOEFL · IELTS"
3. **Filter bar**:
   - Text search input (filters word and translation_pt in real-time)
   - Level pills: All / B1 / B2 / C1
   - Domain dropdown: All domains + each domain name
   - Sort: A–Z / Nível / Domínio / "A Rever"
4. **Word table** — each row:
   - Word (Inter 15/600, clickable → WordDetail)
   - Part of speech label (Inter 13 muted)
   - Translation (Inter 15/400)
   - LevelBadge
   - DomainBadge
   - SRS status icon: ● new / ✓ learnt / ↻ due for review
5. Count label: "A mostrar X de 200 palavras"

**Behaviors:**
- Clicking any row navigates to `/browse/:id`
- All filters combined (AND logic)
- Filter state persists per-session (not in localStorage)

---

### View 5: Word Detail (`04_word_detail.html`)

**Purpose:** full reference view for a single word.

**Layout:**

1. Header
2. Breadcrumb: "Vocabulário / [word]"
3. **Hero section**:
   - Word in Lora 36/700
   - POS + LevelBadge + DomainBadge in one row
4. **Definition card**:
   - Label: "DEFINIÇÃO" uppercase
   - `definition_en` in Inter 16/400
5. **Translation card**:
   - Label: "TRADUÇÃO (PT)"
   - `translation_pt` in Inter 16/500
   - If `note_pt` exists: amber note box below
6. **Examples card**:
   - `example_en` italic with blue left border
   - If `moz_example` exists: second block labelled "Exemplo Moçambique"
7. **Word family card** (only if `word_family` exists):
   - Label: "FAMÍLIA DE PALAVRAS"
   - Pill for each form, each clickable → search Browse for that word
8. **SRS status card**:
   - If never studied: "Ainda não estudaste esta palavra." + "Adicionar à próxima sessão" button
   - If studied: next review date, ease factor (shown as ★★★☆☆ out of 5), total repetitions

---

### View 6: Settings (`05_settings.html`)

**Purpose:** customise study behaviour and appearance.

**Settings groups:**

**Sessão de estudo**
- Nível padrão — All / B1 / B2 / C1 (dropdown)
- Domínio padrão — All / each domain (dropdown)
- Número de cartas — 5 / 10 / 20 (segmented control)

**Apresentação**
- Mostrar definição primeiro — toggle (shows EN definition before asking for translation)
- Mostrar família de palavras — toggle (shows word_family after answering)
- Tema — Auto / Claro / Escuro (segmented control)

**Dados**
- Limpar todo o progresso — destructive button, requires confirmation dialog
- Exportar progresso — downloads a JSON file of `lexis:cards` and `lexis:sessions`
- Sobre — app version, "Vocabulário: AWL + corpus académico"
