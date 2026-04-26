# Licenças

Este repositório usa **licenças mistas** porque combina código original (escrito por nós) com dados e mapas de terceiros que têm requisitos legais próprios.

Lê esta página antes de redistribuir ou modificar qualquer parte do projecto.

---

## Resumo rápido

| Parte do projecto | Licença | O que podes fazer |
|---|---|---|
| Código (`.js`, `.ts`, `.py`, `.html`, `.css`) | **MIT** | Qualquer uso, comercial inclusive |
| Specs (`AGENTS.md`, `ARCHITECTURE.md`, `DESIGN.md`, `PLAN.md`, `PROJECT-IDEAS.md`, `skills/`) | **MIT** | Qualquer uso, comercial inclusive |
| `sample-data/quiz-mocambique.json` | **MIT** | Qualquer uso |
| `sample-data/glossario-economico-mz.json` | **MIT** | Qualquer uso |
| `sample-data/academic-vocabulary-en.json` | **MIT** | Qualquer uso |
| `sample-data/changana-dataset.json` | **CC BY-NC-SA 4.0** | Uso não comercial, com atribuição, partilha igual |
| `sample-data/mocambique-provincias.svg` | **CC BY 4.0** | Qualquer uso, com atribuição |
| XMLs do BCM (`sample-data/*.xml`) | **Termos do BCM** (uso privado/interno; comercial requer autorização escrita) | Ver secção 4 |

---

## 1. MIT License (código + specs + maioria dos dados)

Aplica-se a tudo excepto o que está listado nas secções 2 e 3 abaixo.

```
MIT License

Copyright (c) 2026 Workshop Spec-Driven Development Moçambique

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 2. CC BY-NC-SA 4.0 (Changana dataset)

**Ficheiro:** `sample-data/changana-dataset.json`

Este dataset é distribuído sob **Creative Commons Atribuição-Não Comercial-Partilha Igual 4.0 Internacional** (CC BY-NC-SA 4.0).

Texto completo: https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.pt

**Em resumo, podes:**
- Usar livremente para fins educativos, pessoais e não-comerciais
- Adaptar, traduzir, ampliar
- Partilhar publicamente

**Tens de:**
- **Atribuir:** mencionar o repositório original e o autor
- **Não-comercial:** não vender o dataset nem produtos/serviços que dependam dele directamente como conteúdo principal
- **Partilha igual:** se ampliares ou modificares, distribui sob a mesma licença CC BY-NC-SA 4.0

**Porquê esta escolha:** o Changana é uma língua oral viva de Moçambique. Curar este dataset é trabalho de comunidade e o objectivo é circular livremente entre estudantes e curiosos. Uma licença não-comercial protege o material de ser empacotado e revendido em apps fechadas que não devolvem nada à comunidade.

**Atribuição sugerida:**
> Dataset Changana: Workshop Spec-Driven Development Moçambique (2026), CC BY-NC-SA 4.0. https://github.com/<utilizador>/<repo>

---

## 3. CC BY 4.0 (Mapa de Moçambique)

**Ficheiros:**
- `sample-data/mocambique-provincias.svg`

Distribuídos sob **Creative Commons Atribuição 4.0 Internacional** (CC BY 4.0).

Texto completo: https://creativecommons.org/licenses/by/4.0/legalcode.pt

**Podes:**
- Usar para qualquer fim, incluindo comercial
- Adaptar, modificar, distribuir

**Tens de:**
- **Atribuir** as fontes originais

**Atribuição obrigatória para o SVG e GeoJSON:**

> Fronteiras provinciais: geoBoundaries (Runfola et al. 2020), CC BY 4.0.
> https://www.geoboundaries.org/
>
> Maputo Cidade: simplemaps.com (CC BY 4.0).
> https://simplemaps.com/gis/country/mz

Esta atribuição já está embebida nas tags `<desc>` do próprio SVG. Se modificares o ficheiro, mantém estas referências.

**Citação académica para geoBoundaries:**
> Runfola, D. et al. (2020). geoBoundaries: A global database of political administrative boundaries. PLoS ONE 15(4): e0231866. https://doi.org/10.1371/journal.pone.0231866

---

## 4. Snapshots do Banco de Moçambique (BCM)

**Ficheiros:** `sample-data/daily.xml`, `weekly.xml`, `monthly.xml`, `inflation-monthly.xml`, `inflation-yearly.xml`, `interest-rates.xml`, `pib.xml`

**Endpoints originais:**
- https://www.bancomoc.mz/bmapi/exchangerates/
- https://www.bancomoc.mz/bmapi/exchangerates-weekly/
- https://www.bancomoc.mz/bmapi/exchangerates-monthly/
- https://www.bancomoc.mz/bmapi/inflation-rates/monthly
- https://www.bancomoc.mz/bmapi/inflation-rates/yearly
- https://www.bancomoc.mz/bmapi/interest-rates
- https://www.bancomoc.mz/bmapi/pib

### Termos de utilização do BCM (transcrição literal)

A página de Informação Legal do BCM (https://www.bancomoc.mz/pt/informacao-legal/) define os seguintes termos:

> "Todos os direitos de propriedade intelectual relativos ao Sítio e ao seu conteúdo são detidos pelo Banco de Moçambique, excepto nos casos em que se dispõe de forma contrária.
>
> O Banco de Moçambique autoriza a utilização dos conteúdos disponibilizados no Sítio para fins privados ou internos das entidades ou organizações, sendo vedada a utilização para fins comerciais.
>
> A utilização para fins distintos dos autorizados carece de solicitação do utilizador e de prévia autorização por escrito do Banco de Moçambique.
>
> Nos casos de utilização do conteúdo do Sítio, o utilizador deve citar o Banco de Moçambique como fonte e, nos casos de modificação do conteúdo, tal facto deve ser expressamente referido."

### O que isto significa para ti, na prática

**Podes usar livremente:**
- ✅ Em projectos pessoais, de aprendizagem, ou exercícios de portfólio
- ✅ Em projectos internos de uma empresa ou organização
- ✅ Em material educativo (este workshop é um exemplo)
- ✅ Em ferramentas que não comercializam o acesso aos dados

**Não podes, sem autorização escrita do BCM:**
- ❌ Vender uma app cujo valor central é redistribuir estes dados
- ❌ Cobrar pelo acesso aos dados em si
- ❌ Incorporá-los num produto comercial sem antes pedir permissão por escrito ao BCM

**Tens sempre de:**
- 📌 **Citar o BCM como fonte** em qualquer app, dashboard, ou material que use estes dados
- 📌 **Indicar modificações** se transformares os dados (agregações, médias, conversões)

### Atribuição sugerida

Em apps web, no rodapé ou ecrã "Sobre":

> Fonte dos dados: **Banco de Moçambique** (https://www.bancomoc.mz/)

Se modificares (ex: calcular médias móveis, agregar trimestres):

> Fonte dos dados: Banco de Moçambique. Dados originais transformados localmente para [médias móveis de 7 dias / agregação mensal / etc].

### Querer ir mais longe (uso comercial)

Se a tua app for evoluir para um produto comercial que dependa destes dados, escreve ao BCM e pede autorização. O contacto institucional está no site (https://www.bancomoc.mz/). Pedidos claros e bem fundamentados costumam ter resposta.

---

## 5. Quando estás em dúvida

Se uma parte do repositório não tem licença explícita, **assume MIT por omissão**, excepto para os ficheiros listados nas secções 2 e 3.

Se queres usar este material num contexto que não está claramente coberto, abre uma issue no repositório.
