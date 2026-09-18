Cuestionario de repaso completo de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revisar.

## Dos hooks, dos direcciones, dos poderes distintos

> [!question]- ¿Cuál es la diferencia de timing entre `PreToolUse` y `PostToolUse`?
> `PreToolUse` corre antes de que la herramienta se ejecute. `PostToolUse` corre después de que ya se ejecutó, pero antes de que el modelo procese el resultado.

> [!question]- Si un hook `PreToolUse` bloquea una llamada, ¿qué pasa con el efecto secundario de esa herramienta?
> No pasa nada — la herramienta nunca corre, así que no hay ningún efecto secundario que deshacer. El bloqueo es preventivo.

> [!question]- Si un hook `PostToolUse` "bloquea" después de detectar un problema, ¿qué es lo que realmente logra y qué es lo que NO logra?
> Logra detener el loop del agente (evitar que el modelo siga procesando ese resultado). NO logra deshacer el efecto secundario real, porque la herramienta ya se ejecutó antes de que el hook se disparara.

> [!question]- ¿Qué devuelve un hook `PreToolUse` y qué opciones tiene ese valor?
> Devuelve `permissionDecision`, con valores `allow`, `deny`, `ask` o `defer`, y opcionalmente `updatedInput` para reescribir los argumentos de la llamada antes de que corra.

> [!question]- ¿Qué devuelve un hook `PostToolUse` y qué reemplaza ese campo?
> Devuelve `updatedToolOutput`, que sustituye el resultado que verá el modelo. Reemplaza al campo ya obsoleto `updatedMCPToolOutput`.

## `PostToolUse`: normalización de datos heterogéneos

> [!question]- ¿Por qué distintas herramientas MCP devolviendo datos en formatos distintos es un problema, aunque cada herramienta individualmente funcione bien?
> Porque el modelo tiene que interpretar formatos mixtos en cada iteración, y esa interpretación es inconsistente — puede leer bien un timestamp Unix una vez y mal la siguiente, sin que haya cambiado nada en el dato mismo.

> [!question]- ¿En qué momento del ciclo actúa el hook `PostToolUse` para resolver ese problema?
> Intercepta el resultado crudo justo después de que la herramienta corrió y antes de que el modelo lo vea, y lo convierte a un formato único antes de entregarlo.

> [!question]- Da dos ejemplos de transformaciones típicas que resuelve un hook `PostToolUse` de normalización.
> Timestamps Unix → fechas ISO 8601, y códigos de estado numéricos → strings legibles (cualquier par válido: también montos de moneda → formato decimal consistente, o fechas regionales → un estándar único).

> [!question]- ¿Por qué dejar la normalización de datos en manos del modelo (en vez de un hook) es una mala idea, aunque el modelo "sepa" interpretar todos los formatos?
> Porque la interpretación del modelo es probabilística — puede acertar la mayoría de las veces pero fallar ocasionalmente, generando inconsistencia. El hook garantiza el mismo formato siempre, sin depender de que el modelo interprete bien cada vez.

## `PreToolUse`: enforcement de políticas antes de que algo ocurra

> [!question]- ¿Qué tienen en común los tres patrones de uso de `PreToolUse` (umbral de reembolso, prerrequisito AML, aprobación gerencial)?
> Los tres interceptan la llamada a la herramienta, evalúan una condición, y bloquean o dejan pasar la ejecución **antes** de que ocurra — nunca actúan sobre algo que ya pasó.

> [!question]- En el caso del umbral de reembolso, ¿qué pasa exactamente cuando el monto supera $500?
> El hook bloquea la llamada a `process_refund` y la redirige a un flujo de escalación humana. La herramienta nunca se ejecuta para ese monto.

> [!question]- ¿Cómo se relaciona `PreToolUse` con el concepto de "prerequisite gate" visto en Workflow Enforcement and Handoff?
> Es el mecanismo concreto detrás de esa idea: la prerequisite gate es el patrón general de bloquear una herramienta hasta que se cumple un prerrequisito, y `PreToolUse` es el hook específico del Agent SDK que la implementa interceptando la llamada.

> [!question]- ¿Por qué usar `PostToolUse` para intentar bloquear un reembolso que viola una política sería un error, aunque el hook detecte correctamente el problema?
> Porque para cuando `PostToolUse` se dispara, `process_refund` ya se ejecutó — el reembolso ya salió. Detectar el problema después no lo previene; se necesita `PreToolUse` para bloquear antes de que la herramienta corra.

## El framework de decisión: hooks vs. prompts

> [!question]- ¿Cuál es la pregunta central que determina si algo necesita un hook o basta con un prompt?
> Si la consecuencia de un solo fallo aislado ya justifica una garantía determinista — no si el prompt "es lo bastante bueno" en términos de porcentaje de acierto.

> [!question]- Da dos condiciones bajo las cuales la guía dice que se debe usar un hook en vez de un prompt.
> Si el negocio perdería dinero por un solo fallo, o si el negocio enfrentaría riesgo legal por un solo fallo.

> [!question]- ¿Por qué el escenario de "las respuestas deben tener formato markdown" NO necesita un hook, aunque un prompt no garantice el 100%?
> Porque una respuesta ocasional en texto plano no es un riesgo de negocio real — es una preferencia de formato, y aplicar un hook ahí sería sobre-ingeniería sin ninguna consecuencia que justifique la garantía determinista.

> [!question]- Si un prompt logra 95% de cumplimiento en un chequeo AML antes de transferencias internacionales, ¿por qué ese 95% no es aceptable?
> Porque el 5% restante ya representa transferencias reales que se ejecutaron sin verificación AML — una violación regulatoria real, no un promedio aceptable. En cumplimiento, un solo fallo aislado ya es inaceptable.

> [!question]- ¿Qué deberías sospechar si el examen presenta "reforzar el prompt con instrucciones más detalladas o enfáticas" como solución ante un escenario financiero o de cumplimiento?
> Que es un distractor. Un prompt más detallado sigue siendo probabilístico — mejora el porcentaje de acierto, pero nunca llega al 100% que exige un requisito financiero o de cumplimiento.

## Más allá de `PreToolUse`/`PostToolUse`

> [!question]- ¿Qué puede hacer un hook `PreCompact` que un hook `PostCompact` no puede?
> `PreCompact` puede inspeccionar el estado antes de la compactación y prevenirla (código de salida 2 o `"decision": "block"`). `PostCompact` solo puede observar el resumen ya generado, sin poder evitarlo.

> [!question]- ¿Qué patrón general comparten `PreCompact`/`PostCompact` con `PreToolUse`/`PostToolUse`?
> El mismo principio de dirección: un evento `Pre` puede inspeccionar y bloquear antes de que algo ocurra; un evento `Post` solo puede observar lo que ya pasó, sin poder deshacerlo.

---
> [!tip] Sigue con este tema
> Repasa el resumen completo en [[1 resumen]], aplica estos conceptos en código en [[2 example]], y evalúate con [[4 test]].
