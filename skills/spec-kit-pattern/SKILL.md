# Skill: Spec-Kit Pattern - Como Replicar para Novo Projecto

O `metical-lab/` é um exemplo do padrão. Esta skill ensina como aplicar o mesmo padrão a qualquer projecto novo. É a essência do "desenvolvimento spec-driven".

---

## Hard-and-fast rules

1. **NUNCA começar a programar antes do spec-kit estar completo.** Se Claude Code começa sem specs, o resultado vai precisar de ser deitado fora.
2. **A pasta de specs é a "fonte da verdade".** Quando Claude e o utilizador divergem, **AMBOS recorrem aos ficheiros `.md`**, não às memórias da conversa.
3. **Cada ficheiro tem um propósito ÚNICO.** Não misturar arquitectura no `DESIGN.md` nem desenho no `ARCHITECTURE.md`. Disciplina.
4. **`AGENTS.md` é a primeira coisa que o agente lê.** Por isso é curto e taxativo.
5. **Mocks HTML são autoritários para layout/comportamento; specs `.md` são autoritários para tokens/estrutura.** Se houver conflito, o `.md` vence.

---

## Estrutura mínima do spec-kit

```
my-project/
├── README.md                # Visão geral, "porquê", como usar
├── AGENTS.md                # Regras curtas para o agente executor
├── DESIGN.md                # Sistema de design (tokens, tipografia)
├── DESIGN-SPECS.md          # Specs de cada vista + referência aos mocks
├── ARCHITECTURE.md          # Stack, estrutura de pastas, modelo de dados, APIs
├── PLAN.md                  # Plano de implementação (Parts + Checkpoints)
├── design-mocks/            # HTML mocks self-contained
│   ├── 00_main.html
│   └── ...
├── sample-data/             # Dados de exemplo para dev offline
│   └── ...
└── skills/                  # Skills relevantes para o projecto
    ├── llm-openrouter/SKILL.md
    └── ...
```

Para projectos pequenos (< 5 vistas), podes fundir `DESIGN.md` e `DESIGN-SPECS.md` num só. Para projectos triviais (1 vista), até podes saltar mocks. Mas o `AGENTS.md`, `ARCHITECTURE.md`, e `PLAN.md` são sempre obrigatórios.

---

## Workflow para criar o spec-kit (com LLM-arquitecto)

Cada participante segue este fluxo de 6 passos:

### 1. Brainstorm da ideia (5-10 min)

Conversa com Claude (chat) ou ChatGPT em formato livre. Descreve a ideia em 1-2 parágrafos. Pede ao LLM-arquitecto para fazer perguntas-chave:

- Quem é o utilizador?
- Qual é o caso de uso principal? Que problema resolve?
- Que dados são necessários? De onde vêm?
- Que constraints (tempo, orçamento, plataforma)?
- Existem skills relevantes? (Cola SKILL.md das skills aplicáveis.)

### 2. Esboço de design (10 min)

Pede ao LLM-arquitecto:

> Vou fazer uma app `<descrição>`. Inspira-te em `<marca/produto conhecido>`. Sugere uma direcção visual (paleta, tipografia, voz), e uma lista das vistas principais. Não escrevas código ainda - só direcção.

Iteração até ficar com algo claro.

### 3. Geração de DESIGN.md e DESIGN-SPECS.md (15 min)

> Cria DESIGN.md no formato Google Stitch / Airbnb (vê o exemplo do metical-lab). Inclui paleta, tipografia, componentes, e princípios de espaçamento. Depois cria DESIGN-SPECS.md com uma secção por vista.

Revê. Adiciona/remove vistas. Pede para gerar mocks HTML self-contained navegáveis entre si.

### 4. ARCHITECTURE.md (15 min)

> Com base no DESIGN.md, escreve ARCHITECTURE.md: stack, estrutura de pastas, modelo de dados, contratos de API. Justifica cada decisão. Aplica restrições: deve correr localmente, deve usar `<data source>`, deve caber em $2 USD de tokens.

Verifica que a stack faz sentido para o teu nível e tempo disponível.

### 5. PLAN.md (15 min)

> Com base em ARCHITECTURE.md e DESIGN-SPECS.md, escreve PLAN.md com Parts ordenadas, cada uma com Goal, Tasks numeradas, Success Criteria, e Checkpoint. O agente deve PARAR em cada Checkpoint.

Cada Part deve resultar em algo testável. Não Part chamada "fazer tudo".

### 6. AGENTS.md (5 min)

> Escreve AGENTS.md curto: que ficheiros ler primeiro (em ordem), que skills relevantes existem em `skills/`, regras de estilo, package manager a usar, dependências permitidas, protocolo de Checkpoint.

---

## Convenções dos ficheiros

### `README.md`

- Para humanos
- Visão geral em 2-3 parágrafos
- Estrutura de ficheiros
- Como usar (instruções de instalação/dev)
- Atribuição (fontes de dados, inspirações)

### `AGENTS.md`

- Para o agente executor
- Curto: 100-200 linhas máximo
- Lista de leitura ordenada
- Skills relevantes a consultar
- Regras hard-and-fast (estilo, dependências, package manager)
- Protocolo de Checkpoint

### `DESIGN.md`

- Sistema de design completo
- Paleta com nomes semânticos (não apenas hex)
- Tipografia (família, escala, princípios)
- Componentes principais com specs
- Do's e Don'ts visuais
- Cheat-sheet para o agente (CSS variables, prompts curtos)

### `DESIGN-SPECS.md`

- Uma secção por vista
- Layout descrito (top-to-bottom)
- Comportamentos
- Estados (loading, empty, error)
- Referência aos mocks (`design-mocks/00_xxx.html`)
- Nota sobre quem ganha em conflito (mock vs spec)

### `ARCHITECTURE.md`

- Stack (com justificação curta)
- Estrutura de pastas (árvore)
- Modelo de dados (tipos partilhados)
- Contratos de API (endpoints, requests, responses)
- Integrações externas
- Estratégia de testes
- Out-of-scope explícito

### `PLAN.md`

- Lista ordenada de Parts
- Cada Part: Goal · Tasks · Success Criteria · Checkpoint
- "Baseline rules" no topo (não-negociáveis)
- "Out of scope" no fim

---

## Template de Part (no PLAN.md)

```markdown
## Part N: [Nome curto e claro]

### Goal
1-2 frases. O que está feito no fim desta part.

### Tasks
1. Tarefa concreta verificável
2. ...

### Success Criteria
- Comando concreto que passa (`pnpm test`, etc)
- Output verificável
- Critério visual se aplicável

### Checkpoint
Stop. Wait for user to say "avança".
```

---

## Template de AGENTS.md mínimo

```markdown
# [Project] - Agentic Build Instructions

## Read first

Read in order before writing any code:

1. `ARCHITECTURE.md`
2. `PLAN.md`
3. `DESIGN.md`
4. `DESIGN-SPECS.md`
5. `design-mocks/*.html`

## Skills

Consult before implementing related features:

- `skills/llm-openrouter/SKILL.md` - if app uses LLM
- `skills/bcm-data/SKILL.md` - if app uses BCM data
- `skills/mzn-formatting/SKILL.md` - for any MT/date display

## Rules

- Package manager: pnpm only
- TypeScript strict mode, no `any`
- All UI text in Portuguese (Moçambique)
- No external UI libraries
- No comments restating what code does

## Checkpoint protocol

At each Part Checkpoint:
1. Run all Success Criteria checks
2. Summarize in 3-5 bullets
3. Stop and wait for "avança"

## When in doubt

- Ambiguous spec? Ask before guessing.
- Mock vs spec conflict? Spec wins, flag it.
- Need a dependency not on the allow-list? Ask.
```

---

## Lista de verificação antes de mandar para Claude Code

Antes de fazer `claude code` na pasta, verifica:

- [ ] Todos os ficheiros `.md` existem e estão preenchidos
- [ ] Mocks HTML existem em `design-mocks/` e abrem self-contained no browser
- [ ] `sample-data/` (se aplicável) existe com dados realistas
- [ ] Dependências previstas estão na allow-list em `AGENTS.md`
- [ ] `PLAN.md` tem pelo menos 3 Parts com Checkpoints
- [ ] `AGENTS.md` lista as skills relevantes em `skills/`
- [ ] `.gitignore` tem `node_modules`, `dist`, `.env`
- [ ] `.env.example` existe com todas as variáveis (sem segredos)

---

## Quando NÃO usar este padrão

- Scripts utilitários de < 50 linhas → escreve directo, não precisas de spec-kit
- Protótipos de 10 minutos para validar uma ideia → usa um único ficheiro `.md` curto
- Edição cirúrgica num projecto existente → o spec-kit do projecto principal já cobre

O spec-kit completo justifica-se quando: (a) vais codar com agente, (b) o projecto tem mais de 1 vista ou 1 endpoint, ou (c) outras pessoas vão contribuir.

---

## Armadilhas comuns

- **Skipping AGENTS.md.** Sem ele, o agente não sabe a ordem de leitura nem as regras. Resultado: caos.
- **PLAN.md genérico.** "Implementar tudo" → o agente adivinha a ordem. Sê específico.
- **Mocks que não abrem standalone.** Self-contained obrigatório (CSS inline, JS inline, sem dependências externas além de fontes Google).
- **Misturar inglês e português nos `.md`.** Escolhe um. Recomendado: português para `README.md` e `PROJECT-IDEAS.md`; inglês ou português para os specs técnicos (qualquer um, mas consistente).
- **Tentar fazer tudo em uma sessão sem Checkpoints.** O agente acumula erros. Checkpoints frequentes salvam.
- **Não copiar as skills relevantes para `skills/`.** Sem isso, o agente "improvisa" detalhes obscuros.

---

## Exemplo "vivo"

`projects/metical-lab/` é a referência canónica. Estuda a sua estrutura antes de criar a tua. Em cada projecto novo, começa por:

```bash
cp -r projects/metical-lab/ projects/my-new-project/
cd projects/my-new-project/
# editar os .md, substituir os mocks, etc.
```

Reaproveitas a estrutura, substituis o conteúdo. Ganhas horas.
