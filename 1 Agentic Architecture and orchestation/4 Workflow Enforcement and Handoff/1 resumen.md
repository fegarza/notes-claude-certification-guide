> [!note] En una frase
> Si un solo fallo puede costar dinero, romper seguridad o violar cumplimiento, no basta con pedírselo bien al modelo en el prompt — hay que bloquear la herramienta en código hasta que se cumpla el requisito previo, porque solo el código garantiza el 100% de las veces.

## Explícamelo como si tuviera 5 años

Imagina un guardia de seguridad en la puerta de una bóveda. Hay dos formas de decirle "no dejes pasar a nadie sin credencial": la primera es un cartel enorme que dice "¡POR FAVOR, PIDE LA CREDENCIAL!" — funciona casi siempre, pero el guardia es humano, se distrae, y de vez en cuando deja pasar a alguien sin revisar nada. La segunda es un torniquete que **físicamente no gira** hasta que se escanea una credencial válida — no importa si el guardia se distrae, se cansa o decide "seguro está bien, lo dejo pasar": el torniquete no se mueve.

El cartel es el prompt del sistema ("siempre verifica la identidad antes de procesar un reembolso"). El torniquete es la compuerta programática (prerequisite gate) que bloquea la herramienta en código. Para una bóveda de banco, nadie instalaría solo el cartel.

## Argumento central

> El examen traza una línea dura entre dos formas de controlar el orden de ejecución de un agente: **guía basada en prompt** (probabilística, falla algún porcentaje de las veces) y **enforcement programático** (determinista, funciona siempre). La regla de decisión del examen es mecánica: si la operación es financiera, de seguridad o de cumplimiento, la respuesta correcta es *siempre* enforcement programático — nunca un prompt más fuerte, por muy bien escrito que esté.

## Ideas clave (Conclusiones)

### 1. El espectro de enforcement: prompt-based vs. programático

- **Guía basada en prompt**: instrucciones en el system prompt (ej. "siempre verifica la identidad del cliente antes de procesar un reembolso"). Funciona la mayoría de las veces — quizás 90-95% de los casos — porque el modelo es **probabilístico**: puede saltarse pasos, reordenarlos, o interpretar la instrucción de forma laxa.
- **Enforcement programático**: hooks, compuertas de prerrequisito (prerequisite gates), o checks a nivel de código que **bloquean físicamente** una herramienta downstream hasta que se cumple un prerrequisito. Ejemplo: `process_refund` no puede ejecutarse hasta que `get_customer` haya devuelto un ID de cliente verificado. Esto funciona **todas las veces** — es determinista, no probabilístico.

> [!note] Analogía clave
> Prompt-based es el cartel; programático es el torniquete. Uno pide, el otro impide.

### 2. La regla de decisión del examen

El examen aplica el mismo criterio en escenarios distintos:

- **Operaciones financieras** (reembolsos, transferencias, pagos) → enforcement programático. Un solo reembolso mal verificado ya es una pérdida financiera real.
- **Operaciones de seguridad** (verificación de identidad, control de acceso) → enforcement programático. Un solo bypass de verificación de identidad ya es una brecha de seguridad.
- **Operaciones de cumplimiento** (AML, requisitos regulatorios) → enforcement programático. Un solo check de cumplimiento omitido puede acarrear sanciones legales.
- **Operaciones de bajo riesgo** (preferencias de formato, estilo de salida) → la guía basada en prompt es aceptable. Una inconsistencia de formato no es un riesgo de negocio.

> [!warning] Los distractores más comunes en este tema
> El examen presenta como opciones "mejorar el prompt del sistema", "agregar ejemplos few-shot" o "usar un clasificador de ruteo" para escenarios de alto riesgo. Todas mejoran la precisión probabilística, pero **ninguna** elimina la tasa de fallo. Ante dinero, seguridad o cumplimiento, la respuesta correcta es siempre enforcement programático.

### 3. Prerequisite gates en la práctica

Una **prerequisite gate** es un check programático que bloquea una herramienta hasta que se cumple una condición previa. En un agente de soporte al cliente con las herramientas `get_customer`, `lookup_order` y `process_refund`:

1. La gate verifica: ¿`get_customer` ya devolvió un ID de cliente verificado para esta sesión?
2. Si sí, `process_refund` se ejecuta normalmente.
3. Si no, `process_refund` devuelve un error: *"Cannot process refund — customer identity not verified. Please call get_customer first."*

- La gate es **código**, no una instrucción de prompt — el modelo no puede saltársela decidiendo omitir la verificación.
- Incluso si el modelo intenta llamar a `process_refund` directamente, la gate bloquea la llamada y fuerza la verificación previa.

### 4. Solicitudes con múltiples asuntos (multi-concern)

Cuando un cliente pide varias cosas a la vez ("quiero devolver mi pedido, actualizar mi dirección, y preguntar por mis puntos de lealtad"), el enfoque correcto tiene tres pasos:

1. **Descomponer** la solicitud en asuntos distintos (devolución, actualización de dirección, consulta de puntos).
2. **Investigar cada uno en paralelo**, usando contexto compartido (la información de cuenta del cliente es relevante para los tres).
3. **Sintetizar una resolución unificada** que atienda todos los asuntos en una sola respuesta.

> [!warning] El enfoque incorrecto
> Manejar cada asunto en conversaciones separadas y secuenciales, o atender solo el primer asunto y olvidar el resto. El examen espera descomposición + investigación paralela + síntesis unificada, no atención parcial ni secuencial.

### 5. Protocolos de handoff estructurado

Cuando un agente no puede resolver un caso y debe escalar a un agente humano, la restricción crítica es: **el agente humano NO tiene acceso a la transcripción de la conversación**. No puede desplazarse por el historial de chat para entender el problema.

Un resumen de handoff correcto debe ser **autocontenido** e incluir:

- **Customer ID** — para que el humano pueda buscar la cuenta.
- **Resumen de la conversación** — qué pidió el cliente y qué se intentó.
- **Análisis de causa raíz** — la evaluación del agente sobre el problema subyacente.
- **Monto del reembolso** (si aplica) — la cifra financiera específica, no una referencia vaga.
- **Acción recomendada** — qué cree el agente que debería hacer el humano.

> [!note] Por qué esto pesa tanto en el examen
> Este resumen es la **única** información que recibe el humano. Si está incompleto, el humano tiene que pedirle al cliente que repita todo — una mala experiencia que el examen penaliza como falla de diseño.

## Evidencia — El caso del 8% de fallo

Datos de producción muestran que un agente de soporte procesa reembolsos sin verificar la titularidad de la cuenta en el **8% de los casos**, a pesar de que el system prompt dice explícitamente "siempre verifica la identidad del cliente antes de procesar cualquier reembolso". El prompt funciona el 92% de las veces, pero ese 8% ya generó reembolsos en cuentas equivocadas — una consecuencia monetaria real.

La solución no es reescribir el prompt: es una **prerequisite gate programática** que verifica, antes de ejecutar `process_refund`, que `get_customer` ya devolvió un ID de cliente verificado en la sesión actual. Esto elimina el 8% de fallo **por completo** — no mejorando el prompt, sino impidiendo físicamente el orden de ejecución incorrecto.

### Configuración de ejemplo (hooks de ciclo de vida de subagentes)

> [!info] Más allá de la guía de certificación
> Lo siguiente no es material evaluado directamente por el Task Statement 1.4 — la guía lo marca como contexto de fondo (el apéndice de hooks del Agent SDK que sí evalúa el examen se limita a `PreToolUse`/`PostToolUse`, cubiertos en el tema [[5 Agent SDK Hooks/1 resumen|Agent SDK Hooks]]). Se incluye aquí porque la guía lo documenta junto al enforcement y el handoff, verificado contra la referencia de hooks el 16 de septiembre de 2026.

`SubagentStart` se dispara cuando se genera un subagente vía el Task tool (hoy `Agent`); recibe tipo e id del subagente, **no puede bloquear** el spawn, pero puede devolver `additionalContext` en su salida JSON para inyectar una instrucción antes del primer prompt del subagente. `SubagentStop` se dispara cuando el subagente termina y devuelve su resultado al coordinador; recibe el id y el mensaje final, y si la validación falla, **sale con código 2**, lo que impide que el subagente termine y lo devuelve a seguir trabajando — no existe un campo `decision` para este evento, y ningún hook reescribe el resultado devuelto.

```json
{
  "hooks": {
    "SubagentStart": [
      { "hooks": [{ "type": "command", "command": "echo \"subagent started $(date)\" >> .claude/spawns.log" }] }
    ],
    "SubagentStop": [
      { "hooks": [{ "type": "command", "command": ".claude/hooks/validate-subagent-output.sh" }] }
    ]
  }
}
```

- Los subagentes pueden definir sus propios hooks en su frontmatter, incluyendo `PreToolUse`/`PostToolUse`, con alcance limitado a las llamadas de herramienta que hace ese subagente específico (no el coordinador ni otros subagentes) — esto habilita políticas por rol, como una gate de reembolsos solo en el subagente de billing.
- Si el frontmatter de un subagente define hooks `Stop`, estos se **auto-convierten** en eventos `SubagentStop`, porque ese es el evento que dispara al completarse un subagente.
- Para bloquear el *spawn* en sí (rate limits, exigir que el coordinador pase cierto contexto), no sirve `SubagentStart` — se necesita un `PreToolUse` sobre la herramienta `Agent`, que sí puede denegar o reescribir la invocación antes de que el subagente arranque.

> [!note] Tabla mental rápida
> `SubagentStart`: observa el spawn, agrega contexto, no bloquea. `SubagentStop`: gatea la finalización con exit code 2, no agrega contexto, no reescribe el resultado. Para bloquear el spawn mismo o reescribir un resultado, se usa `PreToolUse`/`PostToolUse` sobre la herramienta `Agent`, no los hooks de ciclo de vida del subagente.

## Trampas de examen

> [!warning] Trampa 1 — Prompt más fuerte como solución para fallas de cumplimiento de alto riesgo
> Si el prompt actual ya instruye el flujo correcto pero falla el 8% de las veces, un prompt más fuerte podría bajar el fallo a 3-4%, pero **nunca llega a 0%**. Operaciones financieras, de seguridad y de cumplimiento requieren enforcement programático para garantías deterministas.

> [!warning] Trampa 2 — Ejemplos few-shot como suficientes para garantizar cumplimiento
> Los ejemplos few-shot mejoran el comportamiento del modelo, pero siguen siendo probabilísticos. No pueden ofrecer el 100% de enforcement que requieren operaciones financieras y de cumplimiento. La forma correcta es una prerequisite gate programática.

> [!warning] Trampa 3 — Un clasificador de ruteo para arreglar fallas de cumplimiento por agente
> Un clasificador de ruteo decide **qué agente** atiende una solicitud. La falla de cumplimiento ocurre **dentro** de la secuencia de ejecución de un agente, no en el nivel de ruteo. Los clasificadores resuelven ruteo, no enforcement de flujo de trabajo dentro de un agente.

> [!warning] Trampa 4 — Resúmenes de handoff que omiten campos críticos
> Los agentes humanos no tienen acceso a la transcripción de la conversación. El resumen de handoff debe ser autocontenido con los cinco campos requeridos: customer ID, resumen de la conversación, análisis de causa raíz, monto del reembolso, y acción recomendada. Omitir cualquiera fuerza al humano a pedirle al cliente que repita todo.

## Fuentes citadas por la guía
- Claude Certification Guide — módulo "Workflow Enforcement and Handoff" (`/learn/1-agentic-architecture/1-4-workflow-enforcement-handoff`)
- `examguide.pdf` — Task Statement 1.4: "Implement multi-step workflows with enforcement and handoff patterns"
- Claude Agent SDK Overview — Anthropic
- Hooks Reference — Anthropic
- Building with Claude API, incluyendo el escenario "Customer Support Resolution Agent" (Skilljar) — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
