# Resultados — Tutor socrático: Iterative Refinement Techniques

> [!info] Sesión evaluada el 2026-09-29
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 88/100**

Mejora clara respecto a la sesión anterior (68/100). Eliges la técnica correcta en todos los escenarios, justificas bien por qué los ejemplos le ganan a la prosa, aplicas el criterio batch vs secuencial en ambos sentidos y corregiste la dirección del interview pattern. Lo que falta es precisión en tres puntos: qué hacer cuando falla un edge case (volviste a proponer cambiar de técnica), la formulación exacta de la relación test case ↔ ejemplo, y la mitad "del modelo" en la pregunta de diagnóstico interview vs examples. De 14 preguntas: 11 correctas y 3 parciales.

## Conceptos con buen dominio

- **Concrete input/output examples como primera técnica**: la eliges ante interpretación inconsistente y sabes que bastan 2-3 pares.
- **Por qué no refinar la prosa (Trampa 1)**: explicaste que la prosa sigue abierta a interpretación y que los ejemplos eliminan la ambigüedad. Era un área débil y ya la dominas.
- **Proceso de 4 pasos**: sabes verificar la generalización con un caso nuevo y agregar un ejemplo de edge case cuando falla uno.
- **Test-driven iteration**: la eliges para transformaciones complejas, defines primero happy path, edge cases y performance, y compartes el test failure tal cual.
- **Interview pattern**: la dirección es correcta (Claude te pregunta a ti) y lo reconoces como la solución cuando alguien usa ejemplos en un dominio que desconoce (Trampa 3).
- **Batch vs sequential feedback**: batch para arreglos que interactúan, secuencial para independientes, y sabes qué falla si inviertes el criterio.

## Áreas que necesitan revisión

### Cuántos ejemplos y qué hacer cuando falla un edge case (Trampa 4)

- **Qué pasó:** ante alguien que quería dar 30 ejemplos, dijiste bien que 2-3 bastan, pero propusiste "cambiar a test-driven iteration". Para un edge case que falla, lo correcto es agregar un ejemplo específico de ese edge case sin cambiar de técnica. Además justificaste con el consumo de tokens, que no es la razón que da el resumen.
- **Contenido del resumen:**
> [!warning] Trampa 4 — Creer que más ejemplos siempre es mejor
> **Error común**: intentar cubrir cada caso posible con decenas de ejemplos.
> **Correcto**: 2-3 bien elegidos (caso estándar + un edge case clave) bastan; el modelo generaliza el patrón. Si falla un edge case, se agrega un ejemplo específico de ese edge case.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** si le enseñas a un niño a sumar con 3 ejemplos y se equivoca solo cuando hay que "llevar", no le das 30 sumas más ni le cambias el método por un examen. Le enseñas **una** suma donde se lleva.

### Relación entre test cases y concrete examples

- **Qué pasó:** identificaste que los dos definen input y output esperado y que el test verifica, pero no llegaste a la idea exacta: un test case **es** un ejemplo concreto, solo que ejecutable y repetible.
- **Contenido del resumen:**
> [!note] Relación con la técnica 1
> Un test case con input de ejemplo y output esperado **es** un ejemplo concreto, pero ejecutable y repetible. Por eso el examen pide "proveer test cases específicos con input de ejemplo y output esperado" para arreglar el manejo de edge cases (ej. valores `null` en scripts de migración).
- **Sección:** [[1 resumen]] → "3. Test-driven iteration — la técnica para transformaciones complejas"
- **Analogía para reforzarlo:** una receta con foto del platillo terminado es un ejemplo. Un termómetro de cocina programado para pitar si el pollo no llega a 74 °C es la misma información, "así debe quedar", solo que se revisa sola cada vez que cocinas.

### Pregunta de diagnóstico interview vs examples (menor)

- **Qué pasó:** distinguiste bien los dos casos ("no sabes cómo atacarlo" vs "conoces los casos"), pero omitiste que en examples el problema es que **el modelo interpreta de forma inconsistente** lo que tú ya sabes.
- **Contenido del resumen:**
> [!warning] Key Concept de la guía — no confundir interview con examples
> - **Interview pattern** → dominio desconocido; *el developer* podría no ver consideraciones importantes.
> - **Concrete examples** → el developer **sabe exactamente** la transformación que quiere, pero *el modelo* la interpreta de forma inconsistente.
>
> Resuelven problemas distintos. La pregunta clave: ¿el hueco de conocimiento está en ti (→ interview) o en cómo el modelo lee tu descripción (→ examples)?
- **Sección:** [[1 resumen]] → "4. Interview pattern — la técnica para dominios desconocidos"
- **Analogía para reforzarlo:** si no sabes qué pedir en un restaurante extranjero, le pides al mesero que te pregunte qué te gusta (el hueco está en ti). Si sabes exactamente qué quieres y te lo traen mal, le señalas la foto del menú (el hueco está en cómo te entendió).

## Siguiente paso recomendado

Relee la Trampa 4 y el callout "Relación con la técnica 1", y memoriza la frase completa de diagnóstico "¿el hueco está en mí o en cómo el modelo lee mi descripción?". Se cubrió todo el checklist del tema, así que ya estás listo para [[4 test]].
