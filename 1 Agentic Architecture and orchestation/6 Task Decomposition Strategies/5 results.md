# Resultados — Tutor socrático: Task Decomposition Strategies

> [!info] Sesión evaluada el 2026-10-25
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 88/100**

De 14 preguntas, 12 fueron correctas y 2 parcialmente correctas, sin errores conceptuales. El estudiante superó las 4 áreas débiles de la sesión anterior (2026-09-19): el caso límite de la definición de pipeline fijo, por qué mapear antes de priorizar, por qué la inconsistencia prueba un problema de atención y no de comprensión, y las dos mitades del batching. Lo que queda por pulir no es de concepto sino de precisión: a veces llega a la conclusión correcta sin nombrar el mecanismo o el criterio exacto con que la guía lo justifica.

## Conceptos con buen dominio

- Criterio de selección de patrón (¿pasos conocidos de antemano?): descartó de inmediato la complejidad o la importancia como criterio.
- Definición exacta de pipeline fijo: reconoció que agregar un paso según un resultado intermedio ya no es pipeline fijo (no agregar, no saltar, no reordenar).
- Fases de la decomposición dinámica: explicó que priorizar sin mapear deja un plan a ciegas, y que el plan se adapta conforme aparecen hallazgos.
- Síntomas de attention dilution: identificó los tres síntomas como una sola causa raíz.
- Inconsistencia como prueba de atención y no de comprensión: usó el contrato que sí se marcó bien como evidencia de que el modelo sí sabe.
- Arquitectura multi-pass: nombró correctamente las dos capas (local por ítem e integración cruzada) y por qué la integración corre al final.
- Trampa 1 (modelo o ventana de contexto más grande): la refutó con una analogía propia muy buena (un escritorio más grande no da más atención).
- Trampa 3 (pipeline fijo en investigación abierta): la aplicó bien a un escenario nuevo de debugging de un microservicio.
- Trampa 4 (batching): dio completas las dos mitades, lo que mejora y lo que falta.
- Revisión de código multi-archivo como pipeline fijo: entendió que un input variable no implica pasos variables.

## Áreas que necesitan revisión

### Trampa 2 — el mecanismo de por qué "mejorar el prompt" no basta

- **Qué pasó:** Concluyó bien que es un problema de arquitectura y no de instrucciones, pero no explicó el mecanismo: sigue siendo una sola pasada con un presupuesto de atención fijo repartido entre demasiados ítems. En el examen, saber nombrar el mecanismo es lo que permite descartar el distractor de "mejorar el prompt" cuando viene con métricas que sí mejoran.
- **Contenido del resumen:**
> Esto **no es** "escribir un mejor prompt" — un prompt más enfático mejora la calidad promedio, pero no resuelve el problema fundamental de asignación de atención: sigue siendo una sola pasada con presupuesto fijo repartido entre demasiados ítems.
- **Sección:** [[1 resumen]] → "4. La solución: arquitectura multi-pass (no modelo, no prompt)"
- **Analogía para reforzarlo:** Es como repartir una pizza entre 40 personas y pedirles: "por favor, que cada uno coma bien". La petición puede hacer que repartan un poco mejor, pero la pizza sigue siendo una sola. Para que todos coman bien hace falta una pizza por persona, no una mejor petición.

### Razonamiento de la tabla — por qué pipeline fijo en tareas estructuradas

- **Qué pasó:** En el caso de extracción de facturas tocó la idea de "mismos outputs", pero no la nombró con el criterio de la guía (consistencia y confiabilidad por encima de la adaptabilidad), y metió un argumento de "ventana de tiempo" que no viene en el resumen.
- **Contenido del resumen:**

| Características de la tarea | Patrón | Razonamiento |
|---|---|---|
| Pasos conocidos de antemano, entrada estructurada | Pipeline fijo | "La consistencia y confiabilidad pesan más que la adaptabilidad" |
| Alcance abierto, desconocido | Decomposición dinámica | "La adaptabilidad es esencial cuando el problema no está completamente definido" |

- **Sección:** [[1 resumen]] → "2. El framework de decisión: una tabla, un criterio"
- **Analogía para reforzarlo:** Una máquina que sella latas en una fábrica no "improvisa" con cada lata. Su valor está en que la lata 1 y la lata 10 000 quedan selladas exactamente igual. Si cada lata la sellara alguien que decide sobre la marcha, a veces quedaría mejor, pero ya no podrías confiar en que todas quedan bien.

## Siguiente paso recomendado

Repasa en una lectura rápida la tabla del framework de decisión (sobre todo la columna "Razonamiento") y el bullet de "no es escribir un mejor prompt". Con eso el tema queda sólido y estás listo para [[4 test]].
