Aplicación práctica de [[1 resumen]]

## Caso de uso

Vamos a construir, paso a paso, un **pipeline de extracción de datos de facturas (invoices)** en Python usando la API de Claude, aplicando los cuatro conceptos del resumen: la jerarquía `tool_use` sobre JSON basado en prompt, los tres modos de `tool_choice`, la distinción entre errores sintácticos y semánticos, y el diseño de schema para evitar fabricación.

### Paso 0 — La versión ingenua (para ver por qué falla)

Antes de construir la versión correcta, así se ve el enfoque que el resumen descarta: pedir JSON como texto libre dentro del prompt, sin `tool_use`.

```python
import anthropic

client = anthropic.Anthropic()

NAIVE_PROMPT = """Extract the invoice number, vendor name, and total amount
from this document. Respond ONLY with valid JSON."""

response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": f"{NAIVE_PROMPT}\n\n{document_text}"}],
)
raw_text = response.content[0].text
data = json.loads(raw_text)  # puede lanzar json.JSONDecodeError
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> Pedir JSON como texto no da ninguna garantía estructural — ver [[1 resumen#1. La jerarquía de confiabilidad: tool_use sobre JSON basado en prompt]]. `response.content[0].text` puede venir con una llave sin cerrar, una coma sobrante, o texto conversacional antes del JSON ("Here's the extracted data:"). `json.loads` fallará **periódicamente** en producción, no ocasionalmente por mala suerte — es el comportamiento esperado de este enfoque a escala.

### Paso 1 — Definir el tool de extracción con JSON schema

Reemplazamos el prompt de texto libre por un tool cuyo `input_schema` define la forma exacta de la salida.

```python
extract_invoice_tool = {
    "name": "extract_invoice",
    "description": "Extract structured invoice data from a document.",
    "input_schema": {
        "type": "object",
        "properties": {
            "invoice_number": {"type": "string"},
            "vendor_name": {"type": "string"},
            "total_amount": {"type": "number"},
            "payment_terms": {"type": ["string", "null"]},
            "purchase_order": {"type": ["string", "null"]},
        },
        "required": ["invoice_number", "vendor_name", "total_amount"],
    },
}
```

`payment_terms` y `purchase_order` son `["string", "null"]`: son datos que un invoice puede o no incluir, así que se dejan nullable en vez de required.

### Paso 2 — Probar `tool_choice: "auto"` (y ver por qué no garantiza nada)

```python
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[extract_invoice_tool],
    tool_choice={"type": "auto"},
    messages=[{"role": "user", "content": document_text}],
)

print(response.stop_reason)
# A veces: "tool_use"  → el modelo llamó al tool
# A veces: "end_turn"  → el modelo simplemente respondió con texto
```

> [!warning] Trampa de examen aplicada aquí
> Con `"auto"` el modelo puede decidir responder con texto en vez de invocar el tool — ver [[1 resumen#2. tool_choice: los tres modos]]. Si el pipeline asume que `response.content` siempre trae un bloque `tool_use`, este código se romperá justo en los casos donde el modelo "decidió" no usar el tool. `"auto"` es la opción correcta solo cuando una respuesta puramente conversacional también sería aceptable — no es el caso en un pipeline de extracción.

### Paso 3 — Cambiar a `tool_choice: "any"` para garantizar salida estructurada

Cuando el tipo de documento es desconocido de antemano (podría ser un invoice, un receipt, o un contract), se ofrecen varios tools de extracción y se usa `"any"` para garantizar que el modelo invoque *alguno* de ellos.

```python
extract_receipt_tool = {"name": "extract_receipt", "description": "...", "input_schema": {...}}
extract_contract_tool = {"name": "extract_contract", "description": "...", "input_schema": {...}}

response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[extract_invoice_tool, extract_receipt_tool, extract_contract_tool],
    tool_choice={"type": "any"},
    messages=[{"role": "user", "content": document_text}],
)

assert response.stop_reason == "tool_use"  # garantizado por tool_choice: "any"
tool_call = next(b for b in response.content if b.type == "tool_use")
extracted = tool_call.input
```

### Paso 4 — Forzar un tool específico como paso obligatorio

Si el pipeline requiere que la extracción de metadata ocurra siempre antes de pasos de enriquecimiento posteriores, se fuerza ese tool específico en vez de dejarlo a discreción del modelo.

```python
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[extract_invoice_tool],
    tool_choice={"type": "tool", "name": "extract_invoice"},
    messages=[{"role": "user", "content": document_text}],
)
# El modelo no puede elegir responder con texto ni con otro tool:
# solo puede llamar exactamente a "extract_invoice".
```

> [!note] Recordatorio del resumen
> `tool_choice` se especifica en cada request — si un siguiente turno de la conversación necesita otro modo (por ejemplo volver a `"auto"` para una pregunta de seguimiento del usuario), hay que declararlo de nuevo en esa llamada. Ver [[1 resumen#2. tool_choice: los tres modos]].

### Paso 5 — Procesar documentos con campos ausentes y verificar que no se fabrican valores

```python
def extract(document_text: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        tools=[extract_invoice_tool],
        tool_choice={"type": "tool", "name": "extract_invoice"},
        messages=[{"role": "user", "content": document_text}],
    )
    tool_call = next(b for b in response.content if b.type == "tool_use")
    return tool_call.input

# Documento sin purchase_order visible en el texto fuente
result = extract(invoice_without_po_text)
print(result)
# {"invoice_number": "INV-2044", "vendor_name": "Acme Corp",
#  "total_amount": 1200.0, "payment_terms": "net-30", "purchase_order": None}
```

> [!warning] Trampa de examen aplicada aquí
> Si `purchase_order` hubiera sido `required` en vez de nullable, el modelo se ve presionado a inventar un número de orden de compra con tal de satisfacer el schema — ver [[1 resumen#4. Diseño de schema para producción]]. Marcarlo como `["string", "null"]` es lo que le permite responder `None` honestamente cuando el dato simplemente no está en el documento.

### Paso 6 — Categorización extensible: `"unclear"` y `"other"` + detalle

Extendemos el schema para clasificar el tipo de documento, cubriendo casos ambiguos y categorías no anticipadas.

```python
classify_document_tool = {
    "name": "classify_document",
    "description": "Classify the type of a financial document.",
    "input_schema": {
        "type": "object",
        "properties": {
            "category": {
                "type": "string",
                "enum": ["invoice", "receipt", "contract", "unclear", "other"],
            },
            "category_detail": {
                "type": ["string", "null"],
                "description": "Freeform detail when category is 'other'",
            },
        },
        "required": ["category"],
    },
}
```

`"unclear"` cubre documentos genuinamente ambiguos (sin forzar una categoría "real" incorrecta); `"other"` + `category_detail` cubre tipos de documento que no se anticiparon al diseñar el enum, sin tener que rediseñar el schema cada vez que aparece un caso nuevo.

### Paso 7 — Separar la validación sintáctica de la validación semántica

El paso final aplica directamente la Conclusión 3 del resumen: `tool_use` ya eliminó los errores de sintaxis, pero la corrección semántica (¿el total realmente cuadra con las líneas?) necesita una validación aparte.

```python
def validate_semantics(extracted: dict, line_items: list[dict]) -> list[str]:
    """`tool_use` garantiza que `extracted` es sintácticamente válido según
    el schema. Esta función revisa que además sea semánticamente correcto —
    algo que el schema, por diseño, no puede garantizar."""
    errors = []

    computed_total = sum(item["amount"] for item in line_items)
    if extracted["total_amount"] is not None and abs(computed_total - extracted["total_amount"]) > 0.01:
        errors.append(
            f"total_amount ({extracted['total_amount']}) no coincide con la "
            f"suma de line items ({computed_total})"
        )

    if extracted["invoice_number"] in ("", None):
        errors.append("invoice_number vino vacío pese a ser required por el schema")

    return errors
```

> [!warning] Trampa de examen aplicada aquí
> Sería un error asumir que, como `response.stop_reason == "tool_use"` y el JSON parseó sin problema, la extracción ya es "correcta". Eso solo confirma ausencia de errores de **sintaxis** — ver [[1 resumen#3. Lo que tool_use NO previene: errores semánticos]]. `validate_semantics` es el paso que de verdad detecta discrepancias de suma o campos mal ubicados; sin él, un total incorrecto pasaría el pipeline sin que nadie lo note.

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
