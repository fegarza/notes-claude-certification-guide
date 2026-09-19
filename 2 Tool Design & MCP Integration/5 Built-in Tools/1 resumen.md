---
tags:
  - claude-cert/dominio-2
  - task-statement/2.5
---

# 2.5 — Built-in Tools

## Explícamelo como si tuviera 5 años

Imagina que tienes una caja de herramientas con seis herramientas básicas. Si quieres saber **qué hay escrito dentro** de un montón de cajas cerradas, usas una linterna que ve a través de la tapa (eso es Grep: busca *contenido*). Si en cambio quieres encontrar **una caja específica por su etiqueta** —todas las que dicen "TEST" en la etiqueta, sin importar qué haya adentro— usas un catálogo de etiquetas (eso es Glob: busca *nombres/rutas*). Confundir la linterna con el catálogo —usar Glob para ver contenido, o Grep para buscar por nombre— es exactamente el error que el examen pone a propósito frente a ti.

Y para modificar lo que hay dentro de una caja, hay una herramienta de precisión (Edit, que cambia solo el texto exacto que señalas) y una de fuerza bruta (Read + Write, que saca todo el contenido de la caja, lo reescribe completo y lo vuelve a meter). La de precisión es más rápida y barata — se usa primero. La de fuerza bruta es el respaldo, no la opción por defecto.

## Argumento central

> [!note] Idea central
> Claude Code ofrece seis herramientas nativas para trabajar con código — Read, Write, Edit, Bash, Grep y Glob —, cada una optimizada para un propósito distinto. Usar la herramienta equivocada para una tarea desperdicia tiempo, tokens de contexto, o ambos. El examen presenta deliberadamente escenarios donde confundir estas herramientas lleva a una respuesta incorrecta: la distinción Grep-vs-Glob, cuándo usar Edit vs. Read+Write, y cómo explorar un código base de forma incremental en vez de leerlo todo de entrada.

## Conclusiones

### 1. Grep vs. Glob: la distinción que más pesa en este tema

- **Grep busca CONTENIDO** — texto dentro de archivos. Úsalo para encontrar quién llama a una función, mensajes de error, declaraciones de import, asignaciones de variables. Cualquier vez que la pregunta sea "¿qué contienen los archivos?", la respuesta es Grep.
- **Glob busca RUTAS por patrón de nombre** — archivos de test, archivos de configuración, todos los `.ts` de un directorio. Cualquier vez que la pregunta sea "¿qué archivos existen con este nombre/extensión/estructura?", la respuesta es Glob.

> [!note] La distinción en una frase
> Grep encuentra lo que está DENTRO de los archivos. Glob encuentra archivos por sus NOMBRES.

> [!tip] Analogía mental
> Grep es una linterna que ilumina el contenido de cada caja cerrada. Glob es un catálogo que lista cajas por su etiqueta, sin abrirlas.

> [!warning] Contexto de examen: la trampa de "funciona técnicamente pero es la herramienta equivocada"
> El examen presenta escenarios donde se usa la herramienta equivocada. Usar Glob para encontrar quién llama a una función **falla** — Glob compara rutas, no contenido. Usar Grep para encontrar archivos de test por patrón de nombre puede "funcionar" técnicamente (buscando la palabra "test" dentro del contenido), pero es la herramienta incorrecta, y el examen espera que identifiques la correcta (Glob).

### 2. Read, Write y Edit: cada una para un caso de uso distinto

- **Edit** hace modificaciones puntuales usando coincidencia de texto único: especificas el texto exacto a buscar y su reemplazo. Es rápida y precisa porque solo toca el texto que señalas.
- **Read/Write** son para operaciones de archivo completo: Read carga todo el contenido, Write reescribe el archivo entero.

> [!warning] Por qué Edit falla si el texto no es único (no es un bug)
> Edit requiere que el texto especificado sea único en el archivo. Si aparece en más de un lugar, Edit no puede saber cuál es el que quieres cambiar, así que falla. Es un mecanismo de seguridad, no un defecto — evita que cambies texto que nunca quisiste tocar.

#### Qué hacer cuando Edit falla por coincidencia no única

> [!warning] Respuesta del examen vs. Claude Code actual — no son lo mismo
> El **exam guide v1.0** documenta la respuesta esperada como: cuando Edit falla, usar **Read + Write** como fallback — leer el archivo completo y reescribirlo completo. Esa es la respuesta correcta **para el examen**.
> Al **14 de agosto de 2026**, la documentación actual de la herramienta Edit describe un paso intermedio más barato primero: ampliar `old_string` con más contexto de alrededor hasta que apunte a una sola ubicación, o usar `replace_all: true` si en realidad se quiere cambiar cada ocurrencia. Ambas opciones mantienen el trabajo dentro de Edit y casi no cuestan contexto extra.
> **En el examen, responde Read + Write como el fallback documentado. En trabajo real, amplía el ancla (o usa `replace_all`) antes de llegar a Read + Write** — es más barato y evita gastar el contenido de un archivo entero en lo que suele ser un cambio de una línea.

- **Orden en el examen**: Edit primero → Read + Write cuando Edit falla. Dos pasos.
- **Orden en trabajo real**: Edit con el ancla más corta plausible → si falla por no-unicidad, ampliar `old_string` o usar `replace_all: true` → Read + Write solo si ninguna de esas dos opciones puede desambiguar el objetivo.
- Ambos órdenes coinciden en el primer paso: **nunca uses Read + Write como respuesta por defecto para cada modificación** — el examen penaliza eso porque desperdicia tokens de contexto.

### 3. Comprensión incremental del código base: nunca leer todo de entrada

> [!warning] El error más costoso de exploración: leer todos los archivos de entrada
> Cargar cada archivo al contexto antes de saber qué necesitas es un destructor del presupuesto de contexto. Un código base de 200 archivos leído por completo consume toda la ventana de contexto, en su mayoría en archivos que no tienen nada que ver con la tarea. Ningún otro error de exploración cuesta más.

El enfoque correcto es **descubrimiento incremental**: empezar acotado, expandir solo según haga falta, en cuatro pasos:

1. **Grep para encontrar puntos de entrada** — buscar el nombre de función, clase o mensaje de error que ancla la investigación. Esto indica qué archivos son relevantes.
2. **Read para seguir imports y trazar flujos** — una vez identificados los archivos relevantes, leerlos para entender la estructura del código y seguir sus imports hacia archivos relacionados.
3. **Grep otra vez para trazar el uso** — los archivos leídos en el paso 2 pueden exponer la función bajo otro nombre: un wrapper (`submitOrder()` que internamente llama a `processOrder()`) o un archivo barril que la re-exporta (`export { processOrder as submitOrder }`). Quienes llaman al nombre nuevo nunca mencionan el original, así que el primer Grep nunca los vio. Hay que volver a hacer Grep por cada nombre nuevo, en todo el código base, para obtener la lista completa de consumidores.
4. **Read solo lo necesario** — cada archivo leído debe estar justificado por lo que se descubrió en el paso anterior.

> [!tip] Síntesis
> Es contexto mínimo para máxima comprensión: se mapea el código base progresivamente, gastando tokens solo en los archivos que importan para la tarea.

### 4. Trazar el uso de una función a través de módulos wrapper y archivos barril

Un patrón común en código bases: una función se define en un módulo, se re-exporta a través de un wrapper, y se consume por el nombre del wrapper. Un simple Grep del nombre original **se pierde** a todos los consumidores que importan a través del wrapper.

El enfoque correcto, en secuencia:

1. Grep por la definición de la función, para encontrar dónde se define.
2. Read del archivo que la define, para identificar los nombres exportados.
3. Grep por cada nombre exportado en todo el código base, para encontrar todos los consumidores.
4. Si la función se re-exporta a través de un archivo barril (ej. `index.ts`), Grep por el nombre del módulo barril para encontrar a los consumidores que importan desde ahí.

> [!note] Caso concreto
> `processOrder` se define en `orders.ts`. El barril `utils/index.ts` la re-exporta como `submitOrder`, y tres de sus cinco consumidores importan `submitOrder` desde `utils`. Un Grep de `processOrder` encuentra la definición, la línea del barril y los dos consumidores que importan el nombre original — pero **no** puede encontrar a los otros tres, porque el string `processOrder` nunca aparece en sus archivos. Leer el barril, detectar el renombre, hacer Grep de `submitOrder`, y los tres aparecen.

El rastreo multi-paso atrapa consumidores indirectos que un solo Grep pasaría por alto.

### 5. El escenario de deprecación: el patrón Grep → Glob → Grep otra vez

Escenario recurrente: encontrar todos los archivos que llaman a una función deprecada **y** los archivos de test que la ejercitan. La secuencia correcta:

1. **Grep** por el nombre de la función — encuentra cada archivo cuyo contenido referencia la función, incluyendo cualquier test que la importe directamente (búsqueda de contenido).
2. **Glob** por los archivos de test hermanos — encuentra el archivo de test que corresponde a cada archivo llamador por convención de nombre (ej. `OrderProcessor.ts` → `OrderProcessor.test.tsx`), incluso cuando el test ejercita la función indirectamente a través del módulo fuente (coincidencia de ruta).
3. **Grep otra vez** por nombres de wrapper — cuando un archivo llamador expone la función a través de un wrapper (ej. `applyLegacyOrder` que internamente llama a `processLegacyOrder`), Grep por el nombre del wrapper encuentra tests que cubren la función transitivamente a través de él.

> [!note] Ejemplo del patrón
> Grep revela que `OrderProcessor.ts` y `RefundHandler.ts` llaman a la función deprecada. Glob por `**/OrderProcessor.test.*` y `**/RefundHandler.test.*` trae sus archivos de test hermanos, aunque esos tests nunca mencionen `processLegacyOrder` por nombre. Si alguno de los archivos fuente envuelve la función bajo un nombre nuevo, Grep por ese wrapper atrapa los tests restantes.

> [!warning] No es "Glob primero"
> El patrón es Grep, luego Glob, luego Grep otra vez — búsqueda de contenido para referencias directas, coincidencia de ruta para tests adyacentes, búsqueda de contenido otra vez para cobertura indirecta. **No** es empezar por Glob.

## En una frase

> Elegir bien entre las herramientas nativas de Claude Code —Grep para contenido, Glob para rutas, Edit como modificación por defecto con Read + Write como fallback documentado, y exploración incremental en vez de leer todo el código base de entrada— es lo que separa un uso eficiente del contexto de uno que lo desperdicia sin necesidad.

## Trampas de examen

> [!warning] Trampa 1 — Usar Glob para encontrar quién llama a una función
> Glob compara rutas de archivo por patrón de nombre; no puede buscar dentro del contenido de los archivos. **Por qué es un error:** falla directamente — no hay forma de que Glob encuentre una llamada a función dentro del código. La forma correcta es usar Grep para buscar contenido: nombres de función, declaraciones de import, mensajes de error.

> [!warning] Trampa 2 — Usar Grep para encontrar archivos por extensión o patrón de nombre
> Aunque Grep podría técnicamente encontrar nombres de archivo mencionados en el contenido, Glob es la herramienta diseñada específicamente para comparar rutas. **Por qué es un error:** es la herramienta equivocada para la tarea, aunque "funcione" de forma indirecta. La forma correcta es usar Glob para patrones como `**/*.test.tsx` o `**/config.*`.

> [!warning] Trampa 3 — Leer todos los archivos fuente de entrada
> Cargar cada archivo al contexto antes de saber qué se necesita es el error de exploración más costoso. **Por qué es un error:** consume el presupuesto de contexto en archivos irrelevantes para la tarea. La forma correcta es el enfoque incremental: Grep para encontrar puntos de entrada, luego Read para trazar flujos desde esos puntos específicos.

> [!warning] Trampa 4 — Usar Read + Write como respuesta por defecto para cada modificación
> Edit es más rápido y usa menos contexto porque solo toca el texto específico señalado; Read + Write carga el archivo completo. **Por qué es un error:** el examen penaliza tratar Read + Write como la opción estándar. La forma correcta es intentar Edit primero siempre; Read + Write es el fallback para cuando Edit no encuentra un ancla única, no la respuesta por defecto.

> [!warning] Trampa 5 — Responder "ampliar `old_string`" cuando la pregunta es sobre el examen
> Ampliar el ancla (o usar `replace_all: true`) es lo que hace Claude Code actualmente, y es la jugada más barata en trabajo real. **Por qué es un error en el contexto del examen:** el exam guide nombra Read + Write como el fallback documentado cuando Edit no encuentra texto de anclaje único, y toda respuesta clave del examen sigue esa guía. La forma correcta en el examen es: leer el archivo completo, luego escribir la versión completa modificada de vuelta.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
