# Resultados — Tutor socrático: Custom Slash Commands and Skills

> [!info] Sesión evaluada el 2026-09-26
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 78/100**

De 14 preguntas: 9 correctas, 4 parcialmente correctas y 1 incorrecta. Dominas bien la distinción central (skill bajo demanda vs. `CLAUDE.md` siempre cargado), los niveles de alcance proyecto/usuario y el propósito de las tres opciones de frontmatter. Las respuestas parciales comparten un patrón: el diagnóstico principal es correcto, pero falta la segunda mitad de lo que se pedía (la ruta alternativa válida, la equivalencia funcional de ambas estructuras, la consecuencia concreta de no usar `context: fork`). El único error conceptual fue la trampa del skill "siempre activo".

## Conceptos con buen dominio

- **Skills vs. `CLAUDE.md`** — Aplicaste bien el criterio "¿siempre o solo cuando se invoca?" en ambas direcciones: la convención universal va en `CLAUDE.md` y el procedimiento de rollback va en un skill.
- **Estructura canónica y precedencia** — Identificaste `.claude/skills/<nombre>/SKILL.md` como la canónica, su descubrimiento automático por intención, y que gana ante conflictos de nombre.
- **Alcance proyecto vs. usuario** — Explicaste por qué un skill en `~/.claude/` no llega al equipo y cómo crear una variante personal con un nombre distinto.
- **`context: fork`** — Lo aplicaste correctamente para aislar la salida verbosa de un análisis de codebase.
- **`allowed-tools` como restricción** — Diste la respuesta que espera el examen: limita el acceso para prevenir acciones destructivas.
- **`argument-hint`** — Sabes que actúa en el autocompletado para guiar los inputs esperados.
- **Preferencias personales siempre activas** — Las ubicaste en el `CLAUDE.md` de usuario, no en un skill.

## Áreas que necesitan revisión

### Trampa 3 — Tratar un skill como guía "siempre activa"

- **Qué pasó:** Ante reglas de seguridad puestas en un skill que "a veces" se ignoraban, atribuiste el problema a una descripción poco clara del skill, cuando el problema es el mecanismo elegido.
- **Contenido del resumen:**
  > [!warning] Trampa 3 — Tratar un skill como si fuera guía "siempre activa"
  > Un skill se activa **solo cuando se invoca** (por comando o por coincidencia de intención con descubrimiento automático), nunca de forma persistente como `CLAUDE.md`. Si una convención debe aplicar sin excepción en cada interacción, un skill es la herramienta equivocada.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** Un skill es como un extintor: está en la pared y lo usas cuando hay fuego, por mejor etiquetado que esté. Si quieres que el edificio nunca se incendie, necesitas el reglamento de construcción (`CLAUDE.md`), no un extintor con una etiqueta más clara.

### Las dos estructuras producen el mismo comando

- **Qué pasó:** Al comparar `.claude/commands/release.md` con `.claude/skills/release/SKILL.md`, te enfocaste en las diferencias y no mencionaste que ambos generan un `/release` idéntico en funcionamiento ni que `commands/` existe por compatibilidad hacia atrás.
- **Contenido del resumen:**
  > - **`.claude/skills/<nombre>/SKILL.md`** — estructura **canónica**, basada en directorio.
  > - **`.claude/commands/<nombre>.md`** — archivo **plano**, mantenido por compatibilidad hacia atrás.
  > - Ambas producen un comando `/<nombre>` idéntico en funcionamiento.
- **Sección:** [[1 resumen]] → "1. Sistema unificado de skills: dos estructuras, un mismo resultado"
- **Analogía para reforzarlo:** Es como pagar con tarjeta física o con el celular: el cobro es el mismo. El celular (skills) es el método moderno y trae extras, pero la tarjeta (commands) sigue aceptándose para no dejar fuera a nadie.

### Trampa 1 — La segunda forma válida de arreglar un `.md` suelto en `.claude/skills/`

- **Qué pasó:** Diagnosticaste bien que falta la carpeta contenedora, pero no mencionaste la alternativa de usar un archivo plano en `.claude/commands/`.
- **Contenido del resumen:**
  > [!warning] Trampa 1 — Poner un archivo Markdown plano directamente dentro de `.claude/skills/`
  > Un skill requiere **estructura de directorio**: `.claude/skills/<nombre>/SKILL.md`. Poner un `.md` suelto directamente en `.claude/skills/` (sin la carpeta contenedora) no crea un skill válido. Si quieres un archivo plano sin subcarpeta, esa es la función de `.claude/commands/<nombre>.md`, no de la ruta de skills.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** Una carta tiene dos formas válidas de enviarse: dentro de un sobre en el buzón de sobres (skills con carpeta) o como postal en el buzón de postales (commands plano). Una hoja suelta en el buzón de sobres no llega.

### `allowed-tools`: pre-aprobación y restricción son la misma moneda

- **Qué pasó:** Defendiste correctamente la lectura de restricción, pero no reconociste que la visión de "pre-aprobar sin pedir permiso" también es válida y que ambas describen el mismo mecanismo.
- **Contenido del resumen:**
  > [!note] `allowed-tools`: pre-aprobación y restricción son la misma moneda
  > No lo pienses como "dos comportamientos distintos". Al listar explícitamente qué herramientas puede usar un skill, automáticamente excluyes todas las demás — por eso el examen enmarca `allowed-tools` como una forma de **restringir** el acceso, no solo de agilizar permisos.
- **Sección:** [[1 resumen]] → "3. Opciones críticas de frontmatter"
- **Analogía para reforzarlo:** Una lista de invitados en la puerta de una fiesta hace dos cosas a la vez: quien está en la lista entra sin trámite (pre-aprobación) y quien no está se queda afuera (restricción). Es una sola lista con dos efectos.

### Consecuencia concreta de omitir `context: fork` (Trampa 4)

- **Qué pasó:** En el caso del brainstorming identificaste que todo queda en el contexto principal, pero no nombraste la consecuencia concreta: el gasto del presupuesto de tokens con contenido exploratorio que ya no se necesita.
- **Contenido del resumen:**
  > [!warning] Trampa 4 — Omitir `context: fork` en operaciones verbosas
  > Un skill que produce salida extensa (análisis de codebase, brainstorming con múltiples alternativas) sin `context: fork` contamina el presupuesto de tokens de la conversación principal con contenido exploratorio o intermedio que no necesita quedar ahí. La corrección es aislar esa salida en un sub-agente con `context: fork`.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** Es como hacer las cuentas de borrador en la misma hoja del examen final: la respuesta está ahí, pero te quedas sin espacio para lo que sigue. `context: fork` es usar una hoja de borrador aparte y pasar solo el resultado.

## Siguiente paso recomendado

Relee "Trampas de examen" (sobre todo la Trampa 3) y la Conclusión 1 del resumen, practicando responder cada pregunta completa, no solo su primera mitad. Con eso estás listo para intentar [[4 test]].
