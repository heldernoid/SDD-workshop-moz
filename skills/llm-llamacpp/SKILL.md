# Skill: LLM via LlamaCPP local (workshop endpoint)

Como chamar o endpoint LlamaCPP que o organizador do workshop disponibiliza durante a sessão. É um servidor único, gratuito durante o workshop, com **um modelo apenas** carregado em memória.

---

## Hard-and-fast rules

1. **Endpoint partilhado entre participantes.** Não martelar com pedidos paralelos - o servidor não tem RAM para isso. Faz pedidos sequenciais.
2. **Não há fallback automático.** Se o endpoint estiver down, a app deve mostrar mensagem clara - não tentar OpenRouter sem o utilizador autorizar.
3. **Limite máximo de tokens recomendado:** `max_tokens: 256`. Acima disto a resposta atrasa demasiado.
4. **Apenas um modelo carregado.** Não passes `model` no pedido (ou passa o que o organizador indicou) - o servidor ignora-o de qualquer forma.
5. **Endpoint OpenAI-compatible.** Usa o caminho `/v1/chat/completions`. Não inventes endpoints `/api/chat` etc.

---

## URL do endpoint

O organizador partilha a URL no início da sessão. Tipicamente algo como:

```
https://workshop-llm.<organizador>.cloudflareaccess.com/v1
```

ou via Cloudflare tunnel:

```
https://<random>.trycloudflare.com/v1
```

Configura como variável de ambiente:

```
WORKSHOP_LLM_BASE_URL=https://...trycloudflare.com/v1
```

---

## Exemplo: Node.js (fetch nativo)

```javascript
async function askLocalLLM(prompt) {
  const baseUrl = process.env.WORKSHOP_LLM_BASE_URL;
  if (!baseUrl) throw new Error('WORKSHOP_LLM_BASE_URL não definido');

  const res = await fetch(`${baseUrl}/chat/completions`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      messages: [
        { role: 'system', content: 'Responde em português de Moçambique.' },
        { role: 'user', content: prompt },
      ],
      max_tokens: 256,
      temperature: 0.7,
      stream: false,
    }),
  });

  if (!res.ok) {
    throw new Error(`LLM workshop ${res.status}: ${await res.text()}`);
  }

  const data = await res.json();
  return data.choices[0].message.content;
}
```

## Exemplo: Python

```python
import os
import requests

def ask_local_llm(prompt: str) -> str:
    base_url = os.environ.get("WORKSHOP_LLM_BASE_URL")
    if not base_url:
        raise RuntimeError("WORKSHOP_LLM_BASE_URL não definido")

    response = requests.post(
        f"{base_url}/chat/completions",
        json={
            "messages": [
                {"role": "system", "content": "Responde em português de Moçambique."},
                {"role": "user", "content": prompt},
            ],
            "max_tokens": 256,
            "temperature": 0.7,
            "stream": False,
        },
        timeout=60,
    )
    response.raise_for_status()
    return response.json()["choices"][0]["message"]["content"]
```

## Exemplo: chamada do frontend (sem backend)

Para projectos client-side onde o backend é inviável, podes chamar directamente do browser. CORS está activado no endpoint do workshop.

```javascript
// Em produção (fora do workshop) NÃO faças isto - expor URL é OK aqui porque
// o endpoint é público e descartável após a sessão
const LLM_URL = 'https://....trycloudflare.com/v1/chat/completions';

async function askLLM(prompt) {
  const res = await fetch(LLM_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 256,
    }),
  });
  const data = await res.json();
  return data.choices[0].message.content;
}
```

---

## Diferenças vs OpenRouter

| Aspecto | LlamaCPP local | OpenRouter |
|---------|----------------|------------|
| Custo | Grátis (durante workshop) | Pago por token |
| Modelo | Um único, fixo | 100+ disponíveis |
| Latência | ~1-3s típico | 0.5-2s típico |
| Concorrência | Sequencial recomendada | Paralela OK |
| Disponibilidade | Apenas durante workshop | 24/7 |
| Autenticação | Sem key (URL é o segredo) | API key obrigatória |
| Pós-workshop | Endpoint desaparece | Continua a funcionar |

**Recomendação:** durante o workshop, começa com LlamaCPP. Para o teu repositório final (após o workshop), substitui por OpenRouter para que outros possam usar.

---

## Padrão para suportar ambos

Recomendado: estruturar o código para alternar entre os dois através de uma variável de ambiente.

```javascript
// lib/llm.js
const useLocal = process.env.LLM_PROVIDER === 'local';

const config = useLocal
  ? {
      url: `${process.env.WORKSHOP_LLM_BASE_URL}/chat/completions`,
      headers: { 'Content-Type': 'application/json' },
      model: undefined,
    }
  : {
      url: 'https://openrouter.ai/api/v1/chat/completions',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.OPENROUTER_API_KEY}`,
      },
      model: 'mistralai/ministral-14b-2512',
    };

export async function askLLM(prompt, options = {}) {
  const body = {
    messages: [{ role: 'user', content: prompt }],
    max_tokens: options.maxTokens || 256,
    temperature: options.temperature ?? 0.7,
    ...(config.model && { model: config.model }),
  };

  const res = await fetch(config.url, {
    method: 'POST',
    headers: config.headers,
    body: JSON.stringify(body),
  });

  if (!res.ok) throw new Error(`LLM ${res.status}: ${await res.text()}`);
  const data = await res.json();
  return data.choices[0].message.content;
}
```

`.env` durante o workshop:
```
LLM_PROVIDER=local
WORKSHOP_LLM_BASE_URL=https://....trycloudflare.com/v1
```

`.env` para o repositório público:
```
LLM_PROVIDER=openrouter
OPENROUTER_API_KEY=sk-or-...
```

---

## Armadilhas comuns

- **Esquecer que CORS é o motivo de não funcionar.** Se for um browser → endpoint directo, certifica-te que o endpoint do workshop tem CORS aberto (o organizador confirma).
- **Não tratar timeouts.** O modelo pode demorar até 10s - define timeouts realistas.
- **Pedir pedidos paralelos.** Se a tua app tem N utilizadores em simultâneo, fila os pedidos no backend ou aceita que vão ser lentos.
- **Assumir que o endpoint estará disponível depois.** Após o workshop, o tunnel cai. O teu repo deve usar OpenRouter por defeito.
- **Passar `model` num pedido a um servidor com modelo único.** Não causa erro, mas não tem efeito - não te baseies nessa "config".

---

## Como testar se o endpoint está vivo

```bash
curl https://...trycloudflare.com/v1/models
```

Deve retornar JSON com a lista de modelos disponíveis (provavelmente apenas um).

```bash
curl -X POST https://...trycloudflare.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"olá"}],"max_tokens":50}'
```

Resposta esperada em ~2s com texto em `choices[0].message.content`.
