# Resultados — Tutor socrático: CI-CD Integration

> [!info] Sesión evaluada el 2026-11-03
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 91/100**

De 21 preguntas, 20 fueron correctas y 1 parcialmente correcta. No hubo ninguna incorrecta. Es una mejora clara frente a la sesión anterior (70/100): ahora nombras con exactitud flags e inputs que antes fallabas (`--output-format`, `claude_args`, `actions/checkout`, `isolation: worktree`) y resolviste bien las dos trampas de mayor peso (Sample Questions 10 y 11). Lo que baja el puntaje son detalles de precisión:

- La descripción incompleta de lo que obtiene cada sesión con un worktree.
- Algunas respuestas correctas sin el porqué, como las falsas alternativas a `-p`.
- Matices de redacción menores: el "solo string" de `claude_args` y "archivos" en vez de "ruta" para los tests existentes.

## Conceptos con buen dominio

- **`-p` / `--print`:** identificaste de inmediato por qué el job se cuelga en modo interactivo y cómo arreglarlo.
- **`--output-format json` frente a `--json-schema`:** distinguiste que el primero solo da formato y el segundo valida la estructura.
- **`structured_output`:** sabes que el resultado validado vive en ese campo, no en la raíz del JSON.
- **Aislamiento de contexto:** explicaste que la sesión que generó el código conserva su razonamiento y sesga la revisión.
- **Revisión incremental:** respuesta completa: `${PREVIOUS_FINDINGS}`, reportar solo lo nuevo o lo que sigue sin resolver, y no repetir lo que el equipo decidió no atender.
- **`CLAUDE.md` en CI:** sabes que se carga igual que en modo interactivo y que ahí van los estándares del proyecto.
- **Append frente a replace:** detectaste que `--system-prompt` sobrescribe y elegiste `--append-system-prompt[-file]`.
- **`--bare`:** relacionaste que se salta la carga de las preferencias del proyecto (`CLAUDE.md`).
- **`--allowedTools` frente a `--disallowedTools`/`--tools`:** sabes que preaprobar no equivale a restringir.
- **GitHub Action:** dominaste el input `prompt`, `claude_args`, `actions/checkout` y el uso de secretos para la API key.
- **Coordinación de worktrees:** sabes que el primero hace merge y el segundo hace rebase. También ubicaste `isolation: worktree` en el frontmatter del subagente.
- **Batch API frente a tiempo real (Sample Question 11):** sabes que los checks bloqueantes van síncronos y lo que tolera latencia va a Batch.
- **`--max-turns`:** sabes que limita los turnos agénticos.
- **`-c` frente a `-r`:** distinguiste la sesión más reciente de una sesión específica por `session_id`.

## Áreas que necesitan revisión

### Qué obtiene cada sesión con un git worktree

- **Qué pasó:** identificaste bien el mecanismo (un directorio de trabajo distinto), pero omitiste que cada worktree está en una **rama distinta** y que cada sesión tiene **su propio presupuesto completo de contexto**.
- **Contenido del resumen:**
  > Un git worktree crea un segundo directorio de trabajo para el mismo repositorio, en una rama distinta, lo que permite correr varias instancias de Claude Code en paralelo sin que se pisen los archivos entre sí. Cada sesión obtiene su propio presupuesto completo de contexto y sus propios archivos.
- **Sección:** [[1 resumen]] → "8. Sesiones en paralelo con Git Worktrees"
- **Analogía para reforzarlo:** son dos estudiantes haciendo el mismo proyecto escolar en mesas separadas. Cada uno tiene su propio cuaderno (su rama) y su propia cabeza descansada (su propio presupuesto de contexto); no comparten hojas ni se cansan el uno al otro.

### El porqué de las falsas alternativas a `-p`

- **Qué pasó:** descartaste correctamente las tres opciones, pero no explicaste la razón de cada una. En el examen, saber el porqué te ayuda a descartar variantes nuevas de la misma trampa.
- **Contenido del resumen:**
  > Según la guía, alternativas como `CLAUDE_HEADLESS=true` o `--batch` **no existen**, y redirigir stdin desde `/dev/null` no resuelve el problema real.
- **Sección:** [[1 resumen]] → "1. El flag `-p`: modo no interactivo"
- **Analogía para reforzarlo:** es como intentar apagar la alarma de un coche con un botón que el coche no tiene (`CLAUDE_HEADLESS`, `--batch`), o tapándote los oídos (`/dev/null`). La alarma sigue sonando; solo la llave correcta (`-p`) la apaga.

## Siguiente paso recomendado

Ya estás listo para [[4 test]]. Antes, dale una lectura rápida a la sección 8 (worktrees). Quedaron sin evaluar en esta sesión los valores de `--permission-mode`, `--add-dir`, `--model` y `--input-format`/`stream-json` de la referencia de flags.
