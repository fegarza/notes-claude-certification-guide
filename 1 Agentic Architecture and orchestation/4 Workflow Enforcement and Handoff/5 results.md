# Resultados — Tutor socrático: Workflow Enforcement and Handoff

> [!info] Sesión evaluada el 2026-09-18
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 77/100**

Buen manejo de la regla de decisión central del tema (financiero/seguridad/cumplimiento → enforcement programático; bajo riesgo → prompt aceptable) y de los mecanismos concretos (bloqueo de una prerequisite gate, investigación paralela en multi-concern, restricción de handoff sin acceso a la transcripción). Las fallas se concentraron en explicar el *porqué* detrás de la etiqueta correcta — respuestas que acertaban la conclusión pero con razonamiento impreciso o genérico — y en un detalle puntual (mecanismo exacto para bloquear el spawn de un subagente).

## Conceptos con buen dominio

- Regla de decisión del examen (financiero/seguridad/cumplimiento vs. bajo riesgo) — identificó correctamente por qué un 3% de fallo sigue siendo inaceptable en escenarios de alto riesgo.
- Comportamiento de una prerequisite gate — explicó con precisión que bloquea la llamada y fuerza el cumplimiento del prerrequisito.
- Solicitudes multi-concern — entendió por qué investigar en paralelo evita latencia innecesaria sin perder la ventaja del contexto compartido.
- Restricción de handoff (el humano no ve la transcripción) — respuesta directa y correcta.
- Trampa de few-shot como distractor — articuló bien que mejora precisión sin eliminar la tasa de fallo.
- Conversión de hooks `Stop` de subagente a `SubagentStop` — respuesta correcta.

## Áreas que necesitan revisión

### El mecanismo detrás de "probabilístico" vs. "determinista"

- **Qué pasó:** Ante la pregunta de por qué un prompt nunca garantiza 100%, la respuesta repitió la etiqueta ("es probabilístico") sin explicar el mecanismo de interpretación de lenguaje natural que lo causa.
- **Contenido del resumen:**
  > **Guía basada en prompt**: instrucciones en el system prompt (ej. "siempre verifica la identidad del cliente antes de procesar un reembolso"). Funciona la mayoría de las veces — quizás 90-95% de los casos — porque el modelo es **probabilístico**: puede saltarse pasos, reordenarlos, o interpretar la instrucción de forma laxa.
- **Sección:** [[1 resumen]] → "1. El espectro de enforcement: prompt-based vs. programático" (solo como enlace de navegación).
- **Analogía para reforzarlo:** Es como pedirle a alguien que traduzca una instrucción oral cada vez que la recibe — a veces la entiende distinto, se le olvida un detalle, o decide que "más o menos" es suficiente. El código, en cambio, no traduce nada: ejecuta la misma comparación exacta cada vez.

### Por qué un clasificador de ruteo no resuelve la falla de cumplimiento (Trampa 3)

- **Qué pasó:** Identificó correctamente que el router no es la solución, pero justificó con "más esfuerzo" en vez de la distinción real: el ruteo opera en una capa distinta (qué agente atiende) de donde ocurre la falla (dentro de la secuencia de ejecución de un agente ya seleccionado).
- **Contenido del resumen:**
  > **Trampa 3 — Un clasificador de ruteo para arreglar fallas de cumplimiento por agente**
  > Un clasificador de ruteo decide **qué agente** atiende una solicitud. La falla de cumplimiento ocurre **dentro** de la secuencia de ejecución de un agente, no en el nivel de ruteo. Los clasificadores resuelven ruteo, no enforcement de flujo de trabajo dentro de un agente.
- **Sección:** [[1 resumen]] → "Trampas de examen" (Trampa 3) (solo como enlace de navegación).
- **Analogía para reforzarlo:** El router es como el recepcionista que decide a qué doctor te manda; no puede evitar que ese doctor, ya en el consultorio, se salte un paso del protocolo. Son dos trabajos distintos.

### Por qué el monto del reembolso debe ser una cifra específica en el handoff

- **Qué pasó:** No supo responder por qué una referencia vaga ("el monto que pidió el cliente") es insuficiente en el resumen de handoff.
- **Contenido del resumen:**
  > Un resumen de handoff correcto debe ser **autocontenido** e incluir: [...] **Monto del reembolso** (si aplica) — la cifra financiera específica, no una referencia vaga.
  >
  > Este resumen es la **única** información que recibe el humano. Si está incompleto, el humano tiene que pedirle al cliente que repita todo — una mala experiencia que el examen penaliza como falla de diseño.
- **Sección:** [[1 resumen]] → "5. Protocolos de handoff estructurado" (solo como enlace de navegación).
- **Analogía para reforzarlo:** Es como dejarle una nota a un compañero de turno que dice "cóbrale lo que el cliente mencionó" — si el compañero no estuvo en la llamada, esa nota es inútil; necesita el número exacto para actuar sin tener que llamar al cliente de nuevo.

### Mecanismo exacto para bloquear el spawn de un subagente

- **Qué pasó:** Reconoció que `SubagentStart` no puede bloquear, pero propuso "prerequisite gate" como respuesta genérica en vez de nombrar el mecanismo específico que da el resumen: un hook `PreToolUse` sobre la herramienta `Agent`.
- **Contenido del resumen:**
  > Para bloquear el *spawn* en sí (rate limits, exigir que el coordinador pase cierto contexto), no sirve `SubagentStart` — se necesita un `PreToolUse` sobre la herramienta `Agent`, que sí puede denegar o reescribir la invocación antes de que el subagente arranque.
  >
  > `SubagentStart`: observa el spawn, agrega contexto, no bloquea. `SubagentStop`: gatea la finalización con exit code 2, no agrega contexto, no reescribe el resultado. Para bloquear el spawn mismo o reescribir un resultado, se usa `PreToolUse`/`PostToolUse` sobre la herramienta `Agent`, no los hooks de ciclo de vida del subagente.
- **Sección:** [[1 resumen]] → "Configuración de ejemplo (hooks de ciclo de vida de subagentes)" (solo como enlace de navegación).
- **Analogía para reforzarlo:** "Prerequisite gate" es el concepto general (un torniquete); `PreToolUse` sobre `Agent` es el torniquete específico instalado justo antes de la puerta de spawn — nombrar el concepto general está bien, pero el examen premia saber cuál torniquete exacto va en cuál puerta.

## Siguiente paso recomendado

El checklist de este tema quedó cubierto por completo. Antes de intentar [[4 test]], repasa brevemente las cuatro áreas señaladas arriba — especialmente la distinción de capas (ruteo vs. ejecución) y el hook `PreToolUse` sobre `Agent` — y luego procede al examen de práctica.
