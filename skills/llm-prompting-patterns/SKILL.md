# Skill: LLM Prompting Patterns

Padrões reutilizáveis para extrair outputs fiáveis de LLMs em apps reais. Aplicam-se igualmente ao OpenRouter, LlamaCPP local, ou qualquer outro provider.

---

## Hard-and-fast rules

1. **System prompt é obrigatório em apps de produção.** Define identidade, formato de saída, e linguagem. Não confies no input do utilizador para definir o comportamento.
2. **Para outputs estruturados, pede JSON e parseia com `try/catch`.** Nunca confies que o LLM devolve JSON válido - valida.
3. **Define limite máximo de input do utilizador.** 2000 caracteres é razoável para a maioria dos casos. Mais que isso, fatia ou rejeita.
4. **Sanitiza inputs do utilizador antes de meter no prompt.** Não interpoles HTML, JS, ou tokens de formatação directamente.
5. **Temperature 0.0-0.3 para tarefas determinísticas** (extracção, classificação, validação). 0.6-0.9 para criatividade.

---

## Padrão 1: Output estruturado em JSON

### Prompt

```
Vais classificar despesas pessoais. Recebe uma descrição e devolve JSON com:
- "categoria": uma de ["alimentação", "transporte", "renda", "lazer", "outros"]
- "valor_estimado_mzn": número ou null se não conseguires inferir
- "confianca": "alta", "média", ou "baixa"

Devolve APENAS o objecto JSON, sem texto antes ou depois, sem markdown.

Descrição: "Comi um prego na Cervejaria Continental"
```

### Parsing seguro

```javascript
function parseJsonFromLLM(raw) {
  // Remove markdown fences se aparecerem
  const cleaned = raw.replace(/```json|```/g, '').trim();

  // Tenta extrair primeiro objecto/array JSON do texto
  const match = cleaned.match(/[\{\[][\s\S]*[\}\]]/);
  if (!match) throw new Error('Resposta não contém JSON');

  try {
    return JSON.parse(match[0]);
  } catch (err) {
    throw new Error(`JSON inválido: ${err.message}`);
  }
}

// Uso com retry
async function classifyExpense(description) {
  for (let attempt = 0; attempt < 2; attempt++) {
    const raw = await askLLM(buildClassifyPrompt(description));
    try {
      const parsed = parseJsonFromLLM(raw);
      // Validar shape
      if (!['alimentação', 'transporte', 'renda', 'lazer', 'outros'].includes(parsed.categoria)) {
        throw new Error('categoria inválida');
      }
      return parsed;
    } catch (err) {
      if (attempt === 1) throw err;
      // tenta novamente com prompt mais firme
    }
  }
}
```

---

## Padrão 2: System prompt para localização Moz

```javascript
const SYSTEM_PROMPT_PT_MZ = `És um assistente que comunica em português de Moçambique.

Regras de linguagem:
- Usa "estás" e não "tu estás" repetidamente
- Usa "machimbombo" para autocarro, "chapa" para minibus, "shoprite/chickens" como referências quotidianas quando apropriado
- Os preços são em meticais (MT). Formato: "1 500 MT" (espaço como separador de milhares)
- NUIT tem 9 dígitos e formata-se como "XXX XXX XXX"
- Datas no formato DD/MM/AAAA
- A capital é Maputo. Cidades principais: Beira, Nampula, Matola, Tete, Quelimane, Pemba

Responde de forma directa e útil. Sem preâmbulos longos.`;
```

---

## Padrão 3: Validação de output sensível

Quando o LLM gera conteúdo a mostrar ao utilizador (ex: README, factura, descrição de produto), passa por filtros:

```javascript
function validateLLMOutput(text) {
  const issues = [];

  // Comprimento razoável
  if (text.length > 5000) issues.push('output demasiado longo');
  if (text.length < 10) issues.push('output demasiado curto');

  // Não deve conter prompts internos vazados
  const leakPatterns = [/system prompt/i, /\bAPI key\b/i, /sk-or-/, /sk-ant-/];
  if (leakPatterns.some(p => p.test(text))) {
    issues.push('possível leak de informação interna');
  }

  // Não deve conter instruções para o utilizador clicar em links suspeitos
  if (/click here|clica aqui|http/i.test(text) && !text.includes('https://www.bancomoc.mz')) {
    issues.push('contém URL não autorizado');
  }

  return { valid: issues.length === 0, issues };
}
```

---

## Padrão 4: Reduzir tokens do input

Para apps que enviam contexto longo (ex: histórico de conversas), aplica estas regras antes de chamar o LLM:

```javascript
function trimContext(messages, maxTokens = 2000) {
  // Estimativa simples: 1 token ≈ 4 caracteres
  let totalChars = 0;
  const trimmed = [];

  // Mantém system prompt e mensagem mais recente; corta do meio
  const system = messages.find(m => m.role === 'system');
  if (system) {
    trimmed.push(system);
    totalChars += system.content.length;
  }

  // Adiciona mensagens recentes até atingir limite
  for (let i = messages.length - 1; i >= 0; i--) {
    if (messages[i].role === 'system') continue;
    if (totalChars + messages[i].content.length > maxTokens * 4) break;
    trimmed.unshift(messages[i]);
    totalChars += messages[i].content.length;
  }

  return trimmed;
}
```

---

## Padrão 5: Streaming UX (quando faz sentido)

Para outputs longos (>500 chars) que aparecem na UI, streaming melhora a experiência:

```javascript
async function streamLLM(prompt, onChunk) {
  const res = await fetch(`${baseUrl}/chat/completions`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 1024,
      stream: true,
    }),
  });

  const reader = res.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    // OpenAI/OpenRouter streaming format: lines starting with "data: "
    const lines = chunk.split('\n').filter(l => l.startsWith('data: '));
    for (const line of lines) {
      const data = line.slice(6);
      if (data === '[DONE]') return;
      try {
        const parsed = JSON.parse(data);
        const delta = parsed.choices[0]?.delta?.content;
        if (delta) onChunk(delta);
      } catch {}
    }
  }
}

// UI
let displayed = '';
await streamLLM('Conta-me uma história curta', chunk => {
  displayed += chunk;
  document.getElementById('output').textContent = displayed;
});
```

---

## Padrão 6: Função "few-shot" inline

Quando o LLM não acerta o formato, mostra-lhe 2-3 exemplos:

```javascript
const prompt = `Extrai o NUIT de cada texto. Formato: 9 dígitos seguidos.
Devolve apenas o NUIT, ou "N/D" se não houver.

Texto: "Empresa Lda, NUIT 123456789, sede em Maputo"
NUIT: 123456789

Texto: "Sociedade ABC SA contribuinte 987654321"
NUIT: 987654321

Texto: "Compre na nossa loja em Maputo"
NUIT: N/D

Texto: "${input}"
NUIT:`;
```

Este padrão funciona muito bem com modelos pequenos como o `ministral-14b-2512`.

---

## Armadilhas comuns

- **Confiar que JSON volta válido sem validar.** Sempre wrappa parsing em try/catch.
- **Concatenar input do utilizador directamente no prompt sem escapar.** Risco de prompt injection. Usa formato delimitado: `<input>...</input>`.
- **Pedir output muito longo a modelos pequenos.** O `ministral-14b-2512` perde coerência depois de ~600 tokens. Limita.
- **Não dar contexto sobre Moçambique ao LLM.** Se a app é para Moz, o system prompt deve dizer isso. Caso contrário ele assume Portugal/Brasil.
- **Esquecer-se de validar o output antes de mostrar/guardar.** O LLM pode gerar texto inseguro, ofensivo, ou simplesmente errado.

---

## Cheat sheet de temperaturas

| Tarefa | Temperature |
|--------|-------------|
| Validação de NUIT/email/etc | 0.0 |
| Extracção de campos | 0.0 - 0.2 |
| Classificação | 0.2 - 0.4 |
| Resposta a perguntas factuais | 0.3 - 0.5 |
| Geração de README/descrições | 0.5 - 0.7 |
| Conteúdo criativo | 0.7 - 0.9 |
| Brainstorming | 0.8 - 1.0 |
