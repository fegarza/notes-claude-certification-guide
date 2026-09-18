Aplicación práctica de [[1 resumen]]

## Caso de uso

Vamos a construir, paso a paso, un **sistema de triage de tickets de soporte** que necesita: (1) elegir la herramienta correcta ante solicitudes ambiguas, (2) extraer campos estructurados de tickets con formatos muy variados, y (3) mantener un formato de salida consistente. Aplicaremos los tres disparadores de few-shot del resumen, la construcción correcta de ejemplos (2-4, con razonamiento), y la distinción clave de cuándo *no* usar few-shot.

### Paso 1 — La versión ingenua: solo instrucciones detalladas (para ver por qué falla)

```python
from anthropic import Anthropic

client = Anthropic()

NAIVE_SYSTEM_PROMPT = """
You are a support ticket triage assistant. Read the ticket and decide whether
to route it to `billing_tool` or `technical_tool`. Be careful with ambiguous
tickets and use good judgement. Extract the customer's account ID and the
issue summary as JSON.
"""
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> `"Be careful with ambiguous tickets and use good judgement"` no le da al modelo ningún principio de decisión — es el mismo problema de instrucciones vagas que en [[1 System Prompts with Explicit Criteria/1 resumen|System Prompts with Explicit Criteria]]. En producción, este prompt enruta el mismo tipo de ticket ambiguo ("no puedo pagar mi factura porque el botón de pago da error") a `billing_tool` unas veces y a `technical_tool` otras. Ver [[1 resumen#1. Tres disparadores para usar few-shot]] — esto es exactamente el disparador de "juicio ambiguo".

### Paso 2 — Una forma incorrecta de "arreglarlo" con few-shot: ejemplos sin razonamiento

Antes de la versión correcta, así se ve el error común de construir few-shot sin razonamiento — solo pares entrada-salida.

```python
BAD_FEWSHOT_EXAMPLES = [
    {"role": "user", "content": "Ticket: 'My invoice #4471 shows the wrong amount.'"},
    {"role": "assistant", "content": '{"tool": "billing_tool", "account_id": null, "summary": "Incorrect invoice amount"}'},
    {"role": "user", "content": "Ticket: 'The payment button throws a 500 error.'"},
    {"role": "assistant", "content": '{"tool": "technical_tool", "account_id": null, "summary": "Payment button error"}'},
]
```

> [!warning] Por qué esta forma es insuficiente
> Estos ejemplos muestran el resultado pero no el **por qué**. Ver [[1 resumen#2. Cómo construir buenos ejemplos few-shot]]: sin razonamiento, el modelo puede memorizar literalmente "factura → billing" y "error de botón → technical", pero fallará ante el ticket ambiguo real ("no puedo pagar mi factura porque el botón de pago da error"), que mezcla ambas señales y no aparece tal cual en los ejemplos.

### Paso 3 — La versión correcta: 2-4 ejemplos con razonamiento, enfocados en los casos que fallan

Construimos ejemplos dirigidos específicamente a los tickets ambiguos que el sistema naive fallaba, incluyendo el razonamiento de por qué se elige una herramienta sobre la otra.

```python
FEWSHOT_ROUTING_EXAMPLES = [
    {
        "role": "user",
        "content": "Ticket: 'I can't pay my invoice because the payment button throws an error.'",
    },
    {
        "role": "assistant",
        "content": (
            '{"reasoning": "The customer\'s underlying goal is completing a payment, '
            'not disputing the invoice amount. The error is in the payment mechanism, '
            'so the blocking issue is technical even though billing is the context.", '
            '"tool": "technical_tool", "account_id": null, '
            '"summary": "Payment button error blocking invoice payment"}'
        ),
    },
    {
        "role": "user",
        "content": "Ticket: 'My invoice #4471 shows $200 but I was quoted $150 last month.'",
    },
    {
        "role": "assistant",
        "content": (
            '{"reasoning": "No technical error is mentioned. The dispute is about the '
            'charged amount itself, which is a billing decision, not a system failure.", '
            '"tool": "billing_tool", "account_id": null, '
            '"summary": "Invoice amount discrepancy vs quoted price"}'
        ),
    },
    {
        "role": "user",
        "content": "Ticket: 'The app crashes every time I try to view my billing history.'",
    },
    {
        "role": "assistant",
        "content": (
            '{"reasoning": "The customer is not disputing any charge or amount. The '
            'complaint is that the application itself crashes, which is a technical '
            'defect regardless of which screen triggers it.", '
            '"tool": "technical_tool", "account_id": null, '
            '"summary": "App crash when viewing billing history"}'
        ),
    },
]
```

Tres ejemplos (dentro del rango 2-4), cada uno con `"reasoning"` explícito, y los tres construidos sobre casos donde billing y technical se mezclan — exactamente el patrón que rompía la versión naive. Esto es lo que enseña el **principio de decisión** ("¿la queja es sobre el monto/cargo, o sobre un fallo del sistema?"), no solo tres casos memorizados — ver [[1 resumen#3. El razonamiento enseña generalización, no memorización]].

### Paso 4 — Extracción con documentos de formato variado (tercer disparador)

Ahora atacamos el disparador de "campos de extracción vacíos": la extracción de `account_id` falla cuando el ID viene en formatos distintos al esperado.

```python
FEWSHOT_EXTRACTION_EXAMPLES = [
    {
        "role": "user",
        "content": "Ticket: 'Account #A-99231. My invoice #4471 shows the wrong amount.'",
    },
    {
        "role": "assistant",
        "content": '{"reasoning": "Account ID is explicit and prefixed.", "account_id": "A-99231"}',
    },
    {
        "role": "user",
        "content": "Ticket: 'This is regarding the account tied to my email, jane@example.com, order placed last Tuesday.'",
    },
    {
        "role": "assistant",
        "content": (
            '{"reasoning": "No explicit account ID string exists, but the email '
            'functions as the account identifier in this context.", '
            '"account_id": "jane@example.com"}'
        ),
    },
    {
        "role": "user",
        "content": "Ticket: 'Ref: 2024-INV-0087 — see attached statement.'",
    },
    {
        "role": "assistant",
        "content": (
            '{"reasoning": "The invoice reference number is the only identifier '
            'present and should be captured rather than left null.", '
            '"account_id": "2024-INV-0087"}'
        ),
    },
]
```

> [!warning] Trampa aplicada aquí
> Sin el segundo y tercer ejemplo, el sistema devolvería `"account_id": null` cada vez que el identificador no viene en el formato canónico `A-#####` — no porque falte la información, sino porque nunca vio ese formato. Ver [[1 resumen#4. Reducir alucinación en extracción con documentos variados]]. Agregar más instrucciones ("busca también emails o referencias de factura") ayuda menos que mostrar el patrón directamente con ejemplos.

### Paso 5 — Ensamblar la llamada final combinando ambos bloques de few-shot

```python
def triage_ticket(ticket_text: str) -> dict:
    messages = (
        FEWSHOT_ROUTING_EXAMPLES
        + FEWSHOT_EXTRACTION_EXAMPLES
        + [{"role": "user", "content": f"Ticket: '{ticket_text}'"}]
    )

    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=300,
        system=(
            "You are a support ticket triage assistant. Route each ticket to "
            "`billing_tool` or `technical_tool` and extract the account "
            "identifier and a one-line summary. Always include your reasoning."
        ),
        messages=messages,
    )
    return response.content[0].text
```

Los ejemplos few-shot viven como turnos de conversación previos a la solicitud real — el patrón estándar para demostrar comportamiento con razonamiento, en vez de describirlo en prosa dentro del system prompt.

### Paso 6 — Cuándo *no* usar few-shot: la distinción clave del resumen

Simulamos tres síntomas parecidos a "inconsistencia" pero con causas raíz distintas, para mostrar por qué agregar más few-shot ahí sería el error equivocado.

```python
# --- Síntoma A: el modelo fabrica un account_id cuando no hay ninguno ---
# Ver [[1 resumen#5. Distinción clave: few-shot no es la solución universal]]
#
# INCORRECTO: agregar más ejemplos few-shot esperando que "aprenda a no inventar".
# CORRECTO: hacer el campo explícitamente nullable en el schema de la tool call,
# para que el modelo tenga una salida válida cuando no hay dato (tema de
# [[3 Structured Output with Tool Use/1 resumen|Structured Output with Tool Use]]).
TICKET_SCHEMA = {
    "type": "object",
    "properties": {
        "account_id": {"type": ["string", "null"]},  # nullable, no forzado
        "summary": {"type": "string"},
    },
    "required": ["summary"],
}

# --- Síntoma B: el modelo calcula mal un total de reembolso a partir del ticket ---
# INCORRECTO: mostrarle ejemplos few-shot de sumas correctas.
# CORRECTO: un bucle de validación y reintento que verifique el cálculo
# programáticamente (tema de [[4 Validation, Retry, and Feedback Loops/1 resumen|Validation, Retry, and Feedback Loops]]).
def validate_refund_total(claimed_total: float, line_items: list[float]) -> bool:
    return abs(claimed_total - sum(line_items)) < 0.01

# --- Síntoma C: el modelo confunde billing_tool con un tercer tool "escalation_tool"
# porque ambos tienen descripciones mínimas casi idénticas ---
# INCORRECTO: agregar 5-8 ejemplos few-shot de enrutamiento como primer paso.
# CORRECTO: enriquecer primero la descripción de cada tool (formatos de entrada,
# casos límite, cuándo usar una vs. otra) — few-shot es secundario aquí.
ESCALATION_TOOL_DESCRIPTION = (
    "Use escalation_tool ONLY when the customer explicitly requests a manager, "
    "threatens legal action, or has had 3+ prior unresolved tickets on the same "
    "issue. Do NOT use for routine billing disputes — use billing_tool for those."
)
```

> [!warning] Por qué mezclar estas tres soluciones sería el error del examen
> Si ante los tres síntomas la respuesta fuera "agregar más ejemplos few-shot", se estaría ignorando que cada síntoma tiene una causa raíz distinta. Few-shot es la herramienta correcta solo para el enrutamiento ambiguo (Paso 3) y la extracción con formato variado (Paso 4) — no para fabricación de valores, errores de cálculo, ni para el primer arreglo de descripciones de tools pobres.

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
