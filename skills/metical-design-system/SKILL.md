# Skill: Metical Design System

Sistema de design reutilizável criado para o `metical-lab/`. Pode ser aplicado em qualquer projecto com público moçambicano que pretenda parecer profissional, calmo, e confiável (apps financeiras, fiscais, governamentais, educacionais).

---

## Hard-and-fast rules

1. **Toda renderização de números usa fonte monoespaçada** (JetBrains Mono). Sem excepções, mesmo em espaços apertados.
2. **Apenas UMA cor de marca** (Savanna Gold `#d4a017`). Verde-teal e vermelho são SEMÂNTICOS (positivo/negativo), não decorativos.
3. **Texto principal é `#1c1917`** (warm stone), NUNCA preto puro `#000`.
4. **Cards têm 16px radius e shadow de duas camadas** (ring + ambient blur). Sem cantos rectos.
5. **Background é off-white quente `#fafaf7`** (papel), não branco puro.

---

## Quando usar este sistema

**Adequado:**
- Apps fiscais e financeiras (calculadoras de IRPS, conversores)
- Painéis de dados governamentais ou bancários
- Ferramentas de produtividade para freelancers
- Gestão de pequenos negócios
- Aplicações educacionais formais

**Não adequado (precisa de outro sistema):**
- Jogos
- Redes sociais
- Apps de estilo de vida focadas em jovens
- Lojas online/e-commerce com forte pressão para conversão

Para esses casos, cria um sistema próprio. Mas mantém o princípio: **toda renderização de números usa mono**.

---

## Tokens completos (CSS)

```css
:root {
  /* Brand */
  --brand: #d4a017;          /* Savanna Gold - principal */
  --brand-dark: #a67c00;     /* hover/pressed */
  --brand-light: #fef3c7;    /* tint suave */
  --brand-muted: #fdebb0;    /* fundos de gráfico */

  /* Semantic */
  --positive: #0f766e;       /* Dusk Teal - variação positiva */
  --positive-bg: #ccfbf1;    /* badge fundo */
  --negative: #c13515;       /* Earth Red - variação negativa */
  --negative-bg: #fee4d6;
  --neutral: #78716c;        /* sem variação */

  /* Texto */
  --text: #1c1917;           /* warm stone, principal */
  --text-strong: #44403c;    /* ênfase */
  --text-secondary: #78716c; /* labels */
  --text-muted: #a8a29e;     /* timestamps */
  --text-disabled: #d6d3d1;

  /* Surface */
  --bg-page: #fafaf7;        /* papel */
  --bg-card: #ffffff;
  --bg-subtle: #f5f5f4;
  --bg-hover: #f5f5f4;

  /* Borders */
  --border: #e7e5e4;
  --border-strong: #d6d3d1;
  --border-focus: #d4a017;

  /* Shadows */
  --shadow-card: 0 0 0 1px rgba(28, 25, 23, 0.04), 0 2px 8px rgba(28, 25, 23, 0.06);
  --shadow-hover: 0 0 0 1px rgba(28, 25, 23, 0.06), 0 4px 16px rgba(28, 25, 23, 0.08);
  --ring-focus: 0 0 0 3px rgba(212, 160, 23, 0.2);

  /* Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;
  --radius-pill: 999px;

  /* Fonts */
  --font-ui: Inter, -apple-system, system-ui, Roboto, sans-serif;
  --font-mono: "JetBrains Mono", Menlo, Consolas, monospace;
}
```

---

## Fontes

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap">
```

Usa Inter para UI e JetBrains Mono para números. Não substituas por outras famílias sem motivo forte (ambas são gratuitas, web-safe, e têm bom suporte a caracteres portugueses).

---

## Hierarquia tipográfica

| Role | Família | Tamanho | Peso | Letter-spacing |
|------|---------|---------|------|----------------|
| Page Title (H1) | Inter | 28px | 700 | -0.02em |
| Section Heading (H2) | Inter | 20px | 600 | -0.01em |
| Card Title | Inter | 16px | 600 | normal |
| Body | Inter | 15px | 400 | normal |
| Small/Caption | Inter | 13px | 400 | normal |
| Label uppercase | Inter | 12px | 500 | 0.02em |
| Big Number (hero) | JetBrains Mono | 40px | 600 | -0.01em |
| Rate Number | JetBrains Mono | 18px | 500 | normal |
| Small Mono | JetBrains Mono | 13px | 500 | normal |

---

## Componentes-chave (snippets)

### Button primary

```css
.btn-primary {
  background: var(--brand);
  color: var(--text);
  padding: 10px 20px;
  border-radius: var(--radius-md);
  font: 600 15px var(--font-ui);
}
.btn-primary:hover {
  background: var(--brand-dark);
  color: #fff;
  box-shadow: var(--shadow-hover);
}
```

### Card

```css
.card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 24px;
  box-shadow: var(--shadow-card);
}
```

### Change badge (positive/negative)

```css
.change {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: var(--radius-pill);
  font: 500 12px var(--font-mono);
}
.change-positive { background: var(--positive-bg); color: var(--positive); }
.change-positive::before { content: '▲'; font-size: 10px; }
.change-negative { background: var(--negative-bg); color: var(--negative); }
.change-negative::before { content: '▼'; font-size: 10px; }
```

### Big number (hero)

```css
.big-number {
  font: 600 40px/1.1 var(--font-mono);
  letter-spacing: -0.01em;
  color: var(--text);
}
```

### Mono utility

```css
.mono {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums;  /* dígitos alinhados */
}
```

---

## Princípios espaciais

- Container max-width: **960px** centrado (não dashboard wide)
- Padding horizontal mínimo: **24px** desktop, **16px** mobile
- Espaçamento vertical entre secções: **24-40px**
- Dentro de cards: **20-24px** de padding
- Toque mobile mínimo: **44x44px**

---

## Filosofia visual em 6 princípios

1. **Calmo, não agitado.** O utilizador vem consultar, não vem ser estimulado. Sem animações desnecessárias, sem notificações push, sem nudges.
2. **Confiável, não comercial.** Parece-se mais com uma plataforma do BCM do que com uma fintech sexy. Isto é deliberado.
3. **Os números são os heróis.** Mono, alinhados, generosos em tamanho. Tudo o resto serve para enquadrá-los.
4. **Cor é semântica.** Gold é a marca; teal é positivo; red é negativo. Mais nada. Resistir à tentação de adicionar uma quarta cor.
5. **Espaço é gratuito, mas precioso.** Padding generoso. Não enchas as telas. O cérebro humano lê melhor com folga.
6. **Mobile-first sem ser hostile.** As versões mobile devem ser igualmente respeitáveis - não versões "esmagadas".

---

## Adaptação para outros domínios

Se o teu projecto é **moçambicano mas não financeiro**, podes adaptar:

| Mantém | Pode mudar |
|--------|------------|
| Tipografia (Inter + JetBrains Mono) | Cor de marca (em vez de gold) |
| Background warm `#fafaf7` | - |
| Texto `#1c1917` | - |
| Card radius 16px e shadow warm | - |
| Princípios de espaço | - |
| Mono para números | - |
| Locale moz e formatação | - |

Trocas a cor de marca para algo apropriado (verde para sustentabilidade, azul para tech, roxo para criativo). Mas não mudes mais do que isso ou perdes a coerência.

---

## Cheat-sheet para o agente executor

Quando o agente vai implementar componentes, mostra-lhe estes atalhos:

```
Quero um cartão padrão com sombra e radius:
→ background var(--bg-card), border 1px var(--border), radius 16px, padding 24px, shadow var(--shadow-card)

Quero um botão primário:
→ background var(--brand), text var(--text), padding 10px 20px, radius 8px, font Inter 15 weight 600

Quero mostrar uma variação positiva:
→ pill: bg var(--positive-bg), text var(--positive), font JetBrains Mono 12 weight 500, prefixo ▲

Quero mostrar um número grande:
→ font JetBrains Mono 40 weight 600, line-height 1.1, letter-spacing -0.01em, color var(--text)
```

---

## Exemplo de uso noutro projecto

Calculadora de IRPS (caso uses os tokens do Metical):

```jsx
<div className="card">
  <h2 style={{font: '600 20px Inter', letterSpacing: '-0.01em'}}>
    Cálculo do salário líquido
  </h2>
  <div style={{display: 'grid', gap: 12, marginTop: 20}}>
    <div className="row">
      <span>Salário bruto</span>
      <span className="mono" style={{fontSize: 18, fontWeight: 500}}>25 000.00 MT</span>
    </div>
    <div className="row" style={{color: 'var(--negative)'}}>
      <span>Segurança Social (3%)</span>
      <span className="mono">−750.00 MT</span>
    </div>
    <div className="row" style={{color: 'var(--negative)'}}>
      <span>IRPS</span>
      <span className="mono">−875.00 MT</span>
    </div>
    <hr style={{border: 'none', borderTop: '1px solid var(--border)'}}/>
    <div className="row" style={{fontWeight: 600}}>
      <span>Líquido</span>
      <span className="mono" style={{fontSize: 24, fontWeight: 600}}>23 375.00 MT</span>
    </div>
  </div>
</div>
```

Resultado: parece feito por alguém que conhece design e Moçambique.

---

## Armadilhas comuns

- **Misturar com Bootstrap/Tailwind sem cuidado.** As variáveis CSS dão-te o sistema; não adicionar uma framework por cima excepto se realmente reutilizas componentes complexos.
- **Substituir Inter por "fontes giras".** Tipografia é onde quase todos perdem credibilidade. Inter é boring por design - boring é confiável.
- **Adicionar um quarto tom de marca "porque ficaria bem aqui".** Não. Coerência > conforto pontual.
- **Sombras pesadas (>0.1 opacity).** O sistema usa sombras leves e quentes. Sombras pretas pesadas pertencem a outro estilo.
- **Hardcode de hex em componentes.** Sempre via variáveis CSS. Facilita modo escuro futuramente.
- **Esquecer `tabular-nums`** em colunas de números. Sem isso, mesmo a fonte mono pode desalinhar (em alguns weights).
