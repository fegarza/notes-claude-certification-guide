Cuestionario de repaso completo de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revisar.

## El espectro de enforcement: prompt-based vs. programático

> [!question]- ¿Por qué la guía basada en prompt se describe como "probabilística"?
> Porque depende de que el modelo interprete y siga correctamente una instrucción de lenguaje natural cada vez — y el modelo puede saltarse pasos, reordenarlos o leer la instrucción de forma laxa. Funciona la mayoría de las veces, pero con una tasa de fallo mayor a cero.

> [!question]- ¿Qué hace que el enforcement programático sea "determinista" en vez de probabilístico?
> Que el bloqueo ocurre en código (un hook, una gate, un check), no en la interpretación del modelo. El modelo no tiene forma de "decidir" saltarse una compuerta que está implementada fuera de su control — el resultado es el mismo el 100% de las veces.

> [!question]- Si un prompt bien escrito funciona el 92% de las veces, ¿por qué no basta para una operación financiera?
> Porque el 8% restante ya representa fallos reales con consecuencia monetaria (reembolsos en cuentas equivocadas). Para operaciones de alto riesgo, cualquier tasa de fallo distinta de cero es inaceptable — se necesita una garantía determinista, no una mejora probabilística.

> [!question]- ¿Qué diferencia práctica hay entre "mejorar el prompt" y "agregar una prerequisite gate" ante el mismo problema de verificación faltante?
> Mejorar el prompt reduce la probabilidad de fallo pero nunca la elimina. La prerequisite gate bloquea físicamente la ejecución de la herramienta downstream hasta que se cumple el prerrequisito, eliminando el fallo por completo, sin importar qué decida el modelo.

## La regla de decisión del examen

> [!question]- ¿Cuál es el criterio que usa el examen para decidir entre prompt-based y programático?
> Si la operación es financiera, de seguridad o de cumplimiento, la respuesta correcta es siempre enforcement programático. Si es de bajo riesgo (formato, estilo), la guía basada en prompt es aceptable.

> [!question]- ¿Por qué una inconsistencia de formato de salida no amerita enforcement programático, pero un reembolso sí?
> Porque la consecuencia de un fallo es distinta: un formato inconsistente no genera pérdida de dinero, brecha de seguridad ni violación de cumplimiento — es un riesgo de negocio nulo o mínimo. Un reembolso mal verificado sí tiene consecuencia financiera real y directa.

> [!question]- ¿Qué tienen en común las operaciones financieras, de seguridad y de cumplimiento para que el examen las trate igual?
> En las tres, un solo fallo aislado ya constituye un daño real e irreversible (pérdida de dinero, brecha de seguridad, sanción legal) — no es un problema de "promedio aceptable", es un problema de que cada instancia individual importa.

> [!question]- ¿Qué deberías sospechar si el examen presenta "agregar ejemplos few-shot" o "reforzar el prompt del sistema" como opción ante un escenario de alto riesgo?
> Que es un distractor. Mejora la precisión probabilística, pero no elimina la tasa de fallo — nunca es la respuesta correcta cuando hay dinero, seguridad o cumplimiento en juego.

## Prerequisite gates en la práctica

> [!question]- ¿Qué es, en términos simples, una prerequisite gate?
> Un check programático que bloquea la ejecución de una herramienta hasta que se cumple una condición previa — por ejemplo, que otra herramienta ya haya devuelto un dato verificado.

> [!question]- ¿Por qué se dice que la gate es "código, no una instrucción de prompt"?
> Porque vive fuera del razonamiento del modelo: es lógica de aplicación que intercepta la llamada a la herramienta y la bloquea o permite según un estado verificable, sin depender de que el modelo "decida" respetar un orden.

> [!question]- Si el modelo intenta llamar directamente a la herramienta protegida sin cumplir el prerrequisito, ¿qué pasa con una prerequisite gate bien implementada?
> La gate bloquea la llamada y devuelve un error explicando qué falta, forzando al modelo a cumplir el prerrequisito antes de reintentar — independientemente de la intención del modelo.

## Solicitudes con múltiples asuntos (multi-concern)

> [!question]- ¿Cuáles son los tres pasos correctos para manejar una solicitud con varios asuntos a la vez?
> Descomponer la solicitud en asuntos distintos, investigar cada uno en paralelo usando contexto compartido, y sintetizar una resolución unificada que atienda todos los asuntos en una sola respuesta.

> [!question]- ¿Por qué investigar los asuntos en paralelo tiene sentido si comparten contexto (ej. la cuenta del cliente)?
> Porque no hay dependencia real entre ellos — la información de cuenta es relevante para los tres, pero resolver uno no requiere esperar a que se resuelva otro. Investigar en serie solo agrega latencia sin ninguna ganancia.

> [!question]- ¿Qué dos formas de manejar mal una solicitud multi-concern señala la guía como incorrectas?
> Atenderla en conversaciones separadas y secuenciales, o atender solo el primer asunto mencionado y olvidar el resto.

## Protocolos de handoff estructurado

> [!question]- ¿Cuál es la restricción crítica que define cómo debe ser un resumen de handoff a un humano?
> Que el agente humano no tiene acceso a la transcripción de la conversación — no puede revisar el historial de chat, así que el resumen debe ser autocontenido y darle todo lo necesario para actuar sin más contexto.

> [!question]- ¿Cuáles son los cinco campos que debe incluir un resumen de handoff completo?
> Customer ID, resumen de la conversación, análisis de causa raíz, monto del reembolso (si aplica), y acción recomendada.

> [!question]- ¿Qué pasa en la práctica si un resumen de handoff omite el customer ID o la acción recomendada?
> El humano no puede ubicar la cuenta o no tiene una recomendación clara sobre qué hacer, y probablemente tiene que pedirle al cliente que repita información que ya dio — una mala experiencia y una falla de diseño del handoff.

> [!question]- ¿Por qué "monto del reembolso" debe ser una cifra específica y no una referencia vaga como "el monto que pidió el cliente"?
> Porque el humano no tiene el contexto de la conversación para saber a qué monto se refiere esa referencia — necesita el número concreto para poder actuar sin tener que reconstruir la conversación.

## Hooks de ciclo de vida de subagentes (contenido de fondo, no evaluado directamente)

> [!question]- ¿Qué puede y qué no puede hacer el hook `SubagentStart`?
> Puede observar el spawn de un subagente (recibe tipo e id) y agregar contexto devolviendo `additionalContext`, que se inyecta antes del primer prompt del subagente. No puede bloquear el spawn.

> [!question]- ¿Qué puede y qué no puede hacer el hook `SubagentStop`?
> Puede bloquear la finalización del subagente saliendo con código 2, lo que lo devuelve a seguir trabajando (útil para validar que su salida cumple un esquema esperado). No puede agregar contexto ni reescribir el resultado devuelto.

> [!question]- Si necesitas bloquear el spawn mismo de un subagente (por ejemplo, un rate limit), ¿qué hook usarías y por qué no `SubagentStart`?
> Un `PreToolUse` sobre la herramienta `Agent` (antes `Task`), porque puede denegar o reescribir la invocación antes de que el subagente arranque. `SubagentStart` solo observa — no tiene la capacidad de bloquear el spawn.

> [!question]- ¿Qué significa que los hooks definidos en el frontmatter de un subagente estén "scoped" a ese subagente?
> Que solo interceptan las llamadas a herramienta que hace ese subagente específico durante su ejecución — no las del coordinador ni las de otros subagentes. Esto permite políticas por rol, como una restricción de reembolsos solo para un subagente de billing.

> [!question]- ¿Qué pasa con los hooks `Stop` definidos en el frontmatter de un subagente?
> Se auto-convierten en eventos `SubagentStop`, porque ese es el evento del ciclo de vida que corresponde a la finalización de un subagente — así que la lógica de limpieza o validación definida como `Stop` en el subagente se ejecuta igual al completarse.

---
> [!tip] Sigue con este tema
> Repasa el resumen completo en [[1 resumen]], aplica estos conceptos en código en [[2 example]], y evalúate con [[4 test]].
