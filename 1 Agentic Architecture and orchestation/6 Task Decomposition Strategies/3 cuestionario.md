Cuestionario de repaso completo de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revisar.

## Dos patrones de decomposición, dos definiciones exactas

> [!question]- ¿Cuál es la diferencia esencial entre un pipeline secuencial fijo y una decomposición dinámica adaptativa?
> En el pipeline fijo, el workflow se define de antemano y la secuencia no cambia según los resultados intermedios. En la decomposición dinámica, el agente genera el plan (y lo modifica) según lo que descubre durante la ejecución.

> [!question]- ¿Cómo se llama también al pipeline secuencial fijo en el vocabulario del examen?
> *Prompt chaining* — pasos sucesivos donde la salida de uno alimenta al siguiente.

> [!question]- Si a un pipeline fijo le cambias el orden de los pasos a mitad de ejecución basándote en un resultado intermedio, ¿sigue siendo un pipeline fijo?
> No — en el momento en que la secuencia cambia según un resultado intermedio, dejó de ser un pipeline fijo y se convirtió en decomposición dinámica. La característica definitoria del pipeline fijo es precisamente que la secuencia no cambia.

## El framework de decisión: una tabla, un criterio

> [!question]- ¿Cuál es la única pregunta que determina qué patrón usar, según el framework de decisión?
> ¿Los pasos de la tarea se conocen de antemano, o se van a descubrir durante la ejecución? No depende de qué tan "compleja" parezca la tarea.

> [!question]- ¿Por qué una revisión de código multi-archivo usa pipeline fijo y no decomposición dinámica, si revisar código puede ser complejo?
> Porque los archivos a revisar y los pasos (análisis por archivo + integración cruzada) se conocen de antemano — la complejidad del análisis no cambia que la estructura del trabajo ya está definida antes de empezar.

> [!question]- ¿Por qué explorar un codebase legacy requiere decomposición dinámica y no un pipeline fijo?
> Porque las dependencias y problemas del codebase emergen durante la investigación — no se puede escribir de antemano la lista completa de módulos a revisar sin haber investigado primero.

> [!question]- ¿Cuáles son las tres fases del patrón de decomposición dinámica para una tarea abierta como "agregar tests a un codebase legacy"?
> Primero mapear la estructura, luego identificar las áreas de mayor impacto, y recién ahí crear un plan priorizado que se adapta a medida que se descubren nuevas dependencias.

> [!question]- ¿Por qué memorizar la lista de ejemplos de la tabla de decisión (revisión de código, codebase legacy, extracción de documentos, debugging) no es suficiente para el examen?
> Porque el examen presenta escenarios nuevos, no los mismos ejemplos — lo que hay que aplicar es el criterio subyacente ("¿los pasos se conocen de antemano?"), no reconocer un ejemplo memorizado.

## Dilución de atención: qué es y cómo se ve

> [!question]- ¿Qué es la dilución de atención, en una frase?
> Un modo de fallo que ocurre cuando un agente procesa demasiados ítems en una sola pasada, produciendo una profundidad de análisis inconsistente entre ítems.

> [!question]- Describe los tres síntomas típicos de dilución de atención en una revisión de código.
> Feedback detallado en los primeros archivos pero cada vez más superficial en los últimos; un patrón marcado como problemático en un archivo mientras el mismo código idéntico se aprueba en otro; y bugs obvios pasados por alto en algunos archivos mientras se señalan problemas menores de estilo en otros.

> [!question]- ¿Por qué los tres síntomas de dilución de atención se consideran la misma causa raíz y no tres problemas distintos?
> Porque todos vienen de repartir el mismo presupuesto fijo de atención entre demasiados ítems en una sola pasada — la degradación progresiva, la inconsistencia entre ítems y los bugs pasados por alto son manifestaciones distintas del mismo límite estructural.

## La solución: arquitectura multi-pass (no modelo, no prompt)

> [!question]- ¿Cuáles son las dos capas de la arquitectura multi-pass que resuelve la dilución de atención?
> Pasadas de análisis local por ítem (cada archivo analizado individualmente, con todo el presupuesto de atención enfocado en ese ítem) y una pasada de integración cruzada separada, que corre después de todas las locales y busca preocupaciones transversales entre ítems.

> [!question]- ¿Por qué un modelo con ventana de contexto más grande no resuelve la dilución de atención, aunque le quepan todos los archivos sin truncarse?
> Porque el problema nunca fue que los archivos no cupieran en el contexto — fue que la profundidad de análisis se degrada al procesar muchos ítems en una sola pasada. Es un límite estructural de atención, no de capacidad de contexto.

> [!question]- ¿Por qué un prompt que insiste en "dar el mismo nivel de detalle a todos los archivos" no resuelve el problema de raíz?
> Porque mejora la calidad promedio, pero el modelo sigue repartiendo el mismo presupuesto de atención entre todos los ítems en una sola pasada — ninguna instrucción cambia esa asignación estructural.

> [!question]- Si divides 14 archivos en 3 lotes de ~5 y revisas cada lote por separado, ¿resolviste la dilución de atención?
> Solo parcialmente. El batching reduce la dilución dentro de cada lote, pero sin una pasada de integración cruzada separada, un problema detectado en el lote 1 puede seguir aprobándose sin objeción en el lote 3 — los problemas que cruzan los límites entre lotes se siguen perdiendo.

> [!question]- ¿Qué evalúa específicamente la pasada de integración cruzada que no evalúan las pasadas locales?
> Preocupaciones transversales entre ítems — por ejemplo, si el mismo patrón de código fue marcado como problemático en un archivo pero aprobado en otro, algo que una pasada local (que solo ve un archivo a la vez) no puede detectar por definición.

---
> [!tip] Sigue con este tema
> Repasa el resumen completo en [[1 resumen]], aplica estos conceptos en código en [[2 example]], y evalúate con [[4 test]].
