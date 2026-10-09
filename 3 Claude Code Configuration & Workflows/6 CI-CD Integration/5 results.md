# Resultados — Tutor socrático: CI-CD Integration

> [!info] Sesión evaluada el 2026-09-27
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 70/100**

De 20 preguntas, 12 fueron correctas, 4 parcialmente correctas y 4 incorrectas. Dominas los conceptos de mayor peso en el examen: `-p` y sus falsas alternativas (Sample Question 10), Batch API frente a tiempo real (Sample Question 11), el aislamiento entre sesiones de generación y revisión, y append frente a replace en el system prompt. Las fallas se concentraron en dos frentes:

- **Nombres exactos de flags e inputs:** `--output-format`, `claude_args` y `actions/checkout`. Justo ahí el examen pone distractores con nombres parecidos.
- **Detalles de soporte:** los tests existentes en el contexto, los worktrees y el efecto de `--disallowedTools` con solo el nombre de la herramienta.

## Conceptos con buen dominio

- **Flag `-p` / `--print`:** identificaste de inmediato por qué el job se cuelga y cómo arreglarlo.
- **Falsas alternativas a `-p`:** descartaste `CLAUDE_HEADLESS=true`, `--batch` y la redirección de stdin.
- **`--json-schema` y `structured_output`:** sabes cómo validar la forma de la salida y dónde queda el resultado.
- **Aislamiento de contexto:** entendiste que la sesión que generó el código tiende a justificarlo, y que la revisión debe correr en una sesión independiente.
- **Issues ignorados a propósito:** sabes que no deben volver a reportarse para no erosionar la confianza del equipo.
- **`CLAUDE.md` en CI:** sabes que se lee igual que en modo interactivo y que ahí van los estándares del proyecto.
- **Batch API en checks bloqueantes (trampa):** reconociste que no tiene garantía de latencia y no sirve cuando el desarrollador está esperando.
- **Append frente a replace:** elegiste `--append-system-prompt-file` y explicaste bien el efecto del replace.
- **`--bare`:** relacionaste que se salta la carga de `CLAUDE.md`.
- **GitHub Action con y sin `prompt`:** distinguiste el modo interactivo del modo automatizado.
- **`--tools` frente a `--allowedTools`:** diferenciaste las herramientas disponibles de las preaprobadas.
- **`--max-turns`:** sabes cómo limitar los loops agénticos.

## Áreas que necesitan revisión

### Nombre exacto del flag de formato de salida

- **Qué pasó:** entendiste que hace falta JSON, pero nombraste el flag como `--output-type` en vez de `--output-format`.
- **Contenido del resumen:**
  > - `--output-format json` → envuelve el resultado en un JSON con `session_id`, metadata de costo, etc.
  > - `--json-schema` → valida la salida del agente contra un JSON Schema, en modo `-p`.
- **Sección:** [[1 resumen]] → "2. Salida estructurada para CI"
- **Analogía para reforzarlo:** es como el nombre exacto de una calle en el GPS. "Av. Reforma" te lleva; "Calle Reforma" puede no existir, aunque "se parezca". En la CLI, un flag casi correcto es un flag que no existe.

### Revisión incremental: las instrucciones que acompañan a los hallazgos previos

- **Qué pasó:** dijiste que hay que pasar los hallazgos anteriores, pero no mencionaste la instrucción explícita de reportar solo lo nuevo o lo que sigue sin resolver.
- **Contenido del resumen:**
  > **Solución:** incluir los hallazgos previos (`${PREVIOUS_FINDINGS}`) en el contexto, con instrucciones explícitas de reportar solo issues nuevos o los que sigan sin resolverse — sin repetir issues que el equipo decidió deliberadamente no atender.
- **Sección:** [[1 resumen]] → "4. Revisión incremental (evitar hallazgos duplicados)"
- **Analogía para reforzarlo:** darle a un inspector la lista de la visita pasada no basta; también hay que decirle "solo anota lo nuevo o lo que siga roto". Si no, vuelve a copiar la lista completa.

### Incluir los tests existentes en el contexto

- **Qué pasó:** no supiste qué agregar al contexto para evitar que Claude proponga tests que ya existen.
- **Contenido del resumen:**
  > Al generar tests en CI, incluir los archivos de test ya existentes en el contexto evita que Claude sugiera tests que ya existen, y lo enfoca en encontrar huecos reales de cobertura.
- **Sección:** [[1 resumen]] → "Tests existentes en el contexto"
- **Analogía para reforzarlo:** antes de ir al súper, revisas qué hay en el refrigerador. Si no lo revisas, vuelves a comprar leche que ya tenías y te olvidas de lo que de verdad faltaba.

### GitHub Actions: `claude_args`, `actions/checkout` y secretos

- **Qué pasó:** entendiste que los flags van en un solo input, pero lo llamaste "commands" en vez de `claude_args`. Nombraste el checkout como "check@1" y no respondiste cómo manejar la API key.
- **Contenido del resumen:**
  > - Los flags de CLI se pasan todos juntos como un solo string en `claude_args` (no hay inputs separados para system prompt, allowlist de tools, etc.).
  > - `actions/checkout` es el paso que pone el repositorio en el runner — sin él no hay nada en disco que Claude pueda leer. Las credenciales de API deben ir siempre como secretos del repositorio, nunca escritas literalmente en el YAML.
- **Sección:** [[1 resumen]] → "7. GitHub Actions como paso del workflow"
- **Analogía para reforzarlo:** contratas a un revisor y lo mandas a una oficina vacía: sin `actions/checkout` no le llevaste los documentos. Y la llave de la oficina no se pega en la puerta (el YAML); se guarda en una caja fuerte (los secretos del repo).

### Sesiones en paralelo con git worktrees

- **Qué pasó:** propusiste un fork en lugar de un worktree, y no conocías la coordinación merge/rebase ni `isolation: worktree`.
- **Contenido del resumen:**
  > Un git worktree crea un segundo directorio de trabajo para el mismo repositorio, en una rama distinta, lo que permite correr varias instancias de Claude Code en paralelo sin que se pisen los archivos entre sí. Cada sesión obtiene su propio presupuesto completo de contexto y sus propios archivos. La coordinación se hace por secuencia: la primera sesión en terminar hace merge, la segunda hace rebase antes de seguir trabajando. Los subagentes personalizados pueden declarar `isolation: worktree` en su frontmatter para ejecutarse en paralelo sin colisiones.
- **Sección:** [[1 resumen]] → "8. Sesiones en paralelo con Git Worktrees"
- **Analogía para reforzarlo:** dos cocineros en la misma cocina, cada uno con su propia mesa de trabajo (worktree) pero la misma despensa (el repo). El primero que termina sirve su plato (merge); el segundo ajusta el suyo a lo que ya se sirvió (rebase) antes de servir. Un fork sería mandar a un cocinero a otro restaurante.

### `--disallowedTools` con solo el nombre de la herramienta

- **Qué pasó:** dijiste que poner solo el nombre agrupa un conjunto de comandos. En realidad elimina la herramienta por completo.
- **Contenido del resumen:**

| Flag | Efecto |
|---|---|
| `--disallowedTools "<reglas>"` | Reglas de denegación; el nombre solo elimina la herramienta por completo |

- **Sección:** [[1 resumen]] → "6. Referencia rápida de flags CLI"
- **Analogía para reforzarlo:** con una regla específica le quitas a un niño solo las tijeras de punta; con solo el nombre le quitas la caja de útiles entera.

## Siguiente paso recomendado

Repasa en [[1 resumen]] las secciones 7 y 8 y la tabla de flags, poniendo atención a los nombres **exactos**: `--output-format`, `claude_args`, `actions/checkout` y el efecto de `--disallowedTools` con solo el nombre. Quedaron sin evaluar en esta sesión `-c`/`-r`, los valores de `--permission-mode`, `stream-json`, `--add-dir` y `--model`. Después de ese repaso ya estás listo para [[4 test]].
