# Resultados — Tutor socrático: Human Review & Confidence Calibration

> [!info] Sesión evaluada el 2026-10-22
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 82/100**

De 14 preguntas, 11 fueron correctas, 2 parcialmente correctas y 1 incorrecta. Dominas bien las ideas grandes: el agregado engaña, los `confidence scores` crudos son relativos y hay que calibrarlos con un `labeled validation set`, el muestreo tiene que incluir lo automatizado, la cola de revisión se ordena por incertidumbre y la secuencia de validación va antes de automatizar. Resolviste bien la pregunta tipo Practice Scenario. Los huecos están en el detalle: el mecanismo exacto por el que el agregado se infla (volumen del tipo dominante), uno de los tres estratos del sampling (`confidence band`) y un criterio de priorización (accuracy histórica del tipo de documento).

## Conceptos con buen dominio

- **Argumento central / rol de `human review`**: lo describiste como un recurso escaso que hay que asignar a los casos que realmente lo necesitan.
- **Desglose por dos ejes (tipo de documento Y campo)**: detectaste que promediar los campos de un tipo repite la trampa del agregado.
- **Sampling de alta confianza**: identificaste los `novel error patterns` y la necesidad de seguir verificando lo automatizado.
- **Confidence relativa vs. absoluta**: la aplicaste bien tres veces (calibración, threshold de 0.99 y self-reported confidence de 5.2).
- **Calibración con `labeled validation set`**: sabes qué se necesita y que el resultado se usa para fijar thresholds.
- **Subir el threshold sin calibrar (Trampa 3)**: "cambiar un dato arbitrario por otro arbitrario".
- **Priorización dinámica vs. orden de llegada (Trampa 4)**: entendiste que la cola se ordena por incertidumbre, no cronológicamente.
- **Secuencia `Validation Before Automation`**: diste los pasos 1-4 en el orden correcto.
- **Trampa 1 (97% global + automatizar > 0.95)**: señalaste las dos fallas exactas que da la respuesta correcta de la guía.
- **Confidence por campo vs. por documento**: lo reconociste como la misma trampa del agregado.

## Áreas que necesitan revisión

### Por qué el agregado se ve tan bien (`The Aggregate Metrics Trap`)

- **Qué pasó:** Entendiste que los segmentos pequeños se esconden, pero confundiste los ejes: el 97% era de un solo campo (fechas) promediado entre tipos de documento, y lo explicaste como un promedio de campos. No mencionaste que el número sube porque el **tipo dominante en volumen** pesa más. En otra respuesta también usaste "nivel de confianza" cuando la palabra correcta era accuracy.
- **Contenido del resumen:**
  > - El agregado se ve excelente porque **el tipo de documento dominante en volumen** (ej. facturas estándar) "jala" el promedio hacia arriba.
  > - Los errores de **alto impacto** suelen estar justo en los segmentos minoritarios (recibos manuscritos, facturas internacionales, documentos de compliance escaneados) — y quedan **invisibles**.
- **Sección:** [[1 resumen]] → "1. `The Aggregate Metrics Trap` — el promedio esconde las fallas"
- **Analogía para reforzarlo:** Si en un salón hay 29 niños que miden 1.20 m y un adulto de 1.90 m, la estatura promedio sale 1.22 m: "todos son niños". El grupo grande decide el promedio y el caso raro desaparece. Recuerda también que la **confianza** es lo que el niño *dice* que mide y la **accuracy** es lo que marca la cinta métrica.

### Los tres estratos del `Stratified Random Sampling`

- **Qué pasó:** Mencionaste tipo de documento y campo, pero no la **banda de confianza**, que es justo el estrato que garantiza muestrear las extracciones de alta confianza.
- **Contenido del resumen:**
  > - La verificación continua toma **muestras representativas de cada estrato** (`stratum`): tipo de documento, banda de confianza (`confidence band`), tipo de campo.
  > - **Insight crítico:** hay que muestrear **las extracciones de alta confianza**, no solo las de baja.
- **Sección:** [[1 resumen]] → "2. `Stratified Random Sampling` — seguir midiendo lo automatizado"
- **Analogía para reforzarlo:** Un inspector de fruta no solo toma cajas de cada proveedor y de cada tipo de fruta: también toma de cada "etiqueta de calidad" (primera, segunda, tercera). Si nunca abre las cajas de "primera", nunca sabrá si ya vienen podridas.

### Criterios de priorización: accuracy histórica del tipo de documento

- **Qué pasó:** Con dos ítems de la misma confianza calibrada, uno de factura estándar y otro de formato internacional, no usaste el criterio de **mala accuracy histórica** para desempatar. Volviste a la zona de incertidumbre, que no distinguía entre los dos.
- **Contenido del resumen:**
  > - Qué priorizar:
  >   - Campos con **baja confianza** del modelo.
  >   - Extracciones de documentos fuente **ambiguos o contradictorios**.
  >   - **Tipos de documento con mala accuracy histórica**.
  >   - Campos donde el modelo expresa **incertidumbre** (varias interpretaciones posibles).
- **Sección:** [[1 resumen]] → "4. `Reviewer Capacity Prioritisation` — los revisores son escasos"
- **Analogía para reforzarlo:** Dos alumnos sacan la misma calificación en un examen sorpresa, pero uno siempre ha reprobado esa materia y el otro nunca. El maestro revisa primero el del alumno que suele reprobar: el historial también cuenta como señal de riesgo, no solo la nota de hoy.

## Siguiente paso recomendado

Relee las Conclusiones 1, 2 y 4 del [[1 resumen]] y enfócate en los tres detalles de arriba: volumen dominante, `confidence band` y accuracy histórica. Ya tienes el criterio de fondo para intentar [[4 test]].
