# Resultados — Tutor socrático: Agent SDK Hooks

> [!info] Sesión evaluada el 2026-09-19
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 68/100**

El estudiante domina con solidez la parte conceptual del tema: la distinción direccional entre `PreToolUse` y `PostToolUse`, el problema de normalización de datos heterogéneos, los tres patrones de enforcement (umbral de reembolso, prerrequisito AML, aprobación gerencial), y el framework de decisión hooks vs. prompts — incluyendo reconocer un distractor de examen basado en un prompt "bien escrito". Donde falló consistentemente fue en el detalle concreto y nombrado de la API: los valores exactos de `permissionDecision`, el campo `updatedInput`, y el nombre del campo obsoleto `updatedMCPToolOutput`. Esto sugiere comprensión del "por qué" y el "cuándo", pero memorización incompleta del "cómo se llama exactamente" — justo el tipo de detalle que el examen puede pedir de forma directa.

## Conceptos con buen dominio

- Dirección Pre/Post y sus poderes — identificó sin dudar que `PreToolUse` bloquea antes de que la herramienta corra y que `PostToolUse` no puede deshacer el efecto secundario ya ocurrido.
- Normalización de datos como problema de determinismo — conectó correctamente por qué dejar que el modelo interprete formatos mixtos falla (inconsistencia, no 100% de las veces).
- Los tres patrones de `PreToolUse` (reembolso, AML, aprobación gerencial) — describió correctamente qué dispara cada uno y qué pasa con la ejecución en cada caso.
- Framework de decisión hooks vs. prompts — aplicó correctamente el criterio de "riesgo financiero/legal → hook" tanto en abstracto como en el escenario de distractor del examen (Pregunta 11).

## Áreas que necesitan revisión

### Valores de retorno de `PreToolUse` (`ask`, `defer`, `updatedInput`)

- **Qué pasó:** No supo nombrar los valores de `permissionDecision` distintos de `allow`/`deny` (`ask` y `defer`), ni el campo `updatedInput` que permite reescribir los argumentos de la llamada antes de que corra.
- **Contenido del resumen:**

| Hook          | Qué devuelve                                                                             | Para qué sirve                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `PreToolUse`  | `permissionDecision` (`allow` / `deny` / `ask` / `defer`) + opcionalmente `updatedInput` | Bloquear, aprobar, pedir confirmación, o reescribir los argumentos de la llamada antes de que corra |

- **Sección:** [[1 resumen]] → "1. Dos hooks, dos direcciones, dos poderes distintos" (solo como enlace de navegación).
- **Analogía para reforzarlo:** Piensa en el oficial de aduana de la analogía del resumen: no solo puede decir "pasa" o "no pasa" — también puede decir "espera, llamo a mi supervisor" (`ask`), "vuelve más tarde cuando tenga la info" (`defer`), o "te dejo pasar, pero primero corrígeme esta declaración de aduana" (`updatedInput`).

### Campo `updatedToolOutput` y su predecesor obsoleto

- **Qué pasó:** No recordó que el campo actual de `PostToolUse` (`updatedToolOutput`) reemplazó a uno anterior específico de MCP.
- **Contenido del resumen:**

| Hook          | Qué devuelve                                                          | Para qué sirve                                                                                |
| ------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `PostToolUse` | `updatedToolOutput` (reemplaza el ya obsoleto `updatedMCPToolOutput`) | Sustituir el resultado que el modelo va a ver, sin tocar lo que ya ocurrió en el sistema real |

- **Sección:** [[1 resumen]] → "1. Dos hooks, dos direcciones, dos poderes distintos" (solo como enlace de navegación).
- **Analogía para reforzarlo:** Es como si la etiqueta que usa el empleado de la bodega para reetiquetar cajas cambiara de nombre de "etiqueta-MCP" a simplemente "etiqueta" — porque ya no solo reetiqueta cajas que llegaron por el camión de MCP, sino cualquier caja, venga de donde venga.

### `PreCompact`/`PostCompact` — qué puede bloquear `PreCompact`

- **Qué pasó:** Identificó correctamente el patrón de sincronización (antes de compactar), pero no mencionó que `PreCompact` puede efectivamente **prevenir** la compactación, mientras que `PostCompact` solo observa el resumen ya generado.
- **Contenido del resumen:**
  > **`PreCompact`** corre justo antes de que Claude Code compacte la conversación (ya sea manual, con `/compact`, o automático al llegar al límite de la ventana de contexto). Recibe la ruta del transcript, el tipo de disparador, e instrucciones personalizadas. Se usa para archivar el transcript completo antes de que el resumen descarte detalle. Puede prevenir la compactación devolviendo código de salida 2 o `"decision": "block"`.
  >
  > **`PostCompact`** corre después, y recibe el resumen ya generado.
  >
  > El patrón es el mismo que ya se conoce de los hooks de herramienta: **un evento `Pre` puede inspeccionar y bloquear; un evento `Post` solo puede observar lo que ya pasó.**
- **Sección:** [[1 resumen]] → "5. Más allá de `PreToolUse`/`PostToolUse`: el mismo patrón en otros eventos" (solo como enlace de navegación).
- **Analogía para reforzarlo:** Es como un editor que puede detener la publicación de un artículo antes de que salga a imprenta (`PreCompact` bloqueando), versus un lector que ya recibió el periódico impreso y solo puede subrayar cosas en su copia (`PostCompact` observando).

## Siguiente paso recomendado

Antes de intentar [[4 test]], repasa específicamente la tabla de "Valores de retorno" en la sección 1 del resumen hasta poder nombrar los cuatro valores de `permissionDecision` y los dos campos de modificación (`updatedInput`, `updatedToolOutput`) sin dudar — el resto del tema ya está en buen nivel.
