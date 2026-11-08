# Progreso: CI/CD Integration (Task Statement 3.6)

- **Objetivo**: examen CCAR-F — Dominio 3, Task Statement 3.6
- **Nivel inicial**: sesión socrática previa (2026-09-27) 70/100. Diagnóstico de hoy: `--output-format json` OK; `actions/checkout` OK; secretos → propuso `${VAR:-default}` (incorrecto, es `${{ secrets.X }}`); no identificó `claude_args` ni tras pista; worktrees + merge OK pero dijo "solucionar conflictos" en vez de rebase.
- **Inicio**: 2026-11-03 · **Última sesión**: 2026-11-03
- **Fase actual**: 3 Repaso (conceptos 4 y 8)

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | `-p` / `--print` y sus falsas alternativas (`CLAUDE_HEADLESS`, `--batch`, stdin) | dominado | – | Dominado en sesión previa |
| 2 | Salida estructurada: `--output-format json`, `--json-schema`, `structured_output` | dominado | 2 | Falló nombre en sesión previa (`--output-type`); acertó en diagnóstico |
| 3 | Aislamiento de contexto: generar vs. revisar en sesiones separadas | dominado | – | Dominado en sesión previa |
| 4 | Revisión incremental: hallazgos previos + instrucción explícita de solo nuevos/no resueltos | en curso | 1 | Faltó la instrucción explícita en sesión previa. Hoy: diagnóstico OK; prompt omitía "no resueltos" — corregido con pistas (no cuenta). Ángulo usado: médico con expediente |
| 5 | Contexto para CI: `CLAUDE.md` + tests existentes en el contexto | dominado | 2 | Falló tests existentes en sesión previa. Hoy pidió aclaración; ángulo: tabla reglas vs. archivos + despensa/recetario |
| 6 | System prompt append vs. replace y `--bare` | dominado | 2 | Falló hoy: creyó que --bare vacía todo. Ángulo: oficina sin mochila vs. cambiar empleado. 2do fallo: dijo que con --bare "solo sabe lo de factories"; luego dijo que --system-prompt "agrega". Ángulo 2: celular en modo seguro + tabla de combinaciones |
| 7 | Permisos y herramientas: `--tools`, `--allowedTools`, `--disallowedTools`, `--permission-mode`, `--max-turns` | dominado | 2 | Falló `--disallowedTools` con nombre solo (previa). Hoy: `npm(test)`, luego `--tools "bash"` innecesario + minúscula. Ángulo: edificio/puertas/guardias, Bash = puerta y comandos = personas |
| 8 | GitHub Action: `prompt` vs. sin prompt, `claude_args`, `actions/checkout`, secretos | en curso | 0 | Falló `claude_args` y secretos (previa y hoy). Resumen: metió -p en la action y secretos como "variables de entorno" (3ra vez); luego `${{SECRET_NAME}}` sin `secrets.`. Ángulo: YAML = hoja pública |
| 9 | Worktrees: paralelo, contexto propio, merge/rebase, `isolation: worktree` | dominado | 2 | Previa: propuso fork. Hoy: OK salvo término rebase. Hoy: fork vs worktree con pista (comparten .git) |
| 10 | Batch API vs. tiempo real en CI | dominado | – | Dominado en sesión previa |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-11-03 | 8 Secretos | API key vía `${VAR:-default}` (sintaxis shell) | `${{ secrets.ANTHROPIC_API_KEY }}` — secreto del repo, nunca literal ni default en YAML | Sí |
| 2026-11-03 | 8 `claude_args` | Flags "se mandan junto con -p y --output-format" (sin nombrar input) | Todos los flags en un solo string en `claude_args`; con `prompt` no se pone `-p` | Sí |
| 2026-11-03 | 2 `structured_output` | `jq '.findings'` en la raíz; `--json-schema` recibe ruta de archivo | Resultado validado vive en `.structured_output` (raíz = sobre con session_id/costo); schema se pasa como string JSON | Sí |
| 2026-11-03 | 4 Revisión incremental | Prompt "solo reporta lo que no esté en los anteriores" | Tres casos: nuevo → reportar; previo no resuelto → reportar; won't fix → no repetir. Solo "nuevos" silencia bugs sin corregir | Sí |
| 2026-11-03 | 6 `--bare` | `--bare` no carga nada (ni herramientas ni system prompt), append "a nada" | `--bare` solo salta auto-descubrimiento (hooks, skills, plugins, MCP, memoria, CLAUDE.md); prompt base y tools integradas siguen. Tools "raras" = `--system-prompt-file` reemplaza el prompt base | Sí |
| 2026-11-03 | 7 Permisos | `--allowedTools "npm(test)"`; no vio que `--disallowedTools "Bash"` mata npm test; luego agregó `--tools "bash"` | La tool es `Bash` (mayúscula), comando entre paréntesis: `Bash(npm test)`; nombre solo en disallowed elimina la tool; `--tools` restringe disponibles (quitaría Read/Grep) | Sí |
| 2026-11-03 | 6 `--bare` (2do) | Con --bare + append "solo sabe lo de factories"; --system-prompt "agrega" | --bare = modo seguro (fábrica sí, lo tuyo no); --system-prompt REEMPLAZA el base | Sí |
| 2026-11-03 | 7 `--max-turns` | Escribió `--maxTurns` | Flags en kebab-case: `--max-turns` (excepciones: --allowedTools/--disallowedTools) | Sí |
| 2026-11-03 | 8 Secretos (3ra) + -p en action | Secretos "variables de entorno" / `${{SECRET_NAME}}`; puso -p dentro de la action | `${{ secrets.ANTHROPIC_API_KEY }}` (repo secret: Settings → Secrets and variables → Actions); con input `prompt` no va -p | Sí |
| 2026-11-03 | Resumen parcial | --json-schema con ruta; permisos como "regex"; omitió incremental, CLAUDE.md, --max-turns, tiempo real | Schema como string, valida forma → .structured_output; patrones de permiso tipo `Bash(git push *)` | Sí |
| 2026-11-03 | 4 Revisión incremental (eval final P3) | Vio como problema la sintaxis `${PREVIOUS_FINDINGS}` → `${{ }}`; no detectó falta de instrucción explícita | `${{ }}` es solo para expresiones de GitHub Actions (secrets.X); el problema era pasar hallazgos sin instrucción de los 3 casos | No |
| 2026-11-03 | 8 Action integrador (eval final P11) | Bloque `with:` sin input `prompt`; comillas rotas; `$ {{`; no nombró `anthropic_api_key` | Sin `prompt` la action queda en modo interactivo; bloque completo = prompt + claude_args + anthropic_api_key: ${{ secrets.X }} | No |
| 2026-11-03 | 9 Worktrees | El segundo "soluciona conflictos" | El segundo hace **rebase** sobre lo ya mergeado antes de seguir | Sí |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| 7 Permisos | "Tengo, Abro, Tapo": --tools = lo que Tengo; --allowedTools = lo que Abro sin permiso; --disallowedTools = lo que Tapo (nombre solo = toda la puerta, patrón = guardia a un comando) | Acrónimo/frase (co-construida: cambió Tapio→Tapo) | Sí |
| 6 `--bare` | ~~modo seguro del celular~~ (no funcionó) → "Desnudo, no decapitado": --bare = Claude desnudo (cuerpo/cerebro base y manos/tools siguen, sin ropa/accesorios = CLAUDE.md, plugins, hooks, skills, MCP, memoria); --system-prompt = trasplante de cerebro; --append = nota pegada en la frente | Imagen absurda | Sí |
| 8 Secretos | "El dinero ($) va en una caja fuerte de doble puerta ({{ }}) que se llama secrets" → `${{ secrets.X }}` | Imagen visual | Sí |
| 8 GitHub Action completa | CAPAS: Checkout, Action, Prompt, Args (claude_args, sin -p), Secret (${{ secrets.X }}) | Acrónimo | Sí |
| 4 Revisión incremental | "Datos ≠ órdenes": pegar hallazgos sin instrucción = lista del súper sin decir qué hacer | Contraste | (ángulo, no mnemo formal) |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| 2026-11-03 | Evaluación final | 10/12 | 4 (P3: no vio falta de instrucción explícita; confundió ${VAR} con ${{ }}), 8 (P11: omitió input `prompt`) |

## Resumen final y plan de repaso

