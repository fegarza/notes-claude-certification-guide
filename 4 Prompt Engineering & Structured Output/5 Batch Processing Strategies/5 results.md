# Resultados — Tutor socrático: Batch Processing Strategies

> [!info] Sesión evaluada el 2026-10-14
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 72/100**

Tienes muy claro el núcleo del tema: el trade-off de 50% de ahorro contra 24h sin SLA, la regla de "¿quién está bloqueado esperando?", el uso de `custom_id`, reenviar solo lo fallido y tratar `expired` como fallo. Las debilidades aparecen en los detalles operativos: las modificaciones concretas antes de un reenvío, el tamaño y la representatividad de la muestra, la diferencia entre margen (6h) y cadencia (4h) en el cálculo de SLA, y la mecánica exacta del multi-turn tool calling. De 17 preguntas, 9 fueron correctas, 7 parciales y 1 sin respuesta (server vs. client tools).

## Conceptos con buen dominio

- **Hechos fijos de la Batches API** — identificaste sin dudar el 50% de ahorro, la ventana máxima de 24h y lo que implica no tener SLA.
- **Regla de matching síncrono vs. batch** — formulaste la pregunta correcta ("¿esto bloquea algún trabajo?") y clasificaste bien los 4 workflows del escenario del manager (Trampa 1).
- **`custom_id`** — sabes que sirve para emparejar cada request con su respuesta.
- **Reenviar solo lo fallido** — rechazaste reenviar el batch completo y describiste el patrón correcto.
- **`expired` cuenta como fallo** — detectaste de inmediato qué se le escapaba al pipeline.
- **Costo de saltarse el refinamiento** — calculaste bien los 300 reintentos ahorrados.
- **Costo en tiempo de encadenar batches** — 3 vueltas = hasta 72h contra segundos con la Messages API.
- **Trampa 2 ("normalmente es rápido")** — reconociste que sin SLA no se puede diseñar contra el caso típico.

## Áreas que necesitan revisión

### Modificaciones específicas al reenviar fallos

- **Qué pasó:** cuando se preguntó qué modificación aplicar a documentos que excedieron el contexto y a documentos con formato inusual, respondiste con el paso proactivo de refinar sobre una muestra en lugar de las modificaciones concretas para cada tipo de fallo.
- **Contenido del resumen:**
  > 2. **Reenviar solo los fallidos, con modificaciones** — nunca se reenvía el batch completo. Modificaciones típicas: chunking de documentos que excedieron el límite de contexto, simplificar el prompt de extracción para documentos con estructura inusual, o agregar few-shot examples específicos para el formato que falló.
  > 3. **Refinar el prompt en una muestra ANTES de procesar el batch completo** — este es el paso proactivo, no reactivo: maximiza el éxito en el primer intento y reduce el costo de reenvíos.
- **Sección:** [[1 resumen]] → "4. Manejo de fallos de batch: reenviar solo lo que falló"
- **Analogía para reforzarlo:** es como una mudanza donde algunas cajas no entraron al camión. A las demasiado grandes las **partes en dos cajas más chicas** (chunking). A las de forma rara les **pegas instrucciones claras de cómo cargarlas** (few-shot o prompt simplificado). Revisar las cajas antes de empezar es otra cosa: es prevención, no la reparación de lo que ya falló.

### Muestra representativa: tamaño y característica

- **Qué pasó:** dijiste "4 a 5" documentos y no mencionaste que la muestra debe cubrir el rango de formatos y casos límite.
- **Contenido del resumen:**
  > 1. **Muestra representativa**: tomar 5-10 documentos que cubran el rango de formatos y casos límite del batch completo.
  > 2. **Iterar sobre la muestra**: refinar prompts, agregar few-shot examples, ajustar el schema hasta lograr alta precisión en esa muestra pequeña.
- **Sección:** [[1 resumen]] → "5. Refinar antes de escalar: el paso que más ahorra"
- **Analogía para reforzarlo:** antes de cocinar para una boda, el chef prueba el menú con un grupo pequeño. Si solo invita a gente que come de todo, no se entera de que el platillo falla con los vegetarianos y los alérgicos. El grupo de prueba tiene que parecerse a los invitados reales, raros incluidos.

### Por qué saltarse el refinamiento multiplica el costo

- **Qué pasó:** calculaste bien los 300 reintentos, pero no mencionaste que los documentos fallidos se pagan dos veces (en el batch original y en el reenvío).
- **Contenido del resumen:**
  > Un 90% de éxito en el primer intento sobre 1,000 documentos deja solo 100 reintentos. Un 60% de éxito deja 400 reintentos — cuatro veces el costo de reenvío, más el costo de procesamiento de esos reintentos en el batch original. Saltarse el paso de refinamiento sobre la muestra no ahorra tiempo: multiplica el costo total.
- **Sección:** [[1 resumen]] → "5. Refinar antes de escalar: el paso que más ahorra"
- **Analogía para reforzarlo:** imprimir 1,000 invitaciones sin revisar la prueba. Cada invitación con error la pagas al imprimirla y la vuelves a pagar al reimprimirla. Con 400 errores pagas 400 dos veces.

### Margen vs. cadencia en el cálculo de SLA

- **Qué pasó:** dijiste que el margen era de 4h "por colchón". En realidad el margen es de 6h (30 − 24) y las 4h son la frecuencia de envío de batches recurrentes.
- **Contenido del resumen:**
  > 2. `30h SLA total − 24h peor caso = 6h de margen` para recolectar requests, validar inputs o absorber demoras operativas.
  > 3. Con ese margen de 6 horas, en vez de acumular requests hasta agotarlo y mandar un solo batch al final, se manda un batch nuevo **cada 4 horas** — no como respuesta a un fallo puntual, sino como cronograma fijo y recurrente mientras el sistema opera: un batch a la hora 0, otro a la hora 4, otro a la hora 8, y así sucesivamente sin parar.
- **Sección:** [[1 resumen]] → "3. Cálculo de scheduling contra un SLA"
- **Analogía para reforzarlo:** tienes 6 horas antes de que cierre la oficina de correos (el margen), pero decides que el cartero pase a recoger cartas **cada 4 horas** (la cadencia). Una cosa es cuánto tiempo tienes en total y otra cada cuánto actúas dentro de ese tiempo.

### Ventaja real de enviar cada 4h: detección temprana

- **Qué pasó:** reconociste que un segundo ciclo de 24h no cabe en el SLA, pero describiste la ventaja como "2 horas para enviarlo" en lugar de detección temprana con las 6h completas para reaccionar.
- **Contenido del resumen:**
  > Lo que sí logra el cronograma de "cada 4h" es **detectar el problema antes**: si el primer batch (hora 0) expira, te enteras en la hora 24 y todavía te quedan las 6h completas de margen para reaccionar; si en cambio hubieras esperado a acumular todo durante 6h y mandado un único batch al final, te enterarías del `expired` justo en la hora 30 — el límite del SLA, sin margen para hacer nada. Enviar seguido no elimina el riesgo del peor caso, lo reduce: minimiza cuántas requests quedan expuestas a él y da más tiempo de reacción a las que sí lo sufren.
- **Sección:** [[1 resumen]] → "3. Cálculo de scheduling contra un SLA"
- **Analogía para reforzarlo:** si revisas el horno cada 10 minutos, notas que se apagó con tiempo de prender otro. Si solo lo abres al final, descubres la comida cruda cuando los invitados ya están sentados.

### Mecánica del multi-turn tool calling (Messages API vs. Batches API)

- **Qué pasó:** dijiste que en batch "Claude no puede llamar la tool" (en realidad sí la pide, pero el item termina en `tool_use`) y que en la Messages API funciona "porque hay alguien disponible", sin describir el loop de requests separadas coordinado por tu código.
- **Contenido del resumen:**
  > La clave: **el modelo nunca ejecuta tus tools**. El "multi-turn" en realidad son **varias requests separadas que coordina tu código**:
  >
  > 1. Tu código manda la request con `tools` y el mensaje del usuario.
  > 2. Claude responde con `stop_reason: "tool_use"` y un bloque `tool_use` — la request **termina ahí**, Claude no se queda esperando.
  > 3. Tu código ejecuta la tool en tu propio sistema.
  > 4. Tu código manda una **nueva** request con todo el historial + el bloque `tool_result`.
  > 5. Se repite hasta que `stop_reason` sea `end_turn`.

  > Cada item del batch (identificado por su `custom_id`) es **exactamente una request = una sola vuelta del loop**. El batch procesa el item en los servidores de Anthropic, guarda el resultado y sigue con el siguiente — **no existe un punto donde tu código intervenga a mitad del item**. Si Claude pide una tool, el item termina en `tool_use` sin respuesta.
- **Sección:** [[1 resumen]] → "6. La limitación de multi-turn tool calling"
- **Analogía para reforzarlo:** jugar ajedrez por chat contra un amigo. Cada jugada es un mensaje nuevo, y tú tienes que mover tu pieza y responder antes de que él siga. Por chat se juega en minutos. Por correo postal, la misma partida dura meses: el juego es igual, cambia cuánto tarda cada ida y vuelta.

### Server tools vs. client tools

- **Qué pasó:** no respondiste la pregunta.
- **Contenido del resumen:**
  > La documentación actual de Anthropic distingue entre **server tools** (web search, web fetch, code execution, conectores MCP, tool search) — que sí corren su propio loop agéntico del lado del servidor incluso dentro de una request de batch, y pueden devolver `stop_reason: "pause_turn"` para continuar en una request de seguimiento — y **client tools** (las que ejecuta tu propio código), que siguen sin poder completar un loop dentro de un item de batch: la request termina en `tool_use`, tu código ejecuta la tool fuera del batch, y se envía una request de seguimiento por separado. Para el examen, la regla a aplicar es la de la guía: sin multi-turn tool calling dentro de batch, así que un paso que necesita ejecutar una tool a mitad de proceso va en la API síncrona.
- **Sección:** [[1 resumen]] → "6. La limitación de multi-turn tool calling" → "Vigencia: server tools vs. client tools"
- **Analogía para reforzarlo:** un cocinero en un restaurante cerrado de noche. Puede usar todo lo que hay **dentro de su cocina** (server tools) sin pedirle nada a nadie. Pero si necesita un ingrediente que solo **tú** tienes en tu casa (client tool), deja la receta a medias con una nota y espera al día siguiente.

## Siguiente paso recomendado

Repasa las Conclusiones 3 (margen 6h vs. cadencia 4h y detección temprana) y 6 (el loop de requests separadas y server vs. client tools). Son las que concentraron los errores. Después de eso estás listo para [[4 test]].
