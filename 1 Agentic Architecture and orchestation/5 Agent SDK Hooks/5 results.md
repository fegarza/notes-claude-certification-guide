# Resultados — Tutor socrático: Agent SDK Hooks

> [!info] Sesión evaluada el 2026-10-25
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 85/100**

Hubo una mejora clara frente a la sesión anterior (68/100). Los tres puntos débiles de entonces ya se dominan: los cuatro valores de `permissionDecision` (`allow`/`deny`/`ask`/`defer`), `updatedToolOutput` como reemplazo de `updatedMCPToolOutput`, y el poder de `PreCompact` para bloquear la compactación. El estudiante maneja bien la dirección Pre/Post, los tres patrones de enforcement y el framework hooks vs. prompts, incluido el distractor del prompt "reforzado" al 99%. Lo que queda por pulir es el campo `updatedInput` (confundido con `permissionDecisionReason`, que no aparece en el resumen) y un detalle de sincronización de `PostToolUse`: corre antes de que el modelo procese el resultado. En la última pregunta descartó la opción A con una razón equivocada, pero aclaró que leyó `PreToolUse` donde decía `PostToolUse`. Como en la Pregunta 3 sí explicó correctamente por qué `PostToolUse` no deshace el efecto secundario, se trata como un error de lectura y no de concepto.

## Conceptos con buen dominio

- **Hooks = comportamiento determinista**: explicó sin ayuda que los hooks garantizan el 100% de cumplimiento y por qué eso importa ante riesgo financiero, legal o de seguridad.
- **Limitación crítica de `PostToolUse`**: identificó que bloquear después de `process_refund` detiene el loop pero no deshace el reembolso.
- **Valores de `permissionDecision`**: nombró los cuatro (`allow`/`deny`/`ask`/`defer`). Era un punto débil en la sesión anterior.
- **`updatedToolOutput` / `updatedMCPToolOutput`**: recordó el campo actual y el obsoleto que reemplazó. También era un punto débil antes.
- **Normalización con `PostToolUse` vs. prompt (Trampa 3)**: rechazó normalizar vía prompt por ser no determinista y eligió `PostToolUse`.
- **Patrones de `PreToolUse`**: AML (bloquear y devolver un mensaje que indique qué hacer) y aprobación gerencial (la herramienta corre solo tras aprobar).
- **Framework hooks vs. prompts (Trampa 2)**: descartó un hook para el formato markdown (sobre-ingeniería) y un prompt al 99% para reembolsos mayores a $500.
- **`PreCompact`/`PostCompact`**: explicó que `PreCompact` puede archivar el transcript o bloquear con código de salida 2 / `"decision": "block"`.

## Áreas que necesitan revisión

### El campo `updatedInput` de `PreToolUse`

- **Qué pasó:** Cuando se le preguntó por el campo opcional que, además de `permissionDecision`, permite hacer algo distinto a solo aprobar o bloquear, respondió `permissionDecisionReason` (mensaje de error explicativo). Ese campo no aparece en el resumen, y no es lo que permite **reescribir los argumentos** de la llamada.
- **Contenido del resumen:**

| Hook          | Qué devuelve                                                                             | Para qué sirve                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `PreToolUse`  | `permissionDecision` (`allow` / `deny` / `ask` / `defer`) + opcionalmente `updatedInput` | Bloquear, aprobar, pedir confirmación, o reescribir los argumentos de la llamada antes de que corra |

- **Sección:** [[1 resumen]] → "1. Dos hooks, dos direcciones, dos poderes distintos"
- **Analogía para reforzarlo:** Un mesero que recibe tu pedido antes de mandarlo a la cocina. Puede rechazarlo ("ya no hay salmón"), aceptarlo tal cual, o **corregirlo antes de entregarlo**: tachar "con cacahuate" porque sabe que eres alérgico. Esa corrección del pedido, no la explicación de por qué lo rechazó, es `updatedInput`.

### Cuándo corre exactamente `PostToolUse`

- **Qué pasó:** Respondió que `PostToolUse` corre "después de terminar la ejecución de la herramienta", pero no mencionó que corre **antes de que el modelo procese el resultado**. Esa ventana es la que le permite transformar lo que el modelo ve.
- **Contenido del resumen:**
  > **`PostToolUse`** corre **después** de que la herramienta ya se ejecutó, pero **antes** de que el modelo procese el resultado. Puede **transformar** lo que el modelo ve, pero el efecto secundario real (el cargo procesado, el registro escrito) ya ocurrió — bloquear aquí detiene el *loop*, no deshace la acción.
- **Sección:** [[1 resumen]] → "1. Dos hooks, dos direcciones, dos poderes distintos"
- **Analogía para reforzarlo:** Un traductor en una videollamada internacional. Escucha lo que la otra persona ya dijo (no puede cambiar que lo dijo), pero te lo entrega traducido **antes** de que tú lo oigas. Si llegara después de que ya escuchaste el original, no serviría de nada.

### Por qué falla la interpretación de formatos mixtos (matiz menor)

- **Qué pasó:** Identificó bien que el modelo puede malinterpretar formatos mezclados (confundir día con mes). Le faltó el matiz de que el problema es la **inconsistencia entre iteraciones**: el modelo puede leer bien un formato una vez y mal la siguiente.
- **Contenido del resumen:**
  > Si no hay normalización, **el modelo tiene que interpretar formatos mixtos en cada iteración** — y esa interpretación es inconsistente: puede leer bien un timestamp Unix una vez y mal la siguiente.
- **Sección:** [[1 resumen]] → "2. `PostToolUse`: normalización de datos heterogéneos"
- **Analogía para reforzarlo:** Un cajero que recibe billetes de cinco países distintos sin tabla de conversión. No es que siempre cobre mal, es que a veces acierta y a veces no, y nunca sabes cuándo. Lo peligroso es el error impredecible, no el error constante.

> [!note] Sobre la Pregunta 14
> Descartó la opción A ("`PostToolUse` que bloquea y detiene el loop") con una razón equivocada. Aclaró que leyó `PreToolUse`. La razón correcta es la Trampa 1: para cuando `PostToolUse` se dispara, el registro ya fue borrado. No cuenta como área débil porque en la Pregunta 3 lo explicó bien, pero en el examen real conviene **leer con cuidado el nombre del hook en cada opción**, porque ahí es justo donde el examen pone sus distractores.

## Siguiente paso recomendado

Repasa la tabla de "Valores de retorno" hasta asociar `updatedInput` con "reescribir los argumentos antes de que corra". Con eso el tema queda sólido y ya estás listo para [[4 test]].
