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

Escolhe **um** dos três. Todos são gratuitos para usar (pagas só pelos tokens dos modelos).

### Opção A: Claude Code (recomendado)

```bash
npm install -g @anthropic-ai/claude-code
```

Verifica:

```bash
claude --version
```

Documentação: https://docs.claude.com/en/docs/claude-code/overview

### Opção B: opencode

Agente open-source, suporta múltiplos providers.

```bash
npm install -g opencode-ai
```

Documentação: https://opencode.ai

### Opção C: codex

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

**Claude Code** usa Anthropic API por defeito. Para usar o endpoint local, podes precisar de configurar uma variável de ambiente que aponte para o endpoint OpenAI-compatible. Verifica a documentação actual.

**opencode** suporta endpoints OpenAI-compatible directamente:

```bash
opencode auth login
# Selecciona "Custom OpenAI" e cola a URL
```

**codex** suporta `--baseURL`:

```bash
codex --baseURL https://random-words.trycloudflare.com/v1 --apiKey dummy
```

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

**Claude Code:** suporta OpenAI-compatible providers. Configurar via `ANTHROPIC_BASE_URL` ou flags equivalentes.

**opencode:**

```bash
opencode auth login
# Selecciona "OpenRouter" e cola a key
```

**codex:**

```bash
codex --baseURL https://openrouter.ai/api/v1 --apiKey $OPENROUTER_API_KEY --model mistralai/ministral-14b-2512
```

### Cenário C: Anthropic API directa (Claude Code only)

Se queres usar Claude Code com a API directa da Anthropic:

1. Cria conta em https://console.anthropic.com
2. Adiciona crédito (~5 USD chega)
3. Gera key em https://console.anthropic.com/settings/keys
4. Configura:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

Modelos recomendados (baratos): Claude Haiku 4.5.

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
# Claude Code
claude

# ou opencode
opencode

# ou codex
codex
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
| Claude Haiku 4.5 (Anthropic) | < 0.50 USD |
| Gemini Flash (via OpenRouter) | < 0.30 USD |
| LlamaCPP local (durante workshop) | grátis |

Os 2 USD que mencionamos por participante cobrem largamente uma sessão completa, com margem para erros e iteração.

## Resolução de problemas

### "command not found: claude" / "opencode" / "codex"

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
