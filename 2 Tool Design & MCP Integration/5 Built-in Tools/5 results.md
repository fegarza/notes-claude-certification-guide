# Resultados — Tutor socrático: Built-in Tools

> [!info] Sesión evaluada el 2026-10-19
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 86/100**

De 14 preguntas: 10 correctas, 2 casi completas y 2 parcialmente correctas, sin respuestas incorrectas. La distinción Grep vs. Glob, la mecánica de Edit (unicidad, `replace_all`, fallback del examen vs. trabajo real) y la exploración incremental están bien dominadas. Los fallos aparecieron al justificar *por qué* una secuencia es incorrecta: se razonó con "desperdicio de tokens" donde la razón principal era otra (Edit como opción por defecto; tests indirectos que se pierden). También faltó detallar qué encuentra cada paso del rastreo.

## Conceptos con buen dominio

- **Grep busca contenido, Glob busca rutas.** Identificó la herramienta correcta en todos los escenarios, incluida la trampa de "Grep funciona por suerte para encontrar tests".
- **Edit falla con coincidencias no únicas.** Lo entiende como mecanismo de seguridad y no como bug.
- **Fallback examen vs. trabajo real.** Distingue Read + Write (respuesta del examen) de ampliar `old_string` (práctica actual), y eligió B en la pregunta trampa.
- **`replace_all: true`.** Sabe cuándo usarlo y en qué se diferencia de ampliar el ancla.
- **Exploración incremental.** Grep primero con una pista, Read para seguir imports, y cada lectura justificada.
- **Re-Grep por wrappers.** Entiende que el primer Grep no puede conocer nombres que todavía no se descubren.
- **Glob para tests hermanos.** Sabe que Glob encuentra tests indirectos por convención de nombre, aunque no mencionen la función.

## Áreas que necesitan revisión

### Edit como opción por defecto para modificaciones (Trampa 4)

- **Qué pasó:** ante 15 cambios pequeños hechos con Read + Write, detectó bien el desperdicio. Pero propuso como solución una estrategia de exploración (Grep → Read → Write) en lugar de cambiar la herramienta de modificación a **Edit**.
- **Contenido del resumen:**
  > [!warning] Trampa 4 — Usar Read + Write como respuesta por defecto para cada modificación
  > Edit es más rápido y usa menos contexto porque solo toca el texto específico señalado; Read + Write carga el archivo completo. **Por qué es un error:** el examen penaliza tratar Read + Write como la opción estándar. La forma correcta es intentar Edit primero siempre; Read + Write es el fallback para cuando Edit no encuentra un ancla única, no la respuesta por defecto.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** para corregir una errata en un libro usas un corrector líquido sobre esa palabra. No vuelves a imprimir el libro entero. Reimprimir es solo para cuando no encuentras la palabra exacta que quieres tachar.

### Por qué "Glob primero" es incorrecto en el escenario de deprecación

- **Qué pasó:** justificó el error de "Glob de todos los tests → Grep dentro de ellos" como desperdicio de tokens. La razón de fondo es que esa secuencia **se pierde los tests indirectos**, que nunca mencionan la función por nombre.
- **Contenido del resumen:**
  > 2. **Glob** por los archivos de test hermanos — encuentra el archivo de test que corresponde a cada archivo llamador por convención de nombre (ej. `OrderProcessor.ts` → `OrderProcessor.test.tsx`), incluso cuando el test ejercita la función indirectamente a través del módulo fuente (coincidencia de ruta).

  > [!warning] No es "Glob primero"
  > El patrón es Grep, luego Glob, luego Grep otra vez — búsqueda de contenido para referencias directas, coincidencia de ruta para tests adyacentes, búsqueda de contenido otra vez para cobertura indirecta. **No** es empezar por Glob.
- **Sección:** [[1 resumen]] → "5. El escenario de deprecación: el patrón Grep → Glob → Grep otra vez"
- **Analogía para reforzarlo:** buscas a todos los que conocen a Ana. Si revisas la lista de invitados buscando "Ana", solo encuentras a quien la anotó por nombre. Primero preguntas quiénes son sus amigos directos (Grep). Luego buscas a sus compañeros de cuarto por la dirección (Glob), aunque nunca la hayan nombrado.

### Qué encuentra (y qué no) cada Grep en el rastreo por wrappers y barriles

- **Qué pasó:** acertó en que hace falta un segundo Grep, pero no detalló qué encuentra el primero (definición, línea del barril y los 2 consumidores directos). En el caso del wrapper tampoco dijo qué atrapa el tercer paso (tests que cubren la función transitivamente).
- **Contenido del resumen:**
  > `processOrder` se define en `orders.ts`. El barril `utils/index.ts` la re-exporta como `submitOrder`, y tres de sus cinco consumidores importan `submitOrder` desde `utils`. Un Grep de `processOrder` encuentra la definición, la línea del barril y los dos consumidores que importan el nombre original — pero **no** puede encontrar a los otros tres, porque el string `processOrder` nunca aparece en sus archivos.

  > 3. **Grep otra vez** por nombres de wrapper — cuando un archivo llamador expone la función a través de un wrapper (ej. `applyLegacyOrder` que internamente llama a `processLegacyOrder`), Grep por el nombre del wrapper encuentra tests que cubren la función transitivamente a través de él.
- **Sección:** [[1 resumen]] → "4. Trazar el uso de una función a través de módulos wrapper y archivos barril" y "5. El escenario de deprecación: el patrón Grep → Glob → Grep otra vez"
- **Analogía para reforzarlo:** buscas en la agenda a todos los que llaman a "Roberto". Encuentras a Roberto, la nota que dice "Roberto = Beto" y a quienes le dicen Roberto. Los que solo le dicen "Beto" no aparecen hasta que buscas "Beto".

## Siguiente paso recomendado

Repasa las Trampas 4 y la sección 5 del resumen, y practica justificar cada paso diciendo **qué resultado concreto aporta**, no solo que ahorra tokens. Con eso ya estás listo para [[4 test]].
