Repaso de [[1 resumen]].

## Cómo funcionan los rule files

> [!question]- ¿Dónde viven los path-specific rules y qué extensión de archivo usan?
> En el directorio `.claude/rules/`, como archivos `.md` (Markdown con YAML frontmatter).

> [!question]- ¿Qué campo del frontmatter determina cuándo se activa una regla, y qué tipo de valor tiene?
> El campo `paths`: un array de glob patterns (strings), ej. `["**/*.test.tsx"]`.

> [!question]- ¿Qué dispara la carga de una rule file — invocación manual o algo automático?
> Se carga automáticamente cuando Claude edita/lee un archivo cuyo path matchea alguno de los glob patterns declarados — no requiere invocación manual.

> [!question]- Si tienes un rule file con `paths: ["src/api/**/*"]`, ¿se carga al editar `src/components/Button.tsx`?
> No, porque ese path no matchea el glob pattern; la regla solo se activa con archivos dentro de `src/api/`.

## Ventaja de eficiencia de tokens

> [!question]- ¿Por qué un `CLAUDE.md` raíz es menos eficiente en tokens que un path-specific rule para convenciones de un tipo de archivo?
> Porque el `CLAUDE.md` raíz se carga en cada sesión sin importar qué se edite, mientras que el rule file solo carga cuando el archivo editado matchea su patrón — evita cargar contexto irrelevante.

> [!question]- ¿Cómo se verifica en la práctica que una regla realmente se cargó (o no se cargó) para un archivo dado?
> Con el comando `/context`, que muestra qué configuración está cargada en la sesión actual.

> [!question]- Si editas un archivo `.tf` y luego usas `/context`, ¿qué esperarías ver si tu regla de testing está bien configurada?
> Que la regla de Terraform aparezca cargada, y que la regla de testing (y la de API) NO aparezcan.

## Por qué no alcanza con directory-level CLAUDE.md

> [!question]- ¿Qué tipo de convención resuelve bien un `CLAUDE.md` de directorio, y cuál no?
> Resuelve bien convenciones acotadas a un solo paquete/directorio. No resuelve bien convenciones de un tipo de archivo disperso en muchos directorios (ej. tests junto a su código).

> [!question]- ¿Por qué usar `CLAUDE.md` de directorio para test files esparcidos en 50+ carpetas es un problema de mantenimiento?
> Porque requiere duplicar el mismo contenido en cada una de esas carpetas: cada carpeta nueva necesita su copia, cualquier cambio hay que propagarlo a todas, y con el tiempo las copias inevitablemente se desalinean (drift).

> [!question]- ¿Cómo resume la guía la ventaja de un path-specific rule frente a esa duplicación?
> "Un archivo, un patrón, cobertura universal" — un solo rule file con un glob pattern cubre todos los directorios sin duplicar nada.

> [!question]- ¿Qué pasaría si agregas una nueva carpeta con tests después de haber configurado el rule file de testing?
> Nada especial que hacer: como el glob pattern (`**/*.test.tsx`) no depende del directorio, la nueva carpeta queda cubierta automáticamente sin tocar ningún archivo de configuración.

## Cuándo usar cada mecanismo

> [!question]- ¿Cuándo conviene un `CLAUDE.md` raíz en vez de un path-specific rule?
> Cuando la convención es universal y debe aplicar a todo el código, sin importar el tipo de archivo.

> [!question]- ¿Cuándo conviene un `CLAUDE.md` de directorio en vez de un path-specific rule?
> Cuando la convención es específica de un solo paquete/directorio concreto, no de un tipo de archivo repartido por el proyecto.

> [!question]- ¿Cuál es la diferencia clave entre un path-specific rule y un Skill en cuanto a cuándo se cargan?
> El rule se carga automáticamente y de fondo cada vez que se edita/lee un archivo que matchea su path — moldea cada edición sin invocación. El skill se carga on-demand, como un workflow de tarea, disparado por intención del modelo o invocación explícita del usuario.

> [!question]- Si necesitas que una convención se aplique siempre y automáticamente según el tipo de archivo, sin que nadie tenga que invocarla, ¿usarías un skill o una regla?
> Una regla (path-specific rule) — es justamente el mecanismo diseñado para activación automática y determinística basada en el path del archivo.

> [!question]- ¿Qué tienen en común skills y rules en cuanto a su frontmatter?
> Ambos pueden usar un campo `paths` en el frontmatter para auto-activarse según el archivo — pero eso no los hace equivalentes: la garantía de carga automática y siempre-activa es propia de las rules, no de los skills.
