## Explícamelo como si tuviera 5 años

Imagina que Claude Code es un empleado nuevo que llega a trabajar a una oficina con varios pisos. Antes de sentarse a trabajar, lee **notas pegadas en distintos lugares**: una nota en su casa (sus propias manías, nadie más las ve), una nota en la entrada del edificio (las reglas de toda la empresa, las ve todo el equipo) y una nota pegada en la puerta de un departamento específico (las reglas de ese departamento nada más).

El empleado **lee las tres notas, una tras otra**, y trata de seguir todas. Si dos notas se contradicen, no hay un jefe que decida cuál gana — el empleado improvisa. Por eso estas notas son **sugerencias fuertes, no un candado**. Si de verdad quieres que una regla se cumpla sí o sí, no la escribes en una nota: la pones en el reglamento oficial de la empresa (`settings.json`) o contratas a un guardia que la haga cumplir (un *hook*).

## Argumento central

`CLAUDE.md` implementa un **sistema de contexto en capas** (usuario → proyecto → directorio) que se **concatena, no se sobreescribe**: todos los archivos encontrados se cargan juntos en el contexto, en orden de más general a más específico. Por eso `CLAUDE.md` funciona como **guía de comportamiento, no como aplicación garantizada** — para reglas que deben cumplirse sí o sí existen otros mecanismos (`settings.json`, hooks).

> [!note] Idea clave
> "Más específico" no significa "gana la discusión". Solo significa "se lee después". Si dos instrucciones se contradicen, Claude puede elegir cualquiera de las dos arbitrariamente.

## Conclusiones

### 1. Los tres niveles de la jerarquía

| Nivel | Ubicación | Alcance | ¿Se comparte con el equipo? |
|---|---|---|---|
| Usuario | `~/.claude/CLAUDE.md` | Solo ese usuario, en cualquier proyecto | No — vive fuera del repo, no se versiona con git |
| Proyecto | `.claude/CLAUDE.md` o `CLAUDE.md` en la raíz | Todo el repositorio | Sí — se versiona y llega automáticamente a quien clona el repo |
| Directorio | `CLAUDE.md` en una subcarpeta | Solo esa subcarpeta hacia abajo | Sí, pero solo afecta a quien trabaja dentro de esa carpeta |

**Analogía**: el nivel de usuario es tu libreta personal; el de proyecto es el manual del equipo que va en el repo; el de directorio es la hoja pegada en la puerta de un paquete/módulo específico (por ejemplo, `/packages/api/CLAUDE.md` con convenciones solo de esa API).

### 2. Orden de carga vs. precedencia (la trampa central del tema)

- Los archivos se cargan **de alcance más amplio al más específico**: usuario → proyecto → directorio. Lo que está más cerca de donde lanzaste Claude Code se lee al final.
- Pero cargarse al final **no equivale a ganar**. Todo el contenido se **concatena en el mismo contexto**; no hay un mecanismo que borre o anule instrucciones previas.
- Si dos reglas de niveles distintos se contradicen, la fuente es explícita: *"if two rules contradict each other, Claude may pick one arbitrarily"*.
- Consecuencia práctica: `CLAUDE.md` es una herramienta de **guía de comportamiento**, no de **aplicación determinística**.

### 3. `CLAUDE.local.md`: tus manías, en el repo, sin subirlas a git

- Vive junto al `CLAUDE.md` de proyecto, en el mismo nivel.
- Por convención se agrega a `.gitignore` — el sufijo `.local` marca "esto no se comparte".
- Sirve para ajustes personales dentro de un repo compartido: el `CLAUDE.md` normal son las reglas del equipo; el `CLAUDE.local.md` de al lado son tus propias costumbres para ese repo en particular.

### 4. Organización modular: `@` (sin la palabra "import") y `.claude/rules/`

- Dentro de un `CLAUDE.md` puedes referenciar otros archivos con `@ruta/al/archivo.md` para no tener un solo archivo gigante:

```
Coding standards:
@./standards/naming-conventions.md
@./standards/error-handling.md
@./standards/testing-requirements.md
```

- Estas referencias se cargan **de forma eager (ansiosa) e inline**: en el momento de leer el `CLAUDE.md`, el contenido del archivo importado se inserta ahí mismo. No hay carga condicional ni diferida con `@`.
- Cada paquete/directorio puede importar **solo los estándares que le aplican**, en vez de duplicar contenido o forzar a que todos lean todo.
- Alternativa a un `CLAUDE.md` monolítico: la carpeta **`.claude/rules/`**, donde cada archivo cubre un tema específico (p. ej. `testing.md`, `api-conventions.md`, `deployment.md`) en vez de amontonar todo en un solo documento. La carga condicional por ruta de archivo (frontmatter YAML con glob patterns) es el detalle fino de este mecanismo, y se profundiza en el tema de reglas específicas por ruta.

### 5. Comandos de diagnóstico: `/memory` y `/context`

- **`/memory`**: muestra **qué archivos de configuración están disponibles**.
- **`/context`**: revela **qué se cargó realmente** en la sesión actual, bajo la sección "Memory files".
- Ninguno de los dos **activa** la carga de nada — son **solo diagnóstico**. La carga ocurre automáticamente según la ubicación de los archivos, no porque se haya corrido un comando.

> [!warning] Trampa de examen: "correr `/memory` carga la configuración"
> Falso. `/memory` y `/context` son ventanas para **observar** qué ya se cargó (o qué se podría cargar), no interruptores que disparan la carga. La carga es automática y depende de dónde vive el archivo, no de si ejecutaste algún comando.

### 6. Guía vs. aplicación garantizada: cuándo NO confiar en `CLAUDE.md`

- `CLAUDE.md` moldea el comportamiento de Claude, pero **no es una capa de aplicación forzosa**.
- Si necesitas que una regla se cumpla siempre (sin depender de que Claude "decida" seguirla), usa:
  - **`settings.json`**: el cliente lo aplica sin importar lo que Claude decida (cadena de precedencia: managed policy > local > project > user, y managed siempre gana).
  - **Hooks**: se disparan en puntos fijos del ciclo de vida, sin depender de la interpretación de Claude.

## Trampas de examen

> [!warning] Trampa 1 — Nuevo integrante del equipo recibe comportamiento inconsistente
> Escenario típico: dos desarrolladores en el mismo repo y misma rama, pero Claude Code se comporta distinto con cada uno. La causa casi siempre es que las convenciones viven en el `CLAUDE.md` de **usuario** (`~/.claude/CLAUDE.md`) de uno de ellos, en vez de estar en el `CLAUDE.md` de **proyecto**. El nivel de usuario nunca se compartió vía git, así que el nuevo integrante simplemente no lo tiene.

> [!warning] Trampa 2 — "Lo más específico siempre gana"
> Es tentador pensar que un `CLAUDE.md` de directorio anula al de proyecto, como pasaría con la herencia de estilos CSS. No es así: los archivos **se concatenan**, y ante una contradicción la resolución es **arbitraria**, no determinista por especificidad.

> [!warning] Trampa 3 — Usar un `CLAUDE.md` de directorio para convenciones que cruzan varias carpetas
> Un `CLAUDE.md` de directorio solo aplica a esa carpeta hacia abajo. Si la convención debe aplicar a archivos dispersos en todo el repo (p. ej. todos los `*.test.tsx` sin importar en qué carpeta estén), un `CLAUDE.md` por carpeta no escala — ahí es donde entran las reglas con patrones glob de `.claude/rules/`.

> [!warning] Trampa 4 — Confundir `/memory` o `/context` con "cargar" configuración
> Ambos comandos son de solo lectura/diagnóstico. Ninguno provoca que se lea un archivo nuevo; solo reportan el estado de lo que ya se cargó.

> [!warning] Trampa 5 — Esperar que `CLAUDE.md` "obligue" una regla como lo haría `settings.json`
> `CLAUDE.md` influye en el comportamiento, pero Claude puede desviarse. Para reglas mandatorias e innegociables, la herramienta correcta es `settings.json` (aplicado por el cliente) o un hook (atado al ciclo de vida), no una instrucción en texto dentro de `CLAUDE.md`.

## En una frase

`CLAUDE.md` carga contexto en capas (usuario → proyecto → directorio) que **se concatenan, no se sobreescriben**, por lo que es una herramienta de **guía**, no de **aplicación garantizada** — eso último se logra con `settings.json` o hooks.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
