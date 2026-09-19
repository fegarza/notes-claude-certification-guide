---
tags:
  - claude-cert/dominio-2
  - task-statement/2.3
---

# 3 cuestionario — Tool Distribution & Tool Choice

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## El problema de la sobrecarga de tools

> [!question]- ¿Por qué darle demasiadas tools a un agente degrada su confiabilidad, en vez de simplemente darle más opciones?
> Porque cada tool adicional agrega complejidad de decisión al momento de elegir cuál usar — no es que el modelo se vuelva "menos capaz", es que la tarea de seleccionar entre muchas opciones similares se vuelve más propensa a error conforme crece el toolkit.

> [!question]- ¿Cuál es el rango óptimo de tools por agente según la guía?
> 4-5 tools, acotadas específicamente al rol de ese agente.

> [!question]- ¿Por qué un agente con tools fuera de su especialización tiende a mal-usarlas, en vez de simplemente ignorarlas?
> Porque si la tool está disponible, el modelo a veces la usa aunque no sea su rol — por ejemplo, un agente de síntesis con acceso a búsqueda web puede terminar repitiendo investigación en vez de trabajar con los resultados que ya recibió, duplicando trabajo innecesariamente.

## Consolidar tools casi-duplicadas

> [!question]- ¿Por qué dividir tools por rol no siempre resuelve el problema de sobrecarga?
> Porque si todas las tools de un dominio hacen básicamente el mismo tipo de trabajo (ej. 19 operaciones de transformación de datos), dividir por rol solo mueve las 19 tools a un agente distinto — el agente sigue teniendo que elegir entre 19 opciones casi idénticas.

> [!question]- ¿Cuál es la alternativa a dividir por rol cuando las tools comparten el mismo patrón de entrada-operación-salida?
> Consolidarlas en una sola tool parametrizada con un parámetro enum que selecciona la operación específica (ej. `transform_data` con `transform_type` en vez de 19 tools separadas).

> [!question]- Al consolidar 19 tools en una sola tool parametrizada, ¿se pierde acceso a alguna de las operaciones originales?
> No — cada operación sigue siendo alcanzable, ahora como un valor de enum que el modelo elige dentro de una sola llamada, en vez de tener que encontrar la tool correcta entre diecinueve descripciones casi idénticas.

## Configuración de `tool_choice`

> [!question]- ¿Cuál es la diferencia entre `tool_choice: auto` y `tool_choice: any`?
> Con `auto`, el modelo decide libremente si llama una tool o responde con texto conversacional. Con `any`, el modelo está obligado a llamar alguna tool, pero elige cuál — esto garantiza salida estructurada, algo que `auto` no garantiza.

> [!question]- ¿Cuándo usarías selección forzada (`{"type": "tool", "name": "..."}`) en vez de `any`?
> Cuando necesitas que se ejecute específicamente un paso del workflow que no puede saltarse (ej. forzar `extract_metadata` antes de que corran las tools de enriquecimiento) — `any` deja al modelo elegir cuál tool llamar, pero selección forzada elimina esa elección por completo.

> [!question]- ¿Qué pasaría si usas `tool_choice: auto` en un pipeline de extracción que necesita garantizar salida estructurada siempre?
> El modelo podría responder con texto conversacional en vez de llamar la tool de extracción, rompiendo la garantía de que siempre habrá datos estructurados como salida — para eso se necesita `any` o selección forzada, no `auto`.

> [!question]- Tras forzar la llamada a una tool específica al inicio del workflow, ¿tiene sentido seguir usando selección forzada en los turnos siguientes?
> No necesariamente — una vez que el paso obligatorio ya se ejecutó, los turnos siguientes pueden volver a `auto` para el resto de los pasos de análisis, donde sí conviene flexibilidad.

## Tools acotadas cruzando roles (*scoped cross-role tools*)

> [!question]- ¿Cuál es el problema del enfoque "rutear siempre a través del coordinador" cuando un agente necesita ocasionalmente una capacidad de otro rol?
> Que agrega idas y vueltas (round trips) innecesarias y aumenta la latencia, especialmente cuando la mayoría de esas necesidades son simples y no requieren la investigación completa de un agente especializado.

> [!question]- ¿Cómo se relaciona el porcentaje de casos simples (85%) con la decisión de dar una tool acotada como `verify_fact`?
> Justamente porque la gran mayoría de los casos son simples, tiene sentido resolverlos localmente con una tool acotada en vez de pagar el costo de coordinación en cada uno — el 15% restante, que sí es complejo, sigue mereciendo el pipeline completo a través del coordinador.

> [!question]- ¿Por qué no darle al agente de síntesis acceso completo a todas las tools de búsqueda web, en vez de solo una tool acotada como `verify_fact`?
> Porque eso sobre-provisiona al agente y viola la separación de responsabilidades (y el principio de mínimo privilegio) — el agente de síntesis solo necesita resolver verificaciones simples de un solo dato, no ejecutar investigación web completa, que sigue siendo el rol del agente de búsqueda.

> [!question]- ¿Qué pasaría si en vez de una tool acotada se implementara un "batch" de verificaciones que el agente de síntesis acumula y envía todas juntas al final?
> Crearía dependencias bloqueantes, porque pasos posteriores de la síntesis pueden depender de hechos que aún no se han verificado — acumular y esperar al final no resuelve el problema de latencia, solo lo mueve.

## Reemplazar tools genéricas por alternativas acotadas

> [!question]- ¿Por qué reemplazar `fetch_url` por `load_document` es un ejemplo de mínimo privilegio?
> Porque `load_document` solo permite hacer exactamente lo que el agente necesita (cargar documentos válidos), mientras que `fetch_url` permite traer cualquier cosa de cualquier lugar — la tool acotada reduce la superficie de mal uso posible sin quitarle al agente la capacidad que sí necesita.

> [!question]- Además de prevenir mal uso, ¿qué otro beneficio tiene usar una tool acotada como `load_document` en vez de una genérica como `fetch_url`?
> Clarifica la intención de la tool — una descripción específica hace transparente para qué sirve, en vez de dejarla ambigua como una tool de propósito general.

## Distribución de tools en la práctica

> [!question]- ¿Por qué el agente coordinador de un sistema multi-agente de investigación no tiene tools de dominio propias?
> Porque su rol es orquestar el flujo (delegar, revisar, pedir revisiones) y no ejecutar tareas de investigación él mismo — darle tools de dominio lo convertiría en otro "agente de trabajo" además de coordinador, mezclando responsabilidades.

> [!question]- ¿Por qué el agente de síntesis del ejemplo tiene 4 tools que incluyen una tool acotada (`verify_fact`) en vez de solo tools puramente de síntesis?
> Porque el diseño bien hecho no solo evita darle tools ajenas a su rol — también reconoce cuándo una necesidad cruzada es lo suficientemente frecuente y simple como para merecer su propia tool acotada dentro del rol, en vez de forzar siempre una escalación completa.

## Síntesis

> [!question]- Un agente tiene 18 tools asignadas, varias de ellas fuera de su especialización, y usa `tool_choice: auto` en un pipeline que necesita garantizar datos estructurados. ¿Cuáles son los dos problemas de diseño presentes aquí?
> Primero, sobrecarga de tools (18 en vez de 4-5, y algunas fuera de su rol, invitando a mal uso). Segundo, una configuración de `tool_choice` incorrecta para el objetivo (`auto` no garantiza que el modelo llame una tool, cuando el pipeline necesita esa garantía — se necesitaría `any` o selección forzada).
