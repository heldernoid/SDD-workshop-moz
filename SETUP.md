# SETUP

Como preparar o ambiente para usar este material. Dois cenários cobertos:

1. **Durante o workshop:** usas o endpoint LlamaCPP local que o organizador disponibiliza
2. **Depois do workshop:** usas a tua própria conta OpenRouter (custo baixo)

Tempo de setup: ~15 minutos.

## Pré-requisitos

- **Node.js 20+** ou **Python 3.10+** (depende do projecto que escolheres)
- **git** instalado
- Editor de código (VS Code recomendado)
- Conta GitHub

Verifica versões:

```bash
node --version    # v20+ recomendado
python3 --version # 3.10+ recomendado
git --version
```

## Passo 1: Clonar este repositório

```bash
git clone https://github.com/<user>/<repo>.git workshop
cd workshop
```

## Passo 2: Instalar um agente de coding

Escolhe **um** dos dois. Ambos são gratuitos para usar (pagas só pelos tokens dos modelos).

### Opção A: opencode

Agente open-source, suporta múltiplos providers via configuração JSON.

```bash
npm install -g opencode-ai
```

Documentação: https://opencode.ai

### Opção B: codex

Agente da OpenAI, em Rust, configurável via `~/.codex/config.toml`.

```bash
npm install -g @openai/codex
```

Documentação: https://github.com/openai/codex

## Passo 3: Configurar acesso a um LLM

### Cenário A: Endpoint LlamaCPP local (durante workshop)

O organizador partilha uma URL no início da sessão, tipicamente algo como:

```
https://random-words.trycloudflare.com/v1
```

Cria ficheiro `.env` na raiz do teu projecto:

```bash
LLM_PROVIDER=local
WORKSHOP_LLM_BASE_URL=https://random-words.trycloudflare.com/v1
```

**Importante:** este endpoint é partilhado entre participantes e tem RAM limitada. Faz pedidos sequenciais, não paralelos. Mais detalhes em `skills/llm-llamacpp/SKILL.md`.

Para configurar o agente a usar este endpoint, depende do agente:

**opencode** usa um ficheiro `opencode.json` para definir providers customizados. Cria-o na raiz do teu projecto (ou em `~/.config/opencode/opencode.json` para configuração global):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "llama.cpp": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "llama-server (local)",
      "options": {
        "baseURL": "http://127.0.0.1:8080/v1"
      },
      "models": {
        "qwen-coder": {
          "name": "Qwen3-Coder (local)",
          "limit": {
            "context": 256000,
            "output": 65536
          }
        }
      }
    }
  }
}
```

Notas:
- O nome do modelo (`qwen-coder`) tem de bater certo com o `--alias` que passaste ao `llama-server`
- O `limit.context` deve coincidir (ou ser inferior) ao `-c` do llama-server
- Lança com `opencode` e usa `/models` para escolher "Qwen3-Coder (local)" da lista

**codex** já não suporta as flags `--baseURL` / `--apiKey`. Configuras tudo via `~/.codex/config.toml` e profiles.

Primeiro, lança o `llama-server` com um alias para o modelo (deixa o nome curto e amigável):

```bash
llama-server -m Qwen3-Coder-Next-Q8_0-00001-of-00004.gguf \
  --alias qwen-coder \
  -c 256000 --host 0.0.0.0 --port 8080 \
  --temp 0.6 --top-p 0.95 --top-k 20 --min-p 0.00 \
  --kv-unified --cache-type-k q8_0 --cache-type-v q8_0 \
  --flash-attn on --fit on
```

Depois cria/edita `~/.codex/config.toml`:

```toml
[model_providers.local]
name = "llama-server"
base_url = "http://127.0.0.1:8080/v1"
wire_api = "responses"

[profiles.qwen-coder]
model = "qwen-coder"
model_provider = "local"
web_search = "disabled"
```

E lança com o profile:

```bash
codex -p qwen-coder
```

> **Nota sobre o aviso "Model metadata for `qwen-coder` not found":** o Codex avisa que não tem metadata oficial para modelos não-OpenAI e que isso "pode degradar performance". **Podes ignorar** este aviso. O agente funciona normalmente.

> **Importante:** Codex requer `wire_api = "responses"` (já não suporta `"chat"`). Para isto funcionar com modelos locais, o llama-server tem de ter o endpoint `/v1/responses` (versões recentes têm; se não funcionar, actualiza com `brew upgrade llama.cpp` ou recompila do master). Em algumas combinações de versões pode aparecer o erro `'type' of tool must be 'function'` — neste caso, faz update do llama.cpp.

### Cenário B: OpenRouter (depois do workshop, ou alternativa)

OpenRouter dá acesso a centenas de modelos via uma API única. O modelo recomendado para este material é `mistralai/ministral-14b-2512` (rápido e barato).

1. Cria conta em https://openrouter.ai
2. Adiciona ~5 USD de crédito (chega para muitas horas de uso)
3. Gera uma API key em https://openrouter.ai/keys
4. Adiciona ao `.env`:

```bash
LLM_PROVIDER=openrouter
OPENROUTER_API_KEY=sk-or-v1-...
OPENROUTER_MODEL=mistralai/ministral-14b-2512
```

Para configurar o agente:

**opencode:**

```bash
opencode auth login
# Selecciona "OpenRouter" e cola a key
```

**codex:** adiciona um provider e profile no `~/.codex/config.toml`:

```toml
[model_providers.openrouter]
name = "OpenRouter"
base_url = "https://openrouter.ai/api/v1"
wire_api = "responses"
env_key = "OPENROUTER_API_KEY"

[profiles.openrouter]
model = "mistralai/ministral-14b-2512"
model_provider = "openrouter"
```

E lança com:

```bash
export OPENROUTER_API_KEY=sk-or-v1-...
codex -p openrouter
```

> **Atenção:** nem todos os modelos do OpenRouter suportam Responses API. Se aparecer erro 400 ou de schema, troca o modelo ou usa opencode em vez do Codex para esse provider.

## Passo 4: Verificar que está tudo a funcionar

Cria um pequeno teste:

```bash
mkdir hello-spec
cd hello-spec
cat > AGENTS.md <<EOF
Cria um ficheiro hello.txt que diga "Olá Moçambique" e adiciona a data actual.
EOF
```

Lança o agente:

```bash
# opencode
opencode

# ou codex (com o profile que configuraste no Passo 3)
codex -p qwen-coder
```

Pede ao agente: "Lê AGENTS.md e executa." Deve criar `hello.txt`.

Se isto funciona, estás pronto.

## Passo 5: (Opcional) Configurar Git

Se vais publicar o teu projecto no GitHub:

```bash
git config --global user.name "O Teu Nome"
git config --global user.email "teu@email.com"
```

Em cada projecto:

```bash
git init
git add .
git commit -m "spec-kit inicial"
```

## Tabela de custos esperados

Para um projecto típico do `PROJECT-IDEAS.md` (1-2 vistas, 1 fonte de dados):

| Modelo | Custo aproximado |
|---|---|
| `mistralai/ministral-14b-2512` (OpenRouter) | < 0.20 USD |
| Gemini Flash (via OpenRouter) | < 0.30 USD |
| LlamaCPP local (durante workshop) | grátis |

Os 2 USD que mencionamos por participante cobrem largamente uma sessão completa, com margem para erros e iteração.

## Resolução de problemas

### "command not found: opencode" / "codex"

O `npm install -g` foi para um path que não está em `$PATH`. Corrige com:

```bash
echo 'export PATH="$(npm config get prefix)/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### "401 Unauthorized" ao chamar OpenRouter

A API key está mal copiada ou foi revogada. Gera nova em https://openrouter.ai/keys.

### "402 Payment Required" no OpenRouter

Sem crédito. Adiciona em https://openrouter.ai/credits.

### Endpoint LlamaCPP do workshop dá timeout

Provável causa: muitos pedidos paralelos a saturar a RAM. Espera 30 segundos e tenta novamente, fazendo um pedido de cada vez.

### Agente fica preso ou gera código que não compila

Não é a tua culpa. Os modelos baratos falham por vezes. Soluções:

1. Lê o erro com calma
2. Corrige o spec-kit (`AGENTS.md`, `PLAN.md`) para ser mais explícito
3. Usa `git checkout` para voltar a um estado bom e tentar de novo
4. Se o agente apaga o teu progresso por engano, `git reflog` salva-te

## Ferramentas adicionais úteis

Não obrigatórias, mas tornam a experiência melhor:

- **direnv:** carrega variáveis de ambiente automaticamente por pasta. Instala com `brew install direnv` ou `apt install direnv`
- **gh** (GitHub CLI): facilita criar repositórios. `brew install gh` ou via [cli.github.com](https://cli.github.com)
- **httpie** ou **curl**: para testar APIs manualmente
- **jq:** para inspeccionar respostas JSON. `apt install jq` ou `brew install jq`

## Próximos passos

1. Lê `README.md` para perceberes a filosofia spec-driven
2. Abre `projects/metical-lab/` e explora os ficheiros `.md`
3. Lê `skills/spec-kit-pattern/SKILL.md` para o fluxo completo
4. Escolhe uma ideia em `projects/PROJECT-IDEAS.md` e começa
