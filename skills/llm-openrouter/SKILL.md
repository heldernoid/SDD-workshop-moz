# Skill: LLM via OpenRouter

Como chamar OpenRouter como "brain" da tua aplicação. OpenRouter é um agregador que dá acesso a centenas de modelos via uma única API compatível com OpenAI.

---

## Hard-and-fast rules

1. **Modelo padrão para o workshop:** `mistralai/ministral-14b-2512`. Usa este - é barato, rápido, e o orçamento foi calculado com ele em mente. Não troques sem pedir antes.
2. **Nunca exponhas a API key no frontend.** A chamada à OpenRouter sai SEMPRE do backend. Se a app for puramente client-side (HTML estático), usa o endpoint LlamaCPP local em vez disto (ver `skills/llm-llamacpp/`).
3. **Define `max_tokens` em todas as chamadas.** Sem isto, um modelo pode gerar tokens infinitos e gastar o budget. Para a maioria dos casos: 512 é mais que suficiente.
4. **Trata 429 (rate limit) e 402 (sem crédito) como casos esperados** com mensagens amigáveis ao utilizador, não como crashes.
5. **Para outputs estruturados, pede JSON em vez de XML/markdown.** Ver `skills/llm-prompting-patterns/`.

---

## Endpoint base

```
https://openrouter.ai/api/v1/chat/completions
```

Compatível com OpenAI Chat Completions. Qualquer SDK OpenAI funciona se mudares apenas a `baseURL`.

## Headers obrigatórios

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
HTTP-Referer: <URL do teu projecto> (opcional, ajuda nas estatísticas)
X-Title: <Nome da tua app> (opcional)
```

---

## Exemplo: Node.js (fetch nativo)

```javascript
async function askLLM(prompt) {
  const res = await fetch('https://openrouter.ai/api/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.OPENROUTER_API_KEY}`,
      'Content-Type': 'application/json',
      'HTTP-Referer': 'https://metical-workshop.local',
      'X-Title': 'Metical Workshop',
    },
    body: JSON.stringify({
      model: 'mistralai/ministral-14b-2512',
      messages: [
        { role: 'system', content: 'Responde sempre em português de Moçambique.' },
        { role: 'user', content: prompt },
      ],
      max_tokens: 512,
      temperature: 0.7,
    }),
  });

  if (!res.ok) {
    const errorText = await res.text();
    throw new Error(`OpenRouter ${res.status}: ${errorText}`);
  }

  const data = await res.json();
  return data.choices[0].message.content;
}
```

## Exemplo: Python (`requests`)

```python
import os
import requests

def ask_llm(prompt: str) -> str:
    response = requests.post(
        "https://openrouter.ai/api/v1/chat/completions",
        headers={
            "Authorization": f"Bearer {os.environ['OPENROUTER_API_KEY']}",
            "Content-Type": "application/json",
            "HTTP-Referer": "https://metical-workshop.local",
            "X-Title": "Metical Workshop",
        },
        json={
            "model": "mistralai/ministral-14b-2512",
            "messages": [
                {"role": "system", "content": "Responde em português de Moçambique."},
                {"role": "user", "content": prompt},
            ],
            "max_tokens": 512,
            "temperature": 0.7,
        },
        timeout=30,
    )
    response.raise_for_status()
    return response.json()["choices"][0]["message"]["content"]
```

## Exemplo: SDK OpenAI (Node)

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENROUTER_API_KEY,
  baseURL: 'https://openrouter.ai/api/v1',
  defaultHeaders: {
    'HTTP-Referer': 'https://metical-workshop.local',
    'X-Title': 'Metical Workshop',
  },
});

const completion = await openai.chat.completions.create({
  model: 'mistralai/ministral-14b-2512',
  messages: [{ role: 'user', content: 'Olá' }],
  max_tokens: 512,
});
```

---

## Configuração no projecto

`.env` (NUNCA commitar):

```
OPENROUTER_API_KEY=sk-or-v1-...
OPENROUTER_MODEL=mistralai/ministral-14b-2512
```

`.env.example` (commitar):

```
OPENROUTER_API_KEY=
OPENROUTER_MODEL=mistralai/ministral-14b-2512
```

`.gitignore` deve incluir `.env`.

---

## Tratamento de erros

| Status | Significado | Acção |
|--------|-------------|-------|
| 401 | API key inválida | Mostrar mensagem clara: "Configuração inválida - contactar admin" |
| 402 | Sem crédito na conta | Mostrar: "Serviço temporariamente indisponível" |
| 429 | Rate limit | Esperar 1-2s e tentar 1x; depois mostrar "Estamos a receber muitos pedidos" |
| 500 | Erro do provider | Tentar novamente 1x; depois falhar |
| Timeout | Network | Configurar timeout de 30s; ao falhar, sugerir tentar novamente |

```javascript
async function askLLMSafe(prompt) {
  try {
    return await askLLM(prompt);
  } catch (err) {
    if (err.message.includes('429')) {
      await new Promise(r => setTimeout(r, 1500));
      return askLLM(prompt);  // 1 retry only
    }
    if (err.message.includes('402')) {
      throw new Error('Serviço temporariamente indisponível.');
    }
    throw err;
  }
}
```

---

## Custos esperados (modelo recomendado)

`mistralai/ministral-14b-2512`:
- Input: ~$0.10 por 1M tokens
- Output: ~$0.10 por 1M tokens

Para uma sessão típica de workshop (50 conversas curtas), o custo total é tipicamente **< $0.10**.

Verifica preços actuais em https://openrouter.ai/models antes de assumir.

---

## Armadilhas comuns

- **Não verificar `data.choices[0]` antes de aceder.** Se o modelo der erro de moderação, `choices` pode estar vazio. Verifica.
- **Esquecer o header `HTTP-Referer`.** Sem ele, OpenRouter pode aplicar limites mais restritos.
- **Usar streaming sem necessidade.** Para respostas curtas (<2s) é mais simples ficar com a resposta completa.
- **Hard-coding do modelo.** Mantém o nome do modelo em `.env` ou config - facilita trocar depois.
- **Não logar tokens consumidos.** A resposta inclui `usage.total_tokens` - guarda em log para acompanhar custos.

---

## Quando NÃO usar OpenRouter

- App puramente client-side sem backend → usa `skills/llm-llamacpp/` (endpoint local do workshop)
- Latência crítica (<500ms) → usa LlamaCPP local
- Workload muito grande → considera Anthropic/OpenAI directos para volume
