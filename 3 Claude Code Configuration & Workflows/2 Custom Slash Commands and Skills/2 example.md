Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, un skill de equipo (`/analyse-feature`) y un comando plano de equipo (`/review`), aplicando las dos estructuras de archivo equivalentes, el alcance de proyecto vs. usuario, y las tres opciones críticas de frontmatter (`context: fork`, `allowed-tools`, `argument-hint`).

## Paso 1 — Comando plano de equipo: `/review`

Empezamos con el caso más simple: un checklist de revisión de código que todo el equipo debe tener disponible al clonar el repo. No necesita frontmatter especial, así que usamos la estructura plana en `.claude/commands/`.

```markdown
<!-- .claude/commands/review.md — crea /review -->
Review the staged changes against our team checklist:
1. Check error handling patterns
2. Verify test coverage for new functions
3. Confirm API naming conventions
4. Flag any hardcoded credentials or secrets
```

Como vive en `.claude/commands/` (dentro del repo), se versiona con git y llega automáticamente a cualquiera que clone o haga pull.

## Paso 2 — Skill canónico de equipo: `/analyse-feature`

Ahora un caso que sí necesita configuración fina: un skill que analiza un área del codebase y genera un reporte extenso de estructura, patrones y riesgos. Para esto usamos la estructura canónica con `SKILL.md` dentro de una carpeta:

```
.claude/skills/analyse-feature/SKILL.md
```

```yaml
---
description: "Analyse a feature area of the codebase and report structure, patterns and risks"
context: fork
allowed-tools:
  - Read
  - Grep
  - Glob
argument-hint: "Provide a feature description or area of the codebase to analyse"
---

Analyse the codebase area described by the user's argument. Report:
1. File structure and module boundaries
2. Recurring patterns (naming, error handling, data flow)
3. Potential risks (missing tests, tight coupling, dead code)
```

Cada opción de frontmatter resuelve un problema concreto:

- **`context: fork`** — este skill "produce extensive file listings and code excerpts" (justo el caso que la guía marca como esencial para aislar contexto). Sin esto, cada listado de archivos y extracto de código terminaría en la conversación principal, consumiendo el presupuesto de tokens con contenido intermedio que nadie necesita ver después.
- **`allowed-tools`** — el skill solo necesita **leer** el codebase, nunca modificarlo. Al listar únicamente `Read`, `Grep` y `Glob`, se **restringe** el acceso: aunque el skill "alucinara" un plan para editar un archivo, no tiene `Edit` ni `Write` disponibles.
- **`argument-hint`** — cuando alguien escribe `/analyse-feature` sin argumentos, el autocompletado le recuerda qué input se espera, en vez de dejar que el skill arranque sin saber qué analizar.

> [!warning] Pregunta trampa — "¿un `.md` suelto dentro de `.claude/skills/` también funciona?"
> Forma ingenua: crear `.claude/skills/analyse-feature.md` (un archivo plano, sin carpeta contenedora) esperando que funcione igual que `SKILL.md` dentro de una carpeta. **No es válido** — la ruta de skills requiere la estructura de directorio `.claude/skills/<nombre>/SKILL.md`. Si lo que quieres es un archivo plano sin carpeta, esa es la función de `.claude/commands/<nombre>.md` (como en el Paso 1), no de `.claude/skills/`.

## Paso 3 — Verificar el alcance: ¿a quién le llega cada uno?

Ambos archivos del equipo viven dentro de `.claude/` en la raíz del repo:

```
mi-repo/
└── .claude/
    ├── commands/
    │   └── review.md              # /review — nivel proyecto, compartido vía git
    └── skills/
        └── analyse-feature/
            └── SKILL.md            # /analyse-feature — nivel proyecto, compartido vía git
```

Al estar dentro de `.claude/` (no en `~/.claude/`), ambos se versionan y llegan automáticamente a cualquier desarrollador que clone o haga pull del repo — ninguno de los dos es un ajuste personal.

## Paso 4 — Variante personal sin afectar al equipo

Un desarrollador quiere una versión de `/analyse-feature` más agresiva, que también busque código duplicado, pero sin cambiar el comportamiento que el resto del equipo espera del comando compartido. La solución **no** es editar el `SKILL.md` del proyecto — eso afectaría a todos. En vez de eso, crea una variante personal con **nombre distinto** en su propio nivel de usuario:

```
~/.claude/skills/analyse-feature-deep/SKILL.md
```

```yaml
---
description: "Personal variant: analyse a feature area, including duplicate-code detection"
context: fork
allowed-tools:
  - Read
  - Grep
  - Glob
argument-hint: "Provide a feature description or area of the codebase to analyse"
---

Analyse the codebase area described by the user's argument. Report:
1. File structure and module boundaries
2. Recurring patterns (naming, error handling, data flow)
3. Potential risks (missing tests, tight coupling, dead code)
4. Duplicate or near-duplicate code blocks across files
```

Al vivir en `~/.claude/skills/` con un nombre propio (`analyse-feature-deep`, no `analyse-feature`), este skill es exclusivo de ese desarrollador: no se sube a git y no colisiona ni sobreescribe el skill compartido del equipo.

> [!warning] Pregunta trampa — "¿por qué no simplemente editar el `SKILL.md` del proyecto para agregar mi paso extra?"
> Forma ingenua: modificar directamente `.claude/skills/analyse-feature/SKILL.md` para agregar la detección de duplicados. El problema es que ese archivo vive en `.claude/` (nivel proyecto) y se comparte vía git — el cambio afectaría a **todo el equipo** en el próximo pull, no solo a quien lo pidió. La forma correcta es crear una variante en `~/.claude/skills/` con un nombre distinto, precisamente para personalizar sin tocar lo que comparten los demás.

## Paso 5 — Skill vs. `CLAUDE.md`: dónde NO va cada cosa

Por último, un caso que ilustra el límite entre los dos mecanismos. El equipo quiere que **siempre** se usen commits en formato `tipo: descripción` (convención universal) y, por separado, quiere un flujo bajo demanda para generar el changelog de un release (`/changelog`).

```markdown
<!-- forma ingenua: meter el flujo de /changelog dentro de CLAUDE.md -->
<!-- .claude/CLAUDE.md -->
# Convenciones del equipo
- Los commits siguen el formato "tipo: descripción".
- Cuando se prepare un release, revisa todos los commits desde el último tag,
  agrúpalos por tipo, y genera un CHANGELOG.md con el formato de Keep a Changelog...
```

> [!warning] Pregunta trampa — "¿el procedimiento de changelog no es también una 'convención del equipo'?"
> Forma ingenua: como el flujo de generar el changelog también lo sigue todo el equipo, parece razonable meterlo en `CLAUDE.md` junto a las demás convenciones. El problema es la frecuencia de uso: `CLAUDE.md` se carga **en cada sesión**, aunque no se esté preparando un release — es contexto desperdiciado la mayoría del tiempo. La convención de formato de commits sí aplica siempre y pertenece a `CLAUDE.md`; el procedimiento de changelog es bajo demanda y pertenece a un skill:

```markdown
<!-- .claude/commands/changelog.md — crea /changelog -->
Review all commits since the last git tag, group them by type
(feat, fix, chore, docs...), and generate a CHANGELOG.md entry
following the Keep a Changelog format.
```

Con esto, `CLAUDE.md` se mantiene enfocado en lo que debe aplicar siempre, y `/changelog` solo se activa cuando alguien realmente lo necesita.

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
