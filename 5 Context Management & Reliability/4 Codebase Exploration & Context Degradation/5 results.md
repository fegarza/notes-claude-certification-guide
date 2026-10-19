# Resultados — Tutor socrático: Codebase Exploration & Context Degradation

> [!info] Sesión evaluada el 2026-10-15
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 69/100**

De 17 preguntas: 8 correctas, 8 parcialmente correctas y 1 sin respuesta. La intuición general del tema es sólida: reconoces el síntoma de la `context degradation`, por qué el scratchpad es inmune, que la clave de los subagentes es `context isolation` y que el "cold start" se resuelve con `summary injection`. Las respuestas parciales comparten un patrón: aciertas el "qué" pero omites el "por qué" o el "quién" (por qué ocurre el cold start, el propósito de `/compact`, quién usa el scratchpad vs. el manifest, qué se pierde al reiniciar). Además, al principio atribuiste la degradación a una "ventana limitada", la idea que la guía descarta explícitamente.

## Conceptos con buen dominio

- **Síntoma de `context degradation`**: identificaste el paso de referencias específicas a respuestas genéricas ("typical patterns").
- **Ventana más grande no lo arregla (escenario 200k vs. 1M)**: explicaste que la causa es la acumulación de output verboso, no el tamaño de la ventana.
- **Por qué el scratchpad es inmune**: dijiste que vive fuera del contexto de la conversación.
- **Scratchpad desde el inicio, no como rescate**: detectaste de inmediato el error de crearlo cuando el agente ya se degradó.
- **`Subagent delegation` = `context isolation`, no paralelización**: respondiste con precisión que el contexto del coordinador queda limpio y solo recibe hallazgos clave.
- **Delegar preguntas específicas**: entendiste que "explora todo" reproduce el problema dentro del subagente.
- **Segundo beneficio de la `summary injection`**: les da comprensión arquitectónica para hacer las preguntas correctas.
- **Flujo de `crash recovery`**: cada agente exporta su estado a un manifest en una ubicación conocida, y el coordinador lo carga e inyecta on resume.

## Áreas que necesitan revisión

### El mecanismo: calidad de atención, no límite de tokens

- **Qué pasó:** concluiste bien que una ventana más grande también se llena, pero describiste el mecanismo como consecuencia de una "ventana de contexto limitada". Atribuiste el desplazamiento a "nuevos hallazgos" en vez de al output verboso. Lo corregiste en la pregunta siguiente.
- **Contenido del resumen:**
> 1. Cada paso de exploración genera **output verboso** (contenido de archivos, resultados de búsqueda, listados de directorios).
> 2. Ese output se **acumula** en el contexto de la conversación.
> 3. Los descubrimientos tempranos y precisos quedan **empujados hacia atrás**, mientras el output reciente domina.
> 4. La **atención del modelo se desplaza** a lo reciente y pierde las referencias específicas a lo anterior.
>
> > [!important] El insight crítico
> > `Context degradation` **no es un problema de token limit**. El modelo no se está quedando sin espacio — está perdiendo el rastro de los detalles. **Aumentar la ventana de contexto no lo arregla**: una ventana más grande también se llena de output verboso.
- **Sección:** [[1 resumen]] → "1. `Context degradation` — qué es y por qué pasa"
- **Analogía para reforzarlo:** en una fiesta ruidosa no oyes a tu amigo, aunque el salón sea enorme. El problema no es que falte espacio, sino que el ruido tapa su voz. Un salón más grande con el mismo ruido no te deja oírlo mejor.

### Qué debe contener un scratchpad

- **Qué pasó:** identificaste que un scratchpad genérico no sirve, pero no dijiste qué debe contener en su lugar.
- **Contenido del resumen:**
> - Debe contener **cosas específicas** (nombres de clases, rutas de archivo, cadenas de dependencias, hallazgos críticos), no generalidades.
- **Sección:** [[1 resumen]] → "2. `Scratchpad files` — la mitigación principal"
- **Analogía para reforzarlo:** una lista del súper que dice "comprar comida" no sirve. Una que dice "leche deslactosada 1L, pasillo 4" sí. El scratchpad es útil solo si tiene "pasillo y marca".

### Por qué ocurre el "cold start"

- **Qué pasó:** nombraste bien el problema y la solución (`summary injection`), pero no explicaste la causa raíz.
- **Contenido del resumen:**
> Los subagentes **no heredan** el contexto del coordinador: lo que no se les pasa explícitamente en el prompt, no lo saben. Por eso la `summary injection` es necesaria.
- **Sección:** [[1 resumen]] → "4. `Summary injection` entre fases"
- **Analogía para reforzarlo:** un plomero nuevo que llega a tu casa no sabe lo que te dijo el anterior. Si no le cuentas que la fuga ya se encontró en el baño, va a revisar toda la casa otra vez.

### Propósito de `/compact`: calidad, no cantidad

- **Qué pasó:** sabías que se usa proactivamente, pero no mencionaste su propósito (proteger la calidad del contexto) ni qué hace (resumir la conversación preservando la información clave).
- **Contenido del resumen:**
> - Cuando el contexto se llena de output verboso de descubrimiento (archivos, búsquedas, listados), `/compact` **resume la conversación** para liberar espacio **preservando la información clave**.
> - Se usa **proactivamente** durante la exploración, **no solo al llegar al límite**.
> - Su propósito es proteger la **calidad** del contexto, no solo la **cantidad**.
- **Sección:** [[1 resumen]] → "5. `/compact` — usarlo de forma proactiva"
- **Analogía para reforzarlo:** lavar los platos mientras cocinas, en vez de esperar a que no quede ninguno limpio. Si esperas, ya tienes una cocina caótica en la que no encuentras nada.

### Contenido del `manifest`

- **Qué pasó:** no recordaste qué tipos de información incluye el manifest.
- **Contenido del resumen:**

| Qué incluye el `manifest` | Para qué |
|---|---|
| Qué se ha explorado (archivos leídos, búsquedas hechas) | No repetir exploración |
| Hallazgos clave hasta ahora | No perder conocimiento |
| Fase actual y siguientes pasos | Saber dónde retomar |
| Preguntas pendientes / temas sin resolver | No olvidar cabos sueltos |

- **Sección:** [[1 resumen]] → "6. `Crash recovery` con `structured state manifests`"
- **Analogía para reforzarlo:** es la nota que dejas al compañero del siguiente turno en un hospital: qué pacientes ya revisaste, qué encontraste, en qué ibas y qué quedó pendiente.

### Scratchpad vs. manifest: quién lo usa

- **Qué pasó:** distinguiste bien el momento (durante la sesión vs. al reanudar), pero omitiste quién usa cada uno. También sugeriste que el manifest se escribe solo cuando hay un error, cuando en realidad se va exportando durante el trabajo.
- **Contenido del resumen:**
> Ambos persisten cosas fuera del contexto, pero con propósito distinto: el **scratchpad** es la "libreta de trabajo" que el agente consulta **durante** la sesión para no degradarse; el **manifest** es el **estado estructurado** (qué se exploró, fase, next steps) que el **coordinador** carga para **reanudar** tras una falla.
- **Sección:** [[1 resumen]] → "6. `Crash recovery` con `structured state manifests`"
- **Analogía para reforzarlo:** el scratchpad es la libreta personal del estudiante durante el examen. El manifest es el guardado automático de un videojuego: se va escribiendo solo y lo carga el juego (el coordinador) cuando se va la luz.

### Trampa 3: reiniciar sin guardar estado

- **Qué pasó:** te enfocaste en que el scratchpad debió usarse desde el inicio, pero no señalaste que reiniciar pierde todo el conocimiento acumulado. Tampoco dijiste que lo correcto es persistir antes de reiniciar e inyectarlo en la nueva sesión.
- **Contenido del resumen:**
> [!warning] Trampa 3 — Reiniciar la sesión para "curar" la degradación sin guardar estado
> **El error:** abrir una sesión nueva y limpia (y quizás pedir "explora más eficientemente esta vez"). **Por qué falla:** reiniciar **pierde todo el conocimiento acumulado** y obliga a re-explorar. **Lo correcto:** persistir hallazgos en `scratchpad files` y `state manifests` **antes** de reiniciar, y luego **inyectarlos** en la nueva sesión.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** formatear la computadora porque va lenta está bien, pero solo si antes respaldaste tus archivos. Formatear sin respaldo "arregla" la lentitud y te deja sin nada.

### Trampa 5: relación con la Trampa 1

- **Qué pasó:** explicaste bien que precargar todo causa degradación, pero no la relacionaste con la Trampa 1 ni mencionaste la alternativa correcta.
- **Contenido del resumen:**
> [!warning] Trampa 5 — Precargar todo el codebase en el contexto inicial
> **El error:** "si le doy toda la estructura del codebase desde el inicio, tendrá menos que descubrir". **Por qué falla:** es el mismo error que la Trampa 1 disfrazado — llenas el contexto de volumen desde el arranque, que es justo lo que causa la degradación. **Lo correcto:** explorar de forma dirigida (subagentes con preguntas específicas) y persistir solo los hallazgos clave. *(Aparece como distractor en el escenario de práctica de la guía.)*
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** darle a alguien la enciclopedia completa para que conteste una pregunta no lo ayuda. Es mejor mandarlo directo al tomo y la página correcta.

## Siguiente paso recomendado

Repasa en [[1 resumen]] la tabla del `manifest`, la nota "Scratchpad vs. manifest" y las Trampas 3 y 5, y practica contestando siempre el "por qué" y el "quién", no solo el "qué". Después pasa a [[4 test]]: las preguntas tipo examen suelen poner como distractor justo la respuesta "casi correcta".
