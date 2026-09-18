Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## El Task tool

> [!question]- ¿Qué requisito binario debe cumplirse antes de que un coordinador pueda invocar cualquier subagente?
> `allowedTools` del coordinador debe incluir `"Task"` (o `"Agent"`, su nombre actual) — sin eso, no puede invocar ningún subagente, sin importar cómo estén definidos.

> [!question]- Si te preguntan en el examen por el nombre de la herramienta que invoca subagentes, ¿qué nombre respondes?
> "Task tool" — es el nombre que fija la guía oficial (v1.0), aunque el código actual de Claude Code muestre "Agent" (Task sigue funcionando como alias).

> [!question]- En el Agent SDK actual, ¿qué pasa técnicamente si `Agent` no está en `allowedTools`?
> No se elimina la herramienta; cada invocación pasa por el callback de permisos, que en una ejecución desatendida la deniega — el resultado práctico es el mismo que describe el examen.

## AgentDefinition

> [!question]- ¿Cuáles son los tres componentes que definen a cada subagente?
> Description (qué hace, usada por el coordinador para decidir cuándo invocarlo), system prompt (sus instrucciones) y tool restrictions (qué herramientas puede usar, limitadas a su rol).

## Paso de contexto

> [!question]- Según la Regla 1 de paso de contexto, ¿qué debe hacer el coordinador si el agente de síntesis necesita hallazgos de otros agentes?
> Pasarle esos hallazgos completos, directamente en su prompt — no puede asumir que el agente de síntesis los va a "buscar" por su cuenta.

> [!question]- ¿Qué dice la Regla 2 sobre formatos de datos entre agentes?
> Deben usar formatos estructurados que separen el contenido (la afirmación, el análisis) de la metadata (URL, nombre del documento, número de página).

> [!question]- ¿Qué pasa si se pasa contenido sin metadata al agente receptor?
> No tiene forma de atribuir sus afirmaciones a una fuente — no puede citar lo que no se le dio.

> [!question]- ¿Qué dice la Regla 3 sobre cómo deben escribirse los prompts del coordinador hacia los subagentes?
> Deben especificar objetivos y criterios de calidad, no instrucciones procedimentales paso a paso.

> [!question]- ¿Por qué los prompts orientados a objetivos son mejores que los procedimentales para subagentes?
> Permiten que el subagente se adapte cuando encuentra situaciones inesperadas; las instrucciones procedimentales lo encasillan y se lo impiden.

> [!question]- Un agente de síntesis produce afirmaciones sin fuente, y los agentes de búsqueda y análisis funcionan correctamente. ¿Cuál es la causa raíz más probable?
> El coordinador pasó contenido sin metadata estructurada al agente de síntesis — este literalmente no tenía información de fuente que incluir.

## Formato de metadata estructurada

> [!question]- En el formato de ejemplo de la guía (`findings`), ¿qué campos son "contenido" y cuáles son "metadata"?
> Contenido: `claim`. Metadata: `source_url`, `document_name`, `page_number`, `confidence`, `retrieved_by`.

> [!question]- En el ejemplo de la falla de atribución, ¿por qué falla el reporte de síntesis aunque búsqueda web y análisis de documentos funcionen bien?
> Porque el coordinador les quita la metadata antes de pasar el contenido a síntesis — envía las afirmaciones en texto plano, sin URLs ni referencias de página.

> [!question]- ¿Cuál es la solución correcta ante esa falla de atribución?
> Que el coordinador pase la metadata estructurada junto al contenido — no modificar el prompt del agente de síntesis.

## Invocación paralela vs. secuencial

> [!question]- ¿Cómo debe invocar el coordinador a varios subagentes cuyas tareas son independientes entre sí?
> Emitiendo múltiples llamadas al Task tool en una sola respuesta, no una por una en turnos separados.

> [!question]- ¿Qué problema introduce la invocación secuencial de subagentes independientes?
> Latencia innecesaria — no hay razón para que un subagente espere a otro si sus tareas no dependen entre sí.

> [!question]- En una opción de respuesta del examen, ¿qué frases son señal del patrón correcto de invocación paralela?
> Frases como "en una sola respuesta" o "simultáneamente".

## `fork_session`

> [!question]- ¿Qué hace `fork_session`?
> Crea una rama independiente a partir de una línea base de análisis compartida — cada rama opera de forma independiente después del punto de bifurcación.

> [!question]- ¿Cuál es la diferencia central entre `fork_session` y `--resume`?
> `--resume` continúa (agrega a) una sesión nombrada específica; `fork_session` bifurca, creando una rama nueva e independiente.

> [!question]- En la práctica actual del Agent SDK, ¿`fork_session` es una alternativa a `resume` o un modificador de este?
> Un modificador: se pasan ambos juntos (`resume=session_id, fork_session=True`); si se omite `fork_session`, la misma llamada simplemente agrega a la sesión original en vez de bifurcarla.

> [!question]- ¿Cuándo usarías fork en vez de resume, y viceversa?
> Fork cuando necesitas exploración divergente desde un punto de partida compartido (comparar dos enfoques); resume cuando quieres continuar la misma línea de investigación.

## Trampas de examen

> [!question]- ¿Por qué es un error asumir que un subagente tiene acceso automático al historial del coordinador o a la salida de otros subagentes?
> Porque no existe herencia automática de contexto — cada dato necesario debe incluirse explícitamente en el prompt del subagente.

> [!question]- ¿Por qué es un error culpar al agente de síntesis cuando el reporte final no tiene citas?
> Porque el agente de síntesis solo puede citar fuentes que se le dieron; si el coordinador no pasó metadata, no hay forma de que produzca citas, sin importar qué tan bien redactado esté su prompt.

> [!question]- ¿Por qué proponer invocación secuencial para tareas independientes es una trampa?
> Porque introduce latencia innecesaria cuando las tareas podrían correr en paralelo sin ningún costo adicional.

> [!question]- ¿Por qué es un error tratar `fork_session` y `--resume` como intercambiables?
> Porque uno bifurca y el otro agrega — confundirlos lleva a elegir la herramienta equivocada según si se necesita explorar enfoques divergentes o continuar la misma línea de trabajo.
