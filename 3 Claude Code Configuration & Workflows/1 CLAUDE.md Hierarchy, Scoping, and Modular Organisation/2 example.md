Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, la configuración de `CLAUDE.md` de un repo ficticio con un backend en `/packages/api` y un frontend en `/packages/web`, aplicando los tres niveles de jerarquía, `@` para modularizar, `.claude/rules/`, `CLAUDE.local.md`, y la diferencia entre "guía" y "aplicación garantizada".

## Paso 1 — Estructura del repo

Primero definimos dónde va a vivir cada archivo, respetando los tres niveles: usuario (fuera del repo), proyecto (raíz) y directorio (por paquete).

```bash
~/.claude/CLAUDE.md                     # nivel usuario — fuera del repo, NO se sube a git

mi-repo/
├── .claude/
│   ├── CLAUDE.md                       # nivel proyecto — reglas del equipo
│   ├── CLAUDE.local.md                 # overrides personales para ESTE repo (gitignored)
│   └── rules/
│       ├── testing.md
│       └── api-conventions.md
├── standards/
│   ├── naming-conventions.md
│   ├── error-handling.md
│   └── testing-requirements.md
└── packages/
    ├── api/
    │   └── CLAUDE.md                   # nivel directorio — solo aplica dentro de /packages/api
    └── web/
        └── CLAUDE.md                   # nivel directorio — solo aplica dentro de /packages/web
```

## Paso 2 — `CLAUDE.md` de proyecto, modularizado con `@`

En vez de meter todos los estándares del equipo en un solo archivo gigante, el `CLAUDE.md` de proyecto **importa** archivos con la sintaxis `@` (sin la palabra `import`):

```markdown
<!-- .claude/CLAUDE.md -->
# Convenciones del equipo — mi-repo

Coding standards:
@./standards/naming-conventions.md
@./standards/error-handling.md
@./standards/testing-requirements.md
```

Estas referencias se resuelven **de forma eager e inline**: en el momento en que Claude Code lee `.claude/CLAUDE.md`, el contenido de cada archivo importado se inserta ahí mismo, como si lo hubieras pegado a mano. No hay carga condicional aquí — eso es justamente lo que NO hace el `@`, a diferencia de `.claude/rules/` (paso 4).

## Paso 3 — `CLAUDE.md` de directorio, con importaciones selectivas por dominio

El paquete `api` no necesita las convenciones de componentes de React del paquete `web`, así que su `CLAUDE.md` de directorio importa solo lo que le aplica a su dominio:

```markdown
<!-- packages/api/CLAUDE.md -->
# Convenciones específicas de /packages/api

@../../standards/error-handling.md

## Reglas propias de este paquete
- Todos los endpoints usan el patrón repository para acceso a datos.
- Los handlers son async/await, nunca callbacks.
```

> [!warning] Pregunta trampa — "¿puedo poner esto en un CLAUDE.md de directorio para que aplique a todo el repo?"
> Forma ingenua: crear `packages/api/CLAUDE.md` esperando que su regla de "usar async/await" también gobierne los archivos de test dispersos en `packages/web/**/*.test.tsx`. **No funciona así** — un `CLAUDE.md` de directorio solo aplica hacia abajo desde donde vive. Si la convención debe aplicar a archivos por patrón de nombre sin importar en qué carpeta estén, la herramienta correcta es un archivo en `.claude/rules/` con glob patterns (ver paso 4), no multiplicar `CLAUDE.md` por cada carpeta.

## Paso 4 — `.claude/rules/` como alternativa a un `CLAUDE.md` monolítico

En vez de seguir agregando secciones al `CLAUDE.md` de proyecto hasta que sea inmanejable, dividimos temas específicos en archivos separados dentro de `.claude/rules/`:

```markdown
<!-- .claude/rules/testing.md -->
# Convenciones de testing

- Cada módulo nuevo necesita al menos un test de integración.
- Los mocks de la base de datos están prohibidos en tests de integración.
```

```markdown
<!-- .claude/rules/api-conventions.md -->
# Convenciones de API

- Todas las respuestas de error siguen el formato { "error": { "code", "message" } }.
```

Esto resuelve el problema de un `CLAUDE.md` gigante y difícil de mantener, dividiéndolo por tema (`testing.md`, `api-conventions.md`, `deployment.md`) en vez de por carpeta física del repo.

## Paso 5 — `CLAUDE.local.md`: tus manías, sin ensuciar el repo del equipo

```markdown
<!-- .claude/CLAUDE.local.md -->
# Preferencias personales (no compartidas)

- Prefiero que expliques cada cambio en 2-3 líneas antes de aplicarlo.
- Usa `pnpm` en vez de `npm` en mis ejemplos de terminal.
```

```gitignore
# .gitignore
.claude/CLAUDE.local.md
```

Con esto, el `CLAUDE.md` de proyecto sigue siendo lo que todo el equipo comparte, mientras que tus manías personales viven al lado, sin llegar nunca al repo remoto.

## Paso 6 — Diagnosticar qué se cargó, con `/memory` y `/context`

Dentro de una sesión de Claude Code, corriendo desde `packages/api/`:

```
> /memory
Archivos de configuración disponibles:
  ~/.claude/CLAUDE.md
  .claude/CLAUDE.md
  .claude/CLAUDE.local.md
  packages/api/CLAUDE.md

> /context
Memory files:
  ~/.claude/CLAUDE.md          (~120 tokens)
  .claude/CLAUDE.md            (~450 tokens, incluye @imports resueltos)
  .claude/CLAUDE.local.md      (~30 tokens)
  packages/api/CLAUDE.md       (~180 tokens)
```

> [!warning] Pregunta trampa — "corrí `/memory`, ¿ya se cargó la configuración?"
> Forma ingenua de pensarlo: asumir que ejecutar `/memory` o `/context` es lo que **activa** la carga de estos archivos, y que si no los corres, Claude Code no los lee. Es al revés: la carga ya ocurrió automáticamente al iniciar la sesión, según la ubicación de cada archivo. `/memory` y `/context` son **solo lectura** — te muestran un estado que ya existía, no lo provocan.

## Paso 7 — Diagnosticar el caso real del "nuevo integrante con comportamiento distinto"

Developer A tiene esto en `~/.claude/CLAUDE.md` (su máquina, nunca subido a git):

```markdown
# ~/.claude/CLAUDE.md de Developer A
- Los endpoints nuevos siguen el naming kebab-case: /user-profile, no /userProfile.
```

Developer B clona el mismo repo, misma rama, y Claude Code le genera endpoints en camelCase. La causa no es un bug ni un archivo corrupto: la convención de Developer A **nunca vivió en el repo**, solo en su config de usuario. La corrección es moverla a `.claude/CLAUDE.md` (nivel proyecto), donde sí se versiona y llega a todo el equipo al clonar.

## Paso 8 — Guía vs. aplicación garantizada: por qué `CLAUDE.md` no basta para reglas mandatorias

```markdown
<!-- forma ingenua: intentar "forzar" una regla solo con texto en CLAUDE.md -->
# .claude/CLAUDE.md
- NUNCA se debe hacer commit si los tests fallan. Esto es obligatorio, sin excepciones.
```

> [!warning] Pregunta trampa — "si lo escribo en mayúsculas y digo 'obligatorio', ¿ya queda forzado?"
> No. `CLAUDE.md` sigue siendo texto que **guía** el comportamiento de Claude; Claude puede desviarse (por ejemplo, si el usuario insiste, o si interpreta mal el contexto). No hay ningún mecanismo que impida físicamente el commit solo por tenerlo escrito en `CLAUDE.md`.

La forma correcta de que una regla se cumpla siempre, sin depender de la interpretación de Claude, es moverla a un mecanismo que el cliente aplica de verdad:

```json
// .claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit*)",
        "hooks": [
          { "type": "command", "command": "npm test || exit 1" }
        ]
      }
    ]
  }
}
```

Este hook se dispara en un punto fijo del ciclo de vida (antes de ejecutar `git commit`) y bloquea la acción si los tests fallan — sin importar lo que Claude "decida". Eso es la diferencia entre **guía** (`CLAUDE.md`) y **aplicación garantizada** (`settings.json` / hooks).

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
