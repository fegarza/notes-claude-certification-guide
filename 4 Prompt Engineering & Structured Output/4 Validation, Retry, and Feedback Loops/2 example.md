Aplicación práctica de [[1 resumen]]

## Caso de uso

Vamos a extender el pipeline de extracción de invoices de [[3 Structured Output with Tool Use/2 example|Structured Output with Tool Use]] agregando un **loop de validación-retry** en Python con Pydantic, aplicando los conceptos del resumen: los tres ingredientes de retry-with-error-feedback, la frontera de efectividad del retry, self-correction en el schema (`calculated_total`/`stated_total`, `conflict_detected`), `detected_pattern` para análisis de descartes, y la distinción entre errores de sintaxis y semánticos.

### Paso 0 — La versión ingenua (para ver por qué falla)

Antes de construir el loop correcto, así se ve un reintento sin retroalimentación de error — el enfoque que el resumen descarta.

```python
import anthropic

client = anthropic.Anthropic()

def extract_naive(document_text: str, tool: dict, previous_error: bool = False) -> dict:
    prompt = document_text
    if previous_error:
        prompt += "\n\nThe previous extraction was wrong. Please try again."

    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        tools=[tool],
        tool_choice={"type": "tool", "name": tool["name"]},
        messages=[{"role": "user", "content": prompt}],
    )
    tool_call = next(b for b in response.content if b.type == "tool_use")
    return tool_call.input
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> `"The previous extraction was wrong. Please try again."` no le dice al modelo *qué* estuvo mal — ver [[1 resumen#1. Retry-with-error-feedback: los tres ingredientes]]. Sin el error específico, el modelo típicamente reproduce la misma extracción, porque no tiene ninguna señal nueva para corregirse. Este código tampoco distingue si el fallo es corregible: si vuelve a llamarse en un bucle contra un documento con información realmente ausente, reintentará indefinidamente sin nunca tener éxito.

### Paso 1 — Modelar el schema con Pydantic, incluyendo self-correction

Definimos el modelo de invoice con `calculated_total`/`stated_total` y un validator que aplica la regla semántica que un JSON schema no puede expresar.

```python
from pydantic import BaseModel, ValidationError, model_validator


class LineItem(BaseModel):
    description: str
    amount: float


class Invoice(BaseModel):
    invoice_number: str
    vendor_name: str
    line_items: list[LineItem]
    stated_total: float
    department: str | None = None
    payment_terms_section_a: str | None = None
    payment_terms_section_b: str | None = None
    conflict_detected: bool = False

    @model_validator(mode="after")
    def totals_must_match(self):
        calculated = round(sum(item.amount for item in self.line_items), 2)
        if abs(calculated - self.stated_total) > 0.01:
            raise ValueError(
                f"line items sum to {calculated} but stated_total is {self.stated_total}"
            )
        return self
```

`department` es `str | None`: puede no estar en la fuente, así que se deja opcional en vez de `required` — la misma defensa contra fabricación de [[3 Structured Output with Tool Use/1 resumen#4. Diseño de schema para producción]].

### Paso 2 — Detectar conflictos de fuente sin elegir un valor silenciosamente

Cuando el documento tiene información contradictoria (dos secciones con distintos términos de pago), se extraen ambos valores y se marca el conflicto — en vez de que el modelo "decida" cuál usar.

```python
def check_conflict(invoice: Invoice) -> Invoice:
    if (
        invoice.payment_terms_section_a is not None
        and invoice.payment_terms_section_b is not None
        and invoice.payment_terms_section_a != invoice.payment_terms_section_b
    ):
        invoice.conflict_detected = True
    return invoice
```

> [!note] Conexión con el resumen
> Esto aplica directamente [[1 resumen#3. Diseño de self-correction dentro del schema]]: el objetivo no es que el pipeline "adivine" cuál de los dos valores es el correcto, sino dejar la discrepancia visible para revisión.

### Paso 3 — Construir el mensaje de retry con los tres ingredientes

Al fallar la validación de Pydantic, se arma el mensaje de reintento con documento original + extracción fallida + error específico — no un genérico "inténtalo de nuevo".

```python
import json


def build_retry_message(original_document: str, failed_extraction: dict, error: ValidationError) -> str:
    error_lines = "\n".join(
        f"{'.'.join(map(str, err['loc'])) or 'invoice'}: {err['msg']}"
        for err in error.errors()
    )
    return (
        f"Original document:\n{original_document}\n\n"
        f"Your extraction:\n{json.dumps(failed_extraction)}\n\n"
        f"Validation errors:\n{error_lines}\n\n"
        f"Please re-extract, fixing the identified errors."
    )
```

### Paso 4 — El loop de retry, con la frontera de efectividad aplicada

El loop reintenta solo cuando el error es de los que `tool_use`/Pydantic pueden corregir con retroalimentación (formato, estructura, matemática) — no cuando el dato simplemente no está en el documento.

```python
UNRETRIABLE_HINTS = ("not present in the source", "not mentioned in the document")


def extract_with_retry(
    document_text: str, tool: dict, max_retries: int = 2
) -> tuple[Invoice | None, str | None]:
    messages = [{"role": "user", "content": document_text}]

    for attempt in range(max_retries + 1):
        response = client.messages.create(
            model="claude-sonnet-5",
            max_tokens=1024,
            tools=[tool],
            tool_choice={"type": "tool", "name": tool["name"]},
            messages=messages,
        )
        tool_call = next(b for b in response.content if b.type == "tool_use")
        raw = tool_call.input

        try:
            invoice = Invoice.model_validate(raw)
            return check_conflict(invoice), None
        except ValidationError as e:
            if attempt == max_retries:
                return None, "max_retries_exceeded"

            if raw.get("department") is None and "department" in document_text.lower():
                # el campo no vino, pero el texto sí menciona "department" en otro sentido:
                # no hay evidencia de que la información esté genuinamente ausente, sí vale reintentar
                pass
            elif raw.get("department") is None:
                # información ausente de la fuente: reintentar no la va a crear
                return None, "needs_human_review_missing_data"

            messages.append({
                "role": "user",
                "content": build_retry_message(document_text, raw, e),
            })

    return None, "max_retries_exceeded"
```

> [!warning] Trampa de examen aplicada aquí
> El `else: return None, "needs_human_review_missing_data"` es exactamente [[1 resumen#2. La frontera de efectividad del retry]]: si `department` está ausente y nada en el documento sugiere que la información existe en algún lado, seguir reintentando es inútil — el resultado correcto es enrutar a revisión humana, no gastar otro round-trip a la API con la esperanza de que el modelo "invente" el dato correcto esta vez.

### Paso 5 — `detected_pattern` para findings de revisión de código

En un pipeline de revisión de código (no de invoices), cada finding trae `detected_pattern` para poder analizar después qué patrones generan más falsos positivos.

```python
class Finding(BaseModel):
    finding: str
    severity: str
    detected_pattern: str
    file: str
    line: int
    dismissed: bool = False


def dismissal_rate_by_pattern(findings: list[Finding]) -> dict[str, float]:
    from collections import defaultdict

    totals: dict[str, int] = defaultdict(int)
    dismissed: dict[str, int] = defaultdict(int)

    for f in findings:
        totals[f.detected_pattern] += 1
        if f.dismissed:
            dismissed[f.detected_pattern] += 1

    return {
        pattern: dismissed[pattern] / totals[pattern]
        for pattern in totals
    }
```

> [!note] Conexión con el resumen
> `dismissal_rate_by_pattern` es lo que cierra el loop de [[1 resumen#4. detected_pattern: loops de mejora sistemática]]: si `"variable shadowing en scope anidado"` tiene una tasa de descarte alta, ese `detected_pattern` es candidato prioritario para refinar el prompt del reviewer — no un patrón elegido al azar.

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
