Cuestionario completo de repaso de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revelar.

## Argumento central

> [!question]- ¿Por qué se dice que `CLAUDE.md` es "guía" y no "aplicación garantizada"?
> Porque todos los archivos encontrados se **concatenan** en el mismo contexto en vez de sobreescribirse entre sí, y si hay contradicciones, Claude puede resolverlas arbitrariamente. No hay ningún mecanismo que fuerce el cumplimiento de una regla solo por estar escrita ahí.

> [!question]- ¿Qué diferencia hay entre "orden de carga" y "precedencia" en el contexto de `CLAUDE.md`?
> El orden de carga determina en qué secuencia se leen los archivos (de más general a más específico), pero no determina cuál instrucción "gana" si hay conflicto. No existe precedencia estricta como en CSS.

## Los tres niveles de la jerarquía

> [!question]- ¿Cuáles son los tres niveles de la jerarquía de `CLAUDE.md` y dónde vive cada uno?
> Usuario (`~/.claude/CLAUDE.md`), proyecto (`.claude/CLAUDE.md` o `CLAUDE.md` en la raíz), y directorio (`CLAUDE.md` dentro de una subcarpeta específica).

> [!question]- ¿Cuál de los tres niveles es el único que NO se comparte automáticamente con el resto del equipo al clonar el repo, y por qué?
> El nivel de usuario. Vive fuera del repositorio (en el home del usuario) y nunca se versiona con git, así que cada persona tiene el suyo por separado.

> [!question]- Si quieres que una convención aplique solo al paquete `/packages/api` y no al resto del repo, ¿en qué nivel la pones?
> En el nivel de directorio: un `CLAUDE.md` dentro de `/packages/api/`.

> [!question]- ¿Qué pasaría si pones una convención de equipo (por ejemplo, el formato de mensajes de commit) en tu `CLAUDE.md` de nivel usuario en vez del de proyecto?
> Solo tú la seguirías; el resto del equipo, al clonar el repo, nunca la recibiría, porque el nivel de usuario no se comparte vía git. Es exactamente el escenario típico de "comportamiento inconsistente entre desarrolladores".

## Orden de carga vs. precedencia

> [!question]- ¿Los archivos de `CLAUDE.md` se sobreescriben entre niveles, o se combinan?
> Se combinan (concatenan) todos en el mismo contexto; ninguno "borra" al otro.

> [!question]- Si el `CLAUDE.md` de proyecto dice "usa comillas dobles" y el de directorio dice "usa comillas simples", ¿cuál gana?
> Ninguno gana de forma determinística. Ambos se cargan y, ante la contradicción, Claude puede elegir cualquiera de las dos arbitrariamente.

> [!question]- ¿Por qué decir "los archivos más específicos se leen al final" no es lo mismo que decir "los archivos más específicos tienen prioridad"?
> Porque "leerse al final" solo describe el orden de la secuencia de carga, no un mecanismo de sobreescritura. Todo termina concatenado en el mismo contexto sin que el orden determine quién "gana" en un conflicto.

## `CLAUDE.local.md`

> [!question]- ¿Qué problema resuelve `CLAUDE.local.md` que no resuelve tener solo `CLAUDE.md` de proyecto?
> Permite tener ajustes personales dentro de un repo compartido (por ejemplo, tu estilo preferido de explicaciones) sin mezclarlos con las reglas del equipo ni subirlos accidentalmente a git.

> [!question]- ¿Por qué la mayoría de los equipos agrega `CLAUDE.local.md` a `.gitignore`?
> Porque el sufijo `.local` indica que ese contenido es personal y no debe compartirse; si se subiera a git, dejaría de ser "local" y contaminaría la configuración del equipo con preferencias individuales.

## Organización modular: `@` y `.claude/rules/`

> [!question]- ¿Qué hace la sintaxis `@ruta/archivo.md` dentro de un `CLAUDE.md`?
> Importa el contenido de ese archivo e lo inserta inline, en el momento en que se lee el `CLAUDE.md` que lo referencia — como si lo hubieras copiado y pegado ahí mismo.

> [!question]- ¿La carga de un archivo con `@` es condicional (solo si se necesita) o siempre ocurre al leer el archivo que lo referencia?
> Siempre ocurre — es una carga "eager" (ansiosa) e incondicional, no una carga diferida ni condicional.

> [!question]- ¿Cuándo conviene usar `.claude/rules/` en vez de seguir agregando secciones a un único `CLAUDE.md`?
> Cuando el `CLAUDE.md` se está volviendo difícil de mantener por mezclar muchos temas distintos; dividirlo en archivos por tema (`testing.md`, `api-conventions.md`, `deployment.md`) da más claridad que un solo archivo monolítico.

> [!question]- ¿Qué diferencia hay entre resolver el problema de "convenciones dispersas por todo el repo, sin importar la carpeta" con `CLAUDE.md` de directorio vs. con `.claude/rules/`?
> Un `CLAUDE.md` de directorio solo cubre esa carpeta hacia abajo, así que no sirve si los archivos afectados están repartidos en carpetas distintas. `.claude/rules/` puede aplicar convenciones por patrón de archivo (independiente de en qué carpeta viva cada uno), lo que lo hace más adecuado para ese caso.

## Comandos de diagnóstico: `/memory` y `/context`

> [!question]- ¿Qué muestra `/memory`?
> Qué archivos de configuración están disponibles para la sesión actual.

> [!question]- ¿Qué muestra `/context` que `/memory` no muestra?
> Qué se cargó realmente en el contexto de la sesión actual (bajo la sección de archivos de memoria), no solo qué archivos existen como candidatos.

> [!question]- ¿Correr `/memory` o `/context` provoca que se carguen archivos nuevos?
> No. Ambos son comandos de solo diagnóstico; la carga de archivos ya ocurrió automáticamente antes, según la ubicación de cada archivo.

> [!question]- ¿Cómo usarías `/memory` o `/context` para diagnosticar por qué dos desarrolladores tienen comportamiento distinto con el mismo repo?
> Pidiendo a cada uno que corra el comando en su propia sesión y comparando la lista de archivos cargados; si uno tiene un archivo de usuario que el otro no tiene, ahí está la fuente de la inconsistencia.

## Guía vs. aplicación garantizada

> [!question]- ¿Por qué escribir "esto es obligatorio" en `CLAUDE.md` no garantiza que Claude lo cumpla siempre?
> Porque `CLAUDE.md` es texto que influye en el comportamiento, no un mecanismo de aplicación forzosa; Claude puede desviarse de esa instrucción según el contexto o la interpretación.

> [!question]- ¿Qué dos mecanismos existen para reglas que deben cumplirse sí o sí, sin depender de la interpretación de Claude?
> `settings.json` (aplicado por el cliente, con una cadena de precedencia managed > local > project > user) y los hooks (atados a puntos fijos del ciclo de vida).

> [!question]- ¿Cuándo usarías `settings.json`/hooks en vez de una instrucción en `CLAUDE.md`?
> Cuando la regla no puede tener excepciones y necesitas que se cumpla de forma determinística (por ejemplo, bloquear un commit si fallan los tests), en vez de simplemente esperar que Claude decida seguir la instrucción escrita.

---

> [!tip] Repasa el resumen completo en [[1 resumen]], aplica estos conceptos en [[2 example]] y evalúate estilo examen en [[4 test]]
