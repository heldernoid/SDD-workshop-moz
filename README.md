# Workshop de Spec-Driven Development 

Material aberto do workshop sobre desenvolvimento spec-driven, em português de Moçambique.
Construído para programadores e estudantes de IT moçambicanos que querem aprender a **especificar primeiro, programar depois**, usando LLMs como Claude Code, opencode ou codex.

## O que é spec-driven development

A maior parte das pessoas que usam IA para programar começa por escrever prompts soltos. O resultado é código que cresce sem direcção, com dívida técnica logo no primeiro dia.

Spec-driven development é o oposto: antes de escrever uma única linha de código, escreves uma **pasta de especificações** (`AGENTS.md`, `ARCHITECTURE.md`, `DESIGN.md`, `PLAN.md`, mocks HTML). Só depois passas essa pasta inteira ao agente executor (Claude Code, opencode, codex) e deixas trabalhar.

A diferença é a mesma que existe entre construir uma casa com plantas vs. construir uma casa "no improviso".

## O que está aqui

Este repositório é simultaneamente material de workshop e template reutilizável.

```
.
├── README.md            (este ficheiro)
├── SETUP.md             (como instalar tudo: agente, LlamaCPP, OpenRouter)
├── LICENSE              (licenças mistas - lê antes de redistribuir)
├── projects/
│   ├── PROJECT-IDEAS.md     (40 ideias de projectos para praticar)
│   ├── metical-lab/         (spec-kit de demonstração: conversor MZN)
│   │   ├── AGENTS.md, ARCHITECTURE.md, DESIGN.md, DESIGN-SPECS.md, PLAN.md
│   │   ├── design-mocks/    (8 mockups HTML self-contained navegáveis)
│   │   └── sample-data/     (XMLs do BCM usados por este projecto)
│   └── sample-data/         (datasets temáticos e mapa, partilhados)
└── skills/                  (9 skills reutilizáveis transversais)
```

### O projecto de demonstração: Metical

O coração deste repositório é uma especificação completa de um conversor de meticais (MZN) que usa dados reais do Banco de Moçambique. Não está implementado a propósito. É um spec-kit pronto para tu (ou um agente) construir do zero.

### As 40 ideias

`PROJECT-IDEAS.md` tem 40 projectos curtos categorizados, todos com:

- Descrição num parágrafo
- Fonte de dados concreta (API, ficheiro, sintético)
- Dificuldade indicativa (1 a 3 estrelas)
- Stack sugerida

Construíveis numa tarde, com orçamento abaixo de 2 USD em modelos baratos.

### As 9 skills

`skills/` contém ficheiros SKILL.md reutilizáveis:

**LLM brain:** OpenRouter, LlamaCPP local, prompting patterns
**Dados Moz:** BCM API, NUIT, IRPS, formatação MZN
**Workflow:** spec-kit pattern, Metical design system

Cada skill tem regras hard-and-fast, exemplos copiáveis em Node ou Python, e armadilhas comuns.

### Datasets em `sample-data/`

Para os participantes não terem de partir do zero quando escolherem um projecto do `PROJECT-IDEAS.md`, a pasta `sample-data/` inclui datasets curados prontos a usar:

**Dados públicos do BCM (XMLs):**
- `daily.xml`, `weekly.xml`, `monthly.xml` - taxas de câmbio MZN
- `inflation-monthly.xml`, `inflation-yearly.xml` - inflação
- `interest-rates.xml` - Mimo, FPC, FPD, Prime
- `pib.xml` - PIB trimestral

**Datasets temáticos (JSON):**
- `quiz-mocambique.json` - 50 perguntas de história, presidentes, símbolos, geografia
- `glossario-economico-mz.json` - 100 termos económicos PT-MZ com exemplos moçambicanos
- `academic-vocabulary-en.json` - 200 palavras de vocabulário académico EN para preparar TOEFL/IELTS
- `changana-dataset.json` - 237 entradas de Changana (saudações, números, verbos, provérbios)

**Geografia:**
- `mocambique-provincias.svg` - 11 províncias e capitais com IDs semânticos para quiz interactivo

Cada dataset tem licença própria documentada em `LICENSE`. Os XMLs do BCM e os datasets de geografia têm requisitos de atribuição.

## Como usar este material

### Como participante de workshop

Vens com uma ideia em mente. Segue o fluxo:

1. Lê `SETUP.md` e prepara o ambiente
2. Lê `PROJECT-IDEAS.md` e escolhe uma ideia (ou cria a tua)
3. Conversa com Claude (chat) ou ChatGPT, cola as skills relevantes da pasta `skills/`, e gera o teu próprio spec-kit usando `metical-lab` como modelo
4. Quando o spec-kit estiver completo, abre Claude Code (ou opencode/codex) na tua pasta e deixa o agente trabalhar
5. Em cada Checkpoint do PLAN.md, revês resultados e dás "go ahead"

### Como template para projectos próprios

```bash
git clone https://github.com/heldernoid/SDD-workshop-moz.git
cd SDD-workshop-moz
# Editar os .md e mocks para o teu novo projecto
```

A skill `spec-kit-pattern` explica o processo passo a passo.

### Como referência

Se já estás a trabalhar com Claude Code e queres só aprender o padrão de skills, abre `skills/` e explora.

## Audiência

Este material foi feito a pensar em programadores e estudantes em Moçambique:

- A linguagem é português de Moçambique
- Os exemplos usam dados reais moçambicanos (BCM)
- As convenções são moçambicanas (NUIT, IRPS, formato de números, locale)
- Os projectos resolvem problemas reais (xitique, conversão MZN, calculadora de salário)

Mas qualquer pessoa que queira aprender spec-driven development pode usar. Basta substituir os exemplos.

## Tecnologias e stack

Não há uma stack imposta. As skills cobrem Node.js e Python, mas o padrão é agnóstico. Os agentes suportados são:

- **Claude Code** (Anthropic)
- **opencode**
- **codex**

Qualquer agente que aceite uma pasta com instruções estruturadas funciona.

## Atribuição e fontes de dados

Este repositório usa três fontes externas com requisitos de atribuição:

**Banco de Moçambique** — XMLs em `sample-data/*.xml`. O BCM autoriza uso para fins privados ou internos, exige citação como fonte, e veda uso comercial sem autorização escrita prévia. Lê os termos completos em `LICENSE` (secção 4) ou directamente no [site oficial](https://www.bancomoc.mz/pt/informacao-legal/).

**geoBoundaries** (Runfola et al. 2020) — fronteiras provinciais em `mocambique-provincias.svg`. Licença CC BY 4.0. Atribuição obrigatória.

**simplemaps.com** — usado para extrair Maputo Cidade. Licença CC BY 4.0.

Sempre que mostrares estes dados na tua app (qualquer dashboard, mapa, gráfico), inclui linha de atribuição visível. Detalhes em `LICENSE`.

## Licenças

Este repositório usa **licenças mistas** porque combina código original (escrito por nós) com dados de terceiros que têm requisitos legais próprios. Os principais:

- **Código, specs, skills, JSONs criados por nós:** MIT
- **`changana-dataset.json`:** CC BY-NC-SA 4.0 (uso não comercial, partilha igual) - protege o trabalho linguístico de ser empacotado e revendido
- **Mapa:** CC BY 4.0 (atribuição obrigatória)
- **XMLs do BCM:** termos do BCM (uso privado/interno; comercial requer autorização escrita)

Lê `LICENSE` para detalhes completos antes de redistribuir.

## Contribuir

Pull requests bem-vindos, especialmente:

- Novas skills úteis para o contexto moçambicano
- Correcções a tabelas (IRPS, NUIT) à medida que evoluem
- Mais ideias de projectos com fontes de dados concretas
- Traduções de skills para outras línguas locais

## Comunidade

Se construíres algo a partir deste material, partilha. As melhores criações ficam destacadas no repositório.
