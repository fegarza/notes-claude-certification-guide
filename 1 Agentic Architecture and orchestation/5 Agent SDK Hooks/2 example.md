Aplicación práctica de [[1 resumen]] — un agente de soporte con tres herramientas MCP que devuelven datos en formatos distintos, más dos políticas de negocio, implementadas con hooks `PreToolUse`/`PostToolUse` del Claude Agent SDK.

## Escenario

Un agente de soporte tiene tres herramientas: `get_customer` (timestamps Unix + códigos de estado numéricos), `lookup_order` (fechas ISO 8601 + strings de estado en inglés) y `check_shipping` (fechas `DD/MM/YYYY` + códigos de un solo carácter, `"S"`=shipped, `"P"`=pending). Además, el agente puede llamar `process_refund` y `transfer_funds`, que tienen reglas de negocio que deben cumplirse el 100% de las veces. Vamos a resolver ambos problemas con los dos tipos de hook: `PostToolUse` para normalizar, `PreToolUse` para bloquear.

## Paso 1 — Las herramientas, con sus formatos heterogéneos sin tocar

Este es el punto de partida: cada herramienta devuelve datos "tal como los da" su sistema backend real. Nada aquí normaliza nada todavía — es la fuente del problema que el resumen describe.

```python
from datetime import datetime, timezone

def get_customer(customer_id: str) -> dict:
    return {
        "customer_id": customer_id,
        "created_at": 1710489600,  # timestamp Unix
        "status": 200,             # código numérico
    }

def lookup_order(order_id: str) -> dict:
    return {
        "order_id": order_id,
        "created_at": "2024-03-15T12:00:00Z",  # ISO 8601
        "status": "pending",                    # string en inglés
    }

def check_shipping(order_id: str) -> dict:
    return {
        "order_id": order_id,
        "shipped_at": "15/03/2024",  # DD/MM/YYYY
        "status": "P",               # código de un carácter
    }
```

> [!danger] Pregunta trampa — dejar que el modelo interprete los tres formatos
> Si estas tres funciones se conectan directo al agente sin ningún hook, el modelo tiene que interpretar `1710489600`, `"2024-03-15T12:00:00Z"` y `"15/03/2024"` como la misma clase de dato (una fecha), y `200`, `"pending"` y `"P"` como la misma clase de dato (un estado) — en cada iteración. **¿Por qué sería mala idea confiar en esto?** Porque es exactamente el tipo de interpretación probabilística que el resumen advierte: el modelo puede convertir bien el timestamp Unix una vez y confundir el orden día/mes de `"15/03/2024"` la siguiente, o leer `"P"` como "processed" en vez de "pending". La solución no es "pedirle al modelo que tenga cuidado" — es normalizar antes de que el modelo vea el dato.

## Paso 2 — Hook `PostToolUse`: normalizar todo a un formato único

El hook se registra para las tres herramientas de lectura. Recibe el resultado crudo, lo normaliza, y devuelve `updatedToolOutput` — el modelo nunca ve el formato original.

```python
STATUS_MAP = {
    200: "active",
    404: "not_found",
    500: "error",
    "pending": "pending",
    "P": "pending",
    "S": "shipped",
}

def _to_iso8601(valor) -> str:
    if isinstance(valor, int):  # timestamp Unix
        return datetime.fromtimestamp(valor, tz=timezone.utc).isoformat()
    if isinstance(valor, str) and "/" in valor:  # DD/MM/YYYY
        dia, mes, anio = valor.split("/")
        return datetime(int(anio), int(mes), int(dia), tzinfo=timezone.utc).isoformat()
    return valor  # ya es ISO 8601

def normalizar_resultado_herramienta(nombre_herramienta: str, resultado_crudo: dict) -> dict:
    normalizado = dict(resultado_crudo)
    for campo_fecha in ("created_at", "shipped_at"):
        if campo_fecha in normalizado:
            normalizado[campo_fecha] = _to_iso8601(normalizado[campo_fecha])
    if "status" in normalizado:
        normalizado["status"] = STATUS_MAP.get(normalizado["status"], normalizado["status"])
    return normalizado

def post_tool_use_hook(nombre_herramienta: str, resultado_crudo: dict) -> dict:
    # se dispara después de que la herramienta ya corrió — solo transforma
    # lo que el modelo va a ver, nunca deshace nada
    return {"hookSpecificOutput": {"updatedToolOutput": normalizar_resultado_herramienta(nombre_herramienta, resultado_crudo)}}
```

Con este hook, `get_customer`, `lookup_order` y `check_shipping` siempre entregan `created_at`/`shipped_at` en ISO 8601 y `status` en el mismo vocabulario (`"active"`, `"pending"`, `"shipped"`, etc.), sin importar qué formato tenía el sistema backend original.

> [!danger] Pregunta trampa — normalizar dentro de cada función de herramienta
> ```python
> def get_customer_mal(customer_id: str) -> dict:
>     resultado = {"customer_id": customer_id, "created_at": 1710489600, "status": 200}
>     resultado["created_at"] = datetime.fromtimestamp(resultado["created_at"]).isoformat()  # ❌
>     return resultado
> ```
> **¿Por qué sería mala idea?** Funcionaría para esta herramienta, pero obliga a duplicar la misma lógica de normalización dentro de cada una de las tres herramientas (y de cualquier herramienta nueva que se agregue después), en vez de tener un único punto de transformación. Un hook `PostToolUse` centraliza la normalización para *cualquier* herramienta que pase por él, sin tocar el código de cada herramienta individual — que es justo el punto de usar un hook en vez de resolverlo función por función.

## Paso 3 — Hook `PreToolUse`: bloquear reembolsos sobre $500

Ahora la política de negocio: `process_refund` nunca debe ejecutarse para montos superiores a $500 sin escalación humana. El hook intercepta la llamada **antes** de que `process_refund` corra.

```python
def process_refund(customer_id: str, order_id: str, amount: float) -> dict:
    return {"refunded": True, "customer_id": customer_id, "amount": amount}

def pre_tool_use_refund_hook(nombre_herramienta: str, entrada: dict) -> dict:
    if nombre_herramienta != "process_refund":
        return {"hookSpecificOutput": {"permissionDecision": "allow"}}

    if entrada["amount"] > 500:
        # la herramienta nunca corre — no hay reembolso que deshacer después
        return {
            "hookSpecificOutput": {
                "permissionDecision": "deny",
                "permissionDecisionReason": (
                    f"Refund of ${entrada['amount']} exceeds the $500 threshold. "
                    "Route to human escalation before retrying."
                ),
            }
        }
    return {"hookSpecificOutput": {"permissionDecision": "allow"}}
```

> [!danger] Pregunta trampa — usar `PostToolUse` para "revertir" el reembolso
> ```python
> def post_tool_use_refund_mal(nombre_herramienta: str, resultado: dict) -> dict:
>     if nombre_herramienta == "process_refund" and resultado["amount"] > 500:
>         resultado["refunded"] = False  # ❌ esto no deshace el cargo real
>     return {"hookSpecificOutput": {"updatedToolOutput": resultado}}
> ```
> **¿Por qué sería mala idea?** Para el momento en que este hook se dispara, `process_refund` ya se ejecutó — el dinero ya salió en el sistema de pagos real. Cambiar el campo `refunded` en el resultado solo engaña al modelo sobre lo que pasó; no revierte el cargo. Bloquear un reembolso sobre $500 tiene que pasar en `PreToolUse`, antes de que la herramienta corra, no después.

## Paso 4 — Hook `PreToolUse`: prerrequisito AML antes de transferir fondos

Mismo patrón que el reembolso, pero la condición es "¿ya se completó el chequeo AML en esta sesión?" en vez de un umbral numérico. El estado de la verificación se guarda cuando `aml_check` se ejecuta, usando un `PostToolUse` sobre esa herramienta específica.

```python
sesiones_con_aml_verificado: set[str] = set()

def post_tool_use_aml_hook(nombre_herramienta: str, session_id: str, resultado: dict) -> dict:
    if nombre_herramienta == "aml_check" and resultado.get("passed"):
        sesiones_con_aml_verificado.add(session_id)
    return {"hookSpecificOutput": {"updatedToolOutput": resultado}}

def pre_tool_use_transfer_hook(nombre_herramienta: str, session_id: str) -> dict:
    if nombre_herramienta != "transfer_funds":
        return {"hookSpecificOutput": {"permissionDecision": "allow"}}

    if session_id not in sesiones_con_aml_verificado:
        return {
            "hookSpecificOutput": {
                "permissionDecision": "deny",
                "permissionDecisionReason": (
                    "transfer_funds blocked — AML check has not passed for this "
                    "session. Call aml_check first."
                ),
            }
        }
    return {"hookSpecificOutput": {"permissionDecision": "allow"}}
```

Con esto, `transfer_funds` está bloqueado el 100% de las veces hasta que `aml_check` haya devuelto `passed=True` en la sesión — no depende de que el modelo "recuerde" verificar antes de transferir.

## Paso 5 — Decidir cuándo NO usar un hook

Para cerrar el framework de decisión del resumen: una preferencia de formato (montos en `$X.XX`) no justifica un hook, porque una desviación ocasional no tiene consecuencia de negocio real.

```python
system_prompt_formato = (
    "When mentioning any amount to the customer, format it as USD with two "
    "decimal places (e.g., $42.50), not as a raw float."
)
```

> [!note] Por qué esto NO necesita `PreToolUse` ni `PostToolUse`
> Aplicar un hook para garantizar el formato de un número en texto sería sobre-ingeniería: una respuesta ocasional con formato inconsistente no pierde dinero ni viola cumplimiento. Es exactamente el caso de "preferencia de bajo riesgo" donde el prompt basta.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y practica con preguntas estilo examen en [[4 test]].
