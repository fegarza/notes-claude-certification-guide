# Resultados — Tutor socrático: Validation, Retry, and Feedback Loops

> [!info] Sesión evaluada el 2026-10-11
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 84/100**

De 13 preguntas, 11 fueron correctas (varias con matices menores de precisión) y 2 parcialmente correctas, sin respuestas incorrectas. El dominio del núcleo del tema es sólido: los tres ingredientes del retry, la frontera de efectividad, la distinción sintaxis/semántica y el rol de Pydantic. Los puntos débiles son de precisión, no de concepto: el propósito exacto del self-correction en el schema, completar la respuesta ante un fallo no corregible (qué hacer en vez de reintentar) y el uso de los términos "formato" vs. "semántico".

## Conceptos con buen dominio

- **Retry ingenuo vs. retry-with-error-feedback** — identificó de inmediato que sin el error específico el modelo repite el mismo error.
- **Los tres ingredientes del retry** — nombró documento original, extracción fallida y error de validación específico.
- **Frontera de efectividad** — clasificó correctamente los 4 casos (formato, valor mal ubicado, documento externo, error matemático) y reconoció que la información ausente no se resuelve con más reintentos.
- **`detected_pattern`** — entendió su uso para analizar descartes y el ciclo extraer → validar → recolectar → refinar prompt → repetir.
- **Sintaxis vs. semántica** — distinguió qué elimina `tool_use` con JSON schema y qué no.
- **Pydantic (parsing + validators) y `ValidationError`** — entendió ambas funciones y por qué el error específico encaja con el loop de retry.
- **`strict: true` / `messages.parse`** — reconoció que no eliminan la capa semántica.
- **Datos fabricados (Trampa 3)** — identificó que forzar un campo inexistente provoca valores inventados, que el schema no puede detectar.

## Áreas que necesitan revisión

### Self-correction dentro del schema (`calculated_total`/`stated_total` y `conflict_detected`)

- **Qué pasó:** Describió bien qué representa cada campo, pero planteó la ventaja como "facilitarle las cosas al validador externo (Pydantic)", cuando el punto es que la discrepancia se marca sola **sin lógica externa**. En `conflict_detected` omitió que se extraen **ambos** valores contradictorios, no solo el booleano.
- **Contenido del resumen:**
  > En vez de depender solo de lógica de validación externa, el schema mismo puede incluir campos que se auto-delaten cuando hay un problema:
  >
  > - **`calculated_total` vs. `stated_total`**: se extraen *ambos* — la suma calculada de las líneas individuales y el total que dice el documento. Si difieren, la discrepancia queda marcada automáticamente, sin lógica externa.
  > - **`conflict_detected` (booleano)**: cuando la fuente contiene información contradictoria (ej. "pago a 30 días" en una sección y "net 60" en otra), se extraen ambos valores y se marca `conflict_detected: true`, en vez de que el modelo elija uno silenciosamente.
- **Sección:** [[1 resumen]] → "3. Diseño de self-correction dentro del schema"
- **Analogía para reforzarlo:** Es como un ticket de supermercado que imprime tanto el total de la caja como el conteo de artículos escaneados. Si no cuadran, cualquiera lo nota al mirar el ticket, sin llamar a un auditor. Y si dos etiquetas de precio dicen cosas distintas, el ticket debería mostrar ambas con una marca de "revisar", no quedarse con una en silencio.

### Qué hacer ante un fallo no corregible

- **Qué pasó:** En el escenario integrador reconoció que el `vendor_tax_id` (que vive en un documento externo) nunca se resolverá con más reintentos, pero no planteó la alternativa: marcar `null` o enviar a revisión humana, y clasificar el fallo **antes** de reintentar. Tampoco identificó por sí solo el riesgo de la instrucción "es obligatorio llenarlo", aunque sí lo resolvió bien cuando se le preguntó por separado.
- **Contenido del resumen:**
  > Si el documento no menciona el nombre de un departamento, reintentar no produce un valor correcto — solo hay dos opciones honestas: marcar el campo como `null` (si el schema lo permite) o enviar la extracción a revisión humana.

  > Retry-with-error-feedback funciona enviando el documento original, la extracción fallida y el error de validación específico — pero corrige errores de formato y estructura, no puede crear información ausente de la fuente, así que siempre hay que identificar primero si un fallo es corregible antes de reintentar.
- **Sección:** [[1 resumen]] → "2. La frontera de efectividad del retry" y "En una frase"
- **Analogía para reforzarlo:** Si llamas a un restaurante y te dicen que no tienen tu platillo en el menú, insistir cinco veces no lo hará aparecer, y exigir "tienes que servírmelo" solo hará que te traigan cualquier otra cosa con ese nombre. Lo sensato es preguntar antes si está en el menú y, si no, pedirle al gerente (el humano) o quedarte sin ese platillo (`null`).

### Vocabulario: discrepancia de formato vs. error semántico

- **Qué pasó:** Llamó "error de semántica" a un problema de formato de fecha (`03/15/2024` vs. `2024-03-15`) y, más adelante, dio "estructura/formato" como ejemplo de error semántico. En este tema "semántico" significa algo preciso: JSON válido pero con valores incorrectos.
- **Contenido del resumen:**
  > - **Errores de sintaxis de schema**: JSON malformado, campos requeridos faltantes, tipo de dato incorrecto — **eliminados por completo** por `tool_use` con JSON schema.
  > - **Errores semánticos de validación**: estructura JSON correcta, pero valores incorrectos — líneas que no suman, fechas mal ordenadas, valores en el campo equivocado. Requieren **lógica de validación** aparte del schema, y son el foco de los loops de retry de este tema.
- **Sección:** [[1 resumen]] → "5. Errores de sintaxis de schema vs. errores semánticos de validación"
- **Analogía para reforzarlo:** Un formulario con la letra bien escrita en cada casilla puede tener "sintaxis" perfecta y aun así decir que naciste en 2090: eso es semántico. Escribir la fecha como "15/03" en vez de "03-15" es solo la forma de anotarla, o sea, formato.

## Siguiente paso recomendado

El tema está bien dominado. Repasa rápido las secciones 2, 3 y 5 de [[1 resumen]] para afinar los matices de arriba, y ya estás listo para [[4 test]].
