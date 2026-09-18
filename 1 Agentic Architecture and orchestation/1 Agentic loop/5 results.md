# Resultados — Tutor socrático: Agentic loop

> [!info] Sesión evaluada el 2026-09-18
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 75/100**

Buen dominio de los mecanismos centrales del agentic loop: el ciclo de 4 pasos, el rol de `stop_reason`, la necesidad de agregar resultados de herramientas al historial, y dos de las cuatro trampas de examen (contenido tipo texto y `tool_choice: "any"`) quedaron claras y bien explicadas. Las áreas débiles están en los matices más finos: qué hacer con valores de `stop_reason` distintos a los dos principales, cuándo ceder la flexibilidad del modelo a reglas deterministas, y la diferencia exacta entre "loop descontrolado" y "terminación prematura" al evaluar el distractor de límites de iteración.

## Conceptos con buen dominio

- Los dos valores principales de `stop_reason` (`tool_use`, `end_turn`) y qué acción dispara cada uno — respuesta directa y completa.
- La necesidad de agregar el resultado de la herramienta al historial de conversación — explicó correctamente la consecuencia de omitirlo.
- Trampa 1 (revisar tipo de contenido en vez de `stop_reason`) — identificó el caso exacto (texto + `tool_use` en la misma respuesta) sin necesidad de pistas.
- Trampa 4 (forzar `tool_choice: "any"`) — conectó bien la causa (forzar siempre herramienta) con el riesgo (loop infinito).

## Áreas que necesitan revisión

### Valores adicionales de `stop_reason` en producción

- **Qué pasó:** Al preguntar por la recomendación práctica ante valores como `pause_turn`, `max_tokens` o `refusal`, la respuesta fue que "ninguno es motivo para terminar el ciclo" — correcto en que no son `end_turn`, pero no capturó que la recomendación es "revisar por qué", no simplemente continuar el loop igual que con `tool_use`.
- **Repasa esto en:** [[1 resumen]] → sección "2. `stop_reason` es la única señal confiable"
- **Analogía para reforzarlo:** Es como un semáforo con una luz amarilla intermitente que no es ni "sigue" (verde) ni "alto total" (rojo) — no significa "avanza igual que si fuera verde", significa "detente un momento y evalúa la situación antes de decidir qué hacer".

### Cuándo ceder el enfoque dirigido por el modelo a reglas deterministas

- **Qué pasó:** Al preguntar en qué situación las reglas programáticas deben imponerse sobre la flexibilidad del modelo, la respuesta fue genérica ("cuando queremos que algo pase el 100% de las veces") sin mencionar los dominios específicos que da el resumen: financiero, de seguridad o regulatorio.
- **Repasa esto en:** [[1 resumen]] → sección "3. Toma de decisiones dirigida por el modelo (model-driven decision-making)"
- **Analogía para reforzarlo:** Un piloto de avión puede improvisar la ruta según el clima, pero en el aterrizaje hay un checklist fijo que no se salta nunca — no es cuestión de preferencia, es porque el costo de un error ahí es demasiado alto.

### Distractor de límites de iteración ante terminación prematura

- **Qué pasó:** Ante el escenario específico de terminación prematura del loop, la respuesta describió pros/contras generales de los límites de iteración, sin señalar que el error del distractor es aplicar la solución equivocada al problema equivocado: los límites de iteración resuelven loops que **no terminan** (descontrolados), no loops que **terminan antes de tiempo**.
- **Repasa esto en:** [[1 resumen]] → callout "Distractor frecuente en el examen"
- **Analogía para reforzarlo:** Si el despertador suena demasiado temprano, la solución no es "poner un límite a cuántas veces puede sonar" — el problema es que la hora está mal configurada, no que suene de más.

## Siguiente paso recomendado

Antes de intentar [[4 test]], repasa específicamente el callout de "Distractor frecuente en el examen" y la sección sobre valores adicionales de `stop_reason` — son los dos puntos donde el examen real suele tender la trampa. El resto del tema está en buen nivel.
