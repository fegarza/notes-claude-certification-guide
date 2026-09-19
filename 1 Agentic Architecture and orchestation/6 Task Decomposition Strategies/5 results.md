# Resultados — Tutor socrático: Task Decomposition Strategies

> [!info] Sesión evaluada el 2026-09-19
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 78/100**

El estudiante domina con solidez el criterio central de selección de patrón (pasos conocidos de antemano vs. descubiertos sobre la marcha) y lo aplicó correctamente incluso a un escenario nuevo no memorizado. También entiende bien que la dilución de atención es un problema estructural que ni un modelo más grande ni un mejor prompt resuelven. Las respuestas tendieron a ser correctas pero incompletas en varios casos (dando solo una parte de una respuesta de dos partes), y hubo un error conceptual claro en el caso límite de la definición exacta de "pipeline fijo".

## Conceptos con buen dominio

- Criterio único de decisión (pasos conocidos vs. emergentes) — lo aplicó correctamente a un escenario nuevo (reporte financiero vs. debugging de microservicio) sin depender de los ejemplos memorizados de la tabla.
- Multi-pass como solución estructural — identificó correctamente que sin pasada de integración cruzada no se detectan problemas transversales entre ítems, incluso con análisis local profundo.
- Rechazo de "modelo más grande" y "mejor prompt" como soluciones — entendió que ninguna de las dos cambia la asignación estructural del presupuesto de atención.
- Nombre alternativo de pipeline fijo — identificó correctamente "prompt chaining".
- Generalización del criterio a escenarios fuera de la tabla — reconoció que el examen evalúa el criterio subyacente, no el reconocimiento de ejemplos memorizados.

## Áreas que necesitan revisión

### Definición exacta de "pipeline fijo" — caso límite de secuencia condicional

- **Qué pasó:** Ante un pipeline con un salto condicional ("si el Paso 1 tiene error, salta al Paso 4"), el estudiante respondió que seguía siendo pipeline fijo porque no se agregó ningún paso nuevo. Pero la definición exacta de la guía es que la secuencia *no cambia según resultados intermedios* — y en ese ejemplo sí cambia (se salta el Paso 3) dependiendo de un resultado intermedio, lo que técnicamente ya lo convierte en una forma de decomposición dinámica.
- **Contenido del resumen:** 
> **Fixed Sequential Pipelines** (también llamado *prompt chaining*): "el workflow se define de antemano. El Paso 1 corre, su salida alimenta al Paso 2, la salida del Paso 2 alimenta al Paso 3, y así sucesivamente. La secuencia no cambia según los resultados intermedios."
- **Sección:** [[1 resumen]] → "Dos patrones de decomposición, dos definiciones exactas"
- **Analogía para reforzarlo:** Es como decir que un tren sigue "yendo en línea recta" aunque tenga un desvío que se activa según una señal en la vía — en el momento en que la vía puede cambiar según lo que se detecta, ya no es una ruta fija, es una ruta que reacciona.

### Fases de la decomposición dinámica — por qué mapear primero

- **Qué pasó:** Al preguntar qué problema tendría un plan que se salta la fase de "mapear la estructura" y va directo a "crear un plan priorizado", la respuesta fue vaga ("no sabe por dónde empezar") en vez de señalar que el plan quedaría basado en suposiciones no verificadas sobre dependencias y áreas de impacto reales.
- **Contenido del resumen:**
> Ante una tarea abierta como "agregar tests exhaustivos a un codebase legacy", el patrón correcto es primero ==**mapear la estructura**, luego **identificar las áreas de mayor impacto**, y recién ahí **crear un plan priorizado que se adapta**== a medida que se descubren dependencias — no intentar escribir el plan completo desde el día uno.
- **Sección:** [[1 resumen]] → "Dos patrones de decomposición, dos definiciones exactas"
- **Analogía para reforzarlo:** Es como intentar decidir qué muebles priorizar reparar en una casa sin haber recorrido la casa primero — puedes "priorizar" algo que ni siquiera existe o pasar por alto el problema real porque nunca lo viste.

### Dilución de atención — por qué la inconsistencia (no solo la degradación) prueba que es atención y no comprensión

- **Qué pasó:** Al preguntar por qué el síntoma de "mismo patrón marcado en un archivo y aprobado en otro" prueba específicamente un problema de *presupuesto de atención* y no de "el modelo no entiende el patrón", el estudiante solo describió la degradación general con el tiempo, sin conectar que la inconsistencia misma es la prueba de que el modelo sí sabe identificar el patrón (lo hizo bien al menos una vez).
- **Contenido del resumen:**
> "La dilución de atención es un modo de fallo específico que ocurre cuando un agente procesa demasiados ítems en una sola pasada. El resultado es una profundidad inconsistente — el agente produce un análisis exhaustivo para algunos ítems y pasa por alto problemas obvios en otros."
- **Sección:** [[1 resumen]] → "Dilución de atención: qué es y cómo se ve"
- **Analogía para reforzarlo:** Si un profesor de manejo reprueba a un alumno por no frenar en un alto pero aprueba a otro alumno que hizo exactamente lo mismo, eso no significa que el profesor "no sepa qué es un alto" — sabe perfectamente qué es, solo que dejó de prestarle atención en el segundo caso.

### Batching — qué sí logra, no solo qué le falta

- **Qué pasó:** Al preguntar si dividir 14 archivos en 3 lotes resuelve la dilución de atención, el estudiante respondió correctamente que sigue faltando la integración cruzada, pero omitió la primera mitad de la respuesta: que el batching sí reduce la dilución *dentro* de cada lote.
- **Contenido del resumen:**
> Agrupar en lotes (*batching*) reduce la dilución **dentro** de cada lote, pero por sí solo no es la solución completa: si no se agrega una pasada de integración cruzada, se pierden los problemas que cruzan los límites del lote.
- **Sección:** [[1 resumen]] → "La solución: arquitectura multi-pass (no modelo, no prompt)"
- **Analogía para reforzarlo:** Es como limpiar una casa habitación por habitación en vez de toda de un jalón — cada habitación queda mejor limpia (eso sí mejora), pero si nunca revisas el pasillo que conecta todas, sigue habiendo desorden que se te escapa entre una habitación y otra.

## Siguiente paso recomendado

Repasar puntualmente la definición exacta de "pipeline fijo" (el caso límite de secuencia condicional) y la sección de dilución de atención antes de intentar [[4 test]] — el resto del tema está sólido.
