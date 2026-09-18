> [!note] En una frase
> Dentro de un pipeline de CI/CD, Claude Code deja de ser un asistente interactivo con el que hablas en la terminal y pasa a ser un motor automatizado que corre sin teclado, sin dueño mirando la pantalla, y cuya salida debe poder leer una máquina — eso cambia casi todas las reglas de cómo lo invocas.

## Explícamelo como si tuviera 5 años

Imagina que normalmente hablas con Claude Code como si fuera un compañero de trabajo sentado a tu lado: le preguntas algo, él responde, tú le contestas de vuelta.

Ahora imagina que ese compañero tiene que trabajar solo, de noche, en una fábrica automática (el pipeline de CI), sin nadie ahí para contestarle si pregunta algo. Si intentas ponerlo a trabajar ahí "como si estuviera hablando contigo", se queda esperando una respuesta que nunca llega y se traba para siempre.

Para que funcione en la fábrica necesita: (1) que le digan "no esperes a que nadie te conteste, solo haz el trabajo y entrega el resultado" (`-p`), (2) que entregue el resultado en una forma que otra máquina pueda leer, no en párrafos bonitos (JSON), (3) que no cargue con los recuerdos de lo que hizo antes cuando alguien más viene a revisar su trabajo, y (4) que sepa qué reglas de la casa seguir (`CLAUDE.md`) igual que si estuviera contigo.

## Argumento central

> Claude Code puede actuar como motor de revisión y generación automatizado dentro de un pipeline de CI/CD, pero requiere una configuración distinta a la del uso interactivo: modo no interactivo, salida estructurada, aislamiento de contexto entre generación y revisión, y manejo explícito de hallazgos repetidos.

## Ideas clave (Conclusiones)

### 1. El flag `-p`: modo no interactivo
El modo por defecto espera input de teclado. Un pipeline de CI no tiene teclado, así que sin `-p` el job se queda colgado indefinidamente.

```bash
claude -p "Analiza este pull request en busca de problemas de seguridad"
```

`-p` (o `--print`) cambia Claude Code a modo "imprime y sal": procesa el prompt, escribe el resultado en stdout y termina. Según la guía, alternativas como `CLAUDE_HEADLESS=true` o `--batch` **no existen**, y redirigir stdin desde `/dev/null` no resuelve el problema real.

> [!warning] Dato más evaluado del Dominio 3
> La guía es explícita: `-p` es "el hecho más directamente evaluado del Dominio 3" (es la Pregunta 10 del set de muestra).

### 2. Salida estructurada para CI
En CI el resultado lo procesa una máquina, no una persona — por eso hace falta una salida parseable:

- `--output-format json` → envuelve el resultado en un JSON con `session_id`, metadata de costo, etc.
- `--json-schema` → valida la salida del agente contra un JSON Schema, en modo `-p`.

```bash
claude -p \
  --output-format json \
  --json-schema '{"type":"object","properties":{"findings":{...}}}' \
  "Revisa este PR en busca de problemas de seguridad"
```

El resultado validado queda dentro del campo `structured_output` del JSON de salida, y se extrae con `jq '.structured_output'`.

### 3. Aislamiento de contexto entre sesiones
Revisar el propio código en la misma sesión donde se generó es menos efectivo que una revisión independiente, porque la sesión conserva el razonamiento con el que se justificó cada decisión al generarlo — es más difícil que se cuestione algo que ya se justificó a sí mismo.

```bash
# Paso 1: generar código (sesión A)
claude -p "Implementa el middleware de autenticación"

# Paso 2: revisar código (sesión B — independiente, sin contexto compartido)
claude -p "Revisa el middleware de autenticación en busca de fallas de seguridad y de diseño"
```

### 4. Revisión incremental (evitar hallazgos duplicados)
Si cada corrida analiza el PR completo desde cero, sin saber qué se reportó antes, se generan comentarios duplicados en cada push — lo que erosiona la confianza del equipo en la herramienta.

**Solución:** incluir los hallazgos previos (`${PREVIOUS_FINDINGS}`) en el contexto, con instrucciones explícitas de reportar solo issues nuevos o los que sigan sin resolverse — sin repetir issues que el equipo decidió deliberadamente no atender.

### 5. `CLAUDE.md` como contexto para CI
`CLAUDE.md` se lee igual en CI que en modo interactivo. Ahí es donde el pipeline obtiene contexto propio del proyecto: estándares de testing, fixtures disponibles, criterios de revisión, cobertura esperada.

```markdown
## Testing standards (para CI)
- Usar los patrones factory para crear datos de prueba
- No testear detalles internos de implementación (private methods)
- Cobertura mínima objetivo: 80% branch coverage
- Fixtures disponibles: ver /test/fixtures/README.md
```

### 6. Referencia rápida de flags CLI

**Prompt de sistema (append vs. replace):**

| Flag | Efecto |
|---|---|
| `--system-prompt "<texto>"` | Reemplaza todo el prompt de sistema por defecto |
| `--system-prompt-file <ruta>` | Reemplaza el prompt por defecto con el contenido de un archivo |
| `--append-system-prompt "<texto>"` | Agrega texto al prompt por defecto (no lo reemplaza) |
| `--append-system-prompt-file <ruta>` | Agrega el contenido de un archivo al prompt por defecto |

> [!note]
> Usa "append" cuando quieres que Claude siga siendo un asistente de código normal, pero que además siga tus reglas extra. Usa "replace" cuando quieres control total sobre el comportamiento.

**Salida y límites en modo headless (`-p`):**

| Flag | Efecto |
|---|---|
| `--output-format text\|json\|stream-json` | Formato de salida |
| `--input-format text\|stream-json` | Formato de entrada |
| `--json-schema '<schema>'` | Salida validada contra un schema |
| `--max-turns <n>` | Límite de turnos agénticos antes de salir |

**Permisos, herramientas y contexto:**

| Flag | Efecto |
|---|---|
| `--permission-mode <modo>` | `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`, `manual` |
| `--allowedTools "<reglas>"` | Herramientas que corren sin pedir permiso |
| `--disallowedTools "<reglas>"` | Reglas de denegación; el nombre solo elimina la herramienta por completo |
| `--tools "Bash,Edit,Read"` | Restringe qué herramientas integradas están disponibles |
| `--add-dir <ruta>` | Agrega un directorio adicional de lectura/escritura |
| `--model <alias\|nombre>` | Define el modelo de la sesión |

**Sesión y arranque:**
- `-c` / `--continue` — retoma la conversación más reciente
- `-r` / `--resume <id|nombre>` — retoma una sesión específica
- `--bare` — modo mínimo, se salta el auto-descubrimiento (hooks, skills, plugins, servidores MCP, memoria, `CLAUDE.md`)

### 7. GitHub Actions como paso del workflow

> [!info] Más allá de la guía
> La guía menciona que Anthropic ofrece `anthropics/claude-code-action@v1`, construida sobre el Agent SDK, aceptando los mismos flags de CLI que una corrida headless con `claude -p`. El ejemplo completo de configuración está en [[2 example]].

- Sin el input `prompt`: la acción corre en modo interactivo, esperando frases disparadoras como `@claude` en comentarios.
- Con el input `prompt`: corre en modo automatizado sobre eventos del workflow.
- Los flags de CLI se pasan todos juntos como un solo string en `claude_args` (no hay inputs separados para system prompt, allowlist de tools, etc.).
- `actions/checkout` es el paso que pone el repositorio en el runner — sin él no hay nada en disco que Claude pueda leer. Las credenciales de API deben ir siempre como secretos del repositorio, nunca escritas literalmente en el YAML.

### 8. Sesiones en paralelo con Git Worktrees

> [!info] Más allá de la guía
> Verificado contra la documentación oficial de Claude Code. El ejemplo completo de comandos está en [[2 example]].

Un git worktree crea un segundo directorio de trabajo para el mismo repositorio, en una rama distinta, lo que permite correr varias instancias de Claude Code en paralelo sin que se pisen los archivos entre sí. Cada sesión obtiene su propio presupuesto completo de contexto y sus propios archivos. La coordinación se hace por secuencia: la primera sesión en terminar hace merge, la segunda hace rebase antes de seguir trabajando. Los subagentes personalizados pueden declarar `isolation: worktree` en su frontmatter para ejecutarse en paralelo sin colisiones.

## Evidencia de apoyo

### Tests existentes en el contexto
Al generar tests en CI, incluir los archivos de test ya existentes en el contexto evita que Claude sugiera tests que ya existen, y lo enfoca en encontrar huecos reales de cobertura.

### Batch API vs. tiempo real para workflows de CI
La Message Batches API da 50% de ahorro en costo, pero puede tardar hasta 24 horas en procesar, sin SLA de latencia garantizado.

| Tipo de workflow | API a usar | Por qué |
|---|---|---|
| Checks bloqueantes antes de merge | Tiempo real (síncrona) | El desarrollador está esperando el resultado |
| Reporte nocturno de deuda técnica | Batch API | No es urgente, ahorra 50% |
| Auditoría semanal de código | Batch API | Programada, tolera latencia |
| Generación de tests nocturna | Batch API | Corre de noche, se revisa al día siguiente |

## Trampas de examen

> [!warning] Trampa — Usar la Batch API para checks de pre-merge bloqueantes
> Puede tardar horas en responder cuando el desarrollador necesita el resultado ya. La guía indica que esta distinción se evalúa directamente (Pregunta de muestra 11).

> [!warning] Trampa — Pensar que `CLAUDE_HEADLESS=true`, `--batch` o redirigir stdin arreglan el colgado del pipeline
> Ninguno existe o resuelve el problema real; la única solución es el flag `-p`.

> [!warning] Trampa — Asumir que revisar en la misma sesión donde se generó el código es tan bueno como una revisión independiente
> La sesión conserva el razonamiento usado para generar el código, sesgando la revisión.

> [!warning] Trampa — No incluir los hallazgos de revisiones anteriores
> Genera comentarios duplicados en cada corrida y erosiona la confianza del equipo en la herramienta.

## Fuentes citadas por la guía
- Claude Code CLI Reference — Anthropic
- Claude Code GitHub Actions — Anthropic
- Claude Code: Run parallel sessions with worktrees — Anthropic
- Claude Certified Architect Foundations Exam Guide — Task Statement 3.6 — Anthropic
- Claude Certified Architect Foundations Exam Guide — Sample Questions 10 y 11 — Anthropic
- Anthropic Message Batches API Documentation — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
