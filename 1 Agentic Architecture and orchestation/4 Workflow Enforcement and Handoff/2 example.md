Aplicación práctica de [[1 resumen]] — un agente de soporte al cliente en Python que implementa una prerequisite gate programática para reembolsos, decompone solicitudes multi-concern, y compila un handoff estructurado hacia un humano.

## Escenario

El mismo caso del 8% de fallo del resumen: un agente de soporte tiene las herramientas `get_customer`, `lookup_order` y `process_refund`. El system prompt ya dice "siempre verifica la identidad antes de reembolsar", pero eso solo reduce el fallo, no lo elimina. Vamos a construir la versión que sí lo elimina: enforcement programático, no prompt más fuerte.

## Paso 1 — Las herramientas, sin protección todavía

Primero, las tres herramientas base tal cual las usaría el agente, sin ninguna gate. Este es el punto de partida — y también la versión que produce el 8% de fallo.

```python
sesiones_verificadas: dict[str, str] = {}  # session_id -> customer_id verificado

def get_customer(session_id: str, name_or_email: str) -> dict:
    customer_id = buscar_cliente_en_crm(name_or_email)  # asume que existe en el sistema real
    sesiones_verificadas[session_id] = customer_id
    return {"customer_id": customer_id, "verified": True}

def lookup_order(order_id: str) -> dict:
    return obtener_orden_de_bd(order_id)

def process_refund(session_id: str, customer_id: str, amount: float) -> dict:
    # todavía sin gate: nada impide llamar esto sin haber pasado por get_customer
    return ejecutar_reembolso_en_pasarela_de_pago(customer_id, amount)
```

> [!danger] Pregunta trampa — confiar en que el prompt baste
> ```python
> system_prompt = (
>     "Always verify the customer's identity using get_customer "
>     "before calling process_refund."
> )
> ```
> **¿Por qué sería mala idea dejarlo así?** Esta instrucción es exactamente la que ya existe en el caso del 8% de fallo del resumen — funciona el 92% de las veces, pero el modelo puede saltarse el paso, y ese 8% ya generó reembolsos en cuentas equivocadas. Ninguna reescritura del prompt, por más fuerte que sea, elimina la tasa de fallo a cero. Se necesita una gate en `process_refund`, no una mejora de redacción.

## Paso 2 — La prerequisite gate: bloquear `process_refund` en código

Ahora agregamos el check programático. `process_refund` verifica el estado de la sesión **antes** de ejecutar nada — el modelo no puede evitar este check decidiendo llamar la función distinto.

```python
class RefundBloqueadoError(Exception):
    pass

def process_refund(session_id: str, customer_id: str, amount: float) -> dict:
    verificado = sesiones_verificadas.get(session_id)
    if verificado is None or verificado != customer_id:
        # la gate: código, no una instrucción de prompt — no hay forma de que
        # el modelo "decida" saltarse esto
        raise RefundBloqueadoError(
            "Cannot process refund — customer identity not verified. "
            "Please call get_customer first."
        )
    return ejecutar_reembolso_en_pasarela_de_pago(customer_id, amount)
```

Con esta gate, si el modelo intenta llamar a `process_refund` directamente sin haber invocado `get_customer` en la sesión, la llamada se bloquea con un error explícito — y ese error es lo que fuerza al modelo a verificar primero antes de reintentar. Esto elimina el 8% de fallo por completo, no reduciéndolo, porque ya no depende de que el modelo interprete bien una instrucción.

## Paso 3 — Decisión de cuándo SÍ basta con prompt

No todo necesita una gate. Para ilustrar la regla de decisión del examen, agregamos una preferencia de bajo riesgo (formato de moneda en la respuesta) que sí se resuelve con prompt.

```python
system_prompt_bajo_riesgo = (
    "When mentioning any amount to the customer, format it as USD with two "
    "decimal places (e.g., $42.50), not as a raw number."
)
```

> [!note] Por qué esta parte NO necesita una gate
> Una inconsistencia de formato no genera pérdida de dinero, brecha de seguridad ni violación de cumplimiento — es exactamente el tipo de caso de bajo riesgo donde la guía basada en prompt es aceptable. Aplicarle una gate programática sería sobre-ingeniería para un riesgo que no existe.

## Paso 4 — Decomponer una solicitud multi-concern

El cliente escribe: *"quiero devolver mi pedido #4521, actualizar mi dirección, y preguntar por mis puntos de lealtad"*. El coordinador debe descomponer, investigar en paralelo con contexto compartido, y sintetizar una sola respuesta — no atender solo el primer asunto.

```python
def decomponer_solicitud(mensaje_cliente: str) -> list[dict]:
    # en producción esto lo hace el modelo; aquí se muestra la estructura esperada
    return [
        {"tipo": "devolucion", "order_id": "4521"},
        {"tipo": "actualizar_direccion"},
        {"tipo": "consulta_puntos_lealtad"},
    ]

def investigar_asuntos_en_paralelo(asuntos: list[dict], customer_id: str) -> list[dict]:
    resultados = []
    for asunto in asuntos:
        # cada investigación usa el mismo customer_id ya verificado (contexto compartido)
        resultados.append(investigar_asunto(asunto, customer_id))
    return resultados

def sintetizar_resolucion_unificada(resultados: list[dict]) -> str:
    partes = [formatear_resultado(r) for r in resultados]
    return "\n\n".join(partes)  # una sola respuesta que cubre los tres asuntos
```

> [!danger] Pregunta trampa — atender solo el primer asunto
> ```python
> def manejar_solicitud_mal(mensaje_cliente: str, customer_id: str) -> str:
>     asunto_principal = decomponer_solicitud(mensaje_cliente)[0]  # ❌ ignora los demás
>     return formatear_resultado(investigar_asunto(asunto_principal, customer_id))
> ```
> **¿Por qué sería mala idea?** El cliente pidió tres cosas y solo se le responde una — el examen espera descomposición completa, investigación en paralelo de *todos* los asuntos, y una síntesis unificada, no una resolución parcial ni una conversación secuencial separada por asunto.

## Paso 5 — El handoff estructurado cuando el agente no puede resolver

Si alguno de los asuntos requiere un humano (por ejemplo, una excepción de política que el agente no puede autorizar), el agente compila un resumen autocontenido — recordando que el humano **no** ve la transcripción.

```python
from dataclasses import dataclass

@dataclass
class ResumenHandoff:
    customer_id: str
    resumen_conversacion: str
    analisis_causa_raiz: str
    monto_reembolso: float | None
    accion_recomendada: str

def compilar_handoff(customer_id: str, resultados: list[dict]) -> ResumenHandoff:
    return ResumenHandoff(
        customer_id=customer_id,
        resumen_conversacion=(
            "Cliente solicitó devolución del pedido #4521, actualización de dirección "
            "y consulta de puntos de lealtad. Devolución y actualización de dirección "
            "resueltas; la consulta de puntos requiere excepción de política."
        ),
        analisis_causa_raiz=(
            "El programa de lealtad del cliente fue migrado hace 3 meses y sus puntos "
            "previos no se transfirieron automáticamente al nuevo sistema."
        ),
        monto_reembolso=38.50,
        accion_recomendada=(
            "Aprobar transferencia manual de 1,200 puntos de lealtad del sistema "
            "anterior al nuevo, según el registro de migración adjunto."
        ),
    )
```

> [!danger] Pregunta trampa — un handoff incompleto
> ```python
> def compilar_handoff_mal(customer_id: str) -> str:
>     # ❌ le falta resumen de conversación, causa raíz y acción recomendada
>     return f"Cliente {customer_id} necesita ayuda con puntos de lealtad."
> ```
> **¿Por qué sería mala idea?** El humano no tiene acceso a la conversación — esta única línea no le da nada para actuar. Sin causa raíz ni acción recomendada, el humano tiene que empezar de cero y probablemente le pida al cliente que repita todo lo que ya explicó. El resumen de handoff debe ser autocontenido con los cinco campos, no un aviso genérico.

## Paso 6 (extra) — Validar la salida de un subagente con `SubagentStop`

Como contenido de fondo del resumen: si la síntesis del handoff la produce un subagente dedicado, se puede validar su forma antes de aceptarla, usando el hook `SubagentStop` — que bloquea la finalización devolviendo código 2, sin reescribir el resultado.

```python
import sys
import json

def validar_subagent_output():
    entrada = json.load(sys.stdin)
    mensaje_final = entrada.get("final_message", "")
    campos_requeridos = ["customer_id", "resumen_conversacion", "analisis_causa_raiz", "accion_recomendada"]
    if not all(campo in mensaje_final for campo in campos_requeridos):
        # código 2: bloquea la finalización, el subagente vuelve a trabajar
        sys.exit(2)
    sys.exit(0)

if __name__ == "__main__":
    validar_subagent_output()
```

```json
{
  "hooks": {
    "SubagentStop": [
      { "hooks": [{ "type": "command", "command": "python .claude/hooks/validate-subagent-output.py" }] }
    ]
  }
}
```

> [!danger] Pregunta trampa — esperar que el hook "arregle" la salida
> ```python
> def validar_subagent_output_mal():
>     entrada = json.load(sys.stdin)
>     entrada["final_message"]["accion_recomendada"] = "revisar manualmente"  # ❌
>     print(json.dumps(entrada))  # esto no reescribe nada real
>     sys.exit(0)
> ```
> **¿Por qué sería mala idea?** `SubagentStop` no tiene un mecanismo documentado para reescribir el resultado del subagente en el flujo real — su única palanca es exit code 2 para bloquear la finalización y devolverlo a trabajar. Si el resultado está mal, la corrección ocurre porque el subagente lo reintenta, no porque el hook lo edite.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y practica con preguntas estilo examen en [[4 test]].
