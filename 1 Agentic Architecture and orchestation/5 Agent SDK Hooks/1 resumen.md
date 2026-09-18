> [!note] En una frase
> Un hook `PreToolUse` es un guardia en la puerta que puede impedir que una herramienta se ejecute; un hook `PostToolUse` es un traductor que ya no puede impedir nada, pero limpia y estandariza lo que la herramienta ya devolvió — confundir esas dos direcciones es el error más común del tema.

## Explícamelo como si tuviera 5 años

Imagina una aduana con dos puestos de trabajo distintos. El primero está en la **entrada del país**: un oficial revisa cada camión antes de dejarlo pasar, y si algo no cumple las reglas, el camión **nunca entra** — se queda afuera. El segundo puesto está **dentro de la bodega**, después de que los camiones ya entraron: ahí un empleado recibe cajas con etiquetas en formatos distintos (unas en kilos, otras en libras, unas en español, otras en inglés) y las reetiqueta todas igual antes de mandarlas al inventario. Ese empleado no puede rechazar nada — el camión ya entró — solo puede ordenar lo que ya llegó.

El oficial de la entrada es `PreToolUse`: se ejecuta **antes** de que la herramienta corra, y puede bloquearla, modificarla o redirigirla. El empleado de la bodega es `PostToolUse`: se ejecuta **después** de que la herramienta ya corrió, y solo puede transformar el resultado que el modelo va a ver — nunca deshacer lo que ya pasó.

## Argumento central

> Los hooks del Agent SDK inyectan **comportamiento determinista** dentro de un sistema que, por lo demás, es probabilístico. Lo que determina qué hook usar no es "¿qué tan importante es esto?" en abstracto, sino una pregunta mecánica: **¿necesito actuar antes de que la herramienta corra, o solo necesito limpiar lo que ya devolvió?** `PreToolUse` responde a la primera pregunta (enforcement de políticas); `PostToolUse` responde a la segunda (normalización de datos). El examen prueba específicamente que no se confundan las dos direcciones.

## Ideas clave (Conclusiones)

### 1. Dos hooks, dos direcciones, dos poderes distintos

- **`PreToolUse`** corre **antes** de que la herramienta se ejecute (por eso también se llama *tool-call interception*). Puede **bloquear**, **modificar** o **redirigir** la llamada. Si el hook bloquea, la herramienta **nunca corre** — no hay efecto secundario que deshacer, porque nunca ocurrió.
- **`PostToolUse`** corre **después** de que la herramienta ya se ejecutó, pero **antes** de que el modelo procese el resultado. Puede **transformar** lo que el modelo ve, pero el efecto secundario real (el cargo procesado, el registro escrito) ya ocurrió — bloquear aquí detiene el *loop*, no deshace la acción.

> [!warning] La limitación crítica de `PostToolUse`
> "Para el momento en que `PostToolUse` se dispara, la herramienta ya corrió — así que bloquear ahí detiene el loop pero no deshace el efecto secundario." Si la herramienta era `process_refund`, el reembolso ya salió. `PostToolUse` nunca es la herramienta correcta para *impedir* que algo pase — solo para limpiar cómo se ve después de que ya pasó.

**Valores de retorno** (la forma concreta en que cada hook ejerce su poder):

| Hook | Qué devuelve | Para qué sirve |
|---|---|---|
| `PreToolUse` | `permissionDecision` (`allow` / `deny` / `ask` / `defer`) + opcionalmente `updatedInput` | Bloquear, aprobar, pedir confirmación, o reescribir los argumentos de la llamada antes de que corra |
| `PostToolUse` | `updatedToolOutput` (reemplaza el ya obsoleto `updatedMCPToolOutput`) | Sustituir el resultado que el modelo va a ver, sin tocar lo que ya ocurrió en el sistema real |

> [!note] Idea clave del examen
> "Los hooks `PostToolUse` transforman datos **después** de la ejecución. Los hooks `PreToolUse` aplican políticas **antes** de la ejecución. Saber en qué dirección opera cada uno es exactamente lo que el examen evalúa."

### 2. `PostToolUse`: normalización de datos heterogéneos

**El problema**: distintas herramientas MCP devuelven el mismo tipo de dato en formatos completamente distintos — timestamps Unix (`1710489600`) contra fechas ISO 8601 (`"2024-03-15T12:00:00Z"`); códigos de estado numéricos (`200`, `404`, `500`) contra strings (`"active"`, `"cancelled"`, `"pending"`).

- Si no hay normalización, **el modelo tiene que interpretar formatos mixtos en cada iteración** — y esa interpretación es inconsistente: puede leer bien un timestamp Unix una vez y mal la siguiente.
- La solución es un hook `PostToolUse` que intercepta el resultado crudo de la herramienta y lo convierte a un formato único **antes** de que el modelo lo vea: timestamps Unix → ISO 8601, códigos numéricos → strings legibles, montos de moneda → formato decimal consistente con código de moneda, fechas regionales → un único estándar.
- El resultado: el modelo recibe datos limpios y consistentes **siempre**, sin importar qué herramienta o sistema backend los produjo.

> [!note] Analogía rápida
> Es el empleado de la bodega reetiquetando cajas. No importa en qué idioma o unidad llegó la etiqueta original — todas salen con el mismo formato antes de llegar al inventario (el modelo).

### 3. `PreToolUse`: enforcement de políticas antes de que algo ocurra

`PreToolUse` es el mecanismo concreto detrás de las *prerequisite gates* — la misma idea de enforcement programático que ya se vio en [[4 Workflow Enforcement and Handoff/1 resumen|Workflow Enforcement and Handoff]], pero aplicada específicamente a interceptar la llamada a la herramienta.

Tres patrones de uso, todos con la misma forma (interceptar → evaluar una condición → bloquear o dejar pasar):

1. **Umbral de reembolso**: el hook intercepta `process_refund`. Si el monto supera $500, bloquea la llamada y redirige a un flujo de escalación humana — `process_refund` **nunca se ejecuta**.
2. **Prerrequisito de cumplimiento**: el hook intercepta `transfer_funds`. Si el chequeo AML (anti-lavado de dinero) no se completó en la sesión, bloquea la llamada y devuelve un error indicando al agente que complete el chequeo AML primero.
3. **Aprobación gerencial**: el hook intercepta `approve_discount` para montos sobre 20%. Pausa la ejecución y enruta la solicitud a una cola de aprobación gerencial; la herramienta solo corre después de esa aprobación.

> [!warning] Trampa de examen directa
> "El examen va a presentar hooks `PostToolUse` como solución para bloquear acciones que violan una política. Esto es incorrecto. `PostToolUse` corre **después** de la ejecución — para cuando se dispara, la acción no conforme **ya ocurrió**. Se usa `PreToolUse` (pre-ejecución) para bloquear acciones antes de que pasen."

### 4. El framework de decisión: hooks vs. prompts

La misma lógica de "determinista vs. probabilístico" de [[4 Workflow Enforcement and Handoff/1 resumen|Workflow Enforcement and Handoff]] se aplica aquí en forma de tabla de decisión:

| Requisito | Mecanismo | Garantía |
|---|---|---|
| Debe cumplirse el 100% de las veces | Hooks | Determinista |
| Preferible, pero una desviación ocasional es aceptable | Prompts | Probabilística |

Criterio de decisión:

- Si el negocio **perdería dinero** por un solo fallo → hook.
- Si el negocio **enfrentaría riesgo legal** por un solo fallo → hook.
- Si es una **preferencia de formato o estilo** → una guía en el prompt es aceptable.

> [!warning] El punto que el examen repite constantemente
> "La decisión no es si el prompt es 'lo suficientemente bueno' — es si la consecuencia de un solo fallo justifica una garantía determinista." El examen usa soluciones basadas en prompt como distractor en cualquier escenario que requiera enforcement determinista, sin importar cuán bien redactado esté el prompt propuesto.

**Comparación lado a lado** (tres escenarios de la guía, mismo patrón cada vez):

- *Transferencias internacionales deben pasar chequeo AML*: prompt → ~95% de cumplimiento, el 5% restante ya es una violación regulatoria real. Hook `PreToolUse` que bloquea `transfer_funds` hasta que `aml_check` pase → 100%, ninguna transferencia corre sin verificación.
- *Las respuestas deben tener formato markdown*: prompt → funciona casi siempre, y una respuesta ocasional en texto plano no es un riesgo de negocio. Un hook aquí sería sobre-ingeniería — no hay nada que justifique la garantía determinista.
- *Reembolsos sobre $500 requieren aprobación humana*: prompt → un solo fallo ya significa un reembolso grande sin aprobación. Hook `PreToolUse` que intercepta `process_refund`, revisa el monto, bloquea si supera $500 y enruta a escalación humana → 100% de las veces.

### 5. Más allá de `PreToolUse`/`PostToolUse`: el mismo patrón en otros eventos

> [!info] Más allá de la guía de certificación
> El Task Statement 1.5 se limita explícitamente a `PreToolUse` y `PostToolUse`. Claude Code dispara hooks en otros puntos de una sesión también, y el banco de práctica ha evaluado uno de ellos — `PreCompact` — desde julio. Se incluye aquí como contexto de fondo, no como parte central del tema.

- **`PreCompact`** corre justo antes de que Claude Code compacte la conversación (ya sea manual, con `/compact`, o automático al llegar al límite de la ventana de contexto). Recibe la ruta del transcript, el tipo de disparador, e instrucciones personalizadas. Se usa para archivar el transcript completo antes de que el resumen descarte detalle. Puede prevenir la compactación devolviendo código de salida 2 o `"decision": "block"`.
- **`PostCompact`** corre después, y recibe el resumen ya generado.

El patrón es el mismo que ya se conoce de los hooks de herramienta: **un evento `Pre` puede inspeccionar y bloquear; un evento `Post` solo puede observar lo que ya pasó.**

## Trampas de examen

> [!warning] Trampa 1 — Usar `PostToolUse` para bloquear una acción que viola una política
> `PostToolUse` corre después de la ejecución. Para cuando se dispara, la acción no conforme ya ocurrió. La corrección: usar `PreToolUse` para bloquear antes de que la acción se ejecute.

> [!warning] Trampa 2 — Instrucciones de prompt "mejoradas" como solución para exigir 100% de cumplimiento
> Los prompts, sin importar qué tan enfáticos o detallados, siguen dando cumplimiento probabilístico. Para requisitos financieros, regulatorios o de seguridad, se necesitan las garantías deterministas de un hook.

> [!warning] Trampa 3 — Sugerir que el modelo transforme los datos en vez de usar `PostToolUse`
> Dejar que el modelo normalice datos heterogéneos introduce inconsistencia (interpreta bien una vez, mal la siguiente). `PostToolUse` garantiza datos limpios sin importar qué herramienta los produjo, sin depender de la interpretación del modelo.

> [!warning] Trampa 4 — Confundir la dirección de cada hook
> `PostToolUse` y `PreToolUse` tienen sincronización opuesta: uno corre antes, el otro después. Confundir cuál es cuál es, según la guía, exactamente lo que el examen evalúa en este tema.

## Fuentes citadas por la guía
- Claude Certification Guide — módulo "Agent SDK Hooks" (`/learn/1-agentic-architecture/1-5-agent-sdk-hooks`)
- `examguide.pdf` — Task Statement 1.5: "Apply Agent SDK hooks for tool call interception and data normalization"
- Claude Agent SDK Overview — Anthropic
- Claude Agent SDK Hooks Documentation — Anthropic
- Claude Code Hooks Reference — `PreCompact` — Anthropic
- Building with Claude API (Skilljar) — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
