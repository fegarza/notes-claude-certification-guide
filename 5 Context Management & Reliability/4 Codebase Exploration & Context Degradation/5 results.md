# Resultados — Tutor socrático: Codebase Exploration & Context Degradation

> [!info] Sesión evaluada el 2026-11-08
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 86/100**

Gran mejora respecto a la evaluación anterior (69/100). De 16 preguntas, 13 fueron correctas, 2 parcialmente correctas y 1 sin respuesta. Dominas el núcleo del tema: qué es la `context degradation`, por qué no es un problema de token limit, y cuál es el papel de cada mitigación (scratchpad, subagents, `/compact`, manifest). Las fallas estuvieron en detalles finos: los campos concretos del `manifest`, el segundo beneficio de la `summary injection` y nombrar explícitamente la pérdida de conocimiento al reiniciar sin guardar estado.

## Conceptos con buen dominio

- **Mecanismo de la `context degradation`**: identificaste que el output verboso acumulado desplaza el foco de atención y la información específica se pierde.
- **No es un problema de token limit (Trampa 1)**: rechazaste con claridad la idea de usar una ventana más grande.
- **`Scratchpad files`**: sabes que son inmunes porque viven fuera de la conversación, que deben usarse **desde el inicio** y que deben guardar información específica, no generalidades.
- **`Subagent delegation` y `context isolation` (Trampa 2)**: defendiste que el valor principal es el aislamiento de contexto, aunque los subagentes no corran en paralelo.
- **"Cold start" y `summary injection`**: entendiste que los subagentes no heredan contexto y necesitan que se les inyecte el resumen de la Fase 1.
- **`/compact` proactivo (Trampa 4)**: sabes que se usa antes de llegar al límite y que protege la calidad del contexto.
- **Mecanismo de `crash recovery`**: describiste completo el flujo: export a ubicación conocida, el coordinador carga el manifest on resume y lo inyecta en los agentes.
- **Scratchpad vs. manifest**: distinguiste bien el propósito y quién usa cada uno.
- **Precargar el codebase (Trampa 5)**: la reconociste como la Trampa 1 disfrazada.

## Áreas que necesitan revisión

### Contenido del `manifest`

- **Qué pasó:** no pudiste mencionar qué información debe incluir el manifest para cumplir sus cuatro objetivos al reanudar.
- **Contenido del resumen:**

| Qué incluye el `manifest` | Para qué |
|---|---|
| Qué se ha explorado (archivos leídos, búsquedas hechas) | No repetir exploración |
| Hallazgos clave hasta ahora | No perder conocimiento |
| Fase actual y siguientes pasos | Saber dónde retomar |
| Preguntas pendientes / temas sin resolver | No olvidar cabos sueltos |

> En el ejemplo de la guía, los campos son `sessionId`, `phase`, `exploredPaths`, `keyFindings` y `nextSteps`.

- **Sección:** [[1 resumen]] → "6. `Crash recovery` con `structured state manifests`"
- **Analogía para reforzarlo:** es como el marcador de un videojuego guardado: te dice en qué nivel vas (fase), qué zonas ya limpiaste (exploredPaths), qué objetos tienes en el inventario (keyFindings) y cuál es la siguiente misión (nextSteps). Si al guardar falta cualquiera de esas cuatro cosas, al cargar la partida algo se repite o se pierde.

### Segundo beneficio de la `summary injection`

- **Qué pasó:** respondiste que sin el resumen los subagentes "no estarían sincronizados". Es razonable, pero no es lo que destaca la guía: además de evitar duplicar trabajo, el resumen les da la comprensión arquitectónica para hacer las preguntas correctas.
- **Contenido del resumen:**
> - Evita el problema de **"cold start"**: subagentes de la Fase 2 que **duplican** la exploración de la Fase 1 porque nadie les pasó los hallazgos previos.
> - Además les da la **comprensión arquitectónica** necesaria para hacer las preguntas correctas.
- **Sección:** [[1 resumen]] → "4. `Summary injection` entre fases"
- **Analogía para reforzarlo:** a un médico especialista que recibe a un paciente referido no solo le ahorras repetir los análisis al pasarle el expediente. Sobre todo le dices *dónde buscar*: "ya descartamos esto, sospechamos del hígado". Sin el expediente, revisaría todo al azar.

### Reiniciar la sesión sin guardar estado (Trampa 3)

- **Qué pasó:** describiste bien qué hacer antes y después (scratchpad + inyectarlo), pero no nombraste el problema central: reiniciar **pierde todo el conocimiento acumulado** y obliga a re-explorar. Tampoco mencionaste los `state manifests` junto al scratchpad.
- **Contenido del resumen:**
> [!warning] Trampa 3 — Reiniciar la sesión para "curar" la degradación sin guardar estado
> **El error:** abrir una sesión nueva y limpia (y quizás pedir "explora más eficientemente esta vez"). **Por qué falla:** reiniciar **pierde todo el conocimiento acumulado** y obliga a re-explorar. **Lo correcto:** persistir hallazgos en `scratchpad files` y `state manifests` **antes** de reiniciar, y luego **inyectarlos** en la nueva sesión.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** es como formatear la computadora porque va lenta sin hacer respaldo: sí queda "limpia", pero perdiste todos tus archivos. Primero respaldas (scratchpad + manifest), luego formateas y restauras.

## Siguiente paso recomendado

Repasa la tabla del `manifest` (sus 4 componentes) y los dos beneficios de la `summary injection`. Con eso cubierto, ya estás listo para [[4 test]].
