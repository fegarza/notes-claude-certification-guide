---
tags:
  - claude-cert/dominio-5
  - task-statement/5.1
---

# 5.1 — Context Window Management

## Explícamelo como si tuviera 5 años

Imagina que tienes una libreta pequeñita donde puedes escribir solo un número limitado de palabras, y cada vez que alguien te cuenta algo nuevo, tienes que decidir qué guardar y qué borrar para que quepa lo siguiente. Si borras mal, puedes terminar borrando el número exacto de la tarjeta de crédito de mamá y dejando solo "mamá dio unos números" — y ahora nadie sabe qué números eran.

Eso es exactamente lo que le pasa a Claude con una conversación larga: tiene espacio limitado (el *context window*), y si la estrategia para hacer espacio es "resumir lo viejo", corres el riesgo de borrar justo los datos puntuales — montos, fechas, números de orden — que en realidad importan. El tema completo trata de cómo decidir qué meter en ese espacio limitado sin perder lo que no se puede perder.

## Argumento central

> [!note] Idea central
> El context window management es la base de cualquier sistema confiable basado en Claude: de él depende toda conversación de varios turnos, todo pipeline multi-agente, toda extracción de documentos largos. Fallar aquí no es abstracto — se traduce en fallos concretos y medibles: un agente de soporte que olvida el monto de un reembolso, un pipeline de investigación que pierde citas, un sistema de extracción que pierde precisión justo en los campos que importan.

## Conclusiones

### 1. La trampa de la progressive summarisation

Cuando una conversación crece, resumir los turnos anteriores para liberar presupuesto de tokens parece la solución obvia. **Es una trampa.**

- La progressive summarisation destruye sistemáticamente la información más crítica en sistemas orientados al cliente y de procesamiento de datos: **valores numéricos, fechas, porcentajes y expectativas que el cliente expresó explícitamente**.
- No es un caso raro o extremo — es lo que la summarisation le hace por defecto a cualquier dato transaccional.

> [!note] Ejemplo mental
> Un turno dice "quiero un reembolso de $247.83 para la orden #8891 del 3 de marzo". Tras resumir, queda "el cliente quiere un reembolso de una orden reciente". El monto, el número de orden y la fecha — los tres datos que el agente necesita para procesar el reembolso — desaparecieron.

**La solución: persistent case facts block.** Se extraen los hechos transaccionales (montos, fechas, números de orden, estados) hacia un bloque estructurado que:

- se incluye en **cada prompt**, fuera del historial que sí se resume,
- **nunca se resume**,
- persiste en todos los turnos sin importar qué le pase al resto del historial.

Para sesiones con **múltiples issues** (un cliente que plantea varios problemas en una sola conversación), cada issue se extrae y persiste como una entrada separada en esta capa (order ID, monto, estado propios) — esto evita que la summarisation mezcle los datos de un issue con los de otro.

> [!note] El patrón más importante del tema
> El persistent case facts block es, literalmente, "el patrón más importante en context window management" según la guía. Es la solución a la progressive summarisation trap y la base de cualquier sistema confiable de múltiples turnos.

### 2. El efecto "lost in the middle"

Los modelos procesan de forma confiable la información al **inicio** y al **final** de un input largo — lo que queda enterrado en el medio puede pasarse por alto o recibir menos peso. Es un fenómeno bien documentado, no una suposición.

- **El fix es estructural, no un recordatorio en el prompt.** Pedirle al modelo "presta atención a todo" no resuelve un efecto de posición.
- La solución: colocar un resumen de hallazgos clave al **inicio** de cualquier input agregado, y organizar los resultados detallados debajo con encabezados de sección explícitos.

> [!note] Ejemplo mental
> Si un agente de síntesis recibe la salida de tres subagentes de investigación, no se le pega todo en bruto uno tras otro. Se arma así: primero una sección "Key Findings Summary" con los puntos clave de cada fuente, y después el detalle completo de cada una, con su propio encabezado (`### Source A`, `### Source B`...).

### 3. Tool result trimming

Los resultados de tools son un "asesino silencioso" del presupuesto de contexto.

- Una consulta de orden puede devolver 40+ campos (timestamps de auditoría interna, códigos de almacén, IDs de transportista) cuando el agente solo necesita 5.
- Esos 35 campos irrelevantes consumen tokens en **cada turno subsecuente**, porque quedan en el historial de la conversación conforme esta crece.
- La solución: **recortar los resultados verbosos de las tools a solo los campos relevantes** antes de que se acumulen en el contexto — no es un "nice-to-have", es necesario para que sistemas de varios turnos no terminen ahogados en datos obsoletos.
- Este recorte debe ocurrir en un hook `PostToolUse` o en la propia implementación de la tool, **antes** de que el resultado entre al historial. Una vez que el dato verboso ya está en el contexto, se queda ahí para siempre.

### 4. La API de Claude es stateless: historial completo en cada request

- La API de Claude **no guarda estado en el servidor**. Cada request debe incluir el historial completo de la conversación.
- Omitir turnos anteriores hace que el modelo pierda coherencia conversacional — no hay sesión persistida del lado del servidor, así que cada turno tiene que cargar todo lo que el modelo necesita para seguir la conversación.
- Esto crea una tensión: se necesita el historial completo para mantener coherencia, pero el historial crece con cada turno.
- El persistent case facts block (Conclusión 1) es justamente lo que resuelve esta tensión: separa los hechos críticos de la narrativa resumible, permitiendo resumir el flujo conversacional sin perder ningún detalle transaccional.

> [!warning] No es lo mismo "resumir" que "truncar"
> El historial se puede resumir (comprimir preservando los hechos críticos aparte), pero **no se puede truncar selectivamente sin consecuencias** — quitar turnos completos rompe la coherencia conversacional, porque la API no tiene memoria propia que compense lo que falta.

### 5. Upstream agent optimisation en sistemas multi-agente

En arquitecturas multi-agente, los agentes upstream a menudo devuelven cadenas de razonamiento verbosas y contenido crudo que los agentes downstream no necesitan.

- Si un subagente de investigación le manda su proceso de pensamiento completo a un agente de síntesis con presupuesto de contexto limitado, ese agente de síntesis desperdicia tokens en razonamiento que no puede usar.
- La solución: modificar los agentes upstream para que devuelvan **datos estructurados** — hechos clave, citas, puntajes de relevancia — en vez de contenido y razonamiento verbosos.
- Se debe exigir a los subagentes que incluyan **metadata** (fechas, ubicación de la fuente, contexto metodológico) dentro de esas salidas estructuradas, para que la síntesis downstream sea precisa.
- El beneficio no es solo de tokens: las salidas estructuradas le permiten al agente downstream procesar los hallazgos sin tener que re-parsear prosa verbosa.

### 6. Prompt caching: la otra mitad de la economía de contexto

Mientras las conclusiones anteriores tratan de *recortar* lo que el modelo ve, el `prompt caching` evita *pagar de nuevo* por procesar las partes que no cambian.

- Se marca un prefijo estable del prompt con un breakpoint `cache_control`; la API guarda ese prefijo ya procesado y lo reutiliza en la siguiente request, cobrando una fracción del costo de input por esos tokens cacheados.
- El caching hace match **desde el inicio del prompt, prefijo por prefijo** — el orden del contenido decide si hay hit o no. Lo constante (instrucciones de sistema, definiciones de tools, documentos de referencia largos) va primero; el breakpoint `cache_control` se coloca al final de ese bloque estático; lo volátil (el último mensaje del usuario, cualquier cosa que cambie por request) va después.
- El bloque estático vive en el parámetro de nivel superior `system`, no dentro de `messages` — la Messages API no tiene un rol `"system"` para los mensajes de input; `messages` solo acepta turnos `"user"` y `"assistant"`.
- Si el orden se invierte (contenido dinámico antes del bloque estático), el prefijo cambia en cada request, no hay match, y se pierde el beneficio por completo.
- Un breakpoint `ephemeral` dura unos 5 minutos desde el último uso; uno con `{"type": "ephemeral", "ttl": "1h"}` dura una hora a un costo de escritura mayor. Una request puede llevar como máximo **cuatro** breakpoints.

> [!info] Alcance para el examen
> La guía excluye explícitamente "los detalles de implementación de prompt caching (más allá de saber que existe)" de lo evaluado. Basta con saber que existe, para qué sirve, y la lógica de orden estático-antes-que-dinámico — no se espera dominar la mecánica completa de los breakpoints.

## En una frase

> El context window es un recurso escaso: la trampa de resumir sin cuidado destruye los datos transaccionales que más importan, y la solución sistemática es separar lo que nunca debe resumirse (el persistent case facts block) de lo que sí puede comprimirse, estructurarse o recortarse (historial narrativo, resultados de tools, salidas de subagentes).

## Trampas de examen

> [!warning] Trampa 1 — Pensar que la progressive summarisation es segura para datos transaccionales
> La summarisation destruye sistemáticamente valores numéricos, fechas e identificadores específicos. **Por qué es un error:** un persistent case facts block debe mantener estos datos fuera del historial resumido — asumir que "resumir bien" basta es ignorar que la compresión de lenguaje natural no preserva precisión numérica de forma confiable.

> [!warning] Trampa 2 — Creer que el "lost in the middle" se resuelve pidiéndole al modelo que "preste atención a todo"
> El fix es estructural: colocar los hallazgos clave al inicio de los inputs y usar encabezados de sección explícitos. **Por qué es un error:** los recordatorios dentro del prompt no son confiables contra efectos de posición — es un fenómeno del procesamiento del modelo, no de falta de instrucción.

> [!warning] Trampa 3 — Mantener los resultados completos de una tool en contexto "por si el modelo los necesita después"
> Los resultados sin recortar de una consulta con 40+ campos agotan el presupuesto de tokens a través de los turnos. **Por qué es un error:** hay que recortar a los campos relevantes *antes* de que el resultado entre al historial de la conversación — una vez adentro, se queda ahí indefinidamente, y el costo se paga en cada turno subsecuente.

> [!warning] Trampa 4 — Creer que el historial de conversación se puede truncar selectivamente sin consecuencias
> La API es stateless: cada request necesita el historial completo. **Por qué es un error:** truncar selectivamente rompe la coherencia conversacional. La herramienta correcta para el problema de "el historial crece demasiado" son el case facts block y la summarisation cuidadosa — no la truncación.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
