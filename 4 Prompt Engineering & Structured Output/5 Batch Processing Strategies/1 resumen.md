## Explícamelo como si tuviera 5 años

Imagina que tienes que enviar cartas. Si necesitas que alguien reciba una respuesta *ahora mismo* — como cuando le preguntas algo urgente a un doctor — pagas el mensajero exprés, aunque sea caro. Pero si son cartas que de todos modos nadie va a leer hasta mañana en la mañana — como un reporte semanal que revisa tu jefe el lunes — no tiene sentido pagar el exprés: las metes todas juntas en un solo envío nocturno, más barato, y aceptas que pueden tardar hasta un día en llegar.

Ese es el tema completo: la Message Batches API es el "envío nocturno" de Claude — 50% más barata, pero sin garantía de cuándo llega (puede tardar hasta 24 horas) y sin poder "abrir la carta a medio camino" para hacer algo con lo que dice antes de que termine de procesarse. Usarla bien significa reconocer *cuándo* alguien está esperando la respuesta (ahí no sirve) y cuándo el resultado se consume después, sin prisa (ahí es ideal).

## Argumento central

> [!note] Idea central
> La Message Batches API ofrece **50% de ahorro de costo** a cambio de una ventana de procesamiento de **hasta 24 horas sin SLA de latencia garantizada**. La decisión correcta nunca es "¿cuál API es más barata?" sino **"¿alguien o algo está bloqueado esperando este resultado?"** — los workflows bloqueantes (pre-merge checks) deben quedarse en la API síncrona; los workflows tolerantes a latencia (reportes nocturnos, auditorías semanales) son los candidatos ideales para batch.

## Conclusiones clave

### 1. Message Batches API: los hechos fijos

Son restricciones de diseño, no detalles de implementación — hay que diseñar el pipeline alrededor de ellas:

- **50% de ahorro de costo** comparado con llamadas síncronas.
- **Ventana de procesamiento de hasta 24 horas** — los resultados pueden llegar en minutos o tardar hasta ese máximo.
- **Sin SLA de latencia garantizado** — no se puede diseñar un flujo que dependa de que el resultado llegue en un tiempo específico.
- **Sin multi-turn tool calling dentro de una sola request de batch** — el modelo no puede ejecutar una tool a mitad del procesamiento y usar su resultado para continuar dentro de esa misma request.
- **Campos `custom_id`** — cada request dentro del batch lleva un identificador único que sirve para emparejar esa request con su respuesta correspondiente.

### 2. La regla de matching: síncrono vs. batch

> [!note] La idea más evaluada de este tema
> No es una pregunta de "cuál es más barato", sino de "¿quién está esperando el resultado?".

| Usa la API **síncrona** cuando... | Usa la API de **batch** cuando... |
|---|---|
| Un desarrollador o proceso está bloqueado esperando la respuesta | El resultado se consume después, sin nadie esperando en tiempo real |
| Ej. pre-merge checks en CI/CD | Ej. reportes de deuda técnica nocturnos |
| Ej. feedback de code review en tiempo real | Ej. auditorías de código semanales, generación de tests nocturna |
| — | Ej. extracción de datos de documentos en volumen |

El examen presenta explícitamente el escenario de un manager que propone mover *todo* a batch por el ahorro de costo — la respuesta correcta nunca es "todo" ni "nada", es separar por tolerancia a latencia: batch solo para lo que no bloquea a nadie.

### 3. Cálculo de scheduling contra un SLA

Cuando el negocio exige un SLA total (ej. 30 horas) para cada request individual que llega al sistema, hay que trabajar hacia atrás desde la ventana máxima de batch:

1. Las 24 horas son una **ventana máxima, no una garantía de entrega**. Un batch que no termina dentro de ese plazo vuelve como `expired` — hay que diseñar el schedule contra ese peor caso, y tratar un batch `expired` como algo que se debe **reenviar**, igual que un fallo.
2. `30h SLA total − 24h peor caso = 6h de margen` para recolectar requests, validar inputs o absorber demoras operativas.
3. Con ese margen de 6 horas, en vez de acumular requests hasta agotarlo y mandar un solo batch al final, se manda un batch nuevo **cada 4 horas** — no como respuesta a un fallo puntual, sino como cronograma fijo y recurrente mientras el sistema opera: un batch a la hora 0, otro a la hora 4, otro a la hora 8, y así sucesivamente sin parar. Nunca es "una sola petición reenviada varias veces" — son batches distintos, cada uno con las requests nuevas acumuladas desde el envío anterior.

> [!note] Por qué importa el orden: detección temprana, no un segundo intento gratis
> El colchón de 6h **no alcanza** para absorber un segundo ciclo completo de 24h si un batch expira — `24h + 24h = 48h` supera cualquier SLA razonable de 30h, así que esto no es una garantía matemática contra el doble peor caso. Lo que sí logra el cronograma de "cada 4h" es **detectar el problema antes**: si el primer batch (hora 0) expira, te enteras en la hora 24 y todavía te quedan las 6h completas de margen para reaccionar; si en cambio hubieras esperado a acumular todo durante 6h y mandado un único batch al final, te enterarías del `expired` justo en la hora 30 — el límite del SLA, sin margen para hacer nada. Enviar seguido no elimina el riesgo del peor caso, lo reduce: minimiza cuántas requests quedan expuestas a él y da más tiempo de reacción a las que sí lo sufren.

> [!note] Analogía
> No es un mensajero que reintenta el mismo paquete dos veces — es un servicio de correo que despacha una nueva tanda de cartas cada 4 horas en vez de esperar y mandar una sola tanda grande al final del día. Si una tanda se pierde, te enteras rápido (porque la siguiente ya salió hace rato) y no descubres el problema recién cuando ya no queda tiempo de reaccionar.

### 4. Manejo de fallos de batch: reenviar solo lo que falló

No todos los documentos de un batch tienen éxito. El patrón correcto tiene tres pasos, en este orden:

1. **Identificar los fallos por `custom_id`** — se parsean los resultados del batch y se buscan los `custom_id` cuyo resultado vino con error.
2. **Reenviar solo los fallidos, con modificaciones** — nunca se reenvía el batch completo. Modificaciones típicas: chunking de documentos que excedieron el límite de contexto, simplificar el prompt de extracción para documentos con estructura inusual, o agregar few-shot examples específicos para el formato que falló.
3. **Refinar el prompt en una muestra ANTES de procesar el batch completo** — este es el paso proactivo, no reactivo: maximiza el éxito en el primer intento y reduce el costo de reenvíos.

> [!warning] `expired` cuenta como fallo
> Un batch que se pasa de las 24 horas devuelve resultados marcados `expired` para las requests que no alcanzó a procesar. Un pipeline de manejo de fallos que solo revisa un estado de "error" explícito y no también `expired` deja esas requests sin reenviar silenciosamente.

### 5. Refinar antes de escalar: el paso que más ahorra

La estrategia de batch más costo-efectiva no es técnica de la API — es de proceso:

1. **Muestra representativa**: tomar 5-10 documentos que cubran el rango de formatos y casos límite del batch completo.
2. **Iterar sobre la muestra**: refinar prompts, agregar few-shot examples, ajustar el schema hasta lograr alta precisión en esa muestra pequeña.
3. **Enviar el batch completo** ya con el prompt refinado — el first-pass success rate será mucho mayor.
4. **Manejar fallos** reenviando solo lo que falló (Conclusión 4).

> [!note] Por qué importa el orden de magnitud
> Un 90% de éxito en el primer intento sobre 1,000 documentos deja solo 100 reintentos. Un 60% de éxito deja 400 reintentos — cuatro veces el costo de reenvío, más el costo de procesamiento de esos reintentos en el batch original. Saltarse el paso de refinamiento sobre la muestra no ahorra tiempo: multiplica el costo total.

### 6. La limitación de multi-turn tool calling

La API de batch no soporta tool calling de múltiples turnos dentro de una sola request. En la práctica, esto significa que **no se puede**:

- Definir tools y que el modelo las llame a mitad de la request.
- Procesar el resultado de una tool y continuar la conversación dentro del mismo item del batch.
- Correr un loop agéntico completo dentro de una sola request de batch.

Si un workflow necesita ejecutar una tool a mitad de procesamiento y continuar con ese resultado, ese paso debe correr en la **API síncrona** — no en batch.

> [!note] Vigencia: server tools vs. client tools
> La guía de certificación (v1.0 del exam guide) establece esta limitación tal cual, sin distinguir tipos de tool, y esa es la respuesta que el examen espera. La documentación actual de Anthropic distingue entre **server tools** (web search, web fetch, code execution, conectores MCP, tool search) — que sí corren su propio loop agéntico del lado del servidor incluso dentro de una request de batch, y pueden devolver `stop_reason: "pause_turn"` para continuar en una request de seguimiento — y **client tools** (las que ejecuta tu propio código), que siguen sin poder completar un loop dentro de un item de batch: la request termina en `tool_use`, tu código ejecuta la tool fuera del batch, y se envía una request de seguimiento por separado. Para el examen, la regla a aplicar es la de la guía: sin multi-turn tool calling dentro de batch, así que un paso que necesita ejecutar una tool a mitad de proceso va en la API síncrona.

## Trampas de examen

> [!warning] Trampa 1 — Mover todo a batch por el ahorro de costo
> Los workflows bloqueantes, donde un desarrollador espera el resultado (pre-merge checks, revisión en tiempo real), deben quedarse en la API síncrona. La API de batch no tiene SLA de latencia garantizado y puede tardar hasta 24 horas. Solo los workflows tolerantes a latencia deben moverse a batch. Esta es la pregunta de muestra oficial de la guía (ver Pregunta 1 del test).

> [!warning] Trampa 2 — Asumir que los resultados de batch llegan rápido porque "normalmente" es así
> La API no tiene SLA de latencia. Que los resultados *suelan* llegar más rápido que 24 horas no es una garantía diseñable — cualquier workflow bloqueante construido asumiendo "casi siempre es rápido" puede fallar exactamente cuando más importa. Se diseña siempre contra el máximo de 24 horas, nunca contra el caso típico.

> [!warning] Trampa 3 — Usar la API de batch para workflows que necesitan tool calling de múltiples turnos
> La API de batch no soporta ejecutar una tool a mitad de una request y continuar usando su resultado dentro de esa misma request. Si el workflow necesita eso, el paso correspondiente debe correr en la API síncrona — forzarlo dentro de batch simplemente no es posible con client tools.

## En una frase

> La Message Batches API da 50% de ahorro a cambio de hasta 24 horas sin SLA garantizado y sin multi-turn tool calling dentro de la request — se usa solo para workflows tolerantes a latencia, se emparejan requests y respuestas por `custom_id`, se reenvía solo lo que falla (nunca el batch completo), y siempre se refina el prompt sobre una muestra pequeña antes de escalar al volumen completo.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
