> [!note] En una frase
> El Dominio 1 describe cómo se construye un agente de principio a fin: un **agentic loop** controlado por `stop_reason` es el motor; un coordinador **hub-and-spoke** lo multiplica en subagentes aislados que solo saben lo que se les pasa; el **enforcement programático** (vía hooks `PreToolUse`/`PostToolUse`) pone garantías deterministas donde el modelo es probabilístico; la **decomposición** decide cómo se reparte el trabajo; y el **estado de sesión** decide cómo se retoma sin arrastrar contexto obsoleto.

## Mapa mental del módulo

```mermaid
mindmap
  root((Dominio 1<br/>Agentic Architecture<br/>and Orchestration))
    1.1 Agentic loop
      Ciclo de 4 pasos
        Enviar request
        Inspeccionar stop_reason
        tool_use: ejecutar y agregar tool results al historial
        end_turn: terminar
      stop_reason es la única señal confiable
        No parsear texto
        No content type
        Iteration limit solo como red de seguridad
      Model-driven decision-making
        Excepto financiero, seguridad, compliance
    1.2 Multi-Agent Orchestration
      Hub-and-spoke
        Todo pasa por el coordinador
        Observabilidad
        Manejo de errores consistente
        Flujo de información controlado
      Context isolation
        Sin herencia automática
        Sin memoria compartida
        Sin memoria entre invocaciones
      Responsabilidades del coordinador
        Selección dinámica de subagentes
        Partición del alcance
        Refinamiento iterativo
      Narrow Decomposition Failure
        Huecos de cobertura = culpa del coordinador
    1.3 Subagent Invocation
      Task tool, hoy Agent, en allowedTools
        Compuerta binaria
      AgentDefinition
        Description
        System prompt
        Tool restrictions
      Context passing
        Hallazgos completos
        Contenido + metadata estructurada
        Prompts orientados a objetivos
      Invocación paralela
        Varias llamadas Task en una sola respuesta
      fork_session vs resume
    1.4 Workflow Enforcement and Handoff
      Prompt-based = probabilístico
      Programático = determinista
      Regla: dinero, seguridad, compliance = código
      Prerequisite gates
        process_refund bloqueado hasta get_customer
      Multi-concern
        Descomponer, investigar en paralelo, síntesis unificada
      Handoff estructurado
        Humano sin transcript
        Customer ID, resumen, causa raíz, monto, acción
    1.5 Agent SDK Hooks
      PreToolUse
        Antes de ejecutar
        Bloquear, modificar, redirigir
        permissionDecision, updatedInput
      PostToolUse
        Después de ejecutar
        Normalizar datos heterogéneos
        updatedToolOutput
        No deshace efectos secundarios
      Hooks vs prompts
        100% requerido = hook
    1.6 Task Decomposition
      Fixed sequential pipeline
        Prompt chaining
        Pasos conocidos de antemano
      Dynamic adaptive decomposition
        Alcance abierto
        El plan emerge al investigar
      Attention dilution
        Profundidad inconsistente
        Problema arquitectónico
      Multi-pass
        Pasadas locales por ítem
        Pasada de integración cruzada
    1.7 Session State and Resumption
      resume = continuación
      fork_session = exploración divergente
      Fresh start + summary injection = tool results obsoletos
      Stale context
        Releer archivos no alcanza
      Re-análisis dirigido
        Solo archivos modificados
```

## Cómo se relacionan los temas

Cada flecha es una dependencia conceptual: el tema de origen es la base o el "por qué" del tema destino.

```mermaid
flowchart TD
    T1["1.1 Agentic loop<br/>el motor"]
    T2["1.2 Multi-Agent Orchestration<br/>la arquitectura"]
    T3["1.3 Subagent Invocation<br/>el cableado"]
    T4["1.4 Workflow Enforcement<br/>las garantías"]
    T5["1.5 Agent SDK Hooks<br/>el mecanismo de las garantías"]
    T6["1.6 Task Decomposition<br/>cómo repartir el trabajo"]
    T7["1.7 Session State<br/>cómo retomar el trabajo"]

    T1 -->|"el coordinador es un agentic loop que llama subagentes como herramientas"| T2
    T2 -->|"de la arquitectura a la API real: Task tool y context passing"| T3
    T1 -->|"model-driven, excepto cuando el negocio exige determinismo"| T4
    T4 -->|"las prerequisite gates se implementan con PreToolUse"| T5
    T5 -->|"PostToolUse limpia los tool results antes de volver al loop"| T1
    T2 -->|"Narrow Decomposition Failure = mala decomposición del coordinador"| T6
    T4 -->|"multi-concern: descomponer y resolver en paralelo"| T6
    T3 -->|"fork_session vs resume se introduce aquí"| T7
    T1 -->|"los tool results viven en el historial y pueden quedar obsoletos"| T7
    T3 -.->|"invocación paralela de subagentes independientes"| T6
    T2 -.->|"context isolation: el receptor solo sabe lo que se le pasa"| T4
```

### Lectura rápida de las relaciones

| Relación | Qué conecta | Idea puente |
|---|---|---|
| 1.1 → 1.2 | Loop → coordinador | Un coordinador es un agentic loop cuyo "tool" es invocar subagentes; cada subagente corre su propio loop. |
| 1.2 → 1.3 | Arquitectura → mecánica | 1.2 dice *qué* (hub-and-spoke, context isolation); 1.3 dice *cómo* (`Task`/`Agent` en `allowedTools`, metadata estructurada). |
| 1.1 → 1.4 | Model-driven → determinista | El loop deja que Claude elija herramientas, salvo en operaciones financieras, de seguridad o compliance: ahí manda el código. |
| 1.4 → 1.5 | Principio → implementación | Una prerequisite gate es, en concreto, un hook `PreToolUse` que deniega la llamada. |
| 1.5 → 1.1 | Hook → loop | `PostToolUse` normaliza el tool result *antes* de que se agregue al historial y Claude lo procese en la siguiente iteración. |
| 1.2 → 1.6 | Coordinador → decomposición | Los huecos de cobertura vienen de una decomposición demasiado estrecha; 1.6 da el criterio para descomponer bien. |
| 1.4 → 1.6 | Multi-concern → decomposición | Una solicitud con varios asuntos se descompone, se investiga en paralelo y se sintetiza. |
| 1.3 → 1.7 | fork_session → sesiones | 1.3 introduce `fork_session` vs `--resume`; 1.7 lo convierte en una matriz de decisión completa. |
| 1.1 → 1.7 | Historial → stale context | El loop exige agregar tool results al historial; ese mismo historial es el que se vuelve obsoleto al reanudar. |

## Hilos conductores del módulo

Estas ideas atraviesan varios temas. Si las tienes claras, muchas preguntas del examen se resuelven con el mismo razonamiento.

### 1. Determinista le gana a probabilístico cuando el fallo cuesta

- **1.1** — `stop_reason` (determinista) en vez de parsear texto (ambiguo).
- **1.4** — prerequisite gate en vez de un prompt "más fuerte".
- **1.5** — hook en vez de instrucción en el prompt cuando se necesita el 100%.
- **1.6** — cambiar la arquitectura (multi-pass) en vez de un mejor prompt o un modelo más grande.

> [!note] Analogía
> El cartel pide, el torniquete impide. En todo el módulo, la respuesta correcta ante un fallo "caro" es el torniquete.

### 2. El receptor solo sabe lo que le pasas

- **1.2** — context isolation: los subagentes no heredan nada del coordinador.
- **1.3** — sin metadata (URL, documento, página) el agente de síntesis no puede citar.
- **1.4** — el humano del handoff no ve el transcript: el resumen debe ser autocontenido.
- **1.7** — en un fresh start, el resumen inyectado es todo lo que el agente sabe del trabajo previo.

### 3. Una pregunta mecánica decide, no "qué tan importante es"

| Tema | Pregunta diagnóstica |
|---|---|
| 1.1 | ¿Qué dice `stop_reason`? |
| 1.4 | ¿Un solo fallo cuesta dinero, seguridad o compliance? |
| 1.5 | ¿Necesito actuar *antes* de que corra la herramienta o solo limpiar *después*? |
| 1.6 | ¿Conozco los pasos de antemano? |
| 1.7 | ¿El contexto previo es válido, quiero divergir, o los tool results están obsoletos? |

### 4. Arregla la causa raíz "arriba", no el síntoma "abajo"

- **1.2** — huecos de cobertura: el problema es la decomposición del coordinador, no los subagentes.
- **1.3** — citas faltantes: el problema es que el coordinador quitó la metadata, no el prompt del agente de síntesis.
- **1.6** — análisis inconsistente: el problema es la arquitectura de una sola pasada, no el modelo.
- **1.7** — consejos contradictorios: el problema es el historial obsoleto, no que falte releer archivos.

### 5. Paralelizar lo independiente

- **1.2** — partir el alcance entre subagentes para no duplicar trabajo.
- **1.3** — varias llamadas al Task tool en una sola respuesta.
- **1.4** — investigar cada asunto de una solicitud multi-concern en paralelo.
- **1.6** — pasadas locales por ítem, independientes entre sí, antes de la integración cruzada.

## Trampas transversales

> [!warning] Distractores que reaparecen en varios temas
> - **"Mejora el prompt" / "agrega few-shot"** → incorrecto en 1.4, 1.5 y 1.6 cuando se requiere garantía o el problema es estructural.
> - **"Agrega un iteration limit"** → solo red de seguridad en 1.1; no arregla una terminación prematura.
> - **"Agrega más subagentes" / "usa un modelo más grande"** → no arregla una mala decomposición (1.2) ni la attention dilution (1.6).
> - **"Que los subagentes se comuniquen directo"** → rompe hub-and-spoke (1.2).
> - **"Usa `PostToolUse` para bloquear"** → la acción ya ocurrió (1.5).
> - **"Haz `--resume` y relee los archivos" o "haz fork"** → el historial obsoleto sigue ahí (1.7).

## Temas del módulo

1. [[1 Agentic loop/1 resumen|1.1 Agentic loop]]
2. [[2 Multi-Agent Orchestration/1 resumen|1.2 Multi-Agent Orchestration]]
3. [[3 Subagent Invocation and Context Passing/1 resumen|1.3 Subagent Invocation and Context Passing]]
4. [[4 Workflow Enforcement and Handoff/1 resumen|1.4 Workflow Enforcement and Handoff]]
5. [[5 Agent SDK Hooks/1 resumen|1.5 Agent SDK Hooks]]
6. [[6 Task Decomposition Strategies/1 resumen|1.6 Task Decomposition Strategies]]
7. [[7 Session State and Resumption/1 resumen|1.7 Session State and Resumption]]
