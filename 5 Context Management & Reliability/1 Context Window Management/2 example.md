---
tags:
  - claude-cert/dominio-5
  - task-statement/5.1
---

# 2 example — Context Window Management

Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, el motor de contexto de un agente de soporte al cliente que maneja reembolsos en varios turnos, aplicando las seis conclusiones del resumen: el persistent case facts block, la mitigación del efecto lost-in-the-middle, el tool result trimming, el manejo del historial completo (API stateless), la optimización de agentes upstream, y prompt caching con la Messages API.

Usamos Python plano y el SDK de Anthropic (`anthropic`), siguiendo la forma real de la Messages API.

## Paso 1 — Extraer hechos transaccionales de un resultado de tool

Antes de construir el prompt, necesitamos un extractor que identifique los datos transaccionales (montos, fechas, order IDs, estados) que **nunca** deben perderse en la summarisation — aplicando la [[1 resumen#1. La trampa de la progressive summarisation|Conclusión 1]].

```python
from dataclasses import dataclass, field
from datetime import date


@dataclass
class CaseFact:
    order_id: str
    order_date: str
    refund_amount: str
    status: str
    item_description: str


def extract_case_facts(raw_order_lookup: dict) -> CaseFact:
    """Extrae solo los campos transaccionales de un resultado de tool crudo."""
    return CaseFact(
        order_id=raw_order_lookup["order_id"],
        order_date=raw_order_lookup["order_date"],
        refund_amount=raw_order_lookup["total_amount"],
        status=raw_order_lookup.get("return_status", "pending_refund"),
        item_description=raw_order_lookup["item_description"],
    )
```

> [!warning] Pregunta trampa en código — dejar que el extractor "resuma con sus propias palabras"
> La forma ingenua sería que `extract_case_facts` le pidiera al propio modelo generar una frase tipo "el cliente quiere un reembolso reciente" en vez de copiar los valores exactos. **Por qué es mala idea:** eso reintroduce exactamente el problema del Argumento central — parafrasear un monto o una fecha es otra forma de progressive summarisation. Los campos deben copiarse literales, no reformularse.

## Paso 2 — El persistent case facts block: una capa separada por issue

Para sesiones multi-issue, cada `CaseFact` vive en su propia entrada, evitando que la summarisation mezcle los datos de un issue con otro ([[1 resumen#1. La trampa de la progressive summarisation|Conclusión 1]]).

```python
@dataclass
class CaseFactsBlock:
    customer_id: str
    issues: list[CaseFact] = field(default_factory=list)

    def add_issue(self, fact: CaseFact) -> None:
        self.issues.append(fact)

    def render(self) -> str:
        """Serializa el bloque para incluirlo en cada prompt, fuera del historial resumido."""
        lines = [f"## Persistent Case Facts (customer {self.customer_id})"]
        for fact in self.issues:
            lines.append(
                f"- Order {fact.order_id} ({fact.order_date}): "
                f"refund {fact.refund_amount}, status={fact.status} — {fact.item_description}"
            )
        return "\n".join(lines)
```

## Paso 3 — Tool result trimming antes de tocar el historial

Aplicamos la [[1 resumen#3. Tool result trimming|Conclusión 3]]: un lookup de orden real trae 40+ campos, pero el agente solo necesita 5. El recorte ocurre **antes** de que el resultado entre a la conversación.

```python
RELEVANT_ORDER_FIELDS = [
    "order_id", "order_date", "total_amount",
    "return_eligible", "item_description",
]


def trim_order_result(raw_result: dict, relevant_fields: list[str] = RELEVANT_ORDER_FIELDS) -> dict:
    """Recorta un resultado verboso de tool a solo los campos relevantes."""
    return {k: v for k, v in raw_result.items() if k in relevant_fields}


def post_tool_use_hook(tool_name: str, raw_result: dict) -> dict:
    """Simula un hook PostToolUse: el recorte ocurre antes de entrar al historial."""
    if tool_name == "lookup_order":
        return trim_order_result(raw_result)
    return raw_result
```

> [!warning] Pregunta trampa en código — recortar el resultado *después* de agregarlo al historial
> La forma ingenua sería anexar `raw_result` completo (sus 40+ campos) a `conversation_history` y solo aplicar `trim_order_result` cuando se le muestra al usuario. **Por qué es mala idea:** el dato verboso ya quedó en el historial — como la API es stateless (Conclusión 4), ese historial completo se reenvía en *cada* turno subsecuente, así que el recorte tardío no ahorra nada. El recorte debe pasar antes de que el resultado entre al historial, no antes de mostrarlo.

## Paso 4 — Construcción del prompt: historial completo + case facts block

Aplicamos la [[1 resumen#4. La API de Claude es stateless historial completo en cada request|Conclusión 4]]: la API no tiene estado, así que cada request lleva el historial completo — pero el case facts block va aparte, sin ser parte de lo que se resume.

```python
def build_messages(case_facts: CaseFactsBlock, summarised_history: str, current_turn: str) -> list[dict]:
    """
    El case facts block se antepone en cada prompt, fuera del historial resumible.
    `summarised_history` puede haber comprimido la narrativa — pero NUNCA los datos
    que ya viven en `case_facts`.
    """
    context_prefix = case_facts.render()
    return [
        {
            "role": "user",
            "content": f"{context_prefix}\n\n## Conversation so far\n{summarised_history}\n\n## Current message\n{current_turn}",
        }
    ]
```

> [!warning] Pregunta trampa en código — dejar que el case facts block también entre al proceso de resumen
> La forma ingenua sería sumar el `context_prefix` al texto que se le pasa al modelo para "resumir toda la conversación de un jalón", incluyendo el bloque de hechos. **Por qué es mala idea:** si el case facts block pasa por el mismo pipeline de summarisation que la narrativa, deja de ser "persistente" — se vuelve vulnerable exactamente al mismo riesgo que se diseñó para evitar (Conclusión 1). El bloque debe construirse y añadirse por fuera del paso de resumen, siempre desde los objetos `CaseFact` originales.

## Paso 5 — Mitigar el "lost in the middle" al agregar hallazgos de varios subagentes

Para un caso de investigación (no de soporte), aplicamos la [[1 resumen#2. El efecto lost in the middle|Conclusión 2]]: los hallazgos clave van primero, con encabezados explícitos después.

```python
def aggregate_subagent_findings(findings_by_source: dict[str, str], key_findings: list[str]) -> str:
    """
    `key_findings` son bullets concisos extraídos de cada fuente.
    Van primero — el detalle completo va después, con encabezados de sección.
    """
    summary_section = "## Key Findings Summary\n" + "\n".join(f"- {f}" for f in key_findings)
    detail_sections = "\n\n".join(
        f"### {source}\n{content}" for source, content in findings_by_source.items()
    )
    return f"{summary_section}\n\n## Detailed Findings\n\n{detail_sections}"
```

> [!warning] Pregunta trampa en código — pegar las salidas de los subagentes una tras otra sin resumen inicial
> La forma ingenua sería concatenar `findings_by_source.values()` directamente, en el orden en que llegaron los subagentes, sin una sección de resumen al principio. **Por qué es mala idea:** cualquier hallazgo que caiga en la fuente del medio (ej. el segundo de tres subagentes) corre el riesgo de recibir menos peso del modelo — el fix no es "confiar en que el modelo lo note", es estructurar el input para que lo importante esté al inicio.

## Paso 6 — Optimización de agentes upstream: salidas estructuradas, no razonamiento crudo

Aplicamos la [[1 resumen#5. Upstream agent optimisation en sistemas multi-agente|Conclusión 5]]: el subagente de investigación no le manda su cadena de razonamiento al agente de síntesis, solo hechos estructurados con metadata.

```python
@dataclass
class Finding:
    claim: str
    source: str
    source_url: str
    relevance_score: float
    publication_date: str


def research_subagent_output(raw_reasoning: str, extracted_claims: list[Finding]) -> dict:
    """
    `raw_reasoning` existe solo para debugging local del subagente — nunca se envía
    al agente de síntesis. Lo que sí se envía son los `Finding` estructurados.
    """
    return {
        "findings": [
            {
                "claim": f.claim,
                "source": f.source,
                "sourceUrl": f.source_url,
                "relevanceScore": f.relevance_score,
                "publicationDate": f.publication_date,
            }
            for f in extracted_claims
        ]
    }
```

> [!warning] Pregunta trampa en código — incluir `raw_reasoning` en el payload que recibe el agente de síntesis
> La forma ingenua sería devolver `{"reasoning": raw_reasoning, "findings": [...]}` y dejar que el agente de síntesis "filtre lo que necesite". **Por qué es mala idea:** el agente de síntesis tiene presupuesto de contexto limitado — cada token gastado en la prosa de razonamiento del subagente es un token que no está disponible para procesar los hallazgos reales. La estructura ya resuelta (claim, source, relevance) es lo único que debe cruzar esa frontera.

## Paso 7 — Prompt caching con la Messages API: orden estático-antes-que-dinámico

Aplicamos la [[1 resumen#6. Prompt caching la otra mitad de la economía de contexto|Conclusión 6]]. El bloque estático (instrucciones + documento de referencia) va en `system`, con el breakpoint al final de esa parte; el mensaje del usuario, que cambia cada vez, va en `messages`.

```python
import anthropic

client = anthropic.Anthropic()

SYSTEM_INSTRUCTIONS = "You are a customer support agent for RefundCo. ..."  # bloque estático largo
REFERENCE_POLICY_DOC = "..."  # documento de referencia largo, también estático


def call_with_cached_prefix(case_facts: CaseFactsBlock, summarised_history: str, current_turn: str):
    dynamic_message = build_messages(case_facts, summarised_history, current_turn)[0]["content"]

    return client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        system=[
            {"type": "text", "text": SYSTEM_INSTRUCTIONS},
            {
                "type": "text",
                "text": REFERENCE_POLICY_DOC,
                "cache_control": {"type": "ephemeral"},
            },
        ],
        messages=[{"role": "user", "content": dynamic_message}],
    )
```

> [!warning] Pregunta trampa en código — poner el mensaje dinámico del usuario dentro del bloque `system`, antes del breakpoint
> La forma ingenua sería meter `dynamic_message` en la lista de `system` junto con `SYSTEM_INSTRUCTIONS`, pensando que "total, todo es contexto para el modelo". **Por qué es mala idea:** el caching hace match de prefijo desde el inicio — si algo que cambia en cada request (el mensaje del usuario) queda antes o mezclado con el breakpoint, el prefijo deja de coincidir en cada llamada y se pierde el ahorro de caching por completo. Lo volátil siempre va después del breakpoint, en `messages`.

## Paso 8 — Orquestación de un turno completo

Con todas las piezas, un turno del agente de soporte se ve así: recorte de tool result → actualización del case facts block → construcción del prompt con historial completo + case facts fuera del resumen → llamada con prefijo cacheado.

```python
def handle_turn(case_facts: CaseFactsBlock, summarised_history: str, user_message: str, raw_tool_result: dict | None):
    if raw_tool_result is not None:
        trimmed = post_tool_use_hook("lookup_order", raw_tool_result)
        case_facts.add_issue(extract_case_facts(trimmed))

    response = call_with_cached_prefix(case_facts, summarised_history, user_message)
    return response
```

Con esto, el flujo completo respeta las seis conclusiones del resumen: los datos transaccionales nunca pasan por summarisation, los hallazgos agregados evitan el efecto lost-in-the-middle, los resultados de tools se recortan antes de entrar al historial, cada request lleva el historial completo tal como exige una API stateless, los subagentes upstream mandan estructura en vez de razonamiento crudo, y el prefijo estático se cachea correctamente.
