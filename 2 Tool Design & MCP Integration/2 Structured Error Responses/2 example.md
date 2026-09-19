---
tags:
  - claude-cert/dominio-2
  - task-statement/2.2
---

# 2 example — Structured Error Responses

Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, una tool MCP (`query_refund_database`) que devuelve respuestas de error estructuradas según el protocolo MCP, y un agente que consume esa metadata para tomar decisiones de recuperación — aplicando las cinco conclusiones del resumen: las dos capas de error, las cuatro categorías, la forma estructurada de la respuesta, la distinción fallo-de-acceso-vs-resultado-vacío, y la recuperación local en subagentes con propagación selectiva.

Usamos Python plano para modelar la forma de las respuestas MCP (`isError`, `content`, `structuredContent`), tal como las define el protocolo.

## Paso 1 — Modelar la forma de una respuesta MCP con y sin error

Antes de escribir lógica, fijamos el contrato de datos que sigue la [[1 resumen#3. La forma estructurada de la respuesta|Conclusión 3]]: `isError`, `content`, y `structuredContent` con `errorCategory`, `isRetryable` y `description`.

```python
from dataclasses import dataclass, field
from typing import Any, Literal

ErrorCategory = Literal["transient", "validation", "business", "permission"]


@dataclass
class ToolResponse:
    isError: bool
    content: str
    structuredContent: dict[str, Any] = field(default_factory=dict)


def success(content: str, **extra: Any) -> ToolResponse:
    return ToolResponse(isError=False, content=content, structuredContent=extra)


def error(content: str, category: ErrorCategory, is_retryable: bool, description: str) -> ToolResponse:
    return ToolResponse(
        isError=True,
        content=content,
        structuredContent={
            "errorCategory": category,
            "isRetryable": is_retryable,
            "description": description,
        },
    )
```

> [!warning] Pregunta trampa en código — devolver solo un string en vez de `ToolResponse`
> La forma ingenua sería que `query_refund_database` simplemente hiciera `raise Exception("Operation failed")` o retornara un string plano cuando algo sale mal. **Por qué es mala idea:** esto es exactamente el "Operation failed" genérico del Argumento central del resumen — el agente recibe una excepción o un texto sin `errorCategory` ni `isRetryable`, y no tiene con qué decidir si reintentar, corregir el input, o escalar.

## Paso 2 — La tool: distinguir fallo de acceso de resultado vacío válido

Implementamos `query_refund_database`, aplicando primero la distinción de mayor peso del resumen ([[1 resumen#4. La distinción más evaluada fallo de acceso vs. resultado vacío válido|Conclusión 4]]): una query que corre bien y no encuentra nada **no** es un error.

```python
import random


class RefundDatabase:
    """Simula una base de datos de reembolsos con fallos intermitentes."""

    def __init__(self):
        self._orders = {"ORD-1001": 120.00, "ORD-1002": 750.00}

    def query(self, order_id: str) -> float | None:
        if random.random() < 0.3:
            raise TimeoutError("Connection to the customer database timed out after 5 seconds")
        return self._orders.get(order_id)  # None = no encontrado, no es un fallo


def query_refund_database(order_id: str, db: RefundDatabase) -> ToolResponse:
    try:
        amount = db.query(order_id)
    except TimeoutError as exc:
        # Error de ejecución transitorio: la tool sí se invocó, pero el servicio no respondió a tiempo.
        return error(
            content=str(exc),
            category="transient",
            is_retryable=True,
            description="Database connection timed out. Safe to retry after a short delay.",
        )

    if amount is None:
        # Resultado vacío VÁLIDO — la query tuvo éxito, no hay refund_amount para este order_id.
        # isError=False a propósito: esto NO es un fallo de acceso.
        return success(
            content=f"No refund record found for {order_id}.",
            resultCount=0,
        )

    return success(content=f"Refund amount for {order_id}: {amount}", resultCount=1, amount=amount)
```

> [!warning] Pregunta trampa en código — marcar el resultado vacío como error
> La forma ingenua sería que, cuando `db.query()` devuelve `None`, la tool retorne `error(..., category="business", ...)` porque "no se encontró el pedido". **Por qué es mala idea:** confunde un resultado vacío legítimo (la query corrió bien, simplemente no hay ese `order_id`) con un fallo de acceso real. Es exactamente la Trampa de examen 1 del resumen — el agente, al ver `isError: true`, podría reintentar una query que ya dio su respuesta correcta, desperdiciando esfuerzo en un resultado que nunca va a cambiar.

## Paso 3 — Validación y reglas de negocio: las otras tres categorías

Extendemos la tool para cubrir `validation`, `business` y `permission`, aplicando la tabla de la [[1 resumen#2. Las cuatro categorías de error de ejecución|Conclusión 2]].

```python
REFUND_POLICY_LIMIT = 500.00


def request_refund(order_id: str, amount: float, has_permission: bool, db: RefundDatabase) -> ToolResponse:
    if not order_id.startswith("ORD-"):
        # Validation: el input está mal formado. No tiene sentido reintentar tal cual.
        return error(
            content=f"'{order_id}' is not a valid order ID.",
            category="validation",
            is_retryable=False,
            description="Order IDs must follow the format 'ORD-NNNN'. Correct the input and resend.",
        )

    if not has_permission:
        # Permission: acceso denegado con las credenciales actuales.
        return error(
            content="Access denied for refund operations.",
            category="permission",
            is_retryable=False,
            description="The current credentials lack refund-issuing permissions. Use an account with refund authority.",
        )

    if amount > REFUND_POLICY_LIMIT:
        # Business: viola una regla de negocio, no un problema técnico.
        # Nótese la explicación customer-friendly además del booleano (Conclusión 2).
        return error(
            content=f"Refund of {amount} exceeds the automatic refund limit.",
            category="business",
            is_retryable=False,
            description=(
                f"Refunds above {REFUND_POLICY_LIMIT} require manager approval. "
                f"This refund of {amount} must be escalated to a human agent."
            ),
        )

    db._orders[order_id] = 0.0
    return success(content=f"Refund of {amount} issued for {order_id}.")
```

> [!warning] Pregunta trampa en código — devolver `isRetryable: false` sin `description` en el caso de negocio
> La forma ingenua sería, para el caso de `business`, retornar solo `{"errorCategory": "business", "isRetryable": False}` sin una `description` explicando el porqué. **Por qué es mala idea:** un booleano por sí solo le dice al agente que no reintente, pero no le da con qué comunicarle al usuario que el reembolso de £750 excede el límite de £500 y necesita aprobación — exactamente lo que señala la Conclusión 2 del resumen sobre errores de negocio.

## Paso 4 — El agente: leer `structuredContent` para decidir el siguiente paso

El agente nunca debería ramificar sobre el texto de `content` — debe leer los campos estructurados.

```python
import time


def handle_tool_response(response: ToolResponse, retries_left: int = 3) -> str:
    if not response.isError:
        # Éxito real, incluyendo el caso de resultado vacío legítimo (resultCount == 0).
        return response.content

    category = response.structuredContent["errorCategory"]
    is_retryable = response.structuredContent["isRetryable"]
    description = response.structuredContent["description"]

    if is_retryable and retries_left > 0:
        time.sleep(1)  # backoff real usaría espera exponencial
        return f"Retrying after transient error ({category}): {description}"

    if category in ("business", "permission"):
        return f"Escalating to human agent — {description}"

    # validation, u otros no-retryables sin escalar: se necesita corregir el input.
    return f"Cannot proceed automatically: {description}"
```

## Paso 5 — Recuperación local en subagentes, propagación selectiva al coordinador

Aplicamos la [[1 resumen#5. Recuperación local en subagentes y propagación selectiva|Conclusión 5]]: el subagente resuelve localmente lo transitorio, y solo propaga al coordinador lo que no pudo resolver — junto con resultados parciales y qué se intentó.

```python
from dataclasses import dataclass


@dataclass
class SubagentReport:
    resolved: bool
    partial_results: list[str]
    attempted: list[str]
    escalation: ToolResponse | None = None


def search_subagent_run(order_ids: list[str], db: RefundDatabase, max_local_retries: int = 3) -> SubagentReport:
    partial_results: list[str] = []
    attempted: list[str] = []

    for order_id in order_ids:
        attempted.append(order_id)
        response = None
        for attempt in range(max_local_retries):
            response = query_refund_database(order_id, db)
            if not response.isError:
                break
            if not response.structuredContent.get("isRetryable"):
                break  # no tiene sentido seguir reintentando algo no-retryable

        if response.isError:
            # No se pudo resolver localmente tras los reintentos permitidos:
            # se propaga al coordinador CON el contexto estructurado, no un status genérico.
            return SubagentReport(
                resolved=False,
                partial_results=partial_results,
                attempted=attempted,
                escalation=response,
            )

        partial_results.append(response.content)

    return SubagentReport(resolved=True, partial_results=partial_results, attempted=attempted)
```

> [!warning] Pregunta trampa en código — devolver un status genérico tras agotar los reintentos locales
> La forma ingenua sería que, al agotar `max_local_retries`, el subagente retorne solo `"search unavailable"` como string, sin `partial_results`, `attempted`, ni el `ToolResponse` original. **Por qué es mala idea** (Trampa de examen 2 del resumen, y Sample Question 8 de `examguide.pdf`): eso oculta al coordinador el tipo de fallo, qué se alcanzó a procesar, y qué se intentó — información que necesita para decidir entre reintentar con otro enfoque, continuar con lo parcial, o escalar a un humano. `SubagentReport` existe precisamente para no perder ese contexto al propagar.

> [!warning] Pregunta trampa en código — capturar el fallo final y marcarlo como éxito
> Otra forma ingenua sería que, si tras los reintentos `response.isError` sigue en `True`, el subagente devuelva de todos modos `SubagentReport(resolved=True, ...)` con lo que alcanzó a juntar, para "no molestar" al coordinador con un fallo. **Por qué es mala idea** (Trampa de examen 3): esto invierte la distinción de la Conclusión 4 — disfraza un fallo de acceso real de resultado exitoso, y le quita al coordinador cualquier posibilidad real de recuperación sin que nadie lo note.

## Paso 6 — El coordinador decide con el contexto propagado

```python
def coordinator_handle(report: SubagentReport) -> str:
    if report.resolved:
        return f"Search complete: {report.partial_results}"

    # El coordinador SÍ ve tipo de fallo, query intentada y resultados parciales —
    # exactamente lo que evalúa la Sample Question 8 del examen oficial.
    category = report.escalation.structuredContent["errorCategory"]
    description = report.escalation.structuredContent["description"]
    return (
        f"Subagent could not fully resolve ({category}): {description}. "
        f"Attempted: {report.attempted}. Partial results so far: {report.partial_results}. "
        f"Deciding: retry with modified query, try alternative source, or proceed with partial results."
    )
```

Con esto, el flujo completo respeta las cinco conclusiones del resumen: la tool distingue error de ejecución de error de protocolo (nunca lanza excepciones crudas al agente), categoriza cada fallo correctamente, nunca confunde resultado vacío con fallo de acceso, y el subagente resuelve lo que puede localmente antes de propagar contexto estructurado — nunca un genérico, nunca un éxito disfrazado.
