## Explícamelo como si tuviera 5 años

Imagina que `CLAUDE.md` es el reglamento general que el empleado nuevo lee siempre, todos los días, sin que nadie se lo pida. Un **skill**, en cambio, es como un **manual de procedimiento guardado en un cajón con etiqueta**: el empleado no lo lee todo el tiempo, solo lo saca cuando alguien dice "necesito que sigas el procedimiento de `/deploy`". Ese cajón puede estar en el escritorio compartido de la oficina (todo el equipo lo usa) o en el cajón personal de tu propio escritorio (solo tú lo tienes).

Algunos de esos manuales, además, traen una **etiqueta especial** pegada arriba: "hazlo en un cuarto aparte y solo tráeme el resumen" (`context: fork`), o "para este procedimiento solo puedes usar estas herramientas, ninguna otra" (`allowed-tools`), o "antes de empezar, pregúntame qué necesitas" (`argument-hint`).

## Argumento central

Un **skill** es un conjunto de instrucciones **nombrado y reutilizable** que Claude Code carga **bajo demanda** mediante un archivo Markdown (con YAML frontmatter opcional), a diferencia de `CLAUDE.md`, que se carga **siempre y automáticamente**. El sistema soporta **dos estructuras de archivo equivalentes** (`.claude/skills/<nombre>/SKILL.md` y `.claude/commands/<nombre>.md`), ambas crean el mismo comando `/nombre`, y el comportamiento fino de un skill (aislamiento de contexto, restricción de herramientas, prompts de argumentos) se controla con **opciones de frontmatter específicas**.

> [!note] Idea clave
> La pregunta que separa un skill de `CLAUDE.md` no es "¿qué tan importante es esta instrucción?", sino "¿esto debe aplicar **siempre** o solo **cuando se invoca**?". Procedimientos específicos de una tarea van en skills; convenciones universales van en `CLAUDE.md`.

## Conclusiones

### 1. Sistema unificado de skills: dos estructuras, un mismo resultado

- **`.claude/skills/<nombre>/SKILL.md`** — estructura **canónica**, basada en directorio.
- **`.claude/commands/<nombre>.md`** — archivo **plano**, mantenido por compatibilidad hacia atrás.
- Ambas producen un comando `/<nombre>` idéntico en funcionamiento.
- La ruta de skills es la **preferida** porque habilita **descubrimiento automático** (Claude puede cargar un skill cuando coincide con la intención del usuario, sin que se invoque el `/comando` explícitamente) y tiene **precedencia** cuando hay conflictos de nombre entre ambas estructuras.

**Analogía**: el archivo plano es como una hoja suelta con instrucciones; la carpeta con `SKILL.md` es como una carpeta con pestaña — además de guardar las instrucciones, tiene metadatos (el frontmatter) que le dicen a Claude cuándo y cómo usarla sin que se lo pidas explícitamente.

### 2. Niveles de alcance (scoping): proyecto vs. usuario

| Nivel | Ubicación | ¿Se comparte con el equipo? |
|---|---|---|
| Proyecto | `.claude/` (skills o commands) | Sí — se distribuye vía git a todos los desarrolladores |
| Usuario | `~/.claude/` (skills o commands) | No — permanece personal, no se comparte |

- Esta distinción determina si un flujo de trabajo es **de todo el equipo** o **individual**.
- Un caso de uso típico de nivel usuario: crear una **variante personal** de un skill en `~/.claude/skills/`, con un **nombre distinto** al del equipo, precisamente para no afectar a los compañeros que usan la versión compartida.

### 3. Opciones críticas de frontmatter

- **`context: fork`** — aísla la salida verbosa del skill en el contexto de un **sub-agente**, preservando el presupuesto de tokens de la conversación principal. Esencial para tareas como análisis de codebase (genera listados extensos de archivos y extractos de código) o brainstorming (contexto exploratorio que no debe ensuciar la conversación principal).
- **`allowed-tools`** — pre-aprueba una lista de herramientas específicas para que el skill las use sin pedir permiso en cada paso. La guía de certificación lo describe explícitamente como un mecanismo que **restringe** el acceso a herramientas (por ejemplo, limitar a operaciones de escritura de archivos para prevenir acciones destructivas) — esa es la respuesta esperada en el examen, aunque en la práctica funcione como una lista de "pre-aprobados".
- **`argument-hint`** — muestra en el autocompletado qué inputs se esperan cuando se invoca el skill, guiando al desarrollador a proveer los parámetros correctos.

> [!note] `allowed-tools`: pre-aprobación y restricción son la misma moneda
> No lo pienses como "dos comportamientos distintos". Al listar explícitamente qué herramientas puede usar un skill, automáticamente excluyes todas las demás — por eso el examen enmarca `allowed-tools` como una forma de **restringir** el acceso, no solo de agilizar permisos.

### 4. Skills vs. `CLAUDE.md`: cuándo usar cada uno

- **Skills**: cargan **bajo demanda**, para flujos de trabajo específicos de una tarea (ej. `/review`, `/deploy`, un skill de análisis de codebase).
- **`CLAUDE.md`**: carga **automáticamente y siempre**, para estándares universales del proyecto.
- Nunca coloques procedimientos específicos de una tarea en `CLAUDE.md`, ni convenciones que deben aplicar siempre dentro de un skill — cada mecanismo existe para el extremo opuesto de esa distinción.

### 5. Referencia rápida: dónde colocar cada comando personalizado

| Necesidad                                          | Ubicación canónica                           | También funciona                 | Alcance                       |
| -------------------------------------------------- | -------------------------------------------- | -------------------------------- | ----------------------------- |
| Comando de equipo                                  | `.claude/skills/<nombre>/SKILL.md`           | `.claude/commands/<nombre>.md`   | Proyecto (compartido vía git) |
| Comando de equipo con configuración de frontmatter | `.claude/skills/<nombre>/SKILL.md`           | `.claude/commands/<nombre>.md`   | Proyecto (compartido vía git) |
| Comando personal                                   | `~/.claude/skills/<nombre>/SKILL.md`         | `~/.claude/commands/<nombre>.md` | Usuario (no compartido)       |
| Estándares universales                             | `.claude/CLAUDE.md` o `CLAUDE.md` en la raíz | —                                | Proyecto (siempre cargado)    |
| Preferencias personales                            | `~/.claude/CLAUDE.md`                        | —                                | Usuario (no compartido)       |

## Trampas de examen

> [!warning] Trampa 1 — Poner un archivo Markdown plano directamente dentro de `.claude/skills/`
> Un skill requiere **estructura de directorio**: `.claude/skills/<nombre>/SKILL.md`. Poner un `.md` suelto directamente en `.claude/skills/` (sin la carpeta contenedora) no crea un skill válido. Si quieres un archivo plano sin subcarpeta, esa es la función de `.claude/commands/<nombre>.md`, no de la ruta de skills.

> [!warning] Trampa 2 — Guardar un comando compartido en la ruta de usuario
> Colocar un comando destinado a todo el equipo en `~/.claude/commands/` o `~/.claude/skills/` es el mismo error que colocar convenciones de equipo en el `CLAUDE.md` de usuario: nunca llega a los demás desarrolladores porque el nivel de usuario no se comparte vía git. Para que todos lo reciban al clonar o hacer pull, debe vivir en `.claude/`.

> [!warning] Trampa 3 — Tratar un skill como si fuera guía "siempre activa"
> Un skill se activa **solo cuando se invoca** (por comando o por coincidencia de intención con descubrimiento automático), nunca de forma persistente como `CLAUDE.md`. Si una convención debe aplicar sin excepción en cada interacción, un skill es la herramienta equivocada.

> [!warning] Trampa 4 — Omitir `context: fork` en operaciones verbosas
> Un skill que produce salida extensa (análisis de codebase, brainstorming con múltiples alternativas) sin `context: fork` contamina el presupuesto de tokens de la conversación principal con contenido exploratorio o intermedio que no necesita quedar ahí. La corrección es aislar esa salida en un sub-agente con `context: fork`.

## En una frase

Un **skill** es un procedimiento nombrado y reutilizable que Claude Code carga **bajo demanda** (vía `.claude/skills/<nombre>/SKILL.md`, canónico, o `.claude/commands/<nombre>.md`, plano), con alcance de **proyecto** o **usuario**, y cuyo comportamiento fino se ajusta con `context: fork`, `allowed-tools` y `argument-hint` — todo lo opuesto a `CLAUDE.md`, que se carga siempre.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
