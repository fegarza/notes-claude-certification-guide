> [!note] En una frase
> Un agentic loop es el ciclo de ejecución determinista —definido en código, no en el prompt— que hace que Claude siga trabajando (llamando herramientas) hasta que él mismo señala, con el campo `stop_reason`, que terminó.

## Explícamelo como si tuviera 5 años

Imagina un juego de "caliente o frío" entre tú y un amigo que busca un objeto escondido. Tú no le dices "sigue buscando" a cada rato adivinando si ya terminó por cómo suena su voz — eso sería ambiguo y te equivocarías. En vez de eso, hay una regla clara: tu amigo levanta la mano cuando encuentra el objeto. Mientras no levante la mano, sigue buscando. Cuando la levanta, se acabó el juego.

El "agentic loop" es exactamente eso, pero con Claude: en vez de intentar adivinar si ya terminó leyendo lo que "dice" (texto ambiguo), el código revisa una señal clara y oficial —el campo `stop_reason`— que Claude manda en cada respuesta. Si la señal dice "quiero usar una herramienta", el programa ejecuta esa herramienta, le cuenta a Claude el resultado, y le vuelve a preguntar. Si la señal dice "ya terminé mi turno", el programa se detiene y muestra la respuesta final.

## Argumento central

> El agentic loop es el ciclo determinista de control de flujo (definido en código) detrás de todo agente basado en Claude — no es un truco de prompt, ni un loop de reintentos, ni simplemente un turno de chatbot. El único mecanismo confiable para controlarlo es el campo `stop_reason` de la respuesta de la Messages API.

## Ideas clave (Conclusiones)

### 1. El ciclo de vida del loop (4 pasos que se repiten)

1. **Enviar una solicitud** a Claude vía la Messages API, incluyendo el historial de conversación, el system prompt, mensajes previos y resultados de herramientas previas.
2. **Inspeccionar el campo `stop_reason`** de la respuesta — es la señal autoritativa para decidir la siguiente acción:
   - `"tool_use"` → Claude quiere llamar una o más herramientas; el loop continúa.
   - `"end_turn"` → Claude terminó; el loop se detiene.
3. **Si `stop_reason` es `"tool_use"`**: ejecutar la(s) herramienta(s) solicitada(s), agregar los resultados al historial de conversación como un nuevo mensaje, y reenviar la conversación actualizada a Claude.
4. **Si `stop_reason` es `"end_turn"`**: el agente terminó; se presenta la respuesta final al usuario.

> [!warning] Punto crítico
> Los resultados de las herramientas **deben** agregarse al historial de conversación. Si se omite este paso, Claude no puede razonar sobre la nueva información en la siguiente iteración — es como si nunca hubiera recibido el resultado.

### 2. `stop_reason` es la única señal confiable

`stop_reason` es determinista e inequívoco. Nunca se debe usar como mecanismo principal de control: parsing de lenguaje natural, revisar el contenido de texto de la respuesta, o límites arbitrarios de iteración.

Más allá de los dos valores que evalúa el examen (`tool_use` y `end_turn`), las APIs en producción también devuelven: `pause_turn`, `max_tokens`, `stop_sequence`, `refusal`, y `model_context_window_exceeded`. La recomendación práctica: tratar cualquier valor que no sea `end_turn` como "todavía no terminó, hay que revisar por qué", en vez de asumir automáticamente que es `tool_use`.

### 3. Toma de decisiones dirigida por el modelo (model-driven decision-making)

En un agentic loop, es Claude quien decide qué herramienta llamar según el contexto actual — lee la tarea, evalúa las herramientas disponibles, y elige una. Esto contrasta con árboles de decisión preconfigurados o secuencias fijas de herramientas donde el desarrollador programa a mano el orden de ejecución.

El enfoque dirigido por el modelo es preferido porque es flexible — **excepto** cuando la lógica de negocio exige cumplimiento determinista (temas financieros, de seguridad o regulatorios), donde la aplicación programática de reglas debe imponerse sobre la flexibilidad del modelo.

## Trampas de examen

> [!warning] Trampa 1 — Revisar el tipo de contenido en vez de `stop_reason`
> Usar `response.content[0].type == 'text'` para decidir si el loop terminó. **Por qué falla:** Claude puede devolver texto junto con un bloque `tool_use` en la misma respuesta (ej. "Déjame buscar tu pedido" + la llamada a la herramienta). Ver la presencia de texto no indica que terminó. **La forma correcta:** revisar siempre `stop_reason`.

> [!warning] Trampa 2 — Límites de iteración como mecanismo principal
> Poner "detente después de 10 loops" como forma principal de terminar el ciclo. **Por qué falla:** o corta trabajo útil a la mitad, o deja correr iteraciones innecesarias. **La forma correcta:** los límites de iteración son válidos únicamente como red de seguridad, nunca como control principal.

> [!warning] Trampa 3 — Parsear frases de lenguaje natural
> Buscar frases como "ya terminé" en el texto de Claude para decidir si el loop acabó. **Por qué falla:** el lenguaje natural es ambiguo por naturaleza y poco confiable como señal de control. **La forma correcta:** `stop_reason` es determinista e inequívoco; el texto no.

> [!warning] Trampa 4 — Forzar `tool_choice` a "any" para evitar texto
> Forzar que Claude siempre llame una herramienta, para "evitar" que devuelva texto suelto. **Por qué falla:** esto obliga a usar una herramienta incluso cuando el agente genuinamente ya terminó, lo que puede crear un loop infinito.

> [!note] Distractor frecuente en el examen
> El examen suele presentar "agregar un límite de iteraciones" como arreglo plausible ante una terminación prematura del loop. Es incorrecto: los límites de iteración resuelven loops descontrolados (que no terminan), no terminaciones prematuras (que terminan antes de tiempo). El arreglo correcto ante una terminación prematura siempre es revisar `stop_reason` correctamente.

## Caso de bug real explicado por la guía

Un agente de soporte al cliente funciona bien con consultas simples, pero se detiene a la mitad en consultas complejas. El código revisaba `if response.content[0].type == "text"` para decidir si ya había terminado.

**El bug:** Claude devuelve texto ("Déjame revisar tu pedido") junto con un bloque `tool_use` en la misma respuesta. El código ve el texto, asume que el agente terminó, y devuelve una respuesta incompleta.

**El arreglo:** reemplazar la revisión del tipo de contenido por una revisión de `stop_reason` — continuar cuando `stop_reason == "tool_use"`, terminar cuando `stop_reason == "end_turn"`.

## Fuentes citadas por la guía
- Claude Agent SDK Overview — Anthropic
- Messages API Reference — Anthropic
- Building with Claude API (Skilljar) — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
