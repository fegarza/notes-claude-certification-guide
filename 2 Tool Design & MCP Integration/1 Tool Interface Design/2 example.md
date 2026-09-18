---
tags:
  - claude-cert/dominio-2
  - task-statement/2.1
---

# 2 example — Tool Interface Design

Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, el mismo caso de uso que usa la guía de certificación: un agente de soporte con dos tools (`get_customer` y `lookup_order`) que empieza con descripciones ambiguas, medimos el misrouting, y lo arreglamos aplicando los 6 conceptos del resumen — sin tocar la implementación de las tools, solo su interfaz.

Usamos el SDK oficial de Python de Anthropic (`anthropic`).

## Paso 1 — Definir las tools con descripciones mínimas (el punto de partida ambiguo)

Empezamos a propósito con descripciones de una sola frase, sin formatos de input, sin ejemplos, sin fronteras. Esto es lo que la [[1 resumen#2. El problema del misrouting|Conclusión 2]] llama el escenario clásico de misrouting.

```python
import anthropic

client = anthropic.Anthropic()

# Descripciones mínimas — a propósito ambiguas, para reproducir el misrouting
tools_minimal = [
    {
        "name": "get_customer",
        "description": "Retrieves customer information",
        "input_schema": {
            "type": "object",
            "properties": {"identifier": {"type": "string"}},
            "required": ["identifier"],
        },
    },
    {
        "name": "lookup_order",
        "description": "Retrieves order details",
        "input_schema": {
            "type": "object",
            "properties": {"identifier": {"type": "string"}},
            "required": ["identifier"],
        },
    },
]
```

> [!warning] Pregunta trampa en código — "ya tengo dos tools, ¿para qué más texto?"
> La forma ingenua de "arreglar" esto sería dejar las descripciones así y confiar en que el modelo infiera el propósito por el `name` de la tool (`get_customer` "suena" a cliente, `lookup_order` "suena" a orden). **Por qué es mala idea:** el modelo decide leyendo la `description`, no adivinando semántica a partir del nombre de la función. Un nombre claro con una descripción pobre sigue produciendo misrouting — es exactamente la Conclusión 2 del resumen.

## Paso 2 — Medir el misrouting con un set de queries

Antes de tocar nada, generamos evidencia: mandamos 10 queries representativas y registramos qué tool elige el modelo para cada una.

```python
queries = [
    "What is the status of order #12345?",
    "Look up customer john@example.com",
    "Check my order tracking",
    "Find the account for phone 555-0123",
    "Where is my package?",
    "Is order #67890 eligible for a refund?",
    "What loyalty tier is this customer?",
    "I need details on order #11111",
    "Verify the customer account status",
    "When will order #99999 arrive?",
]

expected = [
    "lookup_order", "get_customer", "lookup_order", "get_customer", "lookup_order",
    "lookup_order", "get_customer", "lookup_order", "get_customer", "lookup_order",
]


def run_selection(tools, system=None):
    results = []
    for query in queries:
        kwargs = {
            "model": "claude-sonnet-5",
            "max_tokens": 1024,
            "tools": tools,
            "messages": [{"role": "user", "content": query}],
        }
        if system:
            kwargs["system"] = system
        response = client.messages.create(**kwargs)
        tool_use = next((b for b in response.content if b.type == "tool_use"), None)
        results.append(tool_use.name if tool_use else None)
    return results


selected_before = run_selection(tools_minimal)
accuracy_before = sum(s == e for s, e in zip(selected_before, expected)) / len(expected)
print(f"Accuracy con descripciones mínimas: {accuracy_before:.0%}")
```

Con descripciones mínimas, este experimento reproduce el hallazgo de la guía: varias queries de orden (ej. "check my order #12345") se enrutan a `get_customer` en vez de `lookup_order`.

## Paso 3 — Reescribir las descripciones a production-grade

Aplicamos los 5 elementos de la [[1 resumen#1. Los 5 elementos de una descripción production-grade|Conclusión 1]]: propósito, inputs con formato, queries de ejemplo, edge cases, y frontera explícita frente a la otra tool.

```python
tools_production_grade = [
    {
        "name": "get_customer",
        "description": (
            "Looks up a customer account by email address, phone number, or "
            "customer ID. Returns the customer profile (name, contact details, "
            "account status, loyalty tier). Use this when the user asks about "
            "their identity or account, e.g. 'what loyalty tier am I?' or "
            "'verify my account'. Do NOT use for order-specific queries — "
            "use lookup_order for those."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "identifier": {
                    "type": "string",
                    "description": "Customer email, phone number, or customer ID",
                }
            },
            "required": ["identifier"],
        },
    },
    {
        "name": "lookup_order",
        "description": (
            "Retrieves order details by order number (format: #NNNNN) or "
            "tracking ID. Returns order status, items, shipping details, and "
            "refund eligibility. Use this when the user asks about a specific "
            "order, e.g. 'where is my package?' or 'is order #12345 refundable?'. "
            "Do NOT use for customer identity verification — use get_customer "
            "for that."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "identifier": {
                    "type": "string",
                    "description": "Order number (#NNNNN) or tracking ID",
                }
            },
            "required": ["identifier"],
        },
    },
]
```

> [!warning] Pregunta trampa en código — "arreglar" esto con few-shot examples en vez de reescribir la descripción
> La forma ingenua alternativa sería dejar `tools_minimal` tal cual y en su lugar meter 5-8 ejemplos de queries de orden en el `system` prompt, esperando que el modelo "aprenda por ejemplo" a enrutar bien. **Por qué es mala idea** (Trampa de examen 1 del resumen): eso añade overhead de tokens en cada request y no arregla la causa raíz — la descripción sigue sin decirle al modelo qué formato de identificador acepta cada tool ni cuándo preferir la otra. Es tratar el síntoma, no la enfermedad.

## Paso 4 — Re-ejecutar y comparar accuracy

```python
selected_after = run_selection(tools_production_grade)
accuracy_after = sum(s == e for s, e in zip(selected_after, expected)) / len(expected)
print(f"Accuracy con descripciones production-grade: {accuracy_after:.0%}")

for query, before, after, exp in zip(queries, selected_before, selected_after, expected):
    marker = "OK" if after == exp else "MISROUTED"
    print(f"{marker:9} | {query!r:45} antes={before} despues={after} esperado={exp}")
```

La guía espera que la accuracy suba a 9/10 o 10/10, y que las queries que antes se enrutaban mal ahora acierten.

## Paso 5 — Revisar el system prompt en busca de conflictos

Aunque las descripciones ya sean production-grade, un system prompt con wording sensible a palabras clave puede seguir generando misrouting — la [[1 resumen#6. Interacciones con el system prompt|Conclusión 6]] del resumen.

```python
conflicting_system_prompt = (
    "Always check customer details before proceeding with any request."
)

selected_with_conflict = run_selection(
    tools_production_grade, system=conflicting_system_prompt
)

# Si get_customer aparece para queries que esperábamos que fueran lookup_order,
# el system prompt está compitiendo con las descripciones ya mejoradas.
for query, tool, exp in zip(queries, selected_with_conflict, expected):
    if tool != exp:
        print(f"Conflicto detectado: {query!r} -> {tool} (esperado {exp})")
```

> [!warning] Pregunta trampa en código — dar el problema por cerrado tras el Paso 4
> La forma ingenua sería detenerse en el Paso 4 apenas la accuracy suba a 9/10 o 10/10 y asumir que el misrouting está resuelto para siempre. **Por qué es mala idea:** si el system prompt de producción trae wording sensible a palabras clave (ej. "always check customer details..."), ese wording puede seguir compitiendo con las descripciones ya mejoradas y reintroducir el mismo misrouting en producción aunque el test aislado del Paso 4 haya pasado. Por eso el Paso 5 es parte del arreglo, no un extra opcional.

## Paso 6 — Splitting: cuando el problema es una tool que hace demasiado

El caso anterior era de dos tools *distintas* con descripciones ambiguas. Un problema relacionado pero diferente ([[1 resumen#4. Tool splitting (dividir tools genéricas)|Conclusión 4]]) es una sola tool que carga varias responsabilidades:

```python
# Antes: una tool genérica que obliga al modelo a adivinar qué operación se quiere
tool_generic = {
    "name": "analyze_document",
    "description": "Analyses a document and returns results",
    "input_schema": {
        "type": "object",
        "properties": {"document_id": {"type": "string"}},
        "required": ["document_id"],
    },
}

# Después: tools de propósito único, cada una con su propio contrato de input/output
tools_split = [
    {
        "name": "extract_data_points",
        "description": (
            "Extracts structured data fields (dates, amounts, names) from a "
            "document. Use when the user wants specific values pulled out, "
            "not a narrative summary."
        ),
        "input_schema": {
            "type": "object",
            "properties": {"document_id": {"type": "string"}},
            "required": ["document_id"],
        },
    },
    {
        "name": "summarize_content",
        "description": (
            "Produces a concise summary of a document's key arguments and "
            "conclusions. Use when the user wants an overview, not individual "
            "data fields."
        ),
        "input_schema": {
            "type": "object",
            "properties": {"document_id": {"type": "string"}},
            "required": ["document_id"],
        },
    },
    {
        "name": "verify_claim_against_source",
        "description": (
            "Checks whether a specific claim is supported by the source "
            "document, returning supporting or contradicting evidence. Use "
            "when the user asks to fact-check a statement against the document."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "document_id": {"type": "string"},
                "claim": {"type": "string"},
            },
            "required": ["document_id", "claim"],
        },
    },
]
```

> [!warning] Pregunta trampa en código — "arreglar" `analyze_document` solo con una descripción más larga
> La forma ingenua sería dejar una sola tool `analyze_document` y escribirle una descripción larguísima que mencione extracción, resumen y verificación de claims. **Por qué es mala idea:** el problema no es falta de texto — es que una sola tool sigue cargando tres responsabilidades distintas, así que el modelo debe seguir adivinando cuál de las tres se quiere en cada llamada. Ninguna descripción, por buena que sea, sustituye dividir la responsabilidad en tools separadas.

## Paso 7 — Renombrar cuando el problema es el nombre, no el alcance

Un tercer caso distinto ([[1 resumen#5. Renombrar tools para dar claridad|Conclusión 5]]): dos tools con alcances ya bien definidos, pero nombres confusamente parecidos.

```python
# Antes: nombre genérico que se confunde con otras tools de "análisis"
# analyze_content -> "Analyses content and returns insights"

# Después: mismo comportamiento, nombre y descripción específicos
tool_renamed = {
    "name": "extract_web_results",
    "description": (
        "Extracts structured results (titles, snippets, URLs) from a web "
        "search response. Use this specifically for web search results, not "
        "for arbitrary text content."
    ),
    "input_schema": {
        "type": "object",
        "properties": {"search_response_id": {"type": "string"}},
        "required": ["search_response_id"],
    },
}
```

Nótese que la implementación detrás de la tool no cambia en nada — el arreglo ocurre completamente a nivel de interfaz, tal como señala el resumen.
