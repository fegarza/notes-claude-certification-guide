Cuestionario completo de repaso de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revelar.

## Argumento central

> [!question]- ¿Cuál es la diferencia fundamental entre un skill y `CLAUDE.md`?
> Un skill se carga **bajo demanda** (cuando se invoca o cuando su descripción coincide con la intención del usuario); `CLAUDE.md` se carga **siempre, automáticamente**, en cada sesión.

> [!question]- ¿Qué pregunta deberías hacerte para decidir si algo va en un skill o en `CLAUDE.md`?
> Si la instrucción debe aplicar **siempre**, va en `CLAUDE.md`; si solo aplica **cuando se invoca** un flujo de trabajo específico, va en un skill.

## Sistema unificado de skills

> [!question]- ¿Cuáles son las dos estructuras de archivo que crean un comando `/deploy` equivalente?
> `.claude/skills/deploy/SKILL.md` (canónica, basada en directorio) y `.claude/commands/deploy.md` (archivo plano, compatibilidad hacia atrás).

> [!question]- ¿Por qué se prefiere la ruta de skills (`.claude/skills/`) sobre la ruta de comandos planos?
> Porque soporta descubrimiento automático (Claude puede cargar el skill cuando coincide con la intención del usuario, sin invocación explícita) y tiene precedencia si hay conflicto de nombres entre ambas estructuras.

> [!question]- ¿Qué pasaría si colocas un archivo `.md` suelto directamente dentro de `.claude/skills/`, sin carpeta contenedora?
> No sería un skill válido — la ruta de skills requiere estructura de directorio (`.claude/skills/<nombre>/SKILL.md`). Un archivo plano sin carpeta corresponde a `.claude/commands/<nombre>.md`, no a `.claude/skills/`.

## Alcance: proyecto vs. usuario

> [!question]- ¿Qué diferencia hay entre un skill en `.claude/` y uno en `~/.claude/`?
> El de `.claude/` (nivel proyecto) se versiona con git y llega a todo el equipo al clonar o hacer pull; el de `~/.claude/` (nivel usuario) es personal y no se comparte.

> [!question]- ¿Qué pasaría si guardas un comando destinado a todo el equipo en `~/.claude/commands/` en vez de `.claude/commands/`?
> Solo tú lo tendrías disponible; el resto del equipo, al clonar el repo, nunca lo recibiría, porque el nivel de usuario no viaja con el control de versiones.

> [!question]- ¿Cómo crearías una variante personal de un skill compartido sin afectar a tus compañeros de equipo?
> Creando el skill en tu nivel de usuario (`~/.claude/skills/`) con un **nombre distinto** al del skill de equipo, en vez de editar directamente el `SKILL.md` que vive en `.claude/` (que sí afectaría a todos).

> [!question]- ¿Por qué editar directamente el `SKILL.md` del proyecto para agregar una preferencia personal es un error?
> Porque ese archivo vive en `.claude/` (nivel proyecto) y se comparte vía git; el cambio se propagaría a todo el equipo en el siguiente pull, no solo a quien quería la personalización.

## Opciones críticas de frontmatter

> [!question]- ¿Qué problema resuelve `context: fork`?
> Aísla la salida verbosa de un skill (por ejemplo, listados extensos de archivos o extractos de código) en un sub-agente, evitando que consuma el presupuesto de tokens de la conversación principal.

> [!question]- Menciona dos tipos de tareas donde `context: fork` es especialmente importante.
> Análisis de codebase (genera listados extensos de archivos y código) y brainstorming (produce contexto exploratorio que no debe ensuciar la conversación principal).

> [!question]- ¿Qué pasaría si omites `context: fork` en un skill que analiza cientos de archivos?
> Toda la salida detallada del análisis quedaría en la conversación principal, consumiendo el presupuesto de tokens con contenido intermedio que no aporta valor una vez que el análisis termina.

> [!question]- ¿Cómo describe la guía de certificación lo que hace `allowed-tools`: como "pre-aprobar" herramientas o como "restringir" el acceso?
> Como **restringir** el acceso a herramientas — aunque en la práctica funcione pre-aprobando una lista, el examen lo enmarca explícitamente como un mecanismo de restricción (todo lo que no está en la lista queda excluido).

> [!question]- Da un ejemplo de por qué `allowed-tools` importa para prevenir acciones destructivas.
> Un skill que solo necesita leer el codebase (por ejemplo, listando solo `Read`, `Grep`, `Glob`) no tiene acceso a `Edit` ni `Write`, así que aunque intentara modificar archivos, no podría hacerlo — la restricción de herramientas actúa como salvaguarda.

> [!question]- ¿Para qué sirve `argument-hint`?
> Muestra en el autocompletado qué inputs espera el skill, guiando al desarrollador a proveer los parámetros correctos cuando invoca el comando sin argumentos.

## Skills vs. `CLAUDE.md` en la práctica

> [!question]- Un equipo quiere que el formato de mensajes de commit se siga siempre, y por separado quiere un flujo para generar el changelog de un release. ¿Dónde va cada uno?
> El formato de commits (aplica siempre) va en `CLAUDE.md`. El flujo de generar el changelog (bajo demanda, solo cuando se prepara un release) va en un skill o comando.

> [!question]- ¿Por qué no conviene meter un procedimiento bajo demanda (como generar un changelog) dentro de `CLAUDE.md`, aunque también lo use todo el equipo?
> Porque `CLAUDE.md` se carga en **cada sesión**, incluso cuando ese procedimiento no se necesita — es contexto desperdiciado la mayoría del tiempo. Un skill solo consume contexto cuando se invoca.

---

> [!tip] Repasa el resumen completo en [[1 resumen]], aplica estos conceptos en [[2 example]] y evalúate estilo examen en [[4 test]]
