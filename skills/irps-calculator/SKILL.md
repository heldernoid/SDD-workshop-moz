# Skill: Cálculo de IRPS Moçambicano

Como calcular IRPS (Imposto sobre o Rendimento das Pessoas Singulares) para apps de calculadora de salário, gestão de RH, e simuladores fiscais.

---

## Hard-and-fast rules

1. **AVISO LEGAL:** as tabelas evoluem com a Lei do Orçamento do Estado anual. **VERIFICA** no Diário da República mais recente antes de publicar uma calculadora pública. Esta skill apresenta a estrutura geral; valores concretos podem estar desactualizados.
2. **IRPS é calculado por escalões progressivos**, NÃO por uma única taxa marginal aplicada ao total. Erro comum.
3. **Aplica-se ao rendimento mensal tributável** (já após deduções obrigatórias).
4. **A Segurança Social do empregado (3%)** é deduzida do bruto antes de calcular IRPS. O empregador contribui adicionalmente 4% (não afecta o líquido do trabalhador).
5. **A app deve mostrar o cálculo passo-a-passo,** não apenas o valor final - ajuda o utilizador a confiar no resultado.

---

## Estrutura dos escalões (referência geral)

A tabela abaixo é a estrutura típica do IRPS mensal moçambicano. **Os valores limite e taxas exactos devem ser confirmados no DR mais recente.**

```javascript
// Estrutura referencial (verificar valores actuais no DR)
const IRPS_BRACKETS_MENSAL = [
  { from: 0,      to: 20250,  rate: 0.00, deduction: 0 },
  { from: 20251,  to: 20750,  rate: 0.10, deduction: 2025 },
  { from: 20751,  to: 20875,  rate: 0.15, deduction: 3062.50 },
  { from: 20876,  to: 21625,  rate: 0.20, deduction: 4106.25 },
  { from: 21626,  to: 32375,  rate: 0.25, deduction: 5187.50 },
  { from: 32376,  to: Infinity, rate: 0.32, deduction: 7453.75 },
];
```

> **Atenção:** os limites acima são puramente ilustrativos. Para uma calculadora real, **pesquisa a tabela actual** ou pede ao utilizador para introduzir as taxas/limites manualmente como configuração.

---

## Algoritmo (método "taxa × valor − dedução")

A fórmula oficial moçambicana é:
**IRPS = (rendimento_tributável × taxa_escalão) − dedução_escalão**

Onde a "dedução" é o valor pré-calculado pelo Ministério das Finanças que corresponde à acumulação progressiva dos escalões anteriores. Isto evita ter de somar manualmente os valores dos escalões inferiores.

```javascript
/**
 * Calcula IRPS mensal moçambicano.
 *
 * @param {number} salarioBruto - Salário bruto mensal em MT
 * @param {object[]} brackets - Tabela de escalões (verificar DR actual)
 * @returns {object} Detalhes do cálculo
 */
export function calcularIRPS(salarioBruto, brackets = IRPS_BRACKETS_MENSAL) {
  if (salarioBruto < 0) throw new Error('Salário não pode ser negativo');

  // 1. Segurança social do trabalhador (3% do bruto)
  const segSocialTrabalhador = salarioBruto * 0.03;

  // 2. Rendimento tributável
  const rendTributavel = salarioBruto - segSocialTrabalhador;

  // 3. Escalão aplicável
  const bracket = brackets.find(b => rendTributavel >= b.from && rendTributavel <= b.to);
  if (!bracket) throw new Error('Não foi possível determinar escalão');

  // 4. IRPS
  const irps = Math.max(0, rendTributavel * bracket.rate - bracket.deduction);

  // 5. Salário líquido
  const liquido = salarioBruto - segSocialTrabalhador - irps;

  return {
    salarioBruto,
    segSocialTrabalhador: round2(segSocialTrabalhador),
    rendTributavel: round2(rendTributavel),
    escalao: {
      from: bracket.from,
      to: bracket.to,
      taxa: bracket.rate,
      deducao: bracket.deduction,
    },
    irps: round2(irps),
    liquido: round2(liquido),
  };
}

function round2(n) { return Math.round(n * 100) / 100; }
```

---

## Versão Python

```python
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class Bracket:
    from_value: float
    to_value: float
    rate: float
    deduction: float

IRPS_BRACKETS_MENSAL: List[Bracket] = [
    Bracket(0, 20250, 0.00, 0),
    Bracket(20251, 20750, 0.10, 2025),
    Bracket(20751, 20875, 0.15, 3062.50),
    Bracket(20876, 21625, 0.20, 4106.25),
    Bracket(21626, 32375, 0.25, 5187.50),
    Bracket(32376, float('inf'), 0.32, 7453.75),
]

def calcular_irps(salario_bruto: float, brackets=IRPS_BRACKETS_MENSAL) -> Dict:
    if salario_bruto < 0:
        raise ValueError('Salário não pode ser negativo')

    seg_social = salario_bruto * 0.03
    rend_tributavel = salario_bruto - seg_social

    bracket = next((b for b in brackets
                    if b.from_value <= rend_tributavel <= b.to_value), None)
    if not bracket:
        raise ValueError('Não foi possível determinar escalão')

    irps = max(0, rend_tributavel * bracket.rate - bracket.deduction)
    liquido = salario_bruto - seg_social - irps

    return {
        'salario_bruto': round(salario_bruto, 2),
        'seg_social_trabalhador': round(seg_social, 2),
        'rend_tributavel': round(rend_tributavel, 2),
        'escalao': {
            'from': bracket.from_value,
            'to': bracket.to_value,
            'taxa': bracket.rate,
            'deducao': bracket.deduction,
        },
        'irps': round(irps, 2),
        'liquido': round(liquido, 2),
    }
```

---

## Padrão de UX para apresentar o cálculo

Mostra o cálculo decomposto, não apenas o valor final. Ganha confiança do utilizador:

```
Salário bruto:               25 000.00 MT

(−) Segurança social (3%):     -750.00 MT
                              ─────────
Rendimento tributável:       24 250.00 MT

Escalão aplicável: 21 626 - 32 375 MT @ 25%
Cálculo: 24 250.00 × 0.25 − 5 187.50

(−) IRPS:                     -875.00 MT
                              ─────────
Salário líquido:             23 375.00 MT
```

---

## Casos especiais a documentar (NÃO incluídos no MVP)

- **Subsídios de férias e Natal:** podem ter regime especial - consultar Código do IRPS
- **Dependentes:** descontos por filhos (verificar tabela actual)
- **Dedução de pensão de alimentos:** abate ao rendimento tributável
- **Trabalhadores não residentes:** taxa fixa diferente
- **Rendimentos de outras categorias** (B - empresariais, E - capitais, etc): este código só calcula categoria A (trabalho dependente)

Para uma calculadora "MVP de workshop", limita-te ao caso simples (trabalhador residente, categoria A, sem dependentes), e deixa um aviso visível no UI.

---

## Componente React (excerto)

```jsx
function IrpsCalculator() {
  const [bruto, setBruto] = useState('');

  const result = useMemo(() => {
    const num = Number(bruto.replace(/\s/g, ''));
    if (!num || num < 0) return null;
    try { return calcularIRPS(num); } catch { return null; }
  }, [bruto]);

  return (
    <div className="card">
      <label>Salário bruto mensal (MT)</label>
      <input
        type="text"
        inputMode="numeric"
        value={bruto}
        onChange={e => setBruto(e.target.value)}
        placeholder="25 000"
      />

      {result && (
        <table className="result-table mono">
          <tbody>
            <tr><td>Salário bruto</td><td>{fmt(result.salarioBruto)} MT</td></tr>
            <tr><td>− Segurança social (3%)</td><td>−{fmt(result.segSocialTrabalhador)} MT</td></tr>
            <tr><td>= Rendimento tributável</td><td>{fmt(result.rendTributavel)} MT</td></tr>
            <tr><td>Escalão</td><td>{(result.escalao.taxa * 100).toFixed(0)}%</td></tr>
            <tr><td>− IRPS</td><td>−{fmt(result.irps)} MT</td></tr>
            <tr className="highlight"><td>= Líquido</td><td>{fmt(result.liquido)} MT</td></tr>
          </tbody>
        </table>
      )}

      <p className="disclaimer">
        Cálculo baseado nas tabelas do IRPS publicadas no Diário da República.
        Verifica os valores actuais antes de tomar decisões financeiras.
      </p>
    </div>
  );
}

function fmt(n) {
  return n.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }).replace(/,/g, ' ');
}
```

---

## Armadilhas comuns

- **Aplicar a taxa do escalão a TODO o salário bruto** (sem deduzir SS, sem usar a fórmula com dedução). Errado por dezenas de pontos percentuais.
- **Esquecer que SS é 3% para o trabalhador e 4% para o empregador.** Total contribuído ao INSS é 7% mas só 3% sai do líquido do trabalhador.
- **Hardcoding dos escalões sem aviso.** As tabelas mudam. Coloca a tabela em ficheiro de config separado e mostra a "vigência" no UI.
- **Não validar input numérico negativo ou texto.** Crash silencioso ou números absurdos.
- **Mostrar apenas o líquido sem decompor.** Utilizadores não confiam num número que aparece sem explicação.
- **Confundir IRPS com IVA.** Coisa completamente diferente - IRPS é sobre rendimento, IVA é sobre consumo.

---

## Onde validar a tabela actual

- Diário da República, séries da Lei do Orçamento do Estado
- Site da Autoridade Tributária de Moçambique (AT)
- Tabelas publicadas pelo Ministério das Finanças
- Calculadoras de RH conhecidas (Primavera, etc) - comparação cruzada

Para uma calculadora pública, considera incluir uma nota "Última actualização da tabela: [data]" e a fonte.
