# Skill: NUIT - Número Único de Identificação Tributária (Moçambique)

Tudo o que precisas para validar e formatar NUIT moçambicano em apps.

---

## Hard-and-fast rules

1. **NUIT tem exactamente 9 dígitos.** Nem 8, nem 10. Sempre 9.
2. **NUIT NÃO tem letras nem caracteres especiais.** Apenas dígitos `[0-9]`.
3. **Formato visual padrão é `XXX XXX XXX`** (espaços) - usado em facturas, NÃO usar pontos ou traços.
4. **Persistir SEM formatação** (apenas os 9 dígitos) na base de dados. Formatar só na apresentação.
5. **Primeiro dígito indica tipo do contribuinte** (ver tabela abaixo) - útil para validação adicional, mas não obrigatório no MVP.

---

## Tipo de contribuinte por primeiro dígito

| Primeiro dígito | Tipo |
|-----------------|------|
| 1 | Pessoa singular nacional |
| 2 | Pessoa singular estrangeira |
| 3 | Pessoa singular nacional (continuação) |
| 4 | Pessoa colectiva - empresa privada |
| 5 | Pessoa colectiva - empresa pública / estatal |
| 6, 7 | Outras pessoas colectivas |
| 8, 9 | Reservados / casos especiais |

**Nota:** estas regras são as documentadas pela Autoridade Tributária mas podem evoluir. Para o workshop, foca apenas em validar o comprimento e que são todos dígitos. Validação por checksum NÃO é pública oficialmente.

---

## Funções de referência

### Validação básica

```javascript
/**
 * Valida formato de NUIT.
 * Aceita NUIT com ou sem espaços; ignora espaços e tabs.
 */
export function isValidNUIT(input) {
  if (typeof input !== 'string') return false;
  const cleaned = input.replace(/\s/g, '');
  if (cleaned.length !== 9) return false;
  if (!/^\d{9}$/.test(cleaned)) return false;
  return true;
}

/**
 * Normaliza para os 9 dígitos sem espaços (para guardar em DB).
 */
export function normalizeNUIT(input) {
  if (!isValidNUIT(input)) throw new Error('NUIT inválido');
  return input.replace(/\s/g, '');
}

/**
 * Formata para apresentação: "123 456 789".
 */
export function formatNUIT(input) {
  const clean = normalizeNUIT(input);
  return `${clean.slice(0, 3)} ${clean.slice(3, 6)} ${clean.slice(6, 9)}`;
}

/**
 * Identifica tipo (informativo, não bloqueante).
 */
export function nuitType(input) {
  const clean = normalizeNUIT(input);
  const first = clean[0];
  const map = {
    '1': 'pessoa singular nacional',
    '2': 'pessoa singular estrangeira',
    '3': 'pessoa singular nacional',
    '4': 'pessoa colectiva privada',
    '5': 'pessoa colectiva pública',
    '6': 'outra pessoa colectiva',
    '7': 'outra pessoa colectiva',
  };
  return map[first] || 'desconhecido';
}
```

### Versão Python

```python
import re

def is_valid_nuit(value: str) -> bool:
    if not isinstance(value, str):
        return False
    cleaned = re.sub(r'\s+', '', value)
    return len(cleaned) == 9 and cleaned.isdigit()

def normalize_nuit(value: str) -> str:
    if not is_valid_nuit(value):
        raise ValueError('NUIT inválido')
    return re.sub(r'\s+', '', value)

def format_nuit(value: str) -> str:
    clean = normalize_nuit(value)
    return f'{clean[0:3]} {clean[3:6]} {clean[6:9]}'

def nuit_type(value: str) -> str:
    clean = normalize_nuit(value)
    types = {
        '1': 'pessoa singular nacional',
        '2': 'pessoa singular estrangeira',
        '3': 'pessoa singular nacional',
        '4': 'pessoa colectiva privada',
        '5': 'pessoa colectiva pública',
        '6': 'outra pessoa colectiva',
        '7': 'outra pessoa colectiva',
    }
    return types.get(clean[0], 'desconhecido')
```

---

## Componente React de input

```jsx
function NuitInput({ value, onChange, onValid }) {
  const [error, setError] = useState('');

  const handleChange = (e) => {
    // Aceitar apenas dígitos e espaços; máximo 11 caracteres ("XXX XXX XXX")
    let raw = e.target.value.replace(/[^\d\s]/g, '').slice(0, 11);

    // Auto-formatação à medida que o utilizador escreve
    const digits = raw.replace(/\s/g, '');
    let formatted = digits;
    if (digits.length > 6) formatted = `${digits.slice(0,3)} ${digits.slice(3,6)} ${digits.slice(6,9)}`;
    else if (digits.length > 3) formatted = `${digits.slice(0,3)} ${digits.slice(3)}`;

    onChange(formatted);

    // Validação com debounce visual
    if (digits.length === 9) {
      setError('');
      onValid?.(true);
    } else if (digits.length > 0) {
      setError(`${digits.length}/9 dígitos`);
      onValid?.(false);
    } else {
      setError('');
    }
  };

  return (
    <div>
      <input
        type="text"
        inputMode="numeric"
        placeholder="XXX XXX XXX"
        value={value}
        onChange={handleChange}
        aria-invalid={!!error}
      />
      {error && <span style={{color: 'red'}}>{error}</span>}
    </div>
  );
}
```

---

## Casos de teste recomendados

```javascript
// Válidos
isValidNUIT('123456789') === true
isValidNUIT('123 456 789') === true
isValidNUIT(' 123 456 789 ') === true

// Inválidos
isValidNUIT('') === false
isValidNUIT('12345678') === false       // 8 dígitos
isValidNUIT('1234567890') === false     // 10 dígitos
isValidNUIT('12345678A') === false      // letra
isValidNUIT('123-456-789') === false    // traços
isValidNUIT(123456789) === false        // tipo errado

// Formatação
formatNUIT('123456789') === '123 456 789'
formatNUIT('  123456789  ') === '123 456 789'
```

---

## Armadilhas comuns

- **Confiar que o utilizador escreve sem espaços.** Não. Sempre limpar com `replace(/\s/g, '')` antes de validar/guardar.
- **Mostrar erro durante a digitação dos primeiros dígitos.** Mau UX. Só mostrar erro quando o utilizador tirar o foco (`onBlur`) ou tentar submeter.
- **Guardar formatado na DB.** Causa problemas em queries. Guarda os 9 dígitos puros; formata na renderização.
- **Validar com regex demasiado restritivo logo no `onChange`.** Bloqueia copy-paste de "123 456 789". Aceita espaços, limpa depois.
- **Assumir que NUIT == BI.** Não. BI é número de bilhete de identidade (formato diferente: 13 dígitos).

---

## Diferenças com NUIB e BI

| Documento | Comprimento | Para quê |
|-----------|-------------|----------|
| NUIT | 9 dígitos | Tributário (faturas, declarações) |
| NUIB | 13 dígitos | Bancário (conta, IBAN base) |
| BI | 13 dígitos | Identificação civil |

Não confundir. Se um formulário pede NUIT, valida 9; se pede BI, valida 13 + dígito de controlo + ano emissão (formato mais complexo).
