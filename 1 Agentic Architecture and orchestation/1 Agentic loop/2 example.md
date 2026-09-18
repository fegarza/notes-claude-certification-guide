Aplicación práctica de [[1 resumen]] — implementación completa, paso a paso, de un agentic loop con la Messages API de Claude en Python.

## Escenario

Un agente de soporte al cliente que puede consultar el estado de un pedido usando una herramienta (`get_order_status`), y responder al usuario una vez que tiene la información. Vamos a construirlo aplicando cada idea clave del resumen: el ciclo de 4 pasos, `stop_reason` como única señal de control, y evitar las 4 trampas de examen — mostrando explícitamente por qué la forma "ingenua" de cada una es mala idea en código.

## Paso 1 — Configurar el cliente y definir la herramienta

Primero se define el cliente de la API y el schema de la herramienta que Claude podrá usar. El `input_schema` sigue el estándar de JSON Schema — así Claude sabe exactamente qué argumentos puede pedir.

```python
import anthropic

client = anthropic.Anthropic()

tools = [
    {
        "name": "get_order_status",
        "description": "Busca el estado de un pedido por su ID",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {"type": "string", "description": "ID del pedido"}
            },
            "required": ["order_id"],
        },
    }
]
```

## Paso 2 — Iniciar el historial de conversación

El historial (`messages`) es lo que se reenvía completo en cada vuelta del loop. Arranca solo con el mensaje del usuario.

```python
messages = [{"role": "user", "content": "¿Dónde está mi pedido #4821?"}]
```

## Paso 3 — Paso 1 del ciclo de vida: enviar la solicitud

Dentro del loop, cada iteración empieza enviando el historial completo a la Messages API, incluyendo las herramientas disponibles.

```python
while True:
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        tools=tools,
        messages=messages,
    )
```

## Paso 4 — Agregar la respuesta de Claude al historial

Este paso es el "Punto crítico" del resumen: la respuesta de Claude (que puede traer texto y un bloque `tool_use` juntos) se agrega al historial antes de decidir qué hacer.

```python
    messages.append({"role": "assistant", "content": response.content})
```

## Paso 5 — Paso 2 del ciclo de vida: inspeccionar `stop_reason`

Aquí se aplica la idea central del resumen: `stop_reason` es la única señal confiable, nunca el tipo de contenido ni el texto.

```python
    if response.stop_reason == "tool_use":
        ...
    elif response.stop_reason == "end_turn":
        ...
    else:
        ...
```

> [!danger] Pregunta trampa — Trampa 1 en código
> ```python
> # ❌ NO HACER: chequear el tipo de contenido en vez de stop_reason
> if response.content[0].type == "text":
>     return response.content[0].text  # loop termina aquí
> ```
> **¿Por qué sería mala idea?** Si `response.content` trae `[texto, tool_use]` (Claude "pensando en voz alta" antes de llamar la herramienta), este chequeo corta el loop en el primer bloque de texto y nunca llega a ejecutar la herramienta — exactamente el bug del caso real del resumen: el agente responde "Déjame revisar tu pedido" y ahí se detiene, sin dar el estado real del pedido.

## Paso 6 — Paso 3 del ciclo de vida: ejecutar la(s) herramienta(s) y agregar resultados

Cuando `stop_reason == "tool_use"`, se recorren los bloques de la respuesta buscando los `tool_use`, se ejecuta cada uno, y el resultado se agrega al historial como un nuevo mensaje — nunca se descarta.

```python
    if response.stop_reason == "tool_use":
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = ejecutar_herramienta(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result),
                })
        messages.append({"role": "user", "content": tool_results})
        continue  # vuelve al paso 3 con el historial actualizado
```

```python
def ejecutar_herramienta(nombre, argumentos):
    if nombre == "get_order_status":
        return {"order_id": argumentos["order_id"], "status": "en camino", "eta_dias": 2}
    raise ValueError(f"Herramienta desconocida: {nombre}")
```

> [!danger] Pregunta trampa — forzar `tool_choice` a "any"
> ```python
> # ❌ NO HACER: forzar que Claude siempre use una herramienta
> response = client.messages.create(
>     model="claude-sonnet-5",
>     max_tokens=1024,
>     tools=tools,
>     tool_choice={"type": "any"},  # obliga a llamar una herramienta SIEMPRE
>     messages=messages,
> )
> ```
> **¿Por qué sería mala idea?** Una vez que Claude ya resolvió la consulta del usuario y quiere terminar (`end_turn` genuino), `tool_choice: any` lo obliga a inventar otra llamada a herramienta en vez de terminar. El loop nunca ve `end_turn`, sigue creyendo que hay más trabajo por hacer, y entra en un ciclo que no se detiene solo — la Trampa 4 del resumen.

## Paso 7 — Paso 4 del ciclo de vida: terminar en `end_turn`

Cuando Claude ya no necesita más herramientas, `stop_reason` cambia a `"end_turn"` y se extrae el texto final para mostrarlo al usuario.

```python
    elif response.stop_reason == "end_turn":
        texto_final = "".join(b.text for b in response.content if b.type == "text")
        break
```

> [!danger] Pregunta trampa — parsear lenguaje natural
> ```python
> # ❌ NO HACER: adivinar el final por frases de texto
> if "ya terminé" in texto_final.lower() or "listo" in texto_final.lower():
>     break
> ```
> **¿Por qué sería mala idea?** Claude podría terminar genuinamente sin usar esas frases exactas (loop nunca termina), o podría mencionar la palabra "listo" en medio de una explicación sin haber terminado (loop termina antes de tiempo). El lenguaje natural no es una señal binaria y confiable — la Trampa 3 del resumen.

## Paso 8 — Manejar cualquier otro `stop_reason`

Siguiendo la idea de "más allá de los dos valores del examen": cualquier valor que no sea `tool_use` ni `end_turn` (`pause_turn`, `max_tokens`, `stop_sequence`, `refusal`, `model_context_window_exceeded`) se trata como "no terminado, hay que investigar" — nunca se asume que es equivalente a `tool_use`.

```python
    else:
        raise RuntimeError(f"stop_reason inesperado, revisar: {response.stop_reason}")
```

## Paso 9 — Red de seguridad: límite de iteraciones (nunca como control principal)

Por último, se agrega un límite de iteraciones, pero solo como red de seguridad — nunca como el mecanismo que decide cuándo parar.

```python
MAX_ITERACIONES = 20

def correr_agente(mensaje_usuario: str) -> str:
    messages = [{"role": "user", "content": mensaje_usuario}]

    for i in range(MAX_ITERACIONES):
        response = client.messages.create(
            model="claude-sonnet-5",
            max_tokens=1024,
            tools=tools,
            messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = ejecutar_herramienta(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result),
                    })
            messages.append({"role": "user", "content": tool_results})
            continue

        elif response.stop_reason == "end_turn":
            return "".join(b.text for b in response.content if b.type == "text")

        else:
            raise RuntimeError(f"stop_reason inesperado: {response.stop_reason}")

    # Solo se llega aquí si algo salió mal — red de seguridad, no control normal
    raise RuntimeError(f"Loop no terminó tras {MAX_ITERACIONES} iteraciones — posible bucle descontrolado")
```

> [!danger] Pregunta trampa — Trampa 2 en código
> ```python
> # ❌ NO HACER: usar el contador como condición principal de salida
> for i in range(10):
>     ...
>     if i == 9:
>         return "Lo siento, no pude completar tu solicitud"  # corta trabajo real
> ```
> **¿Por qué sería mala idea?** Si la tarea necesita 12 llamadas a herramientas para completarse (ej. buscar en varios sistemas), se corta en la iteración 10 con una respuesta incompleta, aunque `stop_reason` nunca haya dicho `end_turn`. El contador reemplazando a `stop_reason` como control principal es la Trampa 2: corta trabajo útil que sí estaba a punto de terminar bien.

## Resultado

```python
respuesta = correr_agente("¿Dónde está mi pedido #4821?")
print(respuesta)
# "Tu pedido #4821 está en camino y llega en aproximadamente 2 días."
```

El flujo completo aplicó: el ciclo de 4 pasos, `stop_reason` como único control, agregar siempre los resultados de herramientas al historial, un límite de iteraciones solo como red de seguridad, y evitó explícitamente las 4 trampas de examen del resumen.
