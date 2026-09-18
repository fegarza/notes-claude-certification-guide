Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Modo no interactivo (`-p`)

> [!question]- ¿Por qué un pipeline de CI se cuelga sin el flag `-p`?
> Porque el modo por defecto espera input de teclado, y un pipeline de CI no tiene a nadie escribiendo.

> [!question]- ¿Qué hace exactamente el flag `-p` / `--print`?
> Procesa el prompt, escribe el resultado en stdout y termina — modo "imprime y sal".

> [!question]- ¿Cuáles son las 3 alternativas incorrectas que el examen suele presentar para este problema?
> `CLAUDE_HEADLESS=true`, `--batch`, y redirigir stdin desde `/dev/null`.

## Salida estructurada

> [!question]- ¿Por qué en CI se necesita salida en JSON y no texto libre?
> Porque quien procesa el resultado es un sistema automatizado, no una persona, y necesita una estructura parseable.

> [!question]- ¿Qué hace `--json-schema` en modo `-p`?
> Valida la salida del agente contra un JSON Schema definido.

> [!question]- ¿En qué campo del JSON de salida queda el resultado validado?
> En `structured_output`.

## Aislamiento de sesiones

> [!question]- ¿Por qué revisar código en la misma sesión donde se generó es menos confiable?
> Porque la sesión conserva el razonamiento con el que se justificó cada decisión al generarlo.

> [!question]- ¿Cómo se logra una revisión verdaderamente independiente?
> Usando una invocación separada de `claude -p`, sin historial compartido con la sesión de generación.

## Revisión incremental

> [!question]- ¿Qué problema genera analizar el PR completo desde cero en cada corrida?
> Comentarios duplicados en cada push, que erosionan la confianza del equipo en la herramienta.

> [!question]- ¿Cuál es la solución a los hallazgos duplicados?
> Incluir los hallazgos previos en el contexto, e instruir a reportar solo issues nuevos o no resueltos.

## `CLAUDE.md` en CI

> [!question]- ¿`CLAUDE.md` se lee distinto en CI que en modo interactivo?
> No, se lee exactamente igual en ambos modos.

> [!question]- ¿Qué tipo de contexto de proyecto aporta `CLAUDE.md` a una revisión automatizada?
> Estándares de testing, fixtures disponibles, criterios de revisión y cobertura esperada.

## Flags de CLI

> [!question]- ¿Cuál es la diferencia entre `--system-prompt` y `--append-system-prompt`?
> `--system-prompt` reemplaza todo el prompt por defecto; `--append-system-prompt` le agrega texto sin quitarlo.

> [!question]- ¿Cuándo conviene usar "append" en vez de "replace"?
> Cuando Claude debe seguir siendo un asistente de código normal, pero además seguir reglas adicionales.

> [!question]- ¿Qué hace `--bare`?
> Corre en modo mínimo, saltándose el auto-descubrimiento de hooks, skills, plugins, servidores MCP, memoria y `CLAUDE.md`.

## GitHub Actions y Git Worktrees

> [!question]- ¿Sobre qué está construida la acción `anthropics/claude-code-action@v1`?
> Sobre el Agent SDK, aceptando los mismos flags de CLI que `claude -p`.

> [!question]- ¿Qué determina si la acción corre en modo interactivo o automatizado?
> La presencia o ausencia del input `prompt`.

> [!question]- ¿Qué pasa si un workflow de GitHub Actions omite el paso `actions/checkout`?
> No hay nada en disco que Claude pueda leer, porque el repositorio nunca se puso en el runner.

> [!question]- ¿Qué permite hacer un git worktree que no se puede hacer con un solo directorio de trabajo?
> Correr varias sesiones de Claude Code en paralelo, cada una en su propio directorio y rama, sin que se pisen los archivos.

## Batch API vs. tiempo real

> [!question]- ¿Qué ventaja y qué desventaja tiene la Message Batches API frente a la API en tiempo real?
> Ventaja: 50% de ahorro en costo. Desventaja: hasta 24 horas de procesamiento, sin SLA de latencia garantizado.

> [!question]- ¿Para qué tipo de workflow de CI es correcto usar la Batch API?
> Para trabajo no bloqueante y no urgente — reportes nocturnos, auditorías semanales, generación de tests que se revisan al día siguiente.

> [!question]- ¿Por qué no se debe usar la Batch API para checks de pre-merge?
> Porque son bloqueantes: el desarrollador espera el resultado en el momento, y una demora de horas rompe el flujo de trabajo.
