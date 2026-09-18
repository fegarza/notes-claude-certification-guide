Aplicación práctica de [[1 resumen]] — construcción paso a paso de un pipeline de revisión de PRs con Claude Code en GitHub Actions, aplicando cada idea clave del resumen (y evitando explícitamente sus trampas).

## Escenario

Un pipeline de CI que, en cada pull request, usa Claude Code para: (1) revisar el PR sin colgarse, (2) devolver hallazgos en un formato que un bot pueda leer y postear como comentarios, (3) revisar con una sesión independiente a la que generó el código, y (4) no repetir hallazgos ya reportados en corridas anteriores.

## Paso 1 — Script no interactivo con `-p`

Todo pipeline de CI arranca aquí: sin `-p`, el job se queda esperando input de teclado para siempre. `-p` procesa el prompt y termina, escribiendo el resultado en stdout.

```bash
#!/usr/bin/env bash
set -euo pipefail

claude -p "Analiza el diff de este PR en busca de problemas de seguridad" \
  > review_raw.txt
```

> [!danger] Pregunta trampa — "arreglos" que no funcionan
> ```bash
> # ❌ NO HACER: ninguna de estas tres "soluciones" existe o resuelve el colgado
> CLAUDE_HEADLESS=true claude "Analiza este PR"
> claude --batch "Analiza este PR"
> claude "Analiza este PR" < /dev/null
> ```
> **¿Por qué serían mala idea?** `CLAUDE_HEADLESS` y `--batch` no son flags reales de Claude Code — el script fallaría o simplemente los ignoraría, dejando el modo interactivo activo. Redirigir stdin desde `/dev/null` tampoco cambia el modo de ejecución: Claude Code seguiría esperando el tipo de entrada que el modo interactivo espera, y el job se colgaría igual (o fallaría de forma distinta) en vez de imprimir y salir. La única forma correcta es `-p`.

## Paso 2 — Salida estructurada con JSON Schema

Como quien procesa el resultado es el propio pipeline (no una persona), se pide salida en JSON validada contra un schema, para poder extraer los hallazgos de forma confiable.

```bash
SCHEMA='{
  "type": "object",
  "properties": {
    "findings": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "file": {"type": "string"},
          "line": {"type": "integer"},
          "severity": {"type": "string", "enum": ["low", "medium", "high"]},
          "message": {"type": "string"}
        },
        "required": ["file", "line", "severity", "message"]
      }
    }
  },
  "required": ["findings"]
}'

claude -p \
  --output-format json \
  --json-schema "$SCHEMA" \
  "Revisa el diff de este PR en busca de problemas de seguridad" \
  > review.json

# Extraer solo los hallazgos validados
jq '.structured_output' review.json > findings.json
```

> [!danger] Pregunta trampa — parsear texto libre en vez de JSON
> ```bash
> # ❌ NO HACER: usar grep/regex sobre texto libre para "extraer" hallazgos
> claude -p "Revisa el PR y lista los problemas encontrados" > review.txt
> grep -oP '(?<=Problema: ).*' review.txt > findings.txt
> ```
> **¿Por qué sería mala idea?** El formato del texto libre puede variar entre corridas (Claude no siempre redacta exactamente igual), rompiendo el `grep` silenciosamente. Sin `--json-schema`, tampoco hay garantía de que existan campos como archivo/línea/severidad de forma consistente — el bot que postea comentarios necesita datos estructurados y predecibles, no texto para "adivinar".

## Paso 3 — Postear los hallazgos como comentarios inline en el PR

Con `findings.json` ya parseado, un paso separado del pipeline recorre cada hallazgo y lo postea en el archivo/línea exactos usando la API de GitHub (ej. `gh pr comment` o la API REST de review comments).

```bash
jq -c '.findings[]' findings.json | while read -r finding; do
  file=$(echo "$finding" | jq -r '.file')
  line=$(echo "$finding" | jq -r '.line')
  message=$(echo "$finding" | jq -r '.message')
  severity=$(echo "$finding" | jq -r '.severity')

  gh api repos/:owner/:repo/pulls/:pr/comments \
    -f body="[$severity] $message" \
    -f path="$file" \
    -F line="$line"
done
```

## Paso 4 — Documentar contexto del proyecto en `CLAUDE.md`

`CLAUDE.md` se lee igual en CI que en modo interactivo, así que ahí van los estándares que la revisión automática debe conocer.

```markdown
## Testing standards (para CI)
- Usar los patrones factory para crear datos de prueba (`tests/factories/`)
- No testear detalles internos de implementación (métodos privados)
- Cobertura mínima objetivo: 80% branch coverage
- Fixtures disponibles: ver /test/fixtures/README.md

## Criterios de revisión de seguridad
- Severidad "high": inyección SQL, secretos hardcodeados, auth bypass
- Severidad "medium": falta de validación de input, manejo de errores débil
```

## Paso 5 — Aislar generación de revisión en sesiones separadas

Para que la revisión sea independiente y no herede el razonamiento con el que Claude justificó el código que generó, se usan dos invocaciones separadas — dos procesos `claude -p` distintos, sin historial compartido.

```bash
# Sesión A — generación (por ejemplo, en un paso previo del pipeline)
claude -p "Implementa el middleware de autenticación según las specs del issue" \
  > /dev/null

# Sesión B — revisión, completamente independiente, sin contexto de la sesión A
claude -p \
  --output-format json \
  --json-schema "$SCHEMA" \
  --append-system-prompt "Revisa contra el checklist de seguridad en CLAUDE.md, sé exigente." \
  "Revisa el middleware de autenticación en busca de fallas de seguridad y de diseño" \
  > review.json
```

> [!danger] Pregunta trampa — reusar la sesión de generación para revisar
> ```bash
> # ❌ NO HACER: usar -c / --continue para revisar en la MISMA sesión que generó el código
> claude -p "Implementa el middleware de autenticación" > /dev/null
> claude -p -c "Ahora revisa el código que acabas de escribir en busca de fallas"
> ```
> **¿Por qué sería mala idea?** `-c` retoma la sesión anterior con todo su razonamiento previo. Claude "recuerda" por qué tomó cada decisión al generar el código, así que es menos probable que se cuestione a sí mismo — la revisión pierde objetividad. Por eso el Paso 5 usa dos invocaciones completamente separadas, sin `-c` ni `-r` entre ellas.

## Paso 6 — Revisión incremental: no repetir hallazgos

Cada corrida guarda sus hallazgos, y la siguiente corrida los incluye en el contexto, pidiendo explícitamente que solo se reporten issues nuevos o los que sigan sin resolver.

```bash
# Guardar hallazgos de esta corrida para la próxima
cp findings.json .ci-cache/previous_findings.json

# En la siguiente corrida:
PREVIOUS_FINDINGS=$(cat .ci-cache/previous_findings.json 2>/dev/null || echo '{"findings":[]}')

claude -p \
  --output-format json \
  --json-schema "$SCHEMA" \
  --append-system-prompt "Hallazgos previos ya reportados: ${PREVIOUS_FINDINGS}. Reporta SOLO issues nuevos o issues previos que sigan presentes en el código. No repitas issues que ya fueron corregidos ni los que el equipo decidió no atender." \
  "Revisa el diff de este PR en busca de problemas de seguridad" \
  > review.json
```

> [!danger] Pregunta trampa — no pasar los hallazgos previos
> ```bash
> # ❌ NO HACER: repetir el mismo comando en cada push, sin memoria de corridas anteriores
> claude -p --output-format json --json-schema "$SCHEMA" \
>   "Revisa el diff de este PR en busca de problemas de seguridad" > review.json
> ```
> **¿Por qué sería mala idea?** Cada push volvería a reportar exactamente los mismos hallazgos ya comentados en el PR, incluyendo los que el equipo decidió deliberadamente no resolver — generando ruido y erosionando la confianza del equipo en la herramienta.

## Paso 7 — Ensamblar todo en un workflow de GitHub Actions

Uniendo los pasos anteriores en un solo workflow, usando la acción oficial `anthropics/claude-code-action@v1` (construida sobre el Agent SDK, acepta los mismos flags que `claude -p` vía `claude_args`).

```yaml
name: Claude Code PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: read
    steps:
      # Sin este paso no hay repositorio en disco que Claude pueda leer
      - uses: actions/checkout@v6

      - name: Restaurar hallazgos previos
        uses: actions/cache@v4
        with:
          path: .ci-cache/previous_findings.json
          key: pr-findings-${{ github.event.pull_request.number }}

      - name: Revisión con Claude Code
        uses: anthropics/claude-code-action@v1
        with:
          # La credencial SIEMPRE como secreto del repo, nunca literal en el YAML
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          claude_args: |
            --output-format json
            --json-schema '${{ toJson(vars.REVIEW_SCHEMA) }}'
            --append-system-prompt "Revisa contra el checklist de seguridad en CLAUDE.md"
            --allowedTools "Read,Grep,Glob,Bash(git diff *)"
            --max-turns 10

      - name: Postear comentarios inline
        run: ./scripts/post_review_comments.sh
```

> [!danger] Pregunta trampa — credenciales literales en el YAML
> ```yaml
> # ❌ NO HACER: escribir la API key directamente en el workflow
> with:
>   anthropic_api_key: "sk-ant-api03-xxxxxxxxxxxx"
> ```
> **¿Por qué sería mala idea?** El YAML del workflow queda versionado en el repositorio (y visible en el historial de git para siempre, incluso si se borra después). Cualquiera con acceso de lectura al repo vería la credencial. Por eso siempre se usa `${{ secrets.ANTHROPIC_API_KEY }}`, gestionado por los secretos del repositorio.

## Paso 8 — Elegir Batch API vs. tiempo real según el tipo de job

Este mismo pipeline de revisión pre-merge usa la API en tiempo real porque es bloqueante (el desarrollador espera el resultado). Para un job distinto y no bloqueante —como una auditoría nocturna de deuda técnica— se usaría la Batch API en su lugar, por el ahorro de costo.

```bash
# Job nocturno, NO bloqueante — sí puede usar Batch API
curl https://api.anthropic.com/v1/messages/batches \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "content-type: application/json" \
  -d '{
    "requests": [
      {
        "custom_id": "nightly-tech-debt-audit",
        "params": {
          "model": "claude-sonnet-5",
          "max_tokens": 4096,
          "messages": [{"role": "user", "content": "Audita el repo en busca de deuda técnica"}]
        }
      }
    ]
  }'
# El resultado puede tardar hasta 24h — no se usa aquí para el check bloqueante de pre-merge
```

> [!danger] Pregunta trampa — usar Batch API en el check de pre-merge
> ```bash
> # ❌ NO HACER: mandar el review bloqueante del PR a la Batch API
> curl https://api.anthropic.com/v1/messages/batches \
>   -d '{"requests":[{"custom_id":"pr-42-review", ...}]}'
> # el desarrollador queda esperando hasta 24h con el PR bloqueado
> ```
> **¿Por qué sería mala idea?** El check de pre-merge bloquea el flujo del desarrollador — necesita el resultado en minutos, no en horas. La Batch API no da garantía de latencia (hasta 24h), así que usarla aquí congelaría el PR indefinidamente. El 50% de ahorro de costo no compensa romper el flujo de trabajo del equipo.

## Paso 9 — Paralelizar trabajo con git worktrees (opcional)

Si además se quiere correr, por ejemplo, la generación de un PR y la revisión de otro PR distinto al mismo tiempo sin que los procesos de Claude Code se pisen los archivos, se usa un worktree por sesión.

```bash
# Cada sesión trabaja en su propio directorio, misma repo, ramas distintas
git worktree add ../repo-review-pr-42 -b review/pr-42
git worktree add ../repo-review-pr-43 -b review/pr-43

# O dejar que Claude Code lo maneje automáticamente:
claude --worktree review-pr-42
```

## Resultado

El pipeline completo aplica todas las ideas clave del resumen: modo no interactivo (`-p`), salida estructurada y validada (`--json-schema`), contexto de proyecto vía `CLAUDE.md`, sesiones aisladas para generación vs. revisión, revisión incremental sin duplicados, integración real vía GitHub Actions, la elección correcta entre API en tiempo real y Batch API según si el job bloquea a un humano o no, y paralelización segura con git worktrees cuando hace falta — evitando explícitamente cada una de las trampas de examen del tema.
