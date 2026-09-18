> [!note] En una frase
> En un sistema multi-agente evaluado por el examen, un coordinador central es el único que habla con los subagentes —que no comparten memoria ni contexto entre sí— y casi cualquier "hueco" de cobertura en el resultado final es culpa de cómo el coordinador repartió el trabajo, no de los subagentes que lo ejecutaron.

## Explícamelo como si tuviera 5 años

Imagina la cocina de un restaurante. Hay un **jefe de cocina** (el coordinador) y varios **cocineros especializados** (los subagentes): uno solo hace las entradas, otro solo los platos fuertes, otro solo los postres. Ningún cocinero le grita a otro cocinero directamente — si el de postres necesita saber qué llevó el plato fuerte, tiene que preguntarle al jefe de cocina, porque es el único que tiene la visión completa del pedido. Y cada cocinero, cuando llega a trabajar, no "recuerda" nada de ayer ni de lo que hicieron los demás hoy — solo sabe lo que el jefe le apuntó en su comanda.

Ahora imagina que el menú de esa noche es "cena de mariscos" y el jefe de cocina solo le reparte a los cocineros: camarones y pulpo. Nadie cocinó el salmón ni las almejas — no porque los cocineros lo hicieron mal, sino porque el jefe nunca los apuntó en ninguna comanda. Ese es, casi siempre, el verdadero problema cuando un sistema multi-agente entrega un resultado incompleto: no falló quien cocinó, falló quien repartió las comandas.

## Argumento central

> El examen evalúa un único patrón de arquitectura multi-agente: **hub-and-spoke**, con un coordinador central que enruta *toda* la comunicación y controla qué contexto recibe cada subagente, porque los subagentes operan con **aislamiento total de contexto** (sin herencia automática, sin memoria compartida). La causa casi universal de una cobertura incompleta en el resultado final es una **descomposición de tarea demasiado estrecha por parte del coordinador** — nunca un fallo de los subagentes que sí ejecutaron lo que se les asignó.

## Ideas clave (Conclusiones)

### 1. Arquitectura Hub-and-Spoke

- El **coordinador** ocupa el centro: recibe la tarea inicial, la descompone, decide qué subagentes invocar, les da contexto, recoge sus resultados, maneja fallos y dirige la información entre ellos.
- Los **subagentes** son los "radios" de la rueda: cada uno resuelve una responsabilidad enfocada (buscar en la web, analizar documentos, sintetizar hallazgos, redactar reportes).
- **Regla absoluta**: toda comunicación pasa exclusivamente por el coordinador. Los subagentes **nunca** se comunican entre sí — ni por velocidad, ni por conveniencia, ni por ninguna razón.

> [!note] Matiz sobre productos reales
> Los productos actuales de Claude Code técnicamente permiten *nested delegation* (un subagente puede invocar a sus propios subagentes). Para efectos del examen, esto no cambia la regla: tratar la comunicación directa subagente-a-subagente como incorrecta.

- Este diseño centralizado da tres beneficios que el examen enfatiza:
  1. **Observabilidad** — todo el tráfico de mensajes queda registrado y supervisado en un solo punto.
  2. **Manejo de errores consistente** — la recuperación ante fallos se centraliza en el coordinador, no se improvisa por subagente.
  3. **Flujo de información controlado** — el coordinador decide, de forma deliberada, qué contexto recibe cada subagente.

### 2. El principio de aislamiento de contexto

- Los subagentes **no heredan automáticamente** el historial de conversación del coordinador.
- Cuando el coordinador invoca un subagente, ese subagente solo tiene lo que el coordinador **incluyó deliberadamente** en sus instrucciones — nada de system prompt del coordinador (salvo que se incluya explícitamente), nada de mensajes previos, nada de resultados de otros subagentes (salvo que se transmitan a propósito).
- No existe ningún "repositorio de memoria colectiva" ni estado universal compartido entre subagentes.
- **Tampoco hay memoria persistente entre invocaciones separadas**: si el coordinador invoca dos veces al subagente de búsqueda web, la segunda invocación no recuerda nada de la primera. Cada invocación es independiente.

> [!warning] Consecuencia práctica
> El coordinador debe ser deliberado sobre qué información transmite. Si el agente de síntesis necesita los hallazgos de la búsqueda web, el coordinador se los tiene que pasar directamente en sus instrucciones — el agente de síntesis **no puede "ir a buscarlos"** a ningún repositorio central, porque ese repositorio no existe.

### 3. Responsabilidades del coordinador (4 áreas)

1. **Selección dinámica de subagentes** — analizar los requerimientos de la consulta y decidir qué subagentes invocar, en vez de enrutar siempre por el pipeline completo.
2. **Partición del alcance de investigación** — repartir subtemas o tipos de fuente distintos entre subagentes para minimizar duplicación de trabajo.
3. **Loops de refinamiento iterativo** — evaluar la salida de síntesis en busca de huecos, re-delegar a los subagentes de búsqueda/análisis con consultas más específicas, y volver a invocar síntesis hasta que la cobertura sea suficiente.
4. **Enrutamiento centralizado de comunicación** — por observabilidad, manejo de errores consistente y flujo de información controlado (la misma razón de ser del hub-and-spoke).

### 4. La falla de descomposición estrecha (Narrow Decomposition Failure)

Este es el patrón de examen más distintivo del tema: cuando un sistema multi-agente entrega un resultado con **alcance incompleto** —no "poco detalle", sino **dominios enteros ausentes**— la causa casi universal es la forma en que el coordinador descompuso la tarea, no algo que los subagentes hicieron mal.

> [!note] Cómo reconocer este patrón
> La señal es: cada subagente ejecutó su tarea asignada correctamente y a fondo, pero el resultado final igual tiene huecos grandes. Si todos los componentes funcionaron bien y aun así falta cobertura, hay que mirar **qué se les asignó**, no **cómo lo hicieron**.

## Evidencia — Ejemplo práctico: hueco de cobertura en un sistema de investigación

Un sistema de investigación sobre energías renovables recibe la tarea de cubrir el tema completo. El coordinador lo descompone en "mejoras en el rendimiento de paneles solares" y "ingeniería de turbinas eólicas". Cada subagente produce investigación extensa y bien documentada sobre su tema asignado.

El resultado final cubre solar y eólica a fondo, pero **no menciona geotérmica, mareomotriz, biomasa ni fusión nuclear**. Esta ausencia no refleja mala investigación ni mala síntesis — es consecuencia directa de que el coordinador nunca asignó esas categorías a ningún subagente.

La solución **no es** mejorar los métodos de investigación, ni una síntesis más sofisticada, ni agregar más subagentes — es mejorar la **descomposición del coordinador** para que cubra el alcance completo del tema.

## Trampas de examen

> [!warning] Trampa 1 — Culpar a los subagentes "aguas abajo" por huecos de cobertura
> Asumir que un subagente (búsqueda, análisis de documentos, síntesis) falló porque el resultado final tiene huecos. **Por qué falla:** los subagentes investigan solo lo que se les asignó; si el coordinador limitó "energías renovables" a solar y eólica, ningún subagente tiene capacidad de cubrir geotérmica o mareomotriz aunque quisiera. **La forma correcta:** rastrear el fallo hasta su origen — casi siempre la descomposición del coordinador.

> [!warning] Trampa 2 — Asumir herencia automática de contexto o memoria compartida
> Suponer que un subagente "ya sabe" algo porque el coordinador lo sabe, o porque otro subagente lo generó antes. **Por qué falla:** los subagentes operan con contexto completamente aislado; no adquieren nada del coordinador de forma automática. **La forma correcta:** cada elemento de información requiere transmisión intencional dentro de las instrucciones del subagente.

> [!warning] Trampa 3 — Proponer comunicación directa entre subagentes "por eficiencia"
> Sugerir que dos subagentes se comuniquen directamente para ahorrar una vuelta por el coordinador. **Por qué falla:** la comunicación directa rompe la observabilidad, el manejo de errores consistente y el flujo de información controlado — los tres beneficios centrales del diseño hub-and-spoke. **La forma correcta:** toda comunicación pasa por el coordinador, sin excepción, sin importar consideraciones de eficiencia.

> [!warning] Trampa 4 — Agregar más subagentes para arreglar un problema de descomposición
> Ante un hueco de cobertura, proponer sumar subagentes adicionales como solución. **Por qué falla:** si el coordinador ya restringió el subtema demasiado, los subagentes nuevos reciben asignaciones igual de limitadas — no aportan nada. **La forma correcta:** el arreglo está en mejorar la metodología de descomposición del coordinador, no en la cantidad de subagentes.

> [!warning] Pregunta de muestra oficial (examguide.pdf, Question 7)
> Un sistema de investigación sobre "el impacto de la IA en las industrias creativas" entrega reportes que cubren solo artes visuales, sin música, escritura ni cine — aunque cada subagente (búsqueda, análisis, síntesis) ejecutó su parte correctamente. Los logs muestran que el coordinador descompuso el tema en tres subtareas: "IA en arte digital", "IA en diseño gráfico" e "IA en fotografía". La respuesta correcta identifica que la causa raíz es que la descomposición del coordinador fue demasiado estrecha — nunca cubrió música, escritura ni cine. Las opciones que culpan al agente de síntesis, al agente de búsqueda o al agente de análisis de documentos son distractores que ignoran que esos agentes trabajaron correctamente dentro del alcance que se les asignó.

## Fuentes citadas por la guía
- Claude Certification Guide — módulo "Multi-Agent Orchestration" (`/learn/1-agentic-architecture/1-2-orchestration-patterns`)
- `examguide.pdf` — Task Statement 1.2: "Orchestrate multi-agent systems with coordinator-subagent patterns"

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
