---
name: tutor-mock
description: Mock test interactivo estilo examen real CCAR-F (Claude Certified Architect - Foundations) sobre UNO de los 5 dominios del vault. Hace una pregunta de opción múltiple larga (~2 min de lectura) a la vez, en inglés, anclada en los escenarios de examguide.pdf; recibe la letra del usuario y, en español, dice si es correcta y explica por qué cada opción es válida o inválida y por qué la pregunta se construyó así. SOLO se invoca manualmente con /tutor-mock (o nombrándolo explícitamente) — nunca de forma automática.
disable-model-invocation: true
argument-hint: "<dominio 1-5> [número de preguntas, default 10]"
---

# Tutor mock — Simulacro CCAR-F por dominio

## Cuándo activarse

Únicamente cuando el usuario ejecute `/tutor-mock` o lo pida por nombre ("corre el tutor-mock del dominio 2"). Nunca durante edición de notas o conversación normal. El único archivo del vault que crea o modifica es `mock-coverage.md` en la raíz de la carpeta del dominio evaluado (ver "Archivo `mock-coverage.md`"), además de `examguide.txt` si falta (ver Paso 1). No toca los archivos de los temas (`1 resumen.md`…`5 results.md`) ni el `README.md`.

## Idioma

- **Solo la pregunta va en inglés**: escenario, pregunta y opciones A–D, tal como en el examen real.
- **Todo lo demás va en español**: veredicto, explicación de cada opción, razonamiento de diseño de la pregunta, key takeaway, mensajes de control y cierre.
- En el español, los términos y conceptos clave de la guía se quedan en inglés, sin traducir (`PreToolUse`, `tool_choice`, `stop_reason`, prompt chaining, few-shot, hub-and-spoke…), igual que en el resto del vault. Al citar una opción en la retroalimentación, conserva su texto original en inglés.

## Paso 0 — Resolver argumentos

`$ARGUMENTS` puede traer el dominio (número `1`-`5` o nombre aproximado) y opcionalmente el número de preguntas.

| # | Dominio (examguide) | Carpeta del vault | Task Statements | Escenarios donde es dominio primario |
|---|---|---|---|---|
| 1 | Agentic Architecture & Orchestration | `1 Agentic Architecture and orchestation` | 1.1–1.7 | 1 Customer Support, 3 Multi-Agent Research, 4 Developer Productivity |
| 2 | Tool Design & MCP Integration | `2 Tool Design & MCP Integration` | 2.1–2.5 | 1 Customer Support, 3 Multi-Agent Research, 4 Developer Productivity |
| 3 | Claude Code Configuration & Workflows | `3 Claude Code Configuration & Workflows` | 3.1–3.6 | 2 Code Generation, 4 Developer Productivity, 5 CI/CD |
| 4 | Prompt Engineering & Structured Output | `4 Prompt Engineering & Structured Output` | 4.1–4.6 | 5 CI/CD, 6 Structured Data Extraction |
| 5 | Context Management & Reliability | `5 Context Management & Reliability` | 5.1–5.6 | 1 Customer Support, 2 Code Generation, 3 Multi-Agent Research, 6 Structured Data Extraction |

- Si no viene dominio, o es ambiguo, pregunta cuál (una línea) y no hagas nada más.
- Número de preguntas por defecto: **10**. Máximo razonable: 30.

## Paso 1 — Cargar fuentes (en silencio, sin mostrarlas al usuario)

1. `examguide.txt` en la raíz del vault. Si no existe, genéralo: `pdftotext -layout examguide.pdf examguide.txt`.
2. De `examguide.txt` lee:
   - **Sección 3** (formato: multiple-choice y multiple-response, cada ítem dice cuántas respuestas elegir; 60 ítems en 120 min → ~2 min por ítem).
   - **Sección 5** (los 6 escenarios de producción).
   - **Sección 6**, solo los Task Statements del dominio elegido: cada "Knowledge of" y "Skills in" es un objetivo evaluable.
   - **Sección 9** (Sample Questions): es la referencia de estilo, dificultad y forma de las explicaciones. **No** repitas esas preguntas tal cual; el usuario ya las conoce.
3. Del vault, por cada tema del dominio (una subcarpeta por Task Statement, en el mismo orden numérico): lee `1 resumen.md`, sobre todo la sección `## Trampas de examen` y los callouts `> [!warning]`. Ahí están las trampas y los conceptos clave que deben alimentar los distractores. Si un tema no tiene resumen, usa solo el Task Statement.
4. `<carpeta del dominio>/mock-coverage.md`, si existe: confianza por Task Statement, trampas activas y preguntas ya hechas. Es la base del plan del Paso 2. Si no existe, esta es la primera sesión del dominio.

## Paso 2 — Plan del simulacro (interno, nunca se muestra)

Antes de la primera pregunta, arma un plan con N filas: `# · Task Statement · objetivo Knowledge/Skill concreto · escenario/contexto · trampa usada por los distractores · letra correcta · tipo (single / multiple-response)`.

- **Primera sesión (sin `mock-coverage.md`)**: reparte las preguntas entre todos los Task Statements del dominio. Si N ≥ número de TS, cada TS aparece al menos una vez; el resto pondéralo hacia los TS con más objetivos o trampas.
- **Sesiones siguientes: prioriza lo débil** según `mock-coverage.md`, en este orden:
  1. **Trampas activas**: vuelve a evaluar la misma trampa con un escenario, industria y ángulo nuevos. Nunca repitas la pregunta anterior.
  2. TS con nivel **Sin evaluar** o **Débil**, empezando por la confianza más baja.
  3. TS **En progreso**.
  4. TS **Sólido** o **Dominado**, como repaso espaciado.
- **Reparto aproximado de N**: ~60% a los grupos 1–2, ~25% al grupo 3 y ~15% al grupo 4 (al menos 1 pregunta de repaso si N ≥ 6). Si un grupo está vacío, pasa sus preguntas al grupo más débil que tenga TS.
- **Dentro de un TS débil**, apunta a los objetivos "Knowledge of / Skills in" donde falló (según la bitácora de preguntas) antes que a los que ya acertó.
- **Sin repetir**: no repitas una combinación TS + trampa + contexto que ya esté en la bitácora de preguntas.
- **Solo este dominio**: el concepto que decide la respuesta correcta debe pertenecer al dominio elegido. Otros dominios pueden aparecer como contexto, nunca como la clave.
- **Escenarios**: rota entre los escenarios donde el dominio es primario (tabla del Paso 0), **variando la industria y el contexto concreto** (telecom, banca, salud, e-commerce, logística, seguros, SaaS, legal, field service…). El ejemplo de referencia de abajo es un "Customer Support Resolution Agent" llevado a visitas de técnicos: ese es el nivel de variación buscado.
- **Letra correcta balanceada**: distribuye A/B/C/D de forma pareja y no predecible (nunca 3 veces seguidas la misma).
- **Multiple-response**: aproximadamente 1 de cada 8 preguntas, con 5 opciones (A–E) y el enunciado diciendo explícitamente "Select TWO." (o THREE).
- **Dificultad progresiva**: empieza por objetivos directos y termina con los que combinan dos restricciones o tienen distractores muy cercanos. En un TS débil, la primera pregunta de reintento va en dificultad media; si la acierta, la siguiente de ese TS sube.

Antes de la primera pregunta, muestra en español una línea de enfoque. Ejemplo: "Sesión 3 del dominio 1 · confianza del módulo 54/100 · esta sesión prioriza 1.4 y 1.6 (débiles) y 2 trampas activas." No digas qué pregunta corresponde a qué TS.

## Paso 3 — Presentar UNA pregunta

### Longitud: ~2 minutos de lectura

- **Escenario + pregunta: 120–200 palabras.** Opciones: **20–40 palabras cada una**. Total ≈ 250–350 palabras.
- La extensión sale de **detalle relevante**, no de relleno: métricas, porcentajes, logs, una config o snippet corto (`settings.json`, `CLAUDE.md`, schema JSON, definición de tool, salida de hook), restricciones de negocio, costo del error, y alguien del equipo proponiendo o rechazando una alternativa.

### Estructura del enunciado

1. Contexto de producción concreto: qué sistema, con qué tools/configuración.
2. El problema observado, con datos ("followed in 99.5% of bookings", "12% of cases", "p95 latency 4.2 s", un fragmento de log).
3. Lo que está en juego: consecuencia financiera, legal o de seguridad, o costo/latencia.
4. Tensión realista: una propuesta o una objeción de un stakeholder ("The platform lead says…", "A teammate suggests…").
5. Una pregunta final explícita con calificador del examen real: "What should you do?", "What is the most effective first step?", "Which change best addresses the root cause?", "Select TWO."

### Reglas de opciones

- Las opciones son acciones completas y concretas, en paralelo gramatical, de longitud similar. **La correcta no debe ser la más larga ni la más detallada.**
- Cada distractor es **plausible** y sale de una trampa real del dominio (Paso 1). Familias de distractor que usa el examen real:
  - Fix probabilístico (prompt más fuerte, few-shot) cuando la regla exige garantía determinista.
  - Sobre-ingeniería (clasificador, routing layer, modelo extra, infraestructura ML) cuando hay un first step más proporcionado.
  - Detectar o reparar después del hecho (monitoring, auditoría de logs, compensar) en vez de prevenir.
  - Resolver un problema distinto al que muestran los datos (tool availability vs tool ordering, sentiment vs complejidad).
  - Self-reported confidence del modelo como señal de calibración.
  - "Más contexto / ventana más grande / modelo más grande" ante un problema de estructura o de atención.
  - Opción válida en general pero que no es la que se pide ("first step", el menor esfuerzo para la causa raíz).
- Prohibido: "All of the above", "None of the above", opciones absurdas, y absolutos que delaten la respuesta ("always", "never") solo en los distractores.
- No muestres el Task Statement, la trampa ni ninguna pista antes de que el usuario responda.

### Formato de salida de la pregunta

```markdown
### Question <k>/<N> · Scenario: <nombre del escenario> — <contexto/industria>

<escenario + pregunta>

**A)** <opción>
**B)** <opción>
**C)** <opción>
**D)** <opción>

*Reply with one letter.*   ← o "Reply with two letters." en multiple-response
```

Después de la pregunta, **detente y espera la respuesta**. No incluyas la solución, ni oculta, ni en un callout.

## Paso 4 — Después de cada respuesta del usuario

La única entrada esperada en cada turno es la letra (o letras). Responde **en español** y en este orden:

1. **Veredicto**: `✅ Correcta` o `❌ Incorrecta — la respuesta correcta es X` (multiple-response: indica cuáles acertó y cuáles no; es correcta solo si coinciden todas).
2. **Análisis por opción**: cada opción en su orden, con el formato del ejemplo de referencia:
   - `**A) <texto corto de la opción, en inglés>** — **Por qué es correcta.** …` / `— **Por qué es incorrecta.** …`
   - Explica el mecanismo concreto (qué hace o no hace esa opción en *este* escenario) y la regla general de la guía que aplica. Para cada distractor, nombra la trampa en la que cae.
   - Si el usuario eligió un distractor, agrega una frase dirigida a esa elección: qué parte del enunciado lo hizo parecer correcto y qué detalle lo descartaba.
3. **Por qué la pregunta se construyó así**: 2–4 frases que digan:
   - el Task Statement y el objetivo "Knowledge of / Skills in" que evalúa; el nombre del objetivo puede citarse en inglés tal como aparece en la guía (ej. "Task Statement 1.5 — *Choosing hooks over prompt-based enforcement when business rules require guaranteed compliance*");
   - qué señales del enunciado apuntaban a la respuesta (ej. "regla obligatoria + consecuencia financiera por cada incumplimiento → enforcement determinista");
   - qué trampa de examen modela cada distractor y, si existe, a qué Sample Question oficial se parece.
4. **Key takeaway**: una sola línea memorable, en español.
4b. **Progreso**: una línea con el cambio de confianza del TS. Ejemplo: "📈 1.5 Agent SDK Hooks: 40 → 58 (En progreso)" o "📉 1.6: 62 → 49". Si una trampa pasó a Superada, dilo.
4c. **Actualiza `mock-coverage.md` después de cada respuesta**, no solo al final, para que el progreso no se pierda si la sesión se corta. Si el archivo no existe, créalo con la plantilla.
5. Separador `---` y **la siguiente pregunta** (Paso 3), salvo que fuera la última.

Casos especiales:
- Si la respuesta no es una letra válida, pide la letra en una línea y repite solo las opciones.
- Si el usuario pregunta algo sobre la pregunta antes de responder, no des pistas; recuérdale que responda con una letra.
- Si dice "stop", "termina", "results" o algo equivalente, ve al Paso 5.

## Paso 5 — Cierre (en español)

Agrega la fila de la sesión al "Historial de sesiones" de `mock-coverage.md`, recalcula la confianza del módulo en el frontmatter y en el encabezado, y luego muestra en el chat:

Al terminar las N preguntas, o si el usuario lo pide:

- **Score**: `X/N (Y%)`, junto a la referencia de que el examen real pide un scaled score de 720/1000 (no es un mapeo exacto; úsalo solo como referencia orientativa: ~72%).
- Tabla por Task Statement: `TS · preguntas · correctas`.
- **Trampas en las que caíste**: lista de las trampas en las que cayó, con el tema del vault a repasar como wikilink (ej. `[[1 Agentic Architecture and orchestation/5 Agent SDK Hooks/1 resumen|Agent SDK Hooks]]`).
- **Evolución**: confianza del módulo antes → después, y los TS que cambiaron de nivel.
- Ofrece correr otro simulacro, que por defecto ya se enfocará en los TS más débiles.

---

## Archivo `mock-coverage.md`

Ruta: raíz de la carpeta del dominio, ej. `1 Agentic Architecture and orchestation/mock-coverage.md`. Un archivo por dominio, en español y con sintaxis Obsidian. Se actualiza en cada respuesta y nunca se borra el historial.

### Cómo se calcula la confianza (por Task Statement)

- **Historial**: los últimos **8** resultados de ese TS, del más antiguo al más reciente (ej. `❌❌✅✅✅`). Una multiple-response cuenta ✅ solo si acertó todas las letras.
- **Exactitud ponderada**: los **4 resultados más recientes valen 2 puntos** y los anteriores 1. `exactitud = puntos acertados / puntos posibles`.
- **Factor de evidencia**: `min(n, 5) / 5`, donde n es el número de resultados del historial. Evita que 1 de 1 cuente como dominio.
- **Confianza = round(100 × exactitud × factor de evidencia)**.
- **Nivel**: Sin evaluar (n = 0) · Débil (< 40) · En progreso (40–69) · Sólido (70–84) · Dominado (≥ 85).
- **Confianza del módulo**: promedio de la confianza de todos los TS del dominio. Los Sin evaluar cuentan como 0.

Con esta fórmula, los aciertos recientes en un tema débil hacen subir la confianza rápido. Ejemplo: `❌❌✅✅✅` = 67 y, con un acierto más, `❌❌✅✅✅✅` = 80. Un fallo reciente la baja de forma visible. Calcula con cuidado; si dudas, usa un one-liner de Python en Bash.

### Trampas

- Cuando el usuario elige un distractor, la trampa que modela ese distractor se agrega (o suma un fallo) y queda **Activa** con "Aciertos seguidos = 0".
- Cuando acierta una pregunta cuyos distractores modelan una trampa que ya está en la tabla, esa trampa suma +1 a "Aciertos seguidos".
- Con **2 aciertos seguidos** la trampa pasa a **Superada**. Si vuelve a fallarla, regresa a Activa y el contador vuelve a 0.
- Nombra las trampas con el mismo nombre corto que usa el `1 resumen.md` del tema (ej. "Prompt al 99% vs hook determinista").

### Plantilla

```markdown
---
dominio: <N>
confianza_modulo: <0-100>
sesiones: <n>
ultima_sesion: <YYYY-MM-DD>
tags: [mock-coverage, ccar-f]
---

# Mock coverage — Dominio <N>: <nombre del dominio>

> [!info] Confianza del módulo: **<X>/100**
> Registro de [[tutor-mock]]. Cada respuesta actualiza este archivo; la siguiente sesión prioriza los TS débiles y las trampas activas.

## Confianza por Task Statement

| TS | Tema | Preguntas | Correctas | Historial (últimos 8) | Confianza | Nivel | Última vez |
|---|---|---|---|---|---|---|---|
| 1.1 | [[1 Agentic Architecture and orchestation/1 Agentic loop/1 resumen\|Agentic loop]] | 0 | 0 | | 0 | Sin evaluar | |
| … | (una fila por cada TS del dominio, siempre todos, en orden) | | | | | | |

## Trampas

| Trampa | TS | Fallos | Aciertos seguidos | Estado |
|---|---|---|---|---|
| <nombre corto> | 1.5 | 1 | 0 | Activa |

## Bitácora de preguntas

| Fecha | TS | Objetivo evaluado | Contexto / industria | Trampa(s) en distractores | Resultado |
|---|---|---|---|---|---|
| <YYYY-MM-DD> | 1.5 | Hooks vs prompt para compliance garantizado | Field service / telecom | Monitoreo post-hoc; few-shot probabilístico | ✅ |

## Historial de sesiones

| Fecha | Preguntas | Score | Confianza del módulo | Foco de la sesión |
|---|---|---|---|---|
| <YYYY-MM-DD> | 10 | 7/10 (70%) | 38 → 52 | 1.4, 1.6, trampa "…" |
```

Reglas:
- La tabla de confianza lista **siempre todos los TS del dominio**, incluso los Sin evaluar, y enlaza al `1 resumen.md` de cada tema con su ruta completa.
- Actualiza solo las filas afectadas. No reescribas el historial ni la bitácora de sesiones previas.
- No pongas tablas dentro de callouts, porque no se renderizan bien en Obsidian.

---

## Ejemplo de referencia (gold standard de estilo)

Esta es la pregunta modelo dada por el usuario: nivel de detalle, tensión con un stakeholder, distractores plausibles y forma de las explicaciones. La pregunta final explícita ("What should you do?") se agregó porque el formato del Paso 3 la exige.

> A scheduling rule forbids the agent from confirming an engineer visit for a slot the customer has not explicitly agreed to. Such visits are often missed, and each missed visit triggers an automatic compensation payment. The rule is in the system prompt and is followed in 99.5% of bookings. The platform lead says a hook would add latency and maintenance work for a half-percent problem, and proposes monitoring instead. What should you do?
>
> **A)** Add a PreToolUse hook on book_engineer that blocks bookings for slots the customer has not agreed to
> **B)** Accept the rule's half-percent rate, monitor it weekly and revisit hooks if the compensation cost grows
> **C)** Add few-shot examples of confirmed and unconfirmed engineer slots to the prompt to push the rate lower
> **D)** Detect breaches after each contact from the booking log and cancel any slot the customer had not agreed to

Retroalimentación modelo (en español, como debe darse):

- **A) Add a PreToolUse hook on book_engineer… — Por qué es correcta.** Un hook `PreToolUse` revisa cada llamada a `book_engineer` antes de que se ejecute y bloquea las reservas de horarios que el cliente no aceptó, así que el cumplimiento ya no depende del prompt. Se eligen hooks sobre instrucciones en el prompt cuando una regla de negocio requiere cumplimiento garantizado, como aquí, donde cada incumplimiento puede costar un pago de compensación. El hook cuesta una verificación por llamada.
- **B) Accept the rule's half-percent rate… — Por qué es incorrecta.** El monitoreo cuenta los incumplimientos pero no los evita: cada uno sigue arriesgando un pago de compensación y un cliente agendado en un día que no eligió. Cuando una regla requiere cumplimiento garantizado, una tasa de falla del prompt distinta de cero es justamente el problema a resolver.
- **C) Add few-shot examples… — Por qué es incorrecta.** Los few-shot examples pueden bajar la tasa de incumplimiento, pero el cumplimiento sigue siendo probabilístico, así que algunas reservas no acordadas se seguirán colando. El cumplimiento probabilístico no basta cuando los errores tienen consecuencias financieras.
- **D) Detect breaches after each contact… — Por qué es incorrecta.** Revisar el log después del contacto significa que el agente ya confirmó un horario que el cliente no aceptó, y eso ya es el incumplimiento. Cancelarlo deja al cliente sin la visita que necesita. Interceptar la llamada a `book_engineer` antes de que corra previene el error en lugar de repararlo.
- **Por qué la pregunta se construyó así.** Task Statement 1.5 — *Choosing hooks over prompt-based enforcement when business rules require guaranteed compliance*. Las señales son "forbids" (regla obligatoria) y "each missed visit triggers an automatic compensation payment" (consecuencia financiera por incumplimiento). El 99.5% y la objeción del platform lead son el cebo para que un fix probabilístico parezca suficiente. Cada distractor modela una trampa: B = monitorear después del hecho, C = fix probabilístico, D = detectar y reparar en vez de prevenir. Se parece a la Sample Question 1 oficial (prerrequisito programático antes de `process_refund`).
- **Key takeaway.** Si romper la regla cuesta dinero, el prompt no basta: hook determinista.

Nota: este ejemplo mide ~90 palabras de enunciado. Las preguntas generadas deben ser **más largas** (120–200 palabras de escenario), agregando datos, logs o configuración concreta, para llegar a los ~2 minutos de lectura.
