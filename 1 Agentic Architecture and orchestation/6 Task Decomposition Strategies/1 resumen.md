> [!note] En una frase
> Elegir entre pipeline fijo y decomposición dinámica es una pregunta mecánica ("¿conozco los pasos de antemano?"), no una cuestión de "qué tan importante es la tarea" — y si un agente pierde profundidad al procesar muchos ítems de golpe (atención diluida), la solución nunca es un modelo más grande o un mejor prompt, sino rediseñar la arquitectura en pasadas separadas.

## Explícamelo como si tuviera 5 años

Imagina dos formas de hacer una tarea larga. La primera es como seguir una **receta de cocina**: ya sabes que primero picas la cebolla, luego la sofríes, luego agregas el arroz — el orden está escrito antes de empezar y no cambia aunque la cebolla huela raro. La segunda es como ser un **detective**: llegas a la escena sin saber qué vas a encontrar, sigues la primera pista, y esa pista te dice a dónde ir después — el plan se escribe *mientras* investigas, no antes.

Ningún método es "mejor" en abstracto: usar la receta para investigar un crimen es absurdo (no sabes los pasos de antemano), y usar el método del detective para cocinar arroz es un desperdicio (ya sabes exactamente qué hacer). La pregunta que decide cuál usar es siempre la misma: **¿conozco los pasos desde el principio, o los voy a descubrir sobre la marcha?**

Ahora imagina un segundo problema, independiente del primero: un maestro que corrige 30 exámenes seguidos en una sola tarde. Los primeros cinco los corrige con calma, comentando cada error. Para el examen 25 ya está agotado y solo pone una nota sin mirar bien — y puede que apruebe en un examen el mismo error que reprobó en otro, porque ya no está comparando, solo está sobreviviendo. Eso no se arregla dándole al maestro más café (un "modelo más grande") ni pidiéndole que "por favor sea consistente" (un "mejor prompt") — se arregla dividiendo el trabajo: corregir cada examen en su propia sesión con atención completa, y al final hacer una revisión aparte comparando los 30 entre sí.

## Argumento central

> **Selección de patrón**: la decisión entre pipeline secuencial fijo (*prompt chaining*) y decomposición adaptativa dinámica no depende de qué tan compleja "se siente" la tarea, sino de un hecho concreto: **¿los pasos se conocen de antemano o emergen durante la investigación?** Tareas estructuradas y predecibles usan pipeline fijo; tareas abiertas de alcance desconocido usan decomposición dinámica.
>
> **Dilución de atención**: cuando un agente procesa demasiados ítems en una sola pasada, la profundidad del análisis se vuelve inconsistente — no porque el modelo sea incapaz, sino porque es un **problema arquitectónico de presupuesto de atención**. La solución es siempre estructural (arquitectura multi-pass), nunca un modelo más potente ni un prompt más insistente.

## Ideas clave (Conclusiones)

### 1. Dos patrones de decomposición, dos definiciones exactas

- **Fixed Sequential Pipelines** (también llamado *prompt chaining*): "el workflow se define de antemano. El Paso 1 corre, su salida alimenta al Paso 2, la salida del Paso 2 alimenta al Paso 3, y así sucesivamente. La secuencia no cambia según los resultados intermedios."
- **Dynamic Adaptive Decomposition**: "el agente empieza con un objetivo de alto nivel, realiza una investigación inicial, y genera un plan basado en lo que encuentra. Mientras ejecuta el plan, descubre información nueva que puede cambiar los pasos restantes."

> [!note] Analogía rápida
> Pipeline fijo = receta de cocina. Decomposición dinámica = investigación de detective. La diferencia no es cuál requiere más "inteligencia" — es si el mapa existe antes de caminar o se dibuja mientras se camina.

### 2. El framework de decisión: una tabla, un criterio

| Características de la tarea | Patrón | Razonamiento |
|---|---|---|
| Pasos conocidos de antemano, entrada estructurada | Pipeline fijo | "La consistencia y confiabilidad pesan más que la adaptabilidad" |
| Alcance abierto, desconocido | Decomposición dinámica | "La adaptabilidad es esencial cuando el problema no está completamente definido" |
| Revisión de código multi-archivo | Pipeline fijo | Análisis por archivo + pasada de integración cruzada es predecible |
| Exploración de un codebase legacy | Decomposición dinámica | Las dependencias y problemas emergen durante la investigación |
| Extracción de datos de documentos | Pipeline fijo | Los campos y el formato están predeterminados |
| Depurar un sistema desconocido | Decomposición dinámica | La causa raíz es desconocida; la investigación debe adaptarse |

- **Skill concreta para decomposición dinámica**: ante una tarea abierta como "agregar tests exhaustivos a un codebase legacy", el patrón correcto es primero **mapear la estructura**, luego **identificar las áreas de mayor impacto**, y recién ahí **crear un plan priorizado que se adapta** a medida que se descubren dependencias — no intentar escribir el plan completo desde el día uno.

> [!warning] Por qué la tabla no es "memoriza estos 6 ejemplos"
> El examen no va a repetir literalmente "revisión de código" o "codebase legacy" — va a describir un escenario nuevo y esperar que apliques el mismo criterio: ¿los pasos se conocen de antemano o no? Memorizar el criterio, no la lista de ejemplos.

### 3. Dilución de atención: qué es y cómo se ve

> [!warning] Definición exacta
> "La dilución de atención es un modo de fallo específico que ocurre cuando un agente procesa demasiados ítems en una sola pasada. El resultado es una profundidad inconsistente — el agente produce un análisis exhaustivo para algunos ítems y pasa por alto problemas obvios en otros."

Síntomas concretos (los tres son la misma causa raíz, no problemas separados):

1. Feedback detallado para los primeros archivos, análisis cada vez más superficial para los últimos.
2. Un patrón marcado como problemático en un archivo, mientras el mismo código idéntico se aprueba en otro archivo del mismo lote.
3. Bugs obvios pasados por alto en algunos ítems, mientras se señalan problemas menores de estilo en otros.

> [!note] Caso guía: revisión de 14 archivos
> Un PR modifica 14 archivos. Una revisión de una sola pasada sobre todos juntos produce feedback detallado para los primeros 5 archivos pero pasa por alto bugs obvios en los archivos 10-14, además de marcar un `forEach` como ineficiente en un archivo mientras aprueba código idéntico en otro. Este es el caso de referencia que el examen usa para evaluar el tema — ver [[4 test]].

### 4. La solución: arquitectura multi-pass (no modelo, no prompt)

> [!warning] El punto más importante del tema
> "Dividir el trabajo en dos capas: **pasadas de análisis local por ítem**: analizar cada archivo (o documento, o módulo) individualmente en su propia pasada, con todo el presupuesto de atención enfocado en un solo ítem. **Pasada de integración cruzada**: después de que todas las pasadas locales terminan, correr una pasada separada que busca preocupaciones transversales entre todos los ítems."

- Esto **no es** "usar un modelo más potente" — la guía es explícita: es un problema estructural, no de capacidad del modelo. Un modelo mejor sigue sufriendo dilución si el diseño sigue siendo una sola pasada sobre demasiados ítems.
- Esto **no es** "escribir un mejor prompt" — un prompt más enfático mejora la calidad promedio, pero no resuelve el problema fundamental de asignación de atención: sigue siendo una sola pasada con presupuesto fijo repartido entre demasiados ítems.
- Agrupar en lotes (*batching*) reduce la dilución **dentro** de cada lote, pero por sí solo no es la solución completa: si no se agrega una pasada de integración cruzada, se pierden los problemas que cruzan los límites del lote.

## Trampas de examen

> [!warning] Trampa 1 — "Sube de modelo" o "usa una ventana de contexto más grande" como arreglo
> La dilución de atención es un problema arquitectónico, no de capacidad del modelo. Un modelo más grande o más caro no cambia que el diseño sigue siendo una sola pasada sobre demasiados ítems.

> [!warning] Trampa 2 — "Mejora el prompt" como equivalente a arquitectura multi-pass
> Un prompt mejor sube la calidad promedio, pero no resuelve el problema fundamental de asignación de atención. Multi-pass es un cambio de arquitectura, no de redacción.

> [!warning] Trampa 3 — Aplicar pipeline fijo a una tarea de investigación abierta
> Si el alcance no se conoce de antemano (exploración de codebase legacy, debugging de un sistema desconocido), un pipeline fijo no puede adaptarse a lo que se descubre. Esas tareas requieren decomposición dinámica.

> [!warning] Trampa 4 — Agrupar en lotes sin agregar una pasada de integración cruzada
> Dividir 14 archivos en lotes de 5 reduce la dilución dentro de cada lote, pero sin una pasada de integración cruzada separada se siguen perdiendo los problemas que cruzan los límites entre lotes — no es equivalente a la arquitectura multi-pass completa.

## Fuentes citadas por la guía
- Claude Certification Guide — módulo "Task Decomposition Strategies" (`/learn/1-agentic-architecture/1-6-task-decomposition`)
- `examguide.pdf` — Task Statement 1.6: "Design task decomposition strategies for complex workflows"
- `examguide.pdf` — Sección 9, Sample Question 12 (revisión de PR de 14 archivos)
- Claude Agent SDK documentation — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
