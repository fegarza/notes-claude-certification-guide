Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Ciclo de vida del loop

> [!question]- ¿Qué se envía a Claude en cada vuelta del loop, además del último mensaje?
> Todo el historial de conversación: system prompt, mensajes previos y resultados de herramientas previas.

> [!question]- ¿Cuáles son los dos valores de `stop_reason` que evalúa el examen?
> `tool_use` y `end_turn`.

> [!question]- ¿Qué pasa cuando `stop_reason` es `tool_use`?
> Se ejecuta la herramienta solicitada, se agrega el resultado al historial, y se reenvía la conversación a Claude.

> [!question]- ¿Qué pasa cuando `stop_reason` es `end_turn`?
> El loop se detiene y se presenta la respuesta final al usuario.

> [!question]- ¿Qué ocurre si no se agregan los resultados de las herramientas al historial de conversación?
> Claude no puede razonar sobre esa información en la siguiente iteración — es como si la herramienta nunca se hubiera ejecutado.

## `stop_reason` como única señal

> [!question]- ¿Por qué `stop_reason` es preferible a analizar el texto de la respuesta?
> Porque es determinista e inequívoco, mientras que el texto es ambiguo y puede coexistir con una llamada a herramienta.

> [!question]- ¿Qué otros valores de `stop_reason` existen más allá de `tool_use` y `end_turn`?
> `pause_turn`, `max_tokens`, `stop_sequence`, `refusal`, `model_context_window_exceeded`.

> [!question]- ¿Cómo se debe tratar cualquier `stop_reason` que no sea `end_turn`?
> Como "todavía no terminó, hay que revisar por qué" — nunca asumir automáticamente que es `tool_use`.

## Model-driven decision-making

> [!question]- ¿Quién decide qué herramienta llamar en un agentic loop?
> Claude, según el contexto de la tarea actual — no una secuencia fija programada de antemano.

> [!question]- ¿Con qué contrasta la toma de decisiones dirigida por el modelo?
> Con árboles de decisión preconfigurados o secuencias fijas de herramientas codificadas a mano.

> [!question]- ¿Cuándo conviene imponer reglas programáticas en vez de dejar que el modelo decida?
> Cuando la lógica de negocio exige cumplimiento determinista y auditable — contextos financieros, de seguridad o regulatorios.

## Trampas de examen

> [!question]- ¿Por qué revisar `content[0].type == "text"` es una trampa?
> Porque Claude puede devolver texto junto con un bloque `tool_use` en la misma respuesta; la presencia de texto no indica que el turno terminó.

> [!question]- ¿Por qué un límite de iteraciones no debe ser el mecanismo principal de control del loop?
> Porque corta trabajo útil a la mitad o deja correr pasos innecesarios — no sabe si el agente realmente terminó.

> [!question]- ¿Cuál es el único uso correcto de un límite de iteraciones?
> Como red de seguridad ante un loop descontrolado, nunca como forma principal de decidir cuándo parar.

> [!question]- ¿Por qué parsear frases como "ya terminé" es una mala señal de control?
> Porque el lenguaje natural es ambiguo por naturaleza y no ofrece una señal confiable o determinista.

> [!question]- ¿Qué problema genera forzar `tool_choice` a "any"?
> Obliga a Claude a llamar siempre una herramienta, incluso cuando ya terminó genuinamente, lo que puede causar un loop infinito.

> [!question]- ¿Qué arreglo suele presentar el examen como distractor ante una terminación prematura del loop, y por qué está mal?
> Agregar un límite de iteraciones — está mal porque los límites resuelven loops que no terminan, no loops que terminan antes de tiempo; el arreglo correcto es siempre revisar `stop_reason`.

## Caso de bug real

> [!question]- En el bug del agente de soporte, ¿qué hacía Claude que confundía al código?
> Devolvía texto ("Déjame revisar tu pedido") junto con un bloque `tool_use` en la misma respuesta.

> [!question]- ¿Cuál fue la corrección exacta del bug?
> Reemplazar la revisión del tipo de contenido por una revisión de `stop_reason`: continuar en `tool_use`, terminar en `end_turn`.
