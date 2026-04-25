# Skill: Formatação MZN (Meticais), Datas e Locale Moçambicano

Convenções de apresentação para apps direccionadas ao mercado moçambicano. Pequenos detalhes que separam apps amadoras das que parecem feitas para o sítio.

---

## Hard-and-fast rules

1. **Símbolo da moeda é "MT"** (não "MZN" excepto em contextos técnicos como APIs ou taxas de câmbio). MT vai depois do número, separado por espaço: `1 500 MT`.
2. **Separador de milhares é o ESPAÇO** (estilo francófono/lusófono), NÃO ponto nem vírgula: `1 500 000.00`. Esta é a convenção do BCM.
3. **Separador decimal é o PONTO** (estilo BCM): `64.54`. NÃO uses vírgula.
4. **Datas em DD/MM/AAAA**, NÃO MM/DD ou ISO no UI. ISO (`YYYY-MM-DD`) só em backend/JSON.
5. **Toda renderização de números monetários usa fonte monoespaçada** (JetBrains Mono, Menlo, ou similar). Dígitos alinhados melhoram leitura.

---

## Funções de referência

### JavaScript

```javascript
/**
 * Formata número como moeda Moz: "1 500.00 MT" ou "1 500.00".
 *
 * @param {number} n - Valor numérico
 * @param {object} opts
 * @param {boolean} opts.suffix - Se true, adiciona " MT" no fim
 * @param {number} opts.decimals - Casas decimais (default 2)
 */
export function formatMZN(n, opts = {}) {
  const { suffix = false, decimals = 2 } = opts;
  if (typeof n !== 'number' || isNaN(n)) return '-';

  const formatted = n.toLocaleString('en-US', {
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals,
  }).replace(/,/g, ' ');  // Vírgulas → espaços (separador moz)

  return suffix ? `${formatted} MT` : formatted;
}

/**
 * Apenas o número (para taxas de câmbio onde MT é redundante).
 */
export function formatRate(n, decimals = 2) {
  return formatMZN(n, { suffix: false, decimals });
}

/**
 * Formata percentagem: "12.5%" ou "+0.42%".
 */
export function formatPercent(n, opts = {}) {
  const { showSign = false, decimals = 2 } = opts;
  if (typeof n !== 'number' || isNaN(n)) return '-';
  const sign = showSign && n > 0 ? '+' : '';
  return `${sign}${n.toFixed(decimals)}%`;
}

/**
 * Formata data: "24/04/2026".
 *
 * @param {string|Date} input - ISO string ou Date
 */
export function formatDate(input) {
  const d = typeof input === 'string' ? new Date(input) : input;
  if (isNaN(d.getTime())) return '-';
  return [
    String(d.getDate()).padStart(2, '0'),
    String(d.getMonth() + 1).padStart(2, '0'),
    d.getFullYear(),
  ].join('/');
}

/**
 * Formata hora: "15:30".
 */
export function formatTime(input) {
  const d = typeof input === 'string' ? new Date(input) : input;
  if (isNaN(d.getTime())) return '-';
  return [
    String(d.getHours()).padStart(2, '0'),
    String(d.getMinutes()).padStart(2, '0'),
  ].join(':');
}

/**
 * Formata data + hora: "24/04/2026 · 15:30".
 */
export function formatDateTime(input) {
  return `${formatDate(input)} · ${formatTime(input)}`;
}

/**
 * Parse de input do utilizador no formato moz: "1 500.00" -> 1500.
 * Aceita também "1.500,00" (estilo PT) por tolerância.
 */
export function parseMZN(input) {
  if (typeof input !== 'string') return NaN;
  const cleaned = input.replace(/\s/g, '');
  // Detectar estilo: se tem vírgula como decimal e ponto como milhares
  if (/^\d{1,3}(\.\d{3})+,\d+$/.test(cleaned)) {
    return Number(cleaned.replace(/\./g, '').replace(',', '.'));
  }
  return Number(cleaned.replace(/,/g, ''));
}
```

### Python

```python
from datetime import datetime
from typing import Union

def format_mzn(value: float, suffix: bool = False, decimals: int = 2) -> str:
    if value is None or (isinstance(value, float) and value != value):
        return '-'
    s = f'{value:,.{decimals}f}'.replace(',', ' ')  # vírgulas → espaços
    return f'{s} MT' if suffix else s

def format_rate(value: float, decimals: int = 2) -> str:
    return format_mzn(value, suffix=False, decimals=decimals)

def format_percent(value: float, show_sign: bool = False, decimals: int = 2) -> str:
    if value is None:
        return '-'
    sign = '+' if show_sign and value > 0 else ''
    return f'{sign}{value:.{decimals}f}%'

def format_date(value: Union[str, datetime]) -> str:
    d = datetime.fromisoformat(value) if isinstance(value, str) else value
    return d.strftime('%d/%m/%Y')

def parse_mzn(value: str) -> float:
    cleaned = value.replace(' ', '')
    # Estilo PT (12.500,00) → converter para padrão BCM
    import re
    if re.fullmatch(r'\d{1,3}(\.\d{3})+,\d+', cleaned):
        return float(cleaned.replace('.', '').replace(',', '.'))
    return float(cleaned.replace(',', ''))
```

---

## Exemplos de aplicação

```javascript
formatMZN(1500)                  // "1 500.00"
formatMZN(1500, { suffix: true })// "1 500.00 MT"
formatMZN(1500000, { decimals: 0 }) // "1 500 000"
formatMZN(64.54)                 // "64.54"

formatPercent(0.42, { showSign: true })  // "+0.42%"
formatPercent(-0.26)             // "-0.26%"

formatDate('2026-04-24T15:30:00Z')       // "24/04/2026"
formatTime('2026-04-24T15:30:00Z')       // "15:30"
formatDateTime('2026-04-24T15:30:00Z')   // "24/04/2026 · 15:30"

parseMZN('1 500.00')             // 1500
parseMZN('1.500,00')             // 1500 (tolera estilo PT)
```

---

## Tabela de referência rápida

| Tipo | Bom | Mau |
|------|-----|-----|
| Moeda inline | `1 500 MT` | `1.500,00 MZN`, `MT 1500.00`, `MZN1500` |
| Taxa de câmbio | `64.54` | `64,54`, `64.54 MT` (redundante em tabelas de câmbio) |
| Data UI | `24/04/2026` | `2026-04-24`, `4/24/2026`, `24-04-2026` |
| Data ISO (backend) | `2026-04-24` | `24/04/2026`, `20260424` |
| Hora | `15:30` | `3:30 PM`, `1530`, `15h30` |
| Percentagem | `+0.42%` | `0,42%`, `0.0042`, `42 bp` |
| Telefone moz | `+258 84 123 4567` | `0841234567`, `84-123-4567` |
| NUIT | `123 456 789` | `123-456-789`, `123.456.789` |

---

## Locale técnico

Em apps Node/browser, **NÃO uses `toLocaleString('pt-MZ')` esperando boa formatação** - o suporte é inconsistente. Implementa as funções acima manualmente.

Em ambiente backend (Django, Flask, Spring), define:

```python
# settings.py / config
TIME_ZONE = 'Africa/Maputo'
LANGUAGE_CODE = 'pt-mz'
USE_TZ = True
```

```javascript
// Node
process.env.TZ = 'Africa/Maputo';
// dayjs
import dayjs from 'dayjs';
import 'dayjs/locale/pt';  // pt-mz não existe, usa pt
dayjs.locale('pt');
```

---

## Componentes React reutilizáveis

```jsx
export function Money({ value, suffix = true, mono = true }) {
  const formatted = formatMZN(value, { suffix });
  return <span className={mono ? 'mono' : ''}>{formatted}</span>;
}

export function Rate({ value, decimals = 2 }) {
  return <span className="mono">{formatRate(value, decimals)}</span>;
}

export function Change({ value }) {
  const cls = value > 0 ? 'change-positive' : value < 0 ? 'change-negative' : 'change-neutral';
  return <span className={`change ${cls}`}>{formatPercent(value, { showSign: true })}</span>;
}
```

---

## Armadilhas comuns

- **Usar `toLocaleString('pt-PT')`.** Resulta em `1 500,00` (vírgula decimal) - NÃO é a convenção do BCM. Implementa manualmente.
- **Hardcoding `'MT'` antes do número.** Convenção é depois: `1 500 MT`.
- **Mostrar 4-5 decimais em valores grandes.** Para amounts em meticais, 2 decimais chega. 4 decimais só em taxas de câmbio precisas.
- **Não usar fonte mono em tabelas de números.** Dígitos não alinham - péssimo para leitura.
- **Datas ISO no UI.** Pode parecer "limpo" para devs, mas confunde utilizadores moçambicanos. Sempre DD/MM/AAAA na UI.
- **Não tratar `NaN` ou `null`.** Sempre fallback para "-".

---

## Internacionalização (quando relevante)

Se a app precisa de suportar **PT-MZ + EN** (ex: para diáspora), separa convenções de moeda da string completa:

```javascript
// i18n
const t = {
  pt: {
    'currency.symbol': 'MT',
    'date.format': 'DD/MM/AAAA',
  },
  en: {
    'currency.symbol': 'MZN',
    'date.format': 'YYYY-MM-DD',
  },
};
```
