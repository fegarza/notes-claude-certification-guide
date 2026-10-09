Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Argumento central

> [!question]- ¿Qué es la `context degradation`?
> El modo de falla en sesiones largas donde el modelo pierde el rastro de los hallazgos específicos que descubrió antes, porque quedan enterrados bajo output verboso más reciente.

> [!question]- ¿Por qué la exploración de codebases es especialmente propensa a la `context degradation`?
> Porque es de las tareas que más contexto consumen: cada paso produce mucho output verboso (archivos, búsquedas, listados).

> [!question]- ¿La `context degradation` es un problema de cantidad o de calidad del contexto?
> De calidad (atención): no es quedarse sin espacio, sino perder los detalles específicos entre tanto volumen.

## `Context degradation`

> [!question]- ¿Cuál es el síntoma observable de la `context degradation`?
> El modelo habla de "typical patterns" en vez de citar las clases, métodos y cadenas de dependencias concretas que ya descubrió, y da respuestas inconsistentes.

> [!question]- ¿Cuál es la diferencia entre "sigue el typical repository pattern" y una respuesta sana?
> La sana cita lo específico: qué clase, en qué archivo, qué interfaz implementa y qué método tiene comportamiento particular.

> [!question]- ¿Cuáles son los 4 pasos del mecanismo de degradación?
> Cada paso genera output verboso → se acumula en el contexto → los hallazgos tempranos quedan empujados atrás → la atención se desplaza a lo reciente y pierde las referencias anteriores.

> [!question]- ¿Por qué aumentar la ventana de contexto no arregla la degradación?
> Porque el problema no es el espacio; una ventana más grande también se llena de output verboso que entierra los detalles.

> [!question]- ¿Qué pasaría si precargas todo el codebase en el contexto inicial?
> Llenarías el contexto de volumen desde el inicio, que es justo lo que causa la degradación; es la trampa de "más contexto" disfrazada.

## `Scratchpad files`

> [!question]- ¿Qué es un `scratchpad file`?
> Un archivo donde el agente escribe los hallazgos clave y que consulta para las preguntas siguientes.

> [!question]- ¿Por qué un scratchpad es inmune a la `context degradation`?
> Porque el conocimiento vive fuera del contexto de la conversación; se lee fresco cuando se necesita, no depende de la atención sobre contexto viejo.

> [!question]- ¿Cuál es la mitigación principal de la `context degradation`?
> Los `scratchpad files`.

> [!question]- ¿Qué tipo de contenido debe ir en un scratchpad?
> Información específica: nombres de clases con su ruta, cadenas de dependencias y hallazgos críticos, no descripciones genéricas.

> [!question]- ¿Cuándo debe empezar a usarse el scratchpad?
> Desde el inicio de cualquier sesión de exploración extendida, como estrategia deliberada, no como rescate cuando ya hay degradación.

> [!question]- ¿Qué hace el agente cuando necesita recordar un hallazgo anterior?
> Lee el scratchpad en vez de confiar en el contexto de la conversación.

## `Subagent delegation`

> [!question]- ¿Cómo ayuda la `subagent delegation` contra la degradación?
> Cada subagente explora en su propio contexto aislado; el output verboso se queda ahí y al coordinador solo le llega un resumen estructurado.

> [!question]- ¿Cuál es el beneficio principal de usar subagentes para explorar codebases: paralelismo o `context isolation`?
> `Context isolation`. El paralelismo es la lectura obvia, pero el valor real es mantener limpio el contexto del agente principal.

> [!question]- ¿Qué tipo de tareas se delegan a un subagente?
> Preguntas específicas y acotadas, como "encuentra todos los test files", "traza el flujo de refunds" o "identifica las integraciones externas".

> [!question]- ¿Qué devuelve un subagente al coordinador?
> Un resumen estructurado con los hallazgos clave, no el output crudo de su exploración.

> [!question]- ¿Qué rol conserva el agente principal cuando delega?
> La coordinación de alto nivel y la comprensión global del sistema.

> [!question]- ¿Qué pasaría si el coordinador hiciera toda la exploración él mismo?
> Su contexto se llenaría de output verboso y terminaría degradándose, perdiendo los hallazgos específicos.

## `Summary injection` entre fases

> [!question]- ¿Qué es la `summary injection` entre fases?
> Resumir los hallazgos de una fase antes de lanzar los subagentes de la siguiente, e inyectar ese resumen en su contexto inicial.

> [!question]- ¿Qué es el problema de "cold start"?
> Que los subagentes de la Fase 2 dupliquen la exploración de la Fase 1 porque no recibieron sus hallazgos.

> [!question]- Además de evitar trabajo duplicado, ¿qué aporta la `summary injection`?
> La comprensión arquitectónica que los subagentes necesitan para hacer las preguntas correctas.

> [!question]- ¿Por qué un subagente no "sabe" lo que encontró la fase anterior?
> Porque no hereda el contexto del coordinador; solo sabe lo que se le pasa explícitamente en su prompt.

> [!question]- ¿Qué debe contener un buen resumen de fase?
> La arquitectura, los flujos y clases relevantes, las preocupaciones clave y el objetivo de la siguiente fase.

## `/compact`

> [!question]- ¿Qué hace `/compact` en Claude Code?
> Resume la conversación para liberar contexto preservando la información clave.

> [!question]- ¿Cuándo conviene usar `/compact`?
> De forma proactiva durante sesiones largas, cuando el contexto se llena de output de descubrimiento verboso, no solo al tocar el límite.

> [!question]- ¿Por qué usar `/compact` solo al llegar al límite es un error?
> Porque para entonces la calidad del contexto ya se degradó; `/compact` está para proteger la calidad, no solo la cantidad.

## `Crash recovery` con `state manifests`

> [!question]- ¿Qué tres cosas pueden interrumpir una sesión larga de exploración?
> Crashes de la sesión, interrupciones de red y agotamiento del contexto.

> [!question]- ¿Qué pasa sin un mecanismo de recuperación?
> Se pierde todo el progreso de exploración.

> [!question]- ¿Qué es la `structured state persistence`?
> Que cada agente exporte su estado actual a una ubicación conocida (un `manifest`).

> [!question]- ¿Qué cuatro cosas incluye un `manifest`?
> Lo explorado (archivos leídos, búsquedas), los hallazgos clave, la fase actual con los siguientes pasos, y las preguntas pendientes.

> [!question]- ¿Por qué el manifest debe estar en una ubicación conocida?
> Para que el coordinador sepa dónde encontrarlo y cargarlo al reanudar.

> [!question]- ¿Quién carga el manifest al reanudar y qué hace con él?
> El coordinador; lo carga y lo inyecta en los prompts de los agentes para que continúen donde se quedaron.

> [!question]- ¿Cuál es la diferencia entre un scratchpad y un manifest?
> El scratchpad es la libreta de hallazgos que el agente consulta durante la sesión; el manifest es el estado estructurado (explorado, fase, next steps) que el coordinador usa para reanudar tras una falla.

> [!question]- ¿Qué pasaría si reinicias la sesión para "curar" la degradación sin guardar estado?
> La degradación desaparece, pero también todo el conocimiento acumulado; hay que re-explorar todo.

> [!question]- ¿Cuál es la forma correcta de reiniciar una sesión degradada?
> Persistir los hallazgos (scratchpad y manifest) antes de reiniciar e inyectarlos en la nueva sesión.

## Relaciones entre conceptos

> [!question]- ¿Qué tienen en común el scratchpad, el manifest y la `summary injection`?
> Los tres mueven el conocimiento importante fuera del contexto verboso y lo vuelven a meter de forma compacta y específica cuando se necesita.

> [!question]- ¿Qué mitigación usarías si el coordinador se llena de output de lectura de archivos?
> `Subagent delegation`, para aislar esa exploración verbosa.

> [!question]- ¿Qué mitigación usarías si los subagentes nuevos repiten trabajo ya hecho?
> `Summary injection` de la fase anterior en sus prompts.

> [!question]- ¿Qué mitigación usarías si el agente olvida clases que encontró hace una hora?
> `Scratchpad file` consultado antes de cada pregunta.

> [!question]- ¿Qué mitigación usarías si un crash de red te hace perder la sesión?
> `State manifests` cargados por el coordinador on resume.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], aplícalo en código en [[2 example]] y evalúate con [[4 test]].
