# Resultados — Tutor socrático: CLAUDE.md Hierarchy, Scoping, and Modular Organisation

> [!info] Sesión evaluada el 2026-09-26
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 78/100**

De 15 preguntas: 11 correctas, 2 parciales y 2 incorrectas. Dominas el núcleo del tema: la jerarquía de tres niveles, el orden de carga, que los archivos se concatenan y se resuelven de forma arbitraria, qué se versiona y qué no, que `/memory` y `/context` son solo diagnóstico, y que `CLAUDE.md` es guía y no aplicación determinística. Los huecos están en la periferia: la cadena de precedencia de `settings.json` y por qué ese archivo sí obliga, y cómo usar `@` para que cada paquete importe solo lo que le aplica.

## Conceptos con buen dominio

- **Los tres niveles y su orden de carga**: identificaste usuario → proyecto → directorio sin dudar.
- **Concatenación vs. precedencia (trampa central)**: explicaste bien que "más específico" no gana y que ante una contradicción Claude elige arbitrariamente.
- **Qué se versiona con git**: proyecto y directorio sí, usuario no. Además mencionaste `CLAUDE.local.md` por iniciativa propia.
- **`CLAUDE.local.md` vs. nivel usuario**: distinguiste "preferencias solo para este repo" de "preferencias para todos mis proyectos".
- **Carga eager de `@`**: tenías claro que importar desde otro archivo no significa carga bajo demanda.
- **Glob en `.claude/rules/` para convenciones que cruzan carpetas**: elegiste el mecanismo correcto para los `*.test.tsx` dispersos.
- **`/memory` vs. `/context`**: diferenciaste "disponible" de "cargado realmente" y sabes que ninguno dispara la carga.
- **Guía vs. aplicación y hooks**: reconociste que `CLAUDE.md` no es determinístico y propusiste hooks.
- **Trampa 1 (comportamiento inconsistente entre devs)**: diagnosticaste la causa (convenciones en el nivel usuario) y la solución (moverlas a proyecto).

## Áreas que necesitan revisión

### Cadena de precedencia de `settings.json` y por qué sí obliga

- **Qué pasó:** Nombraste `settings.json` como alternativa a `CLAUDE.md`, pero no explicaste qué lo hace diferente (quién lo aplica). Al preguntarte por la cadena de precedencia, respondiste solo "usuario y proyecto, gana el más específico". Omitiste managed policy y local, y trasladaste la lógica de "gana el más específico", que tampoco aplica a `CLAUDE.md`. En la pregunta de refuerzo (managed policy vs. `settings.local.json`) sí respondiste bien.
- **Contenido del resumen:**
  > - **`settings.json`**: el cliente lo aplica sin importar lo que Claude decida (cadena de precedencia: managed policy > local > project > user, y managed siempre gana).
  > - **Hooks**: se disparan en puntos fijos del ciclo de vida, sin depender de la interpretación de Claude.
- **Sección:** [[1 resumen]] → "6. Guía vs. aplicación garantizada: cuándo NO confiar en `CLAUDE.md`"
- **Analogía para reforzarlo:** `CLAUDE.md` es como pedirle a un taxista "por favor no tome la autopista": casi siempre te hace caso, pero decide él. `settings.json` es el GPS de la flotilla, que tiene la autopista bloqueada: el taxista no puede tomarla aunque quiera. Y la managed policy es la regla de la central de la flotilla, que ningún conductor puede desactivar desde su propio teléfono.

### Organización modular con `@` por paquete

- **Qué pasó:** Ante un monorepo con estándares compartidos (`naming.md`) y específicos (`error-handling.md` solo para `api`), respondiste que no usarías ningún `@` "porque son reglas distintas". Justo ese es el caso de uso de `@`: cada paquete importa solo los estándares que le aplican.
- **Contenido del resumen:**
  > - Dentro de un `CLAUDE.md` puedes referenciar otros archivos con `@ruta/al/archivo.md` para no tener un solo archivo gigante
  > - Cada paquete/directorio puede importar **solo los estándares que le aplican**, en vez de duplicar contenido o forzar a que todos lean todo.
- **Sección:** [[1 resumen]] → "4. Organización modular: `@` (sin la palabra "import") y `.claude/rules/`"
- **Analogía para reforzarlo:** Es como una playlist. Las canciones (`naming.md`, `error-handling.md`) viven una sola vez en tu biblioteca, y cada playlist (`CLAUDE.md` de cada paquete) solo apunta a las que quiere. No copias el archivo MP3 en cada playlist, y la playlist de "gym" no tiene que incluir las canciones de "dormir".

### Estructura de `.claude/rules/`

- **Qué pasó:** Identificaste "rules" como la alternativa a un `CLAUDE.md` gigante, pero no describiste cómo se estructura. En la pregunta siguiente sí aplicaste bien el glob del frontmatter, así que el hueco es menor.
- **Contenido del resumen:**
  > - Alternativa a un `CLAUDE.md` monolítico: la carpeta **`.claude/rules/`**, donde cada archivo cubre un tema específico (p. ej. `testing.md`, `api-conventions.md`, `deployment.md`) en vez de amontonar todo en un solo documento.
- **Sección:** [[1 resumen]] → "4. Organización modular: `@` (sin la palabra "import") y `.claude/rules/`"
- **Analogía para reforzarlo:** Pasar de un cuaderno único donde anotas todas las materias revueltas a un archivero con una carpeta por materia: "Testing", "API", "Deployment". Cada carpeta trata un solo tema.

## Siguiente paso recomendado

Repasa la sección 6 del resumen (cadena **managed policy > local > project > user** y por qué el cliente aplica `settings.json`) y la sección 4 (importar con `@` solo lo que aplica a cada paquete). Después ya estás listo para [[4 test]].
