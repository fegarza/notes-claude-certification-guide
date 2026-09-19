# C:\notes — Vault de Obsidian

Este directorio es un **vault de Obsidian**, no un proyecto de código. Los archivos `.md` son notas personales, organizadas en carpetas numeradas por tema (ej. `1 Agentic Architecture and orchestation`, `2 Tool Design & MCP Integration`). También puede haber archivos `.base` (vistas de Obsidian Bases).

Estoy conectado a este vault a través de **Claudian**, un plugin de Obsidian que me embebe como colaborador de escritura dentro de la app, usando el vault como directorio de trabajo.

## Cómo ayudar aquí

- El objetivo principal es **escribir, editar y organizar notas en Markdown**, no producir software. Evita sugerir estructuras de "proyecto de código" (package.json, tests, builds, etc.) salvo que se pida explícitamente.
- Respeta y usa la sintaxis de Obsidian: `[[wikilinks]]`, `#tags`, embeds `![[nota]]`, callouts (`> [!note]`), y frontmatter YAML para propiedades.
- Al crear o editar notas, sigue la convención de carpetas numeradas por área temática que ya existe en el vault.
- No crees archivos nuevos de forma proactiva (notas, carpetas, índices) a menos que se pida.

## Plugins instalados (afectan cómo se escriben las notas)

- **Claudian** — soy yo, embebido en el vault; me da lectura/escritura de archivos, búsqueda y comandos multi-paso sobre las notas.
- **Mermaid Tools** — mejora la experiencia con diagramas Mermaid (barra visual de elementos). Si una nota se beneficia de un diagrama (arquitectura, flujos, loops de agentes), usa bloques ` ```mermaid ` en vez de describir el diagrama en prosa.
- **Floating Headings** — muestra un outline flotante y colapsable de los encabezados de la nota. Esto hace valioso usar una jerarquía de encabezados (`##`, `###`) clara y consistente, ya que sirve de navegación real.
- **Collapsible Code Blocks** — los bloques de código son colapsables y con scroll. Está bien usar bloques de código largos (snippets, configs, ejemplos) sin miedo a que "ensucien" la nota.
- **Fast Text Color** — permite colorear texto con sintaxis propia (formato tipo `==texto==` extendido con color). Si se pide resaltar texto con color, usa la sintaxis de este plugin en vez de HTML/CSS inline.

## Estructura observada

- Carpetas de dominio, prefijadas con número (orden de lectura/prioridad): `1 Agentic Architecture and orchestation`, `2 Tool Design & MCP Integration`, `3 Claude Code Configuration & Workflows`, `4 Prompt Engineering & Structured Output`, `5 Context Management & Reliability`.
- Dentro de cada carpeta de dominio, **una subcarpeta por tema/módulo** (ej. `1 Agentic loop/`, numerada según el orden del módulo dentro del dominio). Ver "Estructura por tema" más abajo para lo que va dentro de cada una.
- Archivos `.base` en la raíz para vistas tipo base de datos sobre las notas.

## Fuente del contenido

Estas notas están basadas enteramente en el temario de **https://claudecertificationguide.com/learn** (el programa de certificación "Claude Certified Architect"). Los 5 dominios y sus módulos numerados corresponden 1:1 a la estructura de esa página. Al expandir o escribir contenido en estas notas, mantén la fidelidad a esa fuente como referencia principal.

## Estructura por tema (4 archivos por módulo)

Cada tema/módulo de la guía vive en su propia **subcarpeta** dentro de la carpeta de su dominio, nombrada igual que el tema (ej. `1 Agentic Architecture and orchestation/1 Agentic loop/`). Dentro de esa subcarpeta van siempre estos 4 archivos, con prefijo numérico para forzar ese orden en el explorador de archivos de Obsidian (resumen → ejemplo → cuestionario → test):

- `1 resumen.md`
- `2 example.md`
- `3 cuestionario.md`
- `4 test.md`

Cuando se pida trabajar un tema completo, se generan/actualizan los 4. Cuando se pida específicamente "el cuestionario" o "el test", etc. (sin mencionar el resumen), se trabaja únicamente el archivo correspondiente sin tocar los demás.

**Excepción — pedir "el resumen"**: cuando se pida hacer o actualizar "el resumen" de un tema, se generan/actualizan los 4 archivos del tema (`1 resumen.md`, `2 example.md`, `3 cuestionario.md`, `4 test.md`), no solo `1 resumen.md`. El resumen es la base de la que dependen los otros 3 (el cuestionario repasa sus Conclusiones, el example las aplica en código, el test las evalúa estilo examen), así que pedir "el resumen" implica trabajar el tema completo a partir de él.

Los wikilinks internos entre estos 4 archivos deben usar el nombre completo con su número (ej. `[[1 resumen]]`, `[[2 example]]`), no el alias corto sin número.

### Convención de idioma: conceptos clave en inglés

En `1 resumen.md`, `2 example.md` y `3 cuestionario.md` (los tres archivos en español), el texto en general va en español, pero los **términos/conceptos clave de la guía** se dejan en inglés, sin traducir — tal como aparecen en la fuente (ej. `prompt chaining`, `tool use`, `hooks`, `stop_reason`, nombres de parámetros, flags, campos de API, nombres de patrones de arquitectura como "hub-and-spoke" o "Narrow Decomposition Failure"). No se traduce un término técnico solo por escribir el resumen en español — se mantiene el nombre en inglés que usa la certificación, para que el vocabulario coincida con el que aparece en el examen real (que sí es en inglés, ver `4 test.md`).

### Método ACERO para tomar y organizar apuntes (aplica a `1 resumen.md`)

Antes de redactar, filtra y clasifica el contenido de la fuente con el método **ACERO**:

- **A — Argumento**: responde ¿de qué trata el tema? ¿cuál es la afirmación de mayor peso? Puede haber más de un argumento si hay varios subtemas importantes. Es la idea central alrededor de la cual gira todo lo demás.
- **C — Conclusiones**: las divisiones/ideas que explican y sostienen el argumento — el "cómo" se desarrolla la idea central en varias ideas importantes.
- **E — Evidencia**: de menor peso que el argumento y las conclusiones, pero las complementa. Responde ¿qué más? — ejemplos, casos, estudios, datos duros, referencias.
- **R — Relleno**: todo lo que no aporta a entender el tema de cara a la certificación (repeticiones, marketing, anécdotas irrelevantes, ejercicios interactivos tipo "Build Exercise"/"Practice Scenario", etc.). Se descarta y **no** va en `1 resumen.md`.
- **O — Orden**: una vez identificados A, C y E, dales el mejor formato posible (jerarquía de encabezados, listas, tablas, callouts) para que el resumen tenga sentido y profundidad al leerlo, no solo que "ya tenga un orden".

El resumen final debe reflejar esta jerarquía: el/los Argumento(s) bien visibles, las Conclusiones como la estructura principal del cuerpo, la Evidencia como apoyo (no como protagonista), y sin nada de Relleno.

### `1 resumen.md` — para entender el tema

1. **Explicación tipo "explícamelo como si tuviera 5 años"** — lenguaje simple, sin jerga técnica sin explicar, usando analogías cotidianas para introducir el Argumento antes de dar la versión "formal".
2. **Estructura pensada para memorizar**, usando técnicas de estudio comprobadas:
   - Jerarquía clara de encabezados (aprovecha Floating Headings) para poder navegar y repasar por secciones — refleja el Orden de ACERO.
   - Chunking: cada Conclusión como un bloque pequeño y separado (3-5 ideas clave máximo por bloque), no en un solo párrafo largo.
   - Analogías y ejemplos mentales propios (no inventados ni copiados literal de la fuente) que conecten el concepto con algo ya conocido — esto es parte de la técnica Feynman: si no se puede explicar simple, no se entendió.
   - La Evidencia (ejemplos, datos, casos) se incluye como apoyo puntual de cada Conclusión, nunca como el cuerpo principal del resumen.
   - Un resumen ultra-corto al final de la explicación tipo "en una frase" (para repaso rápido/spaced repetition), que sintetice el Argumento.
   - Usa `> [!note]` u otros callouts de Obsidian para resaltar ideas clave o advertencias/errores comunes.
3. **Trampas de examen con prioridad alta**, cuando la guía las mencione:
   - No son Relleno ni Evidencia menor — trátalas como información de alto peso, casi al nivel de las Conclusiones, porque suelen ser exactamente donde falla el entendimiento superficial del tema.
   - Dales su propia sección visible (`## Trampas de examen`), no las mezcles escondidas dentro de otro párrafo.
   - Resáltalas con un callout de advertencia (`> [!warning]`) e incluye tanto el error común como el porqué es un error y cuál es la forma correcta.
   - Si la guía indica que una trampa corresponde a una pregunta de muestra específica del examen, menciónalo.
4. **100% fiel a la guía de certificación** — nada de ejemplos de código buscados fuera de la página, ni bloques de código propios, dentro de `1 resumen.md`. Si la guía en sí trae un bloque de código o snippet, ese sí se conserva (es contenido de la fuente), pero no se agrega código adicional inventado o buscado externamente en este archivo — ese código vive en `2 example.md`.
5. **No incluir los ejercicios de práctica de la guía** (ej. secciones tipo "Build Exercise", "Practice Scenario", pasos de un ejercicio guiado con dificultad/tiempo estimado, etc.) — cuenta como Relleno.
6. **No incluye el cuestionario** — el cuestionario vive en su propio archivo, `3 cuestionario.md`.
7. Al final, enlaza a los otros 3 archivos del tema con wikilinks (ej. `> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]`).

### `3 cuestionario.md` — para repasar y memorizar

Versión **completa y exhaustiva** del cuestionario de autoevaluación (más completa que la que antes vivía dentro del resumen):

- Cubre **todas** las ideas clave/Conclusiones del resumen, no solo una muestra — el objetivo es que repasar este archivo equivalga a repasar todo el tema.
- **Preguntas cortas y directas** — una idea por pregunta, sin rodeos ni redacción rebuscada, fáciles de leer y responder mentalmente en pocos segundos.
- Igual que antes: comprensión **conceptual y profunda**, nunca recordar un ejemplo o dato específico del texto (evitar "¿qué ejemplo menciona el texto sobre X?").
- Sigue usando los tipos: "¿por qué...?", "¿cuál es la diferencia entre...?", "¿qué pasaría si...?", "¿cómo se relaciona X con Y?", "¿cuándo usarías X en vez de Y?".
- Organiza las preguntas agrupadas por sub-tema/sección (mismos encabezados que el resumen), para poder repasar por bloques.
- Respuestas ocultas en callout colapsable (`> [!question]-`) para forzar recuerdo activo antes de revisar.

### `2 example.md` — aplicación práctica en código

- Es donde vive **todo el código** del tema — `1 resumen.md` no debe tener código propio.
- Formato: **explicación paso a paso**, no solo un bloque de código pegado. Cada paso combina una explicación breve en prosa de qué se está haciendo y por qué, seguida del fragmento de código correspondiente — construyendo el caso de uso incrementalmente.
- El caso de uso debe **aplicar todos los conceptos clave del resumen**, no solo uno aislado, incluyendo explícitamente las **consideraciones/trampas de examen** del resumen dentro del propio código o justo al lado del paso que aplica.
- **Preguntas trampa en código, cuando aplique**: para cada trampa de examen del resumen que tenga una forma "ingenua" de escribirse en código, muestra (en un callout o comentario) esa forma incorrecta y explica por qué sería mala idea implementarla así — no solo se explica en abstracto, se conecta directamente con el código real del ejemplo.
- El lenguaje por defecto es **Python** salvo que el tema sea específico de otro contexto (ej. YAML para GitHub Actions, Bash para CLI de Claude Code) — usa el lenguaje que tenga más sentido para el tema.
- Cuando se use la API/SDK de Claude o de terceros, basa el código en la documentación oficial real (Anthropic, MCP, etc.), no en suposiciones.
- Encabezado inicial: una línea ligando el ejemplo con el resumen del mismo tema, usando wikilink de Obsidian (ej. `Aplicación práctica de [[1 resumen]]`).

### `4 test.md` — examen de práctica estilo certificación

Simulacro de examen de opción múltiple, con el mismo estilo que las preguntas reales de la certificación Claude Certified Architect - Foundations (CCAR-F).

> [!info] Fuente autoritativa: `examguide.pdf`
> En la raíz del vault vive `examguide.pdf`, el **Exam Guide oficial** de Anthropic para la certificación (Task Statements por dominio, formato del examen, y una sección "9. Sample Questions" con preguntas reales completas y su explicación). Es la referencia principal para construir `4 test.md` — más autoritativa incluso que la web de la guía de estudio para todo lo relacionado al formato y estilo real del examen. Antes de escribir o actualizar un `4 test.md`, extraer su texto (ej. `pdftotext -layout examguide.pdf examguide.txt`) y consultarlo.

- **Este archivo va enteramente en inglés** (preguntas, opciones y explicaciones de respuesta) — es el idioma real del examen, así que practicar en inglés es parte del objetivo. Es la única excepción al español del resto de la nota; `1 resumen.md`, `2 example.md` y `3 cuestionario.md` se mantienen en español.
- **Basado en el Task Statement específico de ese tema** dentro de `examguide.pdf` (sección 6, "Detailed Objectives by Domain") — cada tema de la vault corresponde a un Task Statement numerado (ej. la carpeta `1 Agentic loop` = Task Statement 1.1, `6 CI-CD Integration` = Task Statement 3.6). Las preguntas deben evaluar específicamente el "Knowledge of" y "Skills in" listados ahí para ese Task Statement, no mezclar contenido de otros temas/dominios.
- **Formato real del examen** (sección 3 y 5 de `examguide.pdf`): son preguntas ancladas en un **escenario de producción realista** (métricas, logs, código o configuración concreta), seguidas de opciones de respuesta. La mayoría son de una sola respuesta correcta (A-D), pero el examen real también incluye ítems de **respuesta múltiple** ("multiple-response") que indican explícitamente cuántas opciones elegir (ej. "Select the two...") — está bien incluir alguno de este tipo si el tema lo amerita, siempre marcando claramente cuántas respuestas se esperan.
- Cada pregunta plantea un **caso de uso de la vida real** enfocado específicamente en este tema: una situación concreta (ej. un bug en producción, una decisión de arquitectura, un pipeline que falla), su problema, y posibles soluciones/respuestas como opciones A-D.
- Las opciones incorrectas deben ser **distractores plausibles** basados en las trampas de examen reales del tema (errores comunes, no opciones absurdas obvias) — replicando el estilo de las explicaciones reales de `examguide.pdf`, que explican no solo por qué la correcta es correcta sino por qué cada distractor está mal.
- La respuesta correcta y su explicación van ocultas en un callout colapsable (`> [!question]-` o `> [!success]-`) después de cada pregunta, para simular el examen real antes de revisar.
- Si `examguide.pdf` trae una pregunta de muestra oficial que aplique 1:1 a este tema, inclúyela (citando que es de la guía oficial) además de preguntas propias construidas siguiendo el mismo estilo — no reemplaza tener preguntas propias, las complementa.
- Este archivo sí puede basarse en casos hipotéticos construidos para practicar (no tiene que ser 100% transcripción literal de la guía como `1 resumen.md`), siempre que el concepto evaluado sea fiel a lo que enseña la guía y al Task Statement correspondiente en `examguide.pdf`.
