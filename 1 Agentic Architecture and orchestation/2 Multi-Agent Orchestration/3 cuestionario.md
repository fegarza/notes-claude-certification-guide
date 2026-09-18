Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Arquitectura Hub-and-Spoke

> [!question]- ¿Quién ocupa el centro de una arquitectura hub-and-spoke y qué hace?
> El coordinador: recibe la tarea inicial, la descompone, decide qué subagentes invocar, les da contexto, recoge resultados, maneja fallos y dirige la información entre ellos.

> [!question]- ¿Pueden dos subagentes comunicarse directamente entre sí?
> No, nunca — toda comunicación debe pasar exclusivamente por el coordinador, sin importar la razón (ni velocidad, ni conveniencia).

> [!question]- ¿Cambia esta regla el hecho de que Claude Code permita técnicamente "nested delegation" (un subagente invocando a sus propios subagentes)?
> No para efectos del examen — la comunicación directa subagente-a-subagente sigue tratándose como incorrecta.

> [!question]- ¿Cuáles son los tres beneficios que el examen enfatiza del diseño centralizado?
> Observabilidad, manejo de errores consistente, y flujo de información controlado.

## Aislamiento de contexto

> [!question]- ¿Un subagente hereda automáticamente el historial de conversación del coordinador?
> No — solo tiene lo que el coordinador incluyó deliberadamente en sus instrucciones.

> [!question]- ¿Existe algún "repositorio de memoria colectiva" al que un subagente pueda acceder para obtener resultados de otro subagente?
> No, no existe tal repositorio; cualquier resultado de otro subagente debe transmitirse explícitamente en las instrucciones.

> [!question]- Si el coordinador invoca dos veces al mismo subagente, ¿la segunda invocación recuerda algo de la primera?
> No — cada invocación es independiente, no hay memoria persistente entre invocaciones separadas.

> [!question]- Si el agente de síntesis necesita los hallazgos del agente de búsqueda web, ¿cómo los obtiene?
> El coordinador se los pasa directamente en las instrucciones de la invocación — el agente de síntesis no puede "ir a buscarlos" por su cuenta.

## Responsabilidades del coordinador

> [!question]- ¿Qué significa "selección dinámica de subagentes"?
> Analizar los requerimientos de cada consulta y decidir qué subagentes invocar, en vez de enrutar siempre por el pipeline completo.

> [!question]- ¿Para qué sirve partición del alcance entre subagentes?
> Para minimizar duplicación de trabajo, asignando subtemas o tipos de fuente distintos a cada subagente.

> [!question]- ¿En qué consiste un loop de refinamiento iterativo del coordinador?
> Evaluar la síntesis en busca de huecos, re-delegar a los subagentes con consultas más específicas, y volver a invocar síntesis hasta lograr cobertura suficiente.

## La falla de descomposición estrecha

> [!question]- Cuando un sistema multi-agente entrega un resultado con dominios enteros ausentes (no solo poco detalle), ¿cuál es la causa casi universal?
> Que el coordinador descompuso la tarea de forma demasiado estrecha — no un fallo de los subagentes.

> [!question]- ¿Cómo se reconoce este patrón en los logs o en el comportamiento del sistema?
> Cada subagente ejecutó su tarea asignada correctamente y a fondo, pero el resultado final igual tiene huecos grandes de cobertura.

> [!question]- Ante un hueco de cobertura, ¿dónde hay que mirar primero: en cómo los subagentes ejecutaron su tarea, o en qué se les asignó?
> En qué se les asignó — es decir, en la descomposición del coordinador.

## Ejemplo de energías renovables

> [!question]- En el ejemplo del sistema de investigación sobre energías renovables, ¿qué subtemas asignó el coordinador?
> Solo "mejoras en paneles solares" y "ingeniería de turbinas eólicas".

> [!question]- ¿Qué categorías quedaron sin cubrir, y por qué?
> Geotérmica, mareomotriz, biomasa y fusión nuclear — porque el coordinador nunca las asignó a ningún subagente, no por mala investigación o síntesis.

> [!question]- ¿Cuáles son las tres soluciones que NO resuelven este tipo de hueco de cobertura?
> Mejorar los métodos de investigación, usar una síntesis más sofisticada, o agregar más subagentes.

> [!question]- ¿Cuál es la única solución real?
> Mejorar la descomposición del coordinador para que cubra el alcance completo del tema.

## Trampas de examen

> [!question]- ¿Por qué es un error culpar al agente de síntesis o de búsqueda por un hueco de cobertura?
> Porque los subagentes solo investigan lo que se les asignó; si el coordinador nunca asignó un subtema, ningún subagente puede cubrirlo aunque funcione perfectamente.

> [!question]- ¿Por qué es un error asumir que un subagente "ya sabe" algo que el coordinador sabe?
> Porque los subagentes operan con contexto totalmente aislado — nada se hereda automáticamente, todo debe transmitirse a propósito.

> [!question]- ¿Por qué proponer comunicación directa entre subagentes "para ganar eficiencia" es una trampa?
> Porque rompe los tres beneficios centrales del hub-and-spoke: observabilidad, manejo de errores consistente y flujo de información controlado.

> [!question]- ¿Por qué agregar más subagentes no arregla un problema de descomposición estrecha?
> Porque los subagentes nuevos reciben asignaciones igual de limitadas que las que ya existían — el problema está en el reparto de tareas, no en la cantidad de trabajadores.

> [!question]- En la pregunta de muestra oficial sobre "IA en industrias creativas", ¿cuál fue la causa raíz del reporte incompleto?
> El coordinador descompuso el tema solo en subtareas de artes visuales (arte digital, diseño gráfico, fotografía), sin asignar nunca música, escritura ni cine a ningún subagente.
