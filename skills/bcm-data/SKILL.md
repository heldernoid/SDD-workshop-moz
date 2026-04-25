# Skill: Dados do Banco de Moçambique (BCM)

Como aceder, parsear, e usar os dados públicos do Banco de Moçambique. Todos os endpoints retornam XML, são públicos e não precisam de autenticação.

---

## Hard-and-fast rules

1. **Sempre cachear no servidor.** A API do BCM é lenta (5-15s por pedido) e instável. Nunca chames directamente do browser nem em cada page-load.
2. **Sempre fornecer fallback offline.** Os ficheiros XML em `sample-data/` devem ser usáveis se a API falhar.
3. **Atributo XML `<n>` é literalmente esse nome** (não é typo). Não renomear no parsing - a API usa-o para "name".
4. **Datas vêm como `YYYYMMDD` ou `MM/YYYY` ou `YYYYTN`** (trimestre). Normalizar para ISO 8601 (`YYYY-MM-DD`) na primeira oportunidade.
5. **Nunca expor a URL do BCM ao browser.** Sempre via backend proxy.

---

## Endpoints disponíveis

| Endpoint | Conteúdo | Tamanho típico | TTL cache sugerido |
|----------|----------|----------------|--------------------|
| `https://www.bancomoc.mz/bmapi/exchangerates/` | Câmbio diário (snapshot) | ~4 KB | 1h |
| `https://www.bancomoc.mz/bmapi/exchangerates-weekly/` | Câmbio últimos 5 dias | ~19 KB | 1h |
| `https://www.bancomoc.mz/bmapi/exchangerates-monthly/` | Câmbio últimos ~24 dias | ~90 KB | 6h |
| `https://www.bancomoc.mz/bmapi/inflation-rates/monthly` | Inflação mensal (var. M-1) | ~6 KB | 24h |
| `https://www.bancomoc.mz/bmapi/inflation-rates/yearly` | Inflação homóloga (12m) | ~6 KB | 24h |
| `https://www.bancomoc.mz/bmapi/interest-rates` | Taxas Mimo, FPC, FPD, Prime | ~600 B | 24h |
| `https://www.bancomoc.mz/bmapi/pib` | PIB trimestral | ~3 KB | 24h |

---

## Schemas (resumo)

### Exchange rates (daily)

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
    <!-- ~22 currencies -->
  </rates>
  <type>daily</type>
</QuoteCurrency>
```

### Exchange rates (weekly/monthly)

```xml
<QuoteCurrencyByDate>
  <baseCurrency>MZN</baseCurrency>
  <rates>
    <QuoteRatesByDate>
      <date>20260424</date>
      <values>
        <QuoteRates>...</QuoteRates>
        <!-- 22 currencies for this date -->
      </values>
    </QuoteRatesByDate>
    <!-- 5 dates (weekly) or ~24 dates (monthly) -->
  </rates>
</QuoteCurrencyByDate>
```

### Inflation monthly

```xml
<InflationCurrency>
  <currency>MZN</currency>
  <rates>
    <InflationPibRates>
      <month>3</month>
      <rate>0.22</rate>
      <year>2026</year>
    </InflationPibRates>
    <!-- ~24 months back -->
  </rates>
</InflationCurrency>
```

`<rate>` é a variação mensal em % (Março 2026 = +0.22%).

### Inflation yearly (homóloga)

```xml
<InflationCurrency>
  <rates>
    <InflationPibRates>
      <year>03/2026</year>
      <rate>3.37</rate>
    </InflationPibRates>
  </rates>
</InflationCurrency>
```

`<year>` é string `MM/YYYY`. `<rate>` é a inflação homóloga 12m em % (Março 2026 = 3.37%).

### Interest rates

```xml
<InterestPibCurrency>
  <rates>
    <InterestRates><n>Taxa Mimo</n><value>9.25</value></InterestRates>
    <InterestRates><n>FPC</n><value>12.25</value></InterestRates>
    <InterestRates><n>FPD</n><value>6.25</value></InterestRates>
    <InterestRates><n>Prime Rate</n><value>15.5</value></InterestRates>
  </rates>
</InterestPibCurrency>
```

Apenas snapshot actual. Não há histórico. Os 4 indicadores:
- **Taxa Mimo:** taxa de política monetária do BCM
- **FPC:** Facilidade Permanente de Cedência (limite superior)
- **FPD:** Facilidade Permanente de Depósito (limite inferior)
- **Prime Rate:** taxa de referência para empréstimos comerciais

### PIB trimestral

```xml
<InterestPibCurrency>
  <values>
    <InflationPibRates>
      <year>2025T3</year>
      <value>-0.85</value>
    </InflationPibRates>
  </values>
</InterestPibCurrency>
```

`<year>` é `YYYYTN` onde `N` é trimestre 1-4. `<value>` é a variação trimestral em %.

---

## Parsing em Node.js

```javascript
import { XMLParser } from 'fast-xml-parser';

const parser = new XMLParser({
  ignoreAttributes: false,
  attributeNamePrefix: '@_',
  parseTagValue: true,
  // Importante: forçar `rates.QuoteRates` a ser sempre array, mesmo com 1 elemento
  isArray: (name) => ['QuoteRates', 'QuoteRatesByDate', 'InflationPibRates', 'InterestRates'].includes(name),
});

function parseDailyRates(xml) {
  const json = parser.parse(xml);
  const root = json.QuoteCurrency;
  return {
    baseCurrency: root.baseCurrency,
    date: normalizeDate(String(root.date)),
    lastUpdate: normalizeTimestamp(root.lastUpdate),
    rates: root.rates.QuoteRates.map(r => ({
      currency: r.currency,
      name: r.n,
      location: r.location,
      buy: Number(r.buy),
      sell: Number(r.sell),
    })),
  };
}

function normalizeDate(yyyymmdd) {
  // "20260424" -> "2026-04-24"
  return `${yyyymmdd.slice(0, 4)}-${yyyymmdd.slice(4, 6)}-${yyyymmdd.slice(6, 8)}`;
}

function normalizeTimestamp(s) {
  // "20260424 15:30:00" -> "2026-04-24T15:30:00Z"
  const [d, t] = s.split(' ');
  return `${normalizeDate(d)}T${t}Z`;
}

function parseInflationMonthly(xml) {
  const json = parser.parse(xml);
  return json.InflationCurrency.rates.InflationPibRates.map(r => ({
    year: Number(r.year),
    month: Number(r.month),
    rate: Number(r.rate),  // percentual mensal
  }));
}

function parseInflationYearly(xml) {
  const json = parser.parse(xml);
  return json.InflationCurrency.rates.InflationPibRates.map(r => {
    const [month, year] = String(r.year).split('/');
    return {
      year: Number(year),
      month: Number(month),
      rate: Number(r.rate),  // homóloga em %
    };
  });
}

function parsePib(xml) {
  const json = parser.parse(xml);
  return json.InterestPibCurrency.values.InflationPibRates.map(r => {
    const [year, q] = String(r.year).split('T');
    return {
      year: Number(year),
      quarter: Number(q),
      value: Number(r.value),  // variação trimestral %
    };
  });
}

function parseInterestRates(xml) {
  const json = parser.parse(xml);
  return json.InterestPibCurrency.rates.InterestRates.reduce((acc, r) => {
    acc[r.n] = Number(r.value);
    return acc;
  }, {});
  // -> { "Taxa Mimo": 9.25, "FPC": 12.25, "FPD": 6.25, "Prime Rate": 15.5 }
}
```

---

## Parsing em Python

```python
from xml.etree import ElementTree as ET

NS = '{http://schemas.datacontract.org/2004/07/CRT.SmartPortals.Web.Models}'

def parse_daily_rates(xml_text: str):
    root = ET.fromstring(xml_text)
    rates = []
    for rate in root.find(f'{NS}rates').findall(f'{NS}QuoteRates'):
        rates.append({
            'currency': rate.find(f'{NS}currency').text,
            'name': rate.find(f'{NS}n').text,
            'location': rate.find(f'{NS}location').text,
            'buy': float(rate.find(f'{NS}buy').text),
            'sell': float(rate.find(f'{NS}sell').text),
        })
    date = root.find(f'{NS}date').text
    return {
        'base_currency': root.find(f'{NS}baseCurrency').text,
        'date': f'{date[0:4]}-{date[4:6]}-{date[6:8]}',
        'rates': rates,
    }
```

---

## Cache simples em Node

```javascript
class TtlCache {
  constructor() { this.store = new Map(); }
  get(key) {
    const e = this.store.get(key);
    if (!e || Date.now() > e.expiry) return null;
    return e.value;
  }
  getStale(key) { return this.store.get(key)?.value || null; }
  set(key, value, ttlMs) {
    this.store.set(key, { value, expiry: Date.now() + ttlMs });
  }
}

const cache = new TtlCache();
const ONE_HOUR = 60 * 60 * 1000;

async function fetchDailyWithCache() {
  const cached = cache.get('daily');
  if (cached) return cached;

  try {
    const res = await fetch('https://www.bancomoc.mz/bmapi/exchangerates/', {
      signal: AbortSignal.timeout(15000),
    });
    if (!res.ok) throw new Error(`BCM ${res.status}`);
    const xml = await res.text();
    const data = parseDailyRates(xml);
    cache.set('daily', data, ONE_HOUR);
    return data;
  } catch (err) {
    // Fallback to stale cache
    const stale = cache.getStale('daily');
    if (stale) return { ...stale, _stale: true };
    throw err;
  }
}
```

---

## Fixtures para desenvolvimento offline

Os XMLs reais estão em `sample-data/`:

```
sample-data/
├── daily.xml              # Câmbio diário (snapshot)
├── weekly.xml             # Câmbio últimos 5 dias
├── monthly.xml            # Câmbio últimos ~24 dias
├── inflation-monthly.xml  # Inflação mensal
├── inflation-yearly.xml   # Inflação homóloga
├── interest-rates.xml     # Taxas Mimo/FPC/FPD/Prime
└── pib.xml                # PIB trimestral
```

Padrão recomendado: variável `BCM_USE_FIXTURES=true` faz o cliente ler estes ficheiros em vez de chamar a API. Permite desenvolvimento sem rede e testes determinísticos.

---

## Armadilhas comuns

- **Assumir que `<rate>` na inflação é em decimal.** Não - é já em percentagem. `0.22` significa 0.22%, não 22%.
- **Esquecer namespaces no parsing Python.** O XML do BCM tem `xmlns="http://schemas.datacontract.org/..."`. Sem prefixar, `find()` falha silenciosamente.
- **`fast-xml-parser` retorna objecto em vez de array para listas com 1 item.** Configura `isArray` para os tags relevantes (ver código acima).
- **Servidor lento.** Timeouts de 5s não chegam - usa 15-30s.
- **Datas em formatos inconsistentes.** `daily` tem `YYYYMMDD`; `inflation-yearly` tem `MM/YYYY`; `pib` tem `YYYYTN`. Normaliza tudo para ISO no parsing.
- **Não respeitar TTL.** Bater na API a cada request mata o servidor (e o teu UX).

---

## Estrutura recomendada para uma app que use BCM

```
your-app/
├── server/
│   ├── bcm-client.js       # fetchDaily, fetchWeekly, etc com cache
│   ├── bcm-parser.js       # XML -> JSON typed
│   ├── fixtures.js         # leitura dos sample-data/*.xml
│   └── routes.js
├── sample-data/            # copiar de projects/metical-lab/sample-data/ ou projects/sample-data/
│   └── *.xml
└── .env
    BCM_USE_FIXTURES=true   # durante dev
```
