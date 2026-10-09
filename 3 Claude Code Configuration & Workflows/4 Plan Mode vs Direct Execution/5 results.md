# Resultados — Tutor socrático: Plan Mode vs Direct Execution

> [!info] Sesión evaluada el 2026-09-27
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 77/100**

De 13 preguntas, 8 fueron correctas, 4 parcialmente correctas y 1 incorrecta. Dominas el criterio central (ambigüedad, no dificultad), qué hace plan mode, cómo funciona el Explore subagent y la clasificación de tareas del decision framework. Las respuestas parciales tuvieron el mismo patrón: la idea general correcta, pero sin el matiz exacto que el examen usa para separar la opción correcta de un distractor. Esos matices son la inconsistencia entre archivos, el "plan THEN direct", la degradación justo al implementar y la complejidad "ya declarada". El único error fue un detalle fino de activación: `/plan` solo funciona como prefijo.

## Conceptos con buen dominio

- **Ambigüedad, no dificultad:** sabes que un bug difícil pero bien acotado va con direct execution.
- **Qué hace plan mode:** explora y diseña sin modificar archivos, y así evita el costly rework cuando hay varios enfoques válidos.
- **Activar plan mode por modo y no por texto:** reconociste que pedirlo en el prompt no hace nada, y conoces las 3 rutas.
- **`Shift+Tab` y la status bar:** sabes cambiar de modo durante la sesión y confirmarlo en la barra de estado.
- **Explore subagent:** explicaste bien el flujo completo: aislamiento → summaries → solo el resumen vuelve → contexto principal limpio.
- **Trampa 2, plan mode para un single-file fix:** identificaste que solo agrega overhead innecesario.
- **Decision framework:** clasificaste bien el validation conditional, la elección entre arquitecturas de integración y el cambio de configuración.

## Áreas que necesitan revisión

### Activación con `/plan`: solo como prefijo

- **Qué pasó:** afirmaste que `/plan` activa plan mode aunque esté en medio del texto del prompt, "porque viene como comando".
- **Contenido del resumen:**
  > [!warning] Detalle fino
  > Escribir "in plan mode" **dentro del cuerpo** de un prompt **no hace nada**. `/plan` **al inicio** del prompt sí lo activa.

  | Ruta                            | Cuándo                                                              |
  | ------------------------------- | ------------------------------------------------------------------- |
  | `claude --permission-mode plan` | Al iniciar la sesión                                                |
  | `Shift+Tab`                     | Durante la sesión, hasta que la status bar muestre plan mode activo |
  | Prefijo `/plan` en un prompt    | Para un solo prompt                                                 |

- **Sección:** [[1 resumen]] → "Cómo se activa plan mode"
- **Analogía para reforzarlo:** es como el "Asunto:" de un correo. Si escribes "URGENTE" en el asunto, el filtro lo marca. Si lo escribes a la mitad del tercer párrafo, nadie lo toma como etiqueta, es solo una palabra más del texto.

### Multi-file modifications: el riesgo es la inconsistencia

- **Qué pasó:** en la migración de 45 archivos mencionaste las dependencias, pero no el riesgo concreto de ir directo archivo por archivo: que la migración quede aplicada de forma distinta en cada archivo.
- **Contenido del resumen:**
  > - **Multi-file modifications**: una library migration que afecta 45+ archivos necesita una estrategia **consistente**; sin plan, la migración se aplica de forma inconsistente entre archivos.
- **Sección:** [[1 resumen]] → "Plan mode: cuándo usarlo"
- **Analogía para reforzarlo:** pintar 45 cuartos de un hotel sin haber elegido antes el tono exacto. Cada pintor mezcla "más o menos el mismo blanco" y al final ningún cuarto combina con el de al lado.

### Patrón híbrido: plan THEN direct, no plan OR direct

- **Qué pasó:** cuando te preguntaron "¿plan mode o direct execution?" para la migración, listaste los criterios de cada modo por separado en vez de responder que se usan **los dos en secuencia**.
- **Contenido del resumen:**
  > [!warning] Trampa 3 — No reconocer el patrón híbrido plan-then-execute
  > **Error común:** tratar la decisión como excluyente ("¿plan mode **o** direct execution?") para una library migration.
  > **Por qué está mal:** el examen espera que combines ambos — plan mode para investigar y diseñar la estrategia, direct execution para aplicarla.
  > **Forma correcta:** plan **THEN** direct.
- **Sección:** [[1 resumen]] → "El patrón híbrido: plan THEN execute"
- **Analogía para reforzarlo:** una receta de cocina. Primero lees la receta y preparas todos los ingredientes ya picados y medidos (plan), y luego cocinas paso a paso sin pensar (execute). Nadie se pregunta "¿leo la receta **o** cocino?".

### Explore subagent: cuándo duele el contexto lleno

- **Qué pasó:** identificaste que el context window se llena en tareas multi-phase, pero no mencionaste la consecuencia: se degrada la calidad de las respuestas posteriores, justo en la fase de implementación.
- **Contenido del resumen:**
  > - Si todo eso fluye a la conversación principal, **llena el context window** y degrada la calidad de las respuestas posteriores (justo cuando toca implementar).
- **Sección:** [[1 resumen]] → "El Explore subagent"
- **Analogía para reforzarlo:** llenas tu escritorio con todos los papeles de la investigación y, cuando por fin te toca escribir el informe, no encuentras espacio ni la hoja importante. El desorden no molesta mientras investigas; molesta cuando necesitas trabajar enfocado.

### Complejidad declarada upfront (Trampa 4 / Sample Question 5)

- **Qué pasó:** respondiste que "nunca" conviene empezar con direct execution por el rework. "Nunca" es una sobregeneralización, porque direct execution es correcto para cambios acotados. Además, te faltó la idea clave: la complejidad ya está en los requisitos, así que no es especulativa.
- **Contenido del resumen:**
  > [!warning] Trampa 4 — Empezar en direct execution y cambiar a plan mode solo cuando "aparezca" la complejidad
  > **Error común:** "empiezo directo y si se complica, planeo".
  > **Por qué está mal:** cuando la complejidad ya está **declarada en los requisitos** (ej. monolith restructuring), es conocida, no especulativa. Esperar sorpresas es el error.
  > **Forma correcta:** elegir plan mode **upfront**. Esta es exactamente la opción D incorrecta de la **Sample Question 5** oficial.
- **Sección:** [[1 resumen]] → "Reconocer la complejidad desde el inicio" y "Trampas de examen"
- **Analogía para reforzarlo:** si el pronóstico ya dice "tormenta a las 3 pm", no sales sin paraguas "a ver si llueve". La lluvia no es una sorpresa: ya te la anunciaron.

## Siguiente paso recomendado

Repasa en [[1 resumen]] las Trampas 3 y 4 y el detalle de `/plan` como prefijo, poniendo atención a la palabra exacta que las define ("THEN", "declarada", "al inicio"). Después ya estás listo para [[4 test]], donde esos matices son justo los que separan la opción correcta de los distractores.
