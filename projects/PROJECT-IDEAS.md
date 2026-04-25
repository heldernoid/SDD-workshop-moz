# 40 Ideias de Projectos para Praticar Spec-Driven Development

Cada ideia é construível numa tarde com Claude Code, cabe em **menos de $2 USD** de tokens em modelos baratos (Haiku, Gemini Flash, DeepSeek), e tem uma **fonte de dados concreta** identificada - porque sem dados reais um projecto fica em demo de brinquedo.

**Como ler cada ideia:**

- **Dificuldade** ★ (1-2h, junior) · ★★ (meio dia, intermédio) · ★★★ (uma tarde inteira, mais ambicioso)
- **Dados:** API real / ficheiro local / scraping / sintético (gerado) / input do utilizador
- **Stack sugerida:** apenas indicativa - escolhe a que conheces melhor

A regra de ouro: se a ideia não tiver dados claros, não a construas. Pára e define os dados primeiro.

---

## Categoria 1 - Dados Públicos do Banco de Moçambique

> O BCM publica vários endpoints com dados macroeconómicos abertos ao público. Antes de construir, lê a página de Informação Legal do BCM (https://www.bancomoc.mz/pt/informacao-legal/), atribui sempre a fonte na tua app, faz cache local em vez de pedidos repetidos, e respeita um intervalo razoável entre chamadas. Não é correr atrás dos dados: é construir tendo o BCM como parceiro implícito.

### 1. Dashboard Macroeconómico de Moçambique ★★★

Página única que mostra os indicadores-chave da economia moçambicana num só ecrã: taxa de câmbio MZN/USD, taxa Mimo, taxa de inflação homóloga, e variação trimestral do PIB. Cada indicador tem um pequeno gráfico sparkline e um badge de variação. **Dados:** XMLs do BCM (`exchangerates`, `interest-rates`, `inflation-rates/yearly`, `pib`) - todos ficheiros pequenos, parseáveis com `fast-xml-parser`. **Stack:** React + Vite, Express proxy (mesmo padrão do `metical-lab`).

### 2. Calculadora de Empréstimo Bancário com Prime Rate Real ★★

Utilizador insere valor do empréstimo, prazo em meses, e spread do banco; aplicação calcula prestação mensal usando a Prime Rate actual do BCM como base. Mostra tabela de amortização e total de juros pagos. **Dados:** `interest-rates.xml` do BCM (Prime Rate 15.5% actualmente). **Stack:** HTML + JS puro, ou React simples.

### 3. Comparador de Inflação Mensal vs Homóloga ★★

Gráfico que sobrepõe a inflação mensal (variação de mês para mês) com a inflação homóloga (12 meses) num único eixo temporal. Permite ver quando os dois divergem - ferramenta útil para jornalistas económicos e estudantes de economia. **Dados:** `inflation-monthly.xml` e `inflation-yearly.xml` do BCM. **Stack:** Vanilla JS + Chart.js, ou React + Recharts.

### 4. Alerta de Variação Cambial ★★

Página que mostra a taxa MZN/USD actual e calcula a variação % em relação à semana anterior. Quando passa um limiar configurável (ex: 1%), mostra um banner de destaque. **Dados:** `exchangerates-weekly.xml` do BCM. **Stack:** React + cron leve no backend (ou só client-side comparando snapshots).

### 5. Visualizador de Recessão Trimestral ★★

App focada apenas no PIB trimestral, mostrando barras coloridas: verde para crescimento positivo, vermelho para negativo. Destaca visualmente períodos de recessão técnica (2 trimestres negativos consecutivos). **Dados:** `pib.xml` do BCM (dados desde 2010 ou similar). **Stack:** React + SVG nativo (não precisa de biblioteca de gráficos).

### 6. Conversor MZN com Histórico de 1 Ano ★★★

Variante do `metical-lab` mas com vista de 1 ano de histórico em gráfico de área para a moeda escolhida. **Dados:** o BCM só dá 24 dias no `monthly`; truque didáctico - começam por carregar o XML mensal e completam com dados sintéticos para 1 ano, depois discutem em sessão como agendariam um cron job para acumular histórico real. **Stack:** React + Express + ficheiro JSON como "base de dados".

---

## Categoria 2 - Ferramentas Pessoais e Pequenos Negócios

> Tudo o que um freelancer, estudante, ou pequeno comerciante moçambicano usaria semanalmente.

### 7. Gerador de Facturas com NUIT e IVA ★★★

Formulário que recolhe dados do cliente (nome, NUIT formatado `XXX XXX XXX`), itens com descrição e valor, calcula IVA a 17% (taxa moçambicana), e exporta PDF. Guarda histórico em `localStorage`. **Dados:** input do utilizador + biblioteca `pdfmake` ou `jspdf` para o output. **Stack:** React + jspdf + tailwind.

### 8. Validador e Formatador de NUIT ★

Pequena ferramenta que valida o NUIT moçambicano (9 dígitos, com regra de checksum oficial), formata como `XXX XXX XXX`, e identifica se é pessoa singular ou colectiva pelos dígitos iniciais. **Dados:** regras documentadas pela Autoridade Tributária; participantes precisam de pesquisar. **Stack:** HTML + JS puro, deploy em GitHub Pages.

### 9. Calculadora de IRPS Moçambicano ★★★

Utilizador insere salário bruto mensal; aplicação calcula IRPS por escalão (0%, 10%, 15%, 20%, 25%, 32%) conforme tabela oficial moçambicana, segurança social (3% do empregado), e mostra salário líquido. **Dados:** tabelas do IRPS publicadas no Diário da República - participantes têm de pesquisar e codificar como constantes. **Stack:** HTML + JS, ou React.

### 10. Tracker de Despesas em Meticais ★★

App de finanças pessoais simples: utilizador adiciona despesas com categoria (alimentação, transporte, renda, lazer), data, e valor em MT; vê totais por mês e por categoria com gráfico de pizza. **Dados:** `localStorage` para persistência local; sem backend. **Stack:** React + Recharts.

### 11. Conversor de Combustível: Litros para Meticais ★

Calcula o custo total de uma viagem dado a distância em km, consumo do carro em L/100km, e preço actual do combustível por litro. Inclui presets para gasolina e gasóleo com preços recentes em Moz. **Dados:** preços recentes do combustível (input manual ou hard-coded com data). **Stack:** HTML + JS, mobile-first.

### 12. Gestor de Xitique Digital ★★★

Aplicação para grupos de xitique (poupança rotativa moçambicana): regista membros, valor da contribuição, calendário de quem recebe quando, e acompanha pagamentos feitos vs pendentes. **Dados:** input + `localStorage` ou backend simples com SQLite. **Stack:** React + Express + SQLite, ou só client-side.

### 13. Tracker de Horas Freelancer ★★

Cronómetro com botão "Iniciar/Parar"; cada sessão regista cliente, projecto, descrição, e duração. Calcula faturação total à taxa horária definida pelo utilizador. **Dados:** `localStorage`. **Stack:** React + dayjs.

### 14. Calculadora de Taxa Cambial Justa ★★

Quando alguém te paga em USD ou EUR via PayPal/Wise/Remessa, mostra quanto deverias receber em MT usando a taxa de venda real do BCM, e compara com o que cada plataforma oferece (taxas hard-coded ou estimadas). **Dados:** `exchangerates.xml` do BCM + spreads conhecidos das plataformas. **Stack:** React + Express proxy.

### 15. Conversor de Unidades para Construção Civil ★

Em obras moçambicanas mistura-se m, m², m³, sacos de cimento, latas de tinta, etc. App converte entre estas unidades dadas dimensões. **Dados:** constantes (1 saco = 50kg, etc); participantes pesquisam. **Stack:** HTML + JS.

---

## Categoria 3 - Educação e Línguas

### 16. Quiz de História de Moçambique ★★

Banco de 50 perguntas de escolha múltipla sobre datas-chave (independência, guerra civil, Acordos de Roma, presidências), com pontuação e explicações. **Dados:** ficheiro JSON criado pelos participantes com pesquisa. **Stack:** React, ou HTML + JS.

### 17. Flashcards Português ↔ Changana ★★★

App de flashcards com algoritmo simples de spaced repetition (intervalo cresce conforme acertos). Inclui 100-200 palavras frequentes em Changana com tradução PT. **Dados:** ficheiro JSON; participantes podem usar Changana.net como referência ou um pequeno dataset Wikipedia. **Stack:** React + `localStorage`.

### 18. Tabuada Interactiva para Crianças ★

Aplicação infantil que pede multiplicações aleatórias (2x3, 7x8...) com feedback imediato visual e sonoro. Modo competição: quantas acertas em 1 minuto? **Dados:** geração aleatória. **Stack:** HTML + CSS animations + JS.

### 19. Conversor de Notas SNE ↔ ECTS ★

Para estudantes moçambicanos que vão fazer Erasmus ou candidaturas internacionais: converte notas do Sistema Nacional de Educação (0-20) para ECTS (A-F) e GPA americano (0-4.0). **Dados:** tabelas de equivalência publicadas pelo MCTESTP/universidades. **Stack:** HTML + JS.

### 20. Calculadora de Média Ponderada Universitária ★★

Estudante introduz disciplinas (nome, créditos, nota), calcula média ponderada por créditos e simula impacto de notas futuras ("se tirar 16 a Cálculo, fico com média X"). **Dados:** input. **Stack:** React + `localStorage`.

### 21. Treinador de Vocabulário Inglês para Estudantes Moz ★★

App focada em vocabulário académico que aparece em provas de inglês (TOEFL, IELTS) e no acesso ao ensino superior. Mostra palavra, exemplo, e pede tradução. **Dados:** lista JSON de 200-500 palavras com nível (B1, B2, C1) - participantes podem pedir ao Claude para gerar. **Stack:** React + spaced repetition simples.

### 22. Glossário de Termos Económicos em PT-MZ ★★

Site procurável com termos económicos explicados em português moçambicano simples - para estudantes universitários. Cada termo tem definição, exemplo prático aplicado a Moz, e termo equivalente em inglês. **Dados:** JSON criado em colaboração com Claude (~50 termos). **Stack:** Vite + React + Fuse.js para procura.

### 23. Quiz de Geografia Moçambicana ★

Mostra mapa de Moz e pede para identificar provincias, capitais, rios principais, etc. **Dados:** SVG do mapa de Moz (open source) + lista de províncias e capitais. **Stack:** SVG interactivo + JS, sem framework.

---

## Categoria 4 - Negócios e Freelancing

### 24. Templates de Contratos para Freelancers Moz ★★

Site com 5-10 templates editáveis (prestação de serviços, NDA, parceria) adaptados ao contexto legal moçambicano. Utilizador preenche um formulário e descarrega contrato em PDF/Word. **Dados:** templates redigidos pelos participantes (com base em modelos públicos). **Stack:** React + docx.js ou jspdf.

### 25. Calculadora de Pricing para Freelancers ★★

Utilizador insere despesas mensais (renda, internet, saúde), horas que quer trabalhar por mês, e margem de lucro desejada. App calcula taxa horária mínima sustentável em MZN e USD. **Dados:** input + taxa de câmbio do BCM. **Stack:** React + Express proxy do BCM.

### 26. Gestor Simples de Stock para Pequeno Comércio ★★★

App para uma loja pequena: lista de produtos com nome, quantidade, preço de custo, preço de venda, alerta visual quando stock baixo. Histórico de movimentos (entrada/saída). **Dados:** SQLite ou `localStorage`. **Stack:** React + better-sqlite3 (Node), ou só client-side.

### 27. Comparador de Preços de Internet em Moz ★★

Tabela comparativa dos planos da Tmcel, Vodacom e Movitel: GB incluídos, preço, validade, custo por GB. Mostra "melhor para ti" com base em quantos GB precisas/mês. **Dados:** tabelas hard-coded com data, ou scraping leve aos sites. **Stack:** HTML + JS puro.

### 28. Gerador de Recibos de Renda ★

Para senhorios pequenos: formulário com nome do inquilino, mês, valor, e gera PDF com recibo formal. **Dados:** input. **Stack:** HTML + jspdf.

### 29. Tracker de Comissões de Mukheristas ★★

App para vendedores ambulantes/mukheristas: regista venda (item, valor, comissão %), calcula totais diários e mensais. **Dados:** input + `localStorage`. **Stack:** React mobile-first.

### 30. Simulador de Investimento em Obrigações do Tesouro Moz ★★★

Utilizador escolhe obrigação (12, 24, 36 meses), valor a investir, taxa de juro (cap. presets das taxas do tesouro moz recentes); app projecta retorno bruto, IRPS sobre os juros, e líquido final. **Dados:** taxas do tesouro publicadas pelo BCM/MEF - participantes pesquisam. **Stack:** React + cálculo composto.

---

## Categoria 5 - Comunidade e Localização

### 31. Directório de Restaurantes em Maputo ★★

Lista filtrável por bairro (Polana, Sommerschield, Baixa, etc) e tipo de cozinha (moçambicana, indiana, portuguesa). Cada restaurante: nome, descrição, gama de preços, foto. **Dados:** ficheiro JSON com 30-50 entradas curadas pelos participantes (ou geradas com Claude e validadas). **Stack:** React + Tailwind.

### 32. Calculadora de Distâncias entre Cidades Moz ★★

Selecciona cidade de origem e destino; mostra distância rodoviária, tempo estimado de carro, e custo de combustível. **Dados:** matriz de distâncias entre 20-30 cidades principais (JSON), construída a partir do Google Maps ou pesquisa. **Stack:** HTML + JS, ou React.

### 33. Calendário de Feriados Moçambicanos ★

Mostra todos os feriados oficiais do ano (Dia da Independência, Dia da Mulher Moçambicana, etc) com descrição histórica. Conta quantos dias faltam para o próximo. **Dados:** lista hard-coded das datas oficiais. **Stack:** HTML + JS, ou React.

### 34. Mapa de Rotas dos Chapas em Maputo ★★★

Mapa interactivo com rotas principais dos chapas (Magoanine-Baixa, Hulene-Praça, etc) com pontos-chave. **Dados:** dados manualmente curados em GeoJSON, ou fontes da APROCHIM se disponíveis. **Stack:** Leaflet + dados estáticos.

### 35. Tradutor de Saudações para Línguas Moz ★

Seleciona uma língua (Changana, Makua, Sena, Macua, Ndau) e mostra como dizer saudações comuns ("Bom dia", "Como estás?", "Obrigado", "Adeus"). Inclui transcrição fonética. **Dados:** JSON criado pelos participantes com referências bibliográficas. **Stack:** HTML + JS.

### 36. Conversor de Endereços Informais Maputo ★★

Em Maputo muitos endereços são "Av Eduardo Mondlane perto da Brioche", não números. App permite registar pontos de referência (mercado, esquina, etc) e cria descrição estruturada para partilhar. **Dados:** input + `localStorage`. **Stack:** React mobile-first.

---

## Categoria 6 - Ferramentas para Programadores

### 37. Boilerplate Django/FastAPI para Projectos Moz ★★★

Template gerado por CLI: pergunta nome do projecto, e cria pasta com Django/FastAPI configurado com timezone Africa/Maputo, locale `pt_MZ`, suporte para MZN como moeda padrão, validador de NUIT, e migração inicial com modelo `User` extendido com NUIT/NUIB. **Dados:** estrutura escrita pelos participantes. **Stack:** Cookiecutter + scripts Python.

### 38. Visualizador de Logs de Apps Mobile em Português ★★

Ferramenta dev: cola logs do logcat (Android), aplica regex para extrair timestamps, severidade, e mensagens; mostra tabela filtrável com cores por nível. Tradução automática de erros comuns para português. **Dados:** logs de exemplo + regex. **Stack:** React + dayjs.

### 39. Gerador de READMEs em Português ★★

CLI ou web app que faz perguntas (nome do projecto, descrição, stack, instalação, autor) e gera README.md profissional em português, com badges, índice, e secções padrão. **Dados:** input + templates. **Stack:** Node CLI, ou React.

### 40. Monitor de Velocidade de Internet com Histórico ★★★

Roda testes de velocidade periodicamente (ping, download, upload) e guarda histórico. Mostra gráfico das últimas 24h e calcula uptime/médias por operadora. **Dados:** medições reais (web app pode usar `navigator.connection` ou fetch a um endpoint conhecido); CLI pode usar `speedtest-cli`. **Stack:** Node + sqlite + dashboard React.

---

## Notas Finais

**Para o teu workshop de 60 minutos**, recomenda-se que escolhas dos seguintes para a demo ao vivo (todas dentro do orçamento e com dados imediatamente disponíveis):

- Ideia 1 (Dashboard Macro) - usa todos os XMLs já uploadados; fica visualmente impressionante
- Ideia 4 (Alerta Cambial) - versão simples e rápida do `metical-lab`
- Ideia 7 (Facturas com NUIT) - utilidade imediata para qualquer freelancer
- Ideia 10 (Tracker de Despesas) - sem dependências externas, 100% client-side

**Para o desafio semanal pós-workshop**, recomendar aos participantes que escolham qualquer das 40, mas que entreguem **primeiro o spec-kit** (DESIGN-SPECS.md + AGENTS.md + design-mocks/) **antes** de começar a implementar. O objectivo é treinar o reflexo de especificar primeiro.

**Quando uma ideia parecer trivial**, lembra os participantes que o desafio não é o código - é a especificação. Um conversor de combustível trivial torna-se interessante quando a spec inclui: estados de erro, validação de input, suporte mobile-first, persistência de presets, e exportação de viagens.
