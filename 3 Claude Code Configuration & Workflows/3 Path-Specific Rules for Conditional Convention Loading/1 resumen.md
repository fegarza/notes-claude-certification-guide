## Explícamelo como si tuviera 5 años

Imagina que tienes una casa con muchos cuartos: cocina, garage, cuarto de juguetes. En vez de pegar UN cartel gigante con "reglas de toda la casa" en la puerta de entrada (que tienes que leer cada vez que entras, aunque solo vayas a la cocina), pegas carteles específicos: uno en el garage que dice "reglas del garage", uno en el cuarto de juguetes que dice "reglas de los juguetes". Cada cartel solo se activa cuando entras a ESE cuarto.

Los **path-specific rules** son justo eso: en vez de un `CLAUDE.md` gigante que se carga siempre (leas o no ese cartel lo necesites), o pegar el mismo cartel en cada cuarto de la casa (50 copias del mismo cartel de "reglas de testing" en 50 carpetas distintas), escribes UN cartel con un patrón ("esto aplica a cualquier archivo que termine en `.test.tsx`, no importa en qué cuarto esté") y Claude Code lo activa solo cuando corresponde.

## Argumento

> [!note] Idea central
> Los **path-specific rules** (`.claude/rules/`) resuelven el problema de aplicar convenciones a **un tipo de archivo disperso en muchos directorios** — algo que ni el `CLAUDE.md` raíz (se carga siempre, para todo) ni el `CLAUDE.md` de directorio (requiere duplicar el mismo archivo en cada carpeta) resuelven bien.

## Conclusiones

### Cómo funcionan los rule files

- Viven en el directorio **`.claude/rules/`**, como archivos `.md`.
- Cada archivo lleva **YAML frontmatter** con un campo **`paths`**: un array de *glob patterns* (ej. `["**/*.test.tsx", "**/*.test.ts"]`).
- El cuerpo del archivo (debajo del frontmatter) contiene las convenciones en Markdown normal.
- La regla se carga **automáticamente** cuando Claude edita/lee un archivo que hace match con alguno de esos patrones — sin invocación manual.

> [!note] Analogía mental
> Es como una regla de CSS con un *selector*: el `paths` es el selector (`**/*.test.tsx`), y el cuerpo del archivo es el "estilo" que se aplica solo a los elementos que matchean ese selector.

### Ventaja de eficiencia de tokens

- El `CLAUDE.md` raíz se carga **en cada sesión, sin importar qué archivo edites** — si tienes convenciones de Terraform ahí, consumen tokens incluso cuando editas un componente de React.
- Los path-specific rules cargan **solo cuando el archivo editado matchea el patrón** — reducen contexto irrelevante y preservan presupuesto de tokens.
- Verificable con **`/context`**: al editar un `.test.ts`, solo aparece cargada la regla de testing (las de API/Terraform no aparecen); al editar un handler de API, aparece solo la regla de API.

### Por qué no alcanza con directory-level CLAUDE.md

- Un `CLAUDE.md` de directorio cubre **un solo directorio/paquete** — funciona bien para convenciones acotadas a ese paquete.
- El problema aparece cuando el tipo de archivo (ej. tests) está **disperso en 50+ directorios** junto al código que prueban (`Button.test.tsx` junto a `Button.tsx`): habría que poner una copia del mismo `CLAUDE.md` en cada una de esas 50+ carpetas.
- Consecuencias de esa duplicación: mantenimiento imposible, cada carpeta nueva con tests necesita su propia copia, cualquier cambio de convención obliga a actualizar 50+ archivos, y **drift inevitable** (copias que se desactualizan con el tiempo).
- Un solo rule file con un glob pattern elimina esto: "un archivo, un patrón, cobertura universal".

### Cuándo usar cada mecanismo (tabla de decisión)

| Escenario | Mejor opción |
|---|---|
| Estándares universales que aplican a todo el código | `CLAUDE.md` raíz |
| Convenciones para un paquete/directorio específico | `CLAUDE.md` de directorio |
| Convenciones para un **tipo de archivo** disperso en muchos directorios | **Path-specific rules** con glob patterns |
| Workflows invocados **on-demand**, no automáticos | Skills (`.claude/skills/`) |

### Ejemplos de rule files

> [!example] Testing conventions
> ```yaml
> ---
> paths: ["**/*.test.ts", "**/*.test.tsx", "**/*.spec.ts", "**/*.spec.tsx"]
> ---
> # Test Conventions
>
> - Use describe/it blocks with descriptive names that read as sentences
> - Each test file must have at least one happy path and one error case
> - Use factory functions for test data, not inline object literals
> - Mock external services at the module boundary, not individual functions
> - Assert behaviour, not implementation details
> ```

> [!example] API conventions
> ```yaml
> ---
> paths: ["src/api/**/*", "**/routes/**/*", "**/*.controller.ts"]
> ---
> # API Conventions
>
> - All endpoints return { data, error, metadata } response shape
> - Use Zod schemas for request validation at the handler boundary
> - Log request ID on every error response
> - Rate limiting configuration must be explicit, not inherited from defaults
> ```

> [!example] Infrastructure/Terraform conventions
> ```yaml
> ---
> paths: ["terraform/**/*", "**/*.tf", "infrastructure/**/*"]
> ---
> # Infrastructure Conventions
>
> - State files must reference remote backends, never local
> - Use workspaces for environment separation
> - Every module must be versioned with a CHANGELOG
> ```

## Evidencia

- El escenario clásico del examen es: componentes React con hooks, handlers de API con async/await, modelos de datos con repository pattern, y **archivos de test dispersos junto al código que prueban** — la respuesta correcta casi siempre es path-specific rules con glob patterns.
- `/context` es la herramienta para *verificar* que la carga condicional funciona: confirma qué reglas están cargadas según el archivo que se está editando en ese momento.

## Trampas de examen

> [!warning] Trampa 1 — Usar CLAUDE.md de directorio para convenciones que cruzan muchos directorios
> **Error común:** elegir un `CLAUDE.md` por directorio cuando la convención (ej. reglas de testing) debe aplicar a archivos de un tipo esparcidos en 50+ carpetas.
> **Por qué está mal:** obliga a duplicar el mismo archivo en cada carpeta — mantenimiento inviable y drift garantizado con el tiempo.
> **Forma correcta:** un único rule file en `.claude/rules/` con un glob pattern que cubra ese tipo de archivo sin importar el directorio.

> [!warning] Trampa 2 — Meter convenciones de un tipo de archivo en el CLAUDE.md raíz
> **Error común:** poner reglas específicas de un tipo de archivo (ej. Terraform) en el `CLAUDE.md` raíz del proyecto.
> **Por qué está mal:** el `CLAUDE.md` raíz se carga en **cada sesión sin importar qué se edite** — esas convenciones de Terraform consumen tokens incluso al editar componentes de React que nada tienen que ver.
> **Forma correcta:** path-specific rules, que cargan solo cuando el archivo editado matchea el patrón, preservando el presupuesto de tokens.

> [!warning] Trampa 3 — Confundir Skills con Rules para carga automática de convenciones
> **Error común:** pensar que un Skill sirve para aplicar convenciones de forma automática y siempre-activa a un tipo de archivo.
> **Por qué está mal:** las **rules** permanecen en el contexto como guía de fondo — se cargan cuando Claude *lee* un archivo que matchea, y moldean cada edición sin que nadie las invoque. Los **skills** se cargan **on-demand**, como workflows tipo tarea, disparados por intención del modelo o invocación explícita — no garantizan aplicación automática y determinística basada en el path del archivo.
> **Forma correcta:** cuando la pregunta pide carga automática y siempre-activa de convenciones según tipo de archivo → path-specific rules. Cuando pide un workflow invocado a demanda → skills.

## En una frase

Los path-specific rules aplican convenciones **automáticamente por tipo de archivo, sin importar el directorio**, usando glob patterns en `.claude/rules/`, evitando tanto el desperdicio de tokens del `CLAUDE.md` raíz como la duplicación imposible de mantener del `CLAUDE.md` por directorio.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
