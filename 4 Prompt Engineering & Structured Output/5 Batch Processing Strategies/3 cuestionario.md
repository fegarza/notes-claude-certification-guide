## Message Batches API: los hechos fijos

> [!question]- ¿Cuánto ahorro de costo ofrece la Message Batches API comparada con la API síncrona?
> 50% de ahorro comparado con llamadas síncronas.

> [!question]- ¿Cuál es la ventana máxima de procesamiento de un batch, y qué NO garantiza esa cifra?
> Hasta 24 horas. No garantiza un SLA de latencia — los resultados pueden llegar en minutos o tardar hasta ese máximo, y no hay forma de asegurar un tiempo específico.

> [!question]- ¿Para qué sirve el campo `custom_id` en una request de batch?
> Para correlacionar cada request con su respuesta correspondiente — cada item del batch lleva un identificador único que permite emparejarlo con su resultado al leer las respuestas.

## La regla de matching: síncrono vs. batch

> [!question]- ¿Cuál es la pregunta correcta para decidir entre API síncrona y batch — "cuál es más barato" o alguna otra?
> "¿Alguien o algo está bloqueado esperando este resultado?" — no el costo. Si hay un bloqueo, va en síncrona; si el resultado se consume después sin nadie esperando en tiempo real, va en batch.

> [!question]- Da un ejemplo de workflow que debe quedarse en la API síncrona y uno que es candidato ideal para batch.
> Síncrona: un pre-merge check en CI/CD que bloquea el merge hasta tener resultado. Batch: un reporte de deuda técnica generado de noche para revisión al día siguiente.

> [!question]- ¿Por qué la respuesta correcta ante "movamos todo a batch para ahorrar costo" nunca es "sí, todo" ni "no, nada"?
> Porque la decisión depende de la tolerancia a latencia de cada workflow individualmente — algunos son bloqueantes y no pueden tolerar hasta 24 horas sin SLA, otros no tienen esa restricción. Aplicar una regla uniforme a todos ignora esa diferencia.

## Cálculo de scheduling contra un SLA

> [!question]- Si el SLA total exigido es de 30 horas y la ventana máxima de batch es de 24 horas, ¿cuánto margen (buffer) queda?
> 6 horas de margen para recolectar requests, validar inputs o absorber demoras operativas.

> [!question]- ¿Desde cuándo empieza a contar el SLA de 30 horas de una request: desde que llega o desde que se envía el batch?
> Desde que llega. Por eso el peor caso es la request que llega justo después de un envío: espera todo el intervalo hasta el siguiente batch, más las 24 h del batch.

> [!question]- ¿Qué pasaría si mandaras un batch cada 8 horas con un SLA de 30 horas?
> Una request que llega justo después de un envío esperaría 8 h + 24 h = 32 h, rompiendo el SLA. El intervalo entre envíos debe ser ≤ 6 h; cada 4 h deja 2 h de colchón.

> [!question]- ¿Por qué hay que calcular el schedule contra el peor caso de 24 horas y no contra el tiempo típico en que normalmente llegan los resultados?
> Porque la API no da SLA de latencia garantizado — que "normalmente" sea más rápido no es algo diseñable ni confiable; un schedule construido sobre el caso típico puede fallar exactamente el día que un batch sí tarde el máximo.

> [!question]- ¿Qué significa que un batch vuelva con estado `expired`, y cómo debe tratarse?
> Significa que no terminó de procesarse dentro de la ventana máxima de 24 horas. Debe tratarse como un fallo que necesita reenvío, igual que un error explícito — no como un resultado válido ni como algo que se pueda ignorar.

## Manejo de fallos de batch

> [!question]- ¿Cuáles son los tres pasos del patrón correcto de manejo de fallos de batch, en orden?
> 1) Identificar los fallos por `custom_id`. 2) Reenviar solo los documentos fallidos, con modificaciones dirigidas. 3) Haber refinado el prompt sobre una muestra antes de procesar el batch completo (paso proactivo).

> [!question]- ¿Por qué reenviar el batch completo en vez de solo los documentos fallidos es un error costoso?
> Porque se vuelve a pagar el procesamiento de los documentos que ya tuvieron éxito, sin ninguna necesidad — el `custom_id` existe precisamente para poder aislar y reenviar solo lo que falló.

> [!question]- Menciona dos modificaciones típicas que se aplican a un documento antes de reenviarlo tras un fallo.
> Chunking de documentos que excedieron el límite de contexto, y simplificar el prompt de extracción (o agregar few-shot examples específicos) para documentos con estructura inusual que causó el fallo.

## Refinar antes de escalar

> [!question]- ¿Cuál es el paso más costo-efectivo de toda la estrategia de batch processing, y en qué momento del proceso ocurre?
> Refinar el prompt sobre una muestra representativa de 5-10 documentos, iterando hasta lograr alta precisión, **antes** de enviar el batch completo — es el paso proactivo, no el reactivo de manejar fallos después.

> [!question]- Si un batch de 1,000 documentos logra 90% de éxito en el primer intento versus 60%, ¿cuántos reintentos produce cada escenario, y qué implica esa diferencia?
> 90% de éxito deja 100 reintentos; 60% deja 400 reintentos — cuatro veces más. Implica que saltarse el refinamiento sobre la muestra no ahorra tiempo, multiplica el costo total de reenvío y de procesamiento.

## La limitación de multi-turn tool calling

> [!question]- ¿Qué NO se puede hacer dentro de una sola request de la API de batch, relacionado con tool use?
> No se puede definir una tool y que el modelo la llame a mitad de la request, procesar su resultado y continuar la conversación dentro del mismo item — no se puede correr un loop agéntico completo dentro de una sola request de batch.

> [!question]- Si un workflow necesita ejecutar una tool a mitad de procesamiento y usar su resultado para continuar razonando en el mismo turno, ¿en qué API debe correr ese paso?
> En la API síncrona — la de batch no soporta ese patrón dentro de una sola request.

> [!question]- En un loop de tool use con la Messages API, ¿quién ejecuta la tool: el modelo o tu código?
> Tu código. Claude solo devuelve un bloque `tool_use` con `stop_reason: "tool_use"` y la request termina; tu código ejecuta la tool y manda una nueva request con el `tool_result`. El "multi-turn" son varias requests separadas que coordina tu código.

> [!question]- ¿Por qué el multi-turn tool calling funciona bien con la Messages API pero no dentro de un item de batch?
> Porque con la Messages API tu código encadena las requests en tiempo real (segundos por vuelta). En batch, cada item es una sola request = una sola vuelta, y tu código no puede intervenir a mitad del item para ejecutar la tool y devolver el `tool_result`.

> [!question]- ¿Qué pasaría si intentaras completar un loop de 3 vueltas de tool use encadenando batches?
> Cada vuelta sería un batch nuevo de hasta 24 h sin SLA → hasta 72 h en el peor caso, contra unos segundos con la API síncrona. Por eso ese paso va en la API síncrona.

> [!question]- Según la guía de certificación (v1.0), ¿qué tipos de tool distingue la documentación actual de Anthropic que la guía no distingue, y cuál sigue siendo la respuesta correcta para el examen?
> La documentación actual distingue server tools (corren su propio loop agéntico incluso dentro de batch, pueden devolver `stop_reason: "pause_turn"`) de client tools (siguen sin poder completar un loop dentro de un item de batch). Para el examen, la respuesta correcta sigue siendo la de la guía: sin multi-turn tool calling en batch, así que un paso que ejecuta una tool a mitad de proceso va en la API síncrona.

---

> [!tip] Repasa la teoría en [[1 resumen]], aplícala en [[2 example]] y evalúate en [[4 test]]
