# Resultados — Tutor socrático: Escalation & Ambiguity Resolution

> [!info] Sesión evaluada el 2026-10-07
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 91/100**

Dominio sólido del tema: de 9 preguntas, 8 fueron correctas y 1 parcialmente correcta. Identificaste sin dudar los tres triggers válidos, distinguiste bien entre vacío y violación de política, y rechazaste con buenos argumentos los dos antipatrones (sentimiento y confianza autorreportada). La única debilidad fue al justificar la mejor práctica de implementación: elegiste bien, pero la razón que diste fue "es lo más sencillo", no el argumento de fondo de la guía. Además, en el caso de múltiples matches no mencionaste los identificadores concretos ni el riesgo de equivocarse de cuenta.

## Conceptos con buen dominio

- **Los tres triggers válidos de escalación**: los nombraste completos y con precisión (solicitud explícita, vacío de política, incapacidad genuina de progresar).
- **Vacío vs. violación de política (Trampa 5)**: distinguiste bien que la violación se resuelve aplicando la política y que el vacío requiere juicio humano.
- **`Sentiment-based escalation` (Trampa 1)**: explicaste que la emoción no correlaciona con la complejidad y que lo correcto es empatía + acción.
- **Frustración que se convierte en solicitud explícita**: reconociste que cuando el cliente reitera que quiere un humano, el caso pasa a ser trigger 1.
- **Escalar de inmediato ante una solicitud explícita (Trampa 3)**: explicaste que no se investiga antes, aunque el caso parezca trivial.
- **`Self-reported confidence scores` (Trampa 2)**: explicaste la mala calibración y en qué dirección falla (inseguro en lo fácil, confiado en lo difícil).
- **Múltiples matches (Trampa 4)**: tuviste claro que nunca se infiere el registro y que se pide información adicional.

## Áreas que necesitan revisión

### Por qué los criterios explícitos + `few-shot` son el primer paso

- **Qué pasó:** Elegiste correctamente agregar criterios explícitos con `few-shot` al `system prompt`, pero lo justificaste solo por ser "la más sencilla". Te faltó explicar que los ejemplos muestran casos concretos de escalar vs. resolver de forma autónoma, y que las alternativas (clasificador, análisis de sentimiento, scores de confianza) se apoyan en los mismos antipatrones que ya se descartaron. Después, en la pregunta de refuerzo, sí explicaste bien por qué el sentimiento es la señal equivocada.
- **Contenido del resumen:**
  > La forma más efectiva de calibrar cuándo escalar es agregar **criterios explícitos de escalación**, acompañados de **ejemplos `few-shot`** dentro del `system prompt`, que demuestren casos concretos de "esto se escala" vs. "esto se resuelve de forma autónoma". Esto se recomienda **antes** de invertir en infraestructura más compleja (modelos clasificadores dedicados, análisis de sentimiento, scores de confianza), porque esas alternativas son precisamente los antipatrones que ya se descartaron.
- **Sección:** [[1 resumen]] → "5. Mejor práctica de implementación: criterios explícitos + few-shot en el system prompt"
- **Analogía para reforzarlo:** Si un empleado nuevo comete errores, primero le das un manual con ejemplos de "esto sí, esto no". No compras un detector de mentiras para vigilarlo, y menos uno que mide algo que no tiene que ver con hacer bien el trabajo. El manual no solo es más barato, también apunta a lo que de verdad importa.

### Múltiples matches: identificadores concretos y riesgo

- **Qué pasó:** La regla de fondo la tuviste bien (preguntar, no adivinar), pero no dijiste qué identificadores pedir ni qué se arriesga al elegir por heurística.
- **Contenido del resumen:**
  > La acción correcta es pedirle al cliente **identificadores adicionales** — email, teléfono, número de orden — para desambiguar con certeza.

  > [!warning] Por qué importa
  > Elegir heurísticamente entre varios matches puede significar actuar sobre la cuenta equivocada: exponer datos de otra persona o tomar una acción (reembolso, cambio) sobre el pedido incorrecto. La ambigüedad de identidad se resuelve pidiendo más información, no adivinando.
- **Sección:** [[1 resumen]] → "4. Múltiples matches de cliente → pedir identificadores, nunca heurística"
- **Analogía para reforzarlo:** En una paquetería llegan dos personas llamadas "María López" a recoger un paquete. El empleado no se lo da a la que "se ve más probable": les pide su identificación o el número de guía. Si se equivoca, la otra María se queda sin su paquete y alguien más ve lo que no le correspondía.

## Siguiente paso recomendado

Relee la sección 5 del resumen para poder justificar los criterios explícitos + `few-shot` con el argumento de la guía, no solo por ser sencillos. Por lo demás, ya estás listo para [[4 test]].
