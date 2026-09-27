Aplicación práctica de [[1 resumen]] — un agente de soporte al cliente que decide cuándo escalar a un humano y cómo resolver ambigüedad de identidad, usando la Messages API de Claude en Python.

## Escenario

Un agente de soporte responde consultas sobre pedidos. Necesitamos que aplique los tres triggers válidos de escalación, evite los dos antipatrones, maneje frustración vs. solicitud explícita, y resuelva ambigüedad de identidad pidiendo identificadores en vez de adivinar — todo calibrado principalmente vía `system prompt` con `few-shot examples`, siguiendo la mejor práctica del resumen.

## Paso 1 — Definir el `system prompt` con criterios explícitos y `few-shot examples`

Esta es la mejor práctica central del resumen: antes de construir clasificadores o análisis de sentimiento, se calibra el comportamiento con reglas explícitas y ejemplos concretos directamente en el `system prompt`.

```python
import anthropic

client = anthropic.Anthropic()

SYSTEM_PROMPT = """Eres un agente de soporte al cliente. Sigue estas reglas de escalación
de forma estricta:

ESCALA A UN HUMANO únicamente cuando:
1. El cliente pide explícitamente hablar con una persona.
2. La solicitud del cliente no está cubierta por la política documentada (una excepción
   o vacío de política, no una violación clara).
3. Ya intentaste resolver el caso y una herramienta falla, falta acceso a un sistema,
   o hay un bug técnico que requiere ingeniería.

NO escales solo porque el cliente suena frustrado o enojado. La frustración no indica
que el caso sea complejo. Si el problema está dentro de tu capacidad, reconoce la
emoción del cliente y ofrece la solución directamente.

NO uses tu propia confianza sobre el caso como criterio de escalación.

Ejemplos:
- Cliente: "¡Esto es un desastre, mi pedido llegó tarde, arréglenlo YA!"
  -> NO escalar. Reconocer la frustración y ofrecer resolución (ej. reembolso de envío).
- Cliente: "Quiero hablar con una persona, ya."
  -> Escalar de inmediato, sin investigar antes.
- Cliente: "¿Aplican el precio de un competidor a mi compra ya hecha?" (la política solo
  cubre ajustes de precio dentro del propio sitio, no menciona competidores)
  -> Escalar: vacío de política, no violación.
- Cliente: "Quiero un reembolso fuera del plazo de 30 días que dice la política."
  -> NO escalar. Es una violación de política clara: se aplica la política (se niega
     el reembolso o se ofrece la alternativa documentada), no se escala.
"""
```

> [!danger] Pregunta trampa — construir un clasificador de sentimiento en vez de esto
> ```python
> # NO HACER: escalar según un score de sentimiento
> sentimiento = analizar_sentimiento(mensaje_cliente)  # ej. modelo externo -1.0 a 1.0
> if sentimiento < -0.5:
>     escalar_a_humano()
> ```
> **¿Por qué sería mala idea?** Esto es exactamente la Trampa 1 del resumen: la frustración no correlaciona con la complejidad del caso. Un cliente con un envío tardío (caso trivial) puede tener un sentimiento muy negativo, mientras un cliente educado con un problema genuinamente irresoluble puede sonar neutral. El criterio correcto vive en las reglas explícitas del `system prompt`, no en una medición de tono.

## Paso 2 — Definir la herramienta de búsqueda de cliente (que puede devolver múltiples matches)

La herramienta simula un caso real: buscar por nombre puede devolver más de un cliente.

```python
tools = [
    {
        "name": "buscar_cliente",
        "description": "Busca cliente(s) por nombre. Puede devolver 0, 1, o varios matches.",
        "input_schema": {
            "type": "object",
            "properties": {"nombre": {"type": "string"}},
            "required": ["nombre"],
        },
    },
    {
        "name": "buscar_cliente_por_identificador",
        "description": "Busca un cliente único por email, teléfono o número de orden.",
        "input_schema": {
            "type": "object",
            "properties": {
                "identificador": {"type": "string", "description": "email, teléfono u orden"}
            },
            "required": ["identificador"],
        },
    },
]

def buscar_cliente(nombre: str):
    base_datos = [
        {"id": 101, "nombre": "Ana Torres", "email": "ana.t@example.com"},
        {"id": 102, "nombre": "Ana Torres", "email": "ana.torres@example.com"},
    ]
    return [c for c in base_datos if c["nombre"].lower() == nombre.lower()]
```

## Paso 3 — El `system prompt` también cubre ambigüedad de identidad

Se agrega la regla explícita de identificadores al mismo `system prompt`, en vez de programar una heurística de selección en el código del agente.

```python
SYSTEM_PROMPT += """

Si `buscar_cliente` devuelve más de un resultado, NUNCA elijas uno por tu cuenta
(ni el más reciente, ni el primero de la lista). Pide al cliente un identificador
adicional (email, teléfono, o número de orden) y usa `buscar_cliente_por_identificador`
para confirmar el registro correcto antes de continuar.
"""
```

> [!danger] Pregunta trampa — elegir el primer match automáticamente
> ```python
> # NO HACER: quedarse con el primer resultado cuando hay varios
> resultados = buscar_cliente("Ana Torres")
> cliente = resultados[0]  # ¿y si hay dos "Ana Torres" distintas?
> ```
> **¿Por qué sería mala idea?** Es la Trampa 4 del resumen: con dos clientes llamadas "Ana Torres", tomar el primer resultado puede exponer datos de la persona equivocada o aplicar una acción (reembolso, cambio de pedido) sobre la cuenta incorrecta. La forma correcta es pedir un identificador adicional y confirmar con `buscar_cliente_por_identificador`.

## Paso 4 — El loop del agente: dejar que las reglas del `system prompt` decidan, no el código

El agente sigue el `agentic loop` estándar (`stop_reason` como única señal de control); lo que cambia aquí es que la lógica de escalación y desambiguación vive en el `system prompt`, no en `if`s dispersos en Python.

```python
def ejecutar_herramienta(nombre, argumentos):
    if nombre == "buscar_cliente":
        return buscar_cliente(argumentos["nombre"])
    if nombre == "buscar_cliente_por_identificador":
        return {"id": 102, "nombre": "Ana Torres", "email": argumentos["identificador"]}
    raise ValueError(f"Herramienta desconocida: {nombre}")


def correr_agente(mensaje_usuario: str) -> str:
    messages = [{"role": "user", "content": mensaje_usuario}]

    for _ in range(20):  # red de seguridad, nunca control principal
        response = client.messages.create(
            model="claude-sonnet-5",
            max_tokens=1024,
            system=SYSTEM_PROMPT,
            tools=tools,
            messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    resultado = ejecutar_herramienta(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(resultado),
                    })
            messages.append({"role": "user", "content": tool_results})
            continue

        if response.stop_reason == "end_turn":
            return "".join(b.text for b in response.content if b.type == "text")

        raise RuntimeError(f"stop_reason inesperado: {response.stop_reason}")

    raise RuntimeError("Loop no terminó — posible bucle descontrolado")
```

## Paso 5 — Aplicando el criterio de "incapacidad de progresar"

El tercer trigger (fallas técnicas reales) se maneja devolviendo un error estructurado desde la herramienta, para que Claude —siguiendo el `system prompt`— decida escalar solo cuando la herramienta realmente falla, no como reacción al primer obstáculo.

```python
def buscar_cliente_con_falla_simulada(nombre: str):
    try:
        return buscar_cliente(nombre)
    except ConnectionError as e:
        return {
            "error": "sistema_no_disponible",
            "detalle": str(e),
            "intentos_previos": ["buscar_cliente"],
        }
```

> [!danger] Pregunta trampa — escalar ante el primer error de herramienta sin reintentar
> ```python
> # NO HACER: escalar automáticamente en cualquier excepción, sin contexto
> try:
>     resultado = buscar_cliente(nombre)
> except Exception:
>     return "Te voy a transferir con un humano."
> ```
> **¿Por qué sería mala idea?** El trigger 3 exige "intentos reales de resolución" antes de escalar, no una reacción automática a cualquier excepción. Además, no distingue un error transitorio (reintentable) de uno genuinamente bloqueante. La forma correcta es devolver contexto estructurado del error y dejar que la lógica de decisión (reglas del `system prompt`) determine si ya se agotaron los intentos razonables.

## Resultado

```python
print(correr_agente("¡Esto es un desastre, mi pedido llegó tarde, arréglenlo YA!"))
# -> Reconoce la frustración y ofrece resolución (no escala) — regla 1 del ejemplo del prompt.

print(correr_agente("Quiero hablar con una persona, ya."))
# -> Escala de inmediato, sin investigar — trigger 1.

print(correr_agente("Necesito ayuda con el pedido de Ana Torres."))
# -> buscar_cliente devuelve 2 matches; el agente pide email/teléfono/orden en vez de adivinar.
```

El flujo completo aplicó: los tres triggers válidos de escalación, evitó los dos antipatrones (sentimiento, confianza autorreportada), distinguió frustración de solicitud explícita, resolvió ambigüedad de identidad pidiendo identificadores, y calibró todo esto principalmente vía `system prompt` con `few-shot examples` — la mejor práctica del resumen.
