# Skills

Estas pastas contêm conhecimento estruturado que o **agente executor** (Claude Code, opencode, codex) deve consultar **antes de implementar** uma funcionalidade relacionada.

## O que é uma skill

Uma skill é uma pasta com um `SKILL.md` que contém:

- **Regras hard-and-fast** no topo (o que NÃO fazer, o que SEMPRE fazer)
- **Exemplos copiáveis** de código que funciona
- **Esquemas de dados** (quando aplicável)
- **Armadilhas comuns** (gotchas) e como evitá-las

## Como usar (durante o spec)

Quando estás a escrever a spec do teu projecto **com o LLM-arquitecto** (Claude chat / ChatGPT / Gemini), faz isto:

1. Identifica que skills o teu projecto vai precisar (ex: vais usar OpenRouter? lê `llm-openrouter`. Vais usar XMLs do BCM? lê `bcm-data`)
2. Cola o conteúdo do `SKILL.md` relevante na conversa
3. Pede ao LLM-arquitecto para **incluir os passos detalhados nas specs** do teu projecto, referenciando a skill

Exemplo de pergunta ao LLM-arquitecto:

> Estou a escrever DESIGN-SPECS.md e PLAN.md para uma app que usa OpenRouter como LLM brain. Aqui está o SKILL.md do OpenRouter [colar conteúdo]. Por favor incorpora os passos relevantes nas minhas specs (autenticação, modelo padrão, tratamento de erros).

## Como usar (durante a implementação)

O agente executor recebe a pasta inteira (`AGENTS.md` lista as skills relevantes). Antes de implementar uma feature, lê o `SKILL.md` da skill associada.

Em `AGENTS.md`, podes escrever:

> Antes de implementar a integração com LLM, lê `skills/llm-openrouter/SKILL.md` na íntegra. As regras "hard-and-fast" são absolutas.

## Skills disponíveis

### LLM Brain
- `llm-openrouter/` - Como chamar a API do OpenRouter (modelo padrão: `mistralai/ministral-14b-2512`)
- `llm-llamacpp/` - Endpoint local OpenAI-compatible do workshop
- `llm-prompting-patterns/` - JSON outputs, system prompts, structured generation

### Dados & Padrões Moçambicanos
- `bcm-data/` - Todos os XMLs do Banco de Moçambique (câmbios, inflação, juros, PIB)
- `nuit-validation/` - Formato e validação do NUIT
- `irps-calculator/` - Escalões e cálculo do IRPS
- `mzn-formatting/` - Formatação de números, datas e locale pt-MZ

### Workflow
- `spec-kit-pattern/` - Como replicar este pacote para um novo projecto
- `metical-design-system/` - Tokens e padrões visuais do Metical (reutilizáveis)

## Adicionar a tua própria skill

Se descobrires um padrão útil durante o teu projecto, cria `skills/<nome>/SKILL.md` na tua pasta de projecto e partilha com a comunidade. As melhores skills:

- São curtas (80-200 linhas)
- Começam com 3-5 regras hard-and-fast
- Incluem pelo menos 2 exemplos copiáveis
- Listam pelo menos 1 armadilha comum
