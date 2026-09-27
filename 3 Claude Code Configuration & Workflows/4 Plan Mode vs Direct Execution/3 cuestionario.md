Repaso de [[1 resumen]].

## Argumento

> [!question]- ¿Cuál es el criterio decisivo para elegir entre plan mode y direct execution?
> La **ambigüedad** de la tarea, no su dificultad: si está claro qué, dónde y cómo cambiar → direct execution; si no → plan mode.

> [!question]- ¿La elección de modo es cuestión de preferencia personal?
> No. Hay criterios claros para cada modo, y el examen evalúa que se apliquen.

## Plan mode: cuándo usarlo

> [!question]- ¿Qué cinco características de una tarea indican plan mode?
> Large-scale changes, multiple valid approaches, architectural decisions, multi-file modifications y necesidad de codebase exploration.

> [!question]- ¿Por qué las decisiones de arquitectura (service boundaries, API contracts) piden plan mode?
> Porque tienen consecuencias *downstream*; decidir mal y descubrirlo tarde provoca costly rework.

> [!question]- ¿Qué riesgo concreto tiene una library migration de 45+ archivos sin plan?
> Aplicar la migración de forma inconsistente entre archivos.

> [!question]- ¿Qué hace Claude en plan mode y qué NO hace?
> Lee el codebase, analiza dependencias y propone un enfoque; no modifica ningún archivo hasta que apruebas el plan.

> [!question]- ¿Por qué se dice que plan mode permite "safe exploration"?
> Porque explorar y diseñar no tiene efectos secundarios: nada se escribe a disco durante la planeación.

## Cómo se activa plan mode

> [!question]- ¿Cuáles son las tres formas de entrar en plan mode?
> `claude --permission-mode plan` al iniciar, `Shift+Tab` durante la sesión, o prefijar un prompt con `/plan`.

> [!question]- ¿Qué pasa si escribes "in plan mode" dentro del cuerpo del prompt?
> Nada: no activa plan mode. Solo `/plan` al inicio del prompt (o las otras dos rutas) lo activa.

> [!question]- ¿Cuándo usarías `/plan` en vez de `--permission-mode plan`?
> Cuando solo un prompt concreto necesita planeación dentro de una sesión que por lo demás es de direct execution.

## Direct execution: cuándo usarlo

> [!question]- ¿Qué tres condiciones hacen apropiado direct execution?
> Cambio well-scoped, enfoque correcto ya conocido y scope limitado (una función, un archivo, una modificación).

> [!question]- ¿Por qué no usar plan mode siempre "por seguridad"?
> Porque en tareas simples y bien definidas planear no agrega nada; solo es overhead.

> [!question]- Da tres ejemplos típicos de direct execution.
> Single-file bug fix con stack trace claro, agregar un date validation conditional, actualizar un valor de configuración.

## Ambigüedad, no dificultad

> [!question]- Un bug difícil, pero con stack trace claro, una función y causa conocida: ¿qué modo?
> Direct execution: es difícil pero no ambiguo.

> [!question]- Un feature que suena simple pero admite tres implementaciones y toca varios módulos: ¿qué modo?
> Plan mode: es ambiguo aunque parezca fácil.

> [!question]- ¿Cuál es la diferencia entre "difícil" y "ambiguo" en este contexto?
> Difícil = requiere esfuerzo o conocimiento; ambiguo = no está decidido qué/dónde/cómo cambiar. Solo la ambigüedad empuja a plan mode.

## El Explore subagent

> [!question]- ¿Qué problema resuelve el Explore subagent?
> Evita que el output verboso del discovery (file listings, dependency graphs, code excerpts) llene el context window principal.

> [!question]- ¿Qué pasaría si todo el discovery fluye a la conversación principal?
> Se llena el context window y se degrada la calidad de las respuestas posteriores, justo en la fase de implementación.

> [!question]- ¿Qué devuelve el Explore subagent a la conversación principal?
> Solo summaries de sus hallazgos, no el output completo.

> [!question]- ¿En qué tipo de tareas conviene usar el Explore subagent?
> En tareas multi-phase donde el discovery es verboso pero la implementación necesita contexto enfocado.

> [!question]- ¿Cómo se relaciona el Explore subagent con plan mode?
> Lo complementa: plan mode decide el diseño sin tocar archivos, y el Explore subagent aísla la exploración verbosa que ese diseño necesita.

## El patrón híbrido: plan THEN execute

> [!question]- ¿En qué consiste el patrón híbrido?
> Plan mode para investigar y diseñar la estrategia; luego direct execution para implementarla archivo por archivo.

> [!question]- ¿Por qué "plan THEN direct" y no "plan OR direct"?
> Porque no son excluyentes: una vez diseñada la estrategia ya no hay ambigüedad, y la implementación pasa a ser un cambio bien definido.

> [!question]- En una library migration, ¿qué se hace en la fase de plan?
> Identificar los archivos que usan la library vieja, mapear diferencias de API, diseñar el migration pattern y revisar edge cases.

> [!question]- En esa misma migración, ¿qué se hace en la fase de execute?
> Aplicar el migration pattern ya decidido a cada archivo.

## Decision framework

> [!question]- ¿Qué modo corresponde a "codebase exploration necesaria"?
> Plan mode, con el Explore subagent.

> [!question]- ¿Qué modo corresponde a "library migration de muchos archivos"?
> Plan mode, luego direct execution.

> [!question]- ¿Qué modo corresponde a "fix conocido, ubicación conocida, enfoque conocido"?
> Direct execution.

## Reconocer la complejidad desde el inicio

> [!question]- ¿Qué tiene de malo "empiezo en direct execution y si se complica, cambio a plan mode"?
> Si la complejidad ya está en los requisitos, es conocida y no especulativa; esperar "sorpresas" lleva a rework.

> [!question]- Si el requisito dice "restructure the monolith into microservices", ¿cuándo entras en plan mode?
> Desde el inicio (upfront).

> [!question]- ¿Por qué dar "instrucciones detalladas upfront" en direct execution no sustituye a plan mode?
> Porque asume que ya conoces la estructura correcta sin haber explorado el codebase.

## Trampas de examen

> [!question]- ¿Cuál es el riesgo de usar direct execution por default en cambios arquitectónicos multi-archivo?
> Descubrir dependencias tarde y tener costly rework, además de cambios inconsistentes.

> [!question]- ¿Cuál es el error inverso más común?
> Usar plan mode para un single-file bug fix con stack trace claro, agregando overhead innecesario.

> [!question]- ¿Qué trampa corresponde a la opción D de la Sample Question 5 oficial?
> Empezar en direct execution y cambiar a plan mode solo cuando aparezca la complejidad, cuando esta ya estaba declarada en los requisitos.

---

> [!tip] Vuelve a [[1 resumen]], aplícalo en [[2 example]] y evalúate en [[4 test]]
