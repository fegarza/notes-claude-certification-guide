Cuestionario de repaso completo de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revisar.

## Opción 1 — `--resume`: continuación

> [!question]- ¿Qué restaura exactamente `--resume <session-name>` al reanudar una sesión?
> Todo el historial de conversación de esa sesión nombrada, incluyendo los resultados de herramientas y análisis previos — no solo un resumen.

> [!question]- ¿Cuándo es apropiado usar `--resume`?
> Cuando el contexto previo sigue siendo mayormente válido y los archivos no han cambiado de forma significativa desde la última sesión.

> [!question]- ¿Cuál es la diferencia entre `-c` / `--continue` y `--resume <name>`?
> `--continue` retoma automáticamente la conversación más reciente en el directorio actual, sin necesitar un nombre. `--resume <name>` retoma una sesión específica identificada por su nombre.

## Opción 2 — `fork_session`: exploración divergente

> [!question]- ¿Qué hace `fork_session` distinto de simplemente reanudar?
> Crea una sesión nueva a partir de una copia del historial existente, en vez de seguir escribiendo sobre la misma sesión. Los cambios en la rama nueva no afectan la original, y las ramas no pueden ver los resultados de las demás.

> [!question]- ¿`fork_session` reemplaza a `resume` o trabaja junto con él?
> Trabaja junto con él. `resume` selecciona la sesión base y `fork_session: true` hace que, en vez de seguir escribiendo sobre esa sesión, se cree una nueva a partir de una copia de su historial.

> [!question]- Un equipo quiere comparar una estrategia de testing contra una estrategia de documentación, ambas partiendo del mismo análisis de codebase ya hecho. ¿Qué opción usarías y por qué?
> `fork_session` — es exactamente el caso de exploración divergente desde una base de análisis compartida: dos caminos independientes que no deben contaminarse entre sí.

> [!question]- ¿Por qué "solo quiero seguir investigando lo mismo" NO es un caso para `fork_session`?
> Porque fork es para divergencia, no para continuación. Si no hay dos caminos distintos que explorar, bifurcar no aporta nada — lo correcto ahí es `--resume`.

## Opción 3 — inicio limpio con resumen inyectado

> [!question]- ¿Qué contiene el contexto inicial de una sesión nueva con resumen inyectado, y qué NO contiene?
> Contiene un resumen estructurado y curado de los hallazgos previos. NO contiene resultados de herramientas crudos ni obsoletos de la sesión anterior.

> [!question]- ¿Cuándo preferirías empezar de cero con un resumen en vez de reanudar?
> Cuando los resultados de herramientas de la sesión previa están obsoletos (archivos modificados, APIs actualizadas, dependencias cambiadas) o cuando el contexto se degradó tras una sesión muy larga.

## El problema del contexto obsoleto (stale context)

> [!question]- ¿Por qué reanudar una sesión después de modificar archivos puede producir consejos contradictorios?
> Porque reanudar restaura todo el historial, incluyendo resultados de herramientas viejos. Los archivos modificados siguen apareciendo con su contenido antiguo cacheado como salida de herramienta, y el modelo mezcla eso con lecturas nuevas — a veces responde con datos viejos, a veces con datos actuales.

> [!question]- Si reanudas una sesión y le pides al agente que relea los 3 archivos que modificaste, ¿resolviste el problema de contexto obsoleto?
> No del todo. Ayuda, pero los resultados obsoletos de esos archivos siguen presentes en el historial de conversación — el modelo puede seguir refiriéndose a información vieja de más atrás en el contexto. La solución completa es iniciar una sesión nueva con resumen inyectado.

> [!question]- ¿Cuál es la causa raíz del problema de contexto obsoleto, en una frase?
> Reanudar restaura el historial completo de conversación, y ese historial contiene resultados de herramientas que ya no reflejan el estado actual de los archivos modificados.

## Re-análisis dirigido vs. re-exploración completa

> [!question]- ¿En qué se diferencia el re-análisis dirigido de la re-exploración completa?
> El re-análisis dirigido informa al agente específicamente qué archivos cambiaron para que re-analice solo esos, combinando ese análisis fresco con el resumen preservado de los archivos sin cambios. La re-exploración completa re-analiza todo el codebase desde cero, sin aprovechar lo que ya se sabía.

> [!question]- ¿Por qué el re-análisis dirigido es preferible a la re-exploración completa cuando solo cambiaron 3 de 50 archivos?
> Porque re-explorar los 50 archivos es un desperdicio de tiempo y recursos cuando 47 de ellos no cambiaron — basta con re-analizar los 3 que sí cambiaron y preservar el resumen del resto.

## Matriz de decisión

> [!question]- ¿Por qué "reanudar tras actualizar dependencias" cae en la categoría de inicio limpio + resumen, y no en `--resume` directo?
> Porque una actualización de dependencias puede haber cambiado múltiples archivos de forma indirecta (no solo los que se tocaron explícitamente), lo que hace probable que el historial contenga resultados de herramientas ya obsoletos.

> [!question]- ¿Por qué memorizar los ejemplos literales de la matriz de decisión (dependencias, refactorización, etc.) no es suficiente para el examen?
> Porque el examen va a describir un escenario nuevo, no repetir esos ejemplos exactos — lo que hay que aplicar es el criterio detrás de cada fila (¿el contexto sigue siendo válido? ¿hay divergencia? ¿hay obsolescencia?), no reconocer un caso memorizado.

## Caso guía: el bug de los consejos contradictorios

> [!question]- En el caso de los tres archivos de autenticación corregidos durante la noche, ¿por qué el agente recomienda arreglar problemas que ya fueron corregidos al reanudar con `--resume`?
> Porque los resultados de herramientas del día 1 (mostrando el código sin arreglar) siguen en el historial de la sesión reanudada, y el agente mezcla esa información vieja con cualquier lectura nueva que haga, produciendo recomendaciones contradictorias.

> [!question]- ¿Qué resumen estructurado resolvería ese caso, y por qué funciona?
> Un resumen que indique qué archivos tenían problemas, cuáles fueron corregidos, y que pida re-analizar específicamente esos archivos. Funciona porque la sesión nueva no tiene ningún resultado de herramienta obsoleto — el agente parte de una lectura fresca y consistente del estado actual.

---
> [!tip] Sigue con este tema
> Repasa el resumen completo en [[1 resumen]], aplica estos conceptos en código en [[2 example]], y evalúate con [[4 test]].
