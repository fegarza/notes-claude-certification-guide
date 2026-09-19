---
tags:
  - claude-cert/dominio-2
  - task-statement/2.5
---

# 3 cuestionario — Built-in Tools

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## Grep vs. Glob

> [!question]- ¿Cuál es la distinción fundamental entre Grep y Glob?
> Grep busca CONTENIDO dentro de los archivos (texto, patrones, nombres de función). Glob busca archivos por su RUTA o patrón de nombre, sin mirar el contenido.

> [!question]- Si necesitas encontrar todos los archivos que llaman a una función específica, ¿qué herramienta usas y por qué?
> Grep, porque la llamada a la función es texto dentro del contenido de los archivos — Glob no puede ver contenido, solo compara nombres de ruta.

> [!question]- Si necesitas encontrar todos los archivos `.test.tsx` de un proyecto, ¿qué herramienta usas y por qué?
> Glob, porque es una búsqueda por patrón de nombre/extensión de archivo, no por lo que el archivo contiene.

> [!question]- ¿Qué pasaría si usas Glob para intentar encontrar quién llama a una función determinada?
> Falla directamente — Glob compara rutas, no contenido, así que a menos que el nombre de la función forme parte literal del nombre del archivo, la búsqueda no encuentra los llamadores.

> [!question]- ¿Por qué "usar Grep para encontrar archivos de test por nombre" se considera la herramienta equivocada, aunque a veces funcione?
> Porque depende de que la palabra buscada aparezca como contenido dentro del archivo — funciona por coincidencia, no por diseño, y falla en archivos de test que no mencionan ese contenido. Glob es la herramienta construida específicamente para coincidencia de ruta/nombre.

## Read, Write y Edit

> [!question]- ¿Cuál es la ventaja de Edit sobre Read + Write para una modificación?
> Edit toca solo el texto exacto especificado, es más rápida y usa mucho menos contexto que Read + Write, que carga y reescribe el archivo completo.

> [!question]- ¿Por qué Edit falla cuando el texto especificado aparece en más de un lugar del archivo?
> Porque no puede determinar cuál de las ocurrencias es la que se quiere cambiar — es un mecanismo de seguridad para evitar modificar texto que no se pretendía tocar, no un defecto de la herramienta.

> [!question]- Según el exam guide, ¿cuál es el fallback documentado cuando Edit falla por coincidencia no única?
> Read + Write: leer el archivo completo y reescribir la versión modificada completa.

> [!question]- ¿Qué hace la documentación actual de Claude Code (14 de agosto de 2026) ante ese mismo fallo de Edit, antes de llegar a Read + Write?
> Amplía `old_string` con más contexto de alrededor hasta que apunte a una sola ubicación, o usa `replace_all: true` si se quiere cambiar cada ocurrencia — ambas opciones se quedan dentro de Edit.

> [!question]- Si una pregunta del examen pide la respuesta documentada del exam guide ante un fallo de Edit por no-unicidad, ¿qué respondes: ampliar el ancla o Read + Write?
> Read + Write — esa es la respuesta que sigue el exam guide v1.0, aunque en trabajo real ampliar el ancla (o usar `replace_all`) sea la opción más barata.

> [!question]- ¿Cuál es el error que el examen penaliza respecto al uso de Read + Write?
> Usarlo como respuesta por defecto para cada modificación, en vez de intentar Edit primero — eso desperdicia tokens de contexto en cambios que Edit podría resolver de forma puntual.

## Comprensión incremental del código base

> [!question]- ¿Cuál es el error de exploración más costoso, según el resumen?
> Leer todos los archivos del código base de entrada, antes de saber cuáles son relevantes — consume el presupuesto de contexto en archivos que no tienen nada que ver con la tarea.

> [!question]- ¿Cuáles son los cuatro pasos del enfoque de descubrimiento incremental?
> (1) Grep para encontrar puntos de entrada, (2) Read para seguir imports y trazar flujos, (3) Grep otra vez para trazar el uso bajo nombres alternativos, (4) Read solo lo que se justifique por el paso anterior.

> [!question]- ¿Por qué cada Read en el proceso incremental debe estar "justificado por el paso anterior"?
> Porque el objetivo es gastar tokens de contexto solo en archivos que efectivamente importan para la tarea, no leer por precaución o "por si acaso" algo resulta relevante.

## Trazar funciones a través de wrappers y archivos barril

> [!question]- ¿Por qué un Grep del nombre original de una función puede perderse consumidores que sí la están usando?
> Porque esos consumidores importan la función a través de un wrapper o de un archivo barril que la re-exporta bajo otro nombre — el string del nombre original nunca aparece en sus archivos.

> [!question]- ¿Cuáles son los cuatro pasos para rastrear el uso completo de una función expuesta a través de un wrapper?
> (1) Grep por la definición de la función, (2) Read del archivo que la define para identificar nombres exportados, (3) Grep por cada nombre exportado en todo el código base, (4) si hay un archivo barril, Grep por el nombre del módulo barril para encontrar consumidores que importan desde ahí.

> [!question]- En el ejemplo `processOrder` / `submitOrder` del resumen, ¿por qué un Grep de `processOrder` encuentra solo dos de los cinco consumidores?
> Porque los otros tres importan la función bajo el nombre `submitOrder`, re-exportado desde un archivo barril (`utils/index.ts`) — el string `processOrder` nunca aparece en sus archivos, así que el Grep original no los detecta.

## El escenario de deprecación

> [!question]- ¿Cuál es la secuencia correcta de herramientas para encontrar llamadores de una función deprecada y sus tests correspondientes?
> Grep (encuentra llamadores por contenido, incluyendo tests que importan directamente) → Glob (encuentra tests hermanos por convención de nombre) → Grep otra vez (encuentra tests que cubren la función indirectamente a través de un wrapper).

> [!question]- ¿Por qué el patrón no empieza por Glob?
> Porque el punto de partida es encontrar quién referencia la función en su contenido (una búsqueda de Grep); Glob entra después, para encontrar los archivos de test que acompañan a esos llamadores por convención de nombre, no para encontrar la función misma.

> [!question]- ¿Qué pasaría si, en el escenario de deprecación, un archivo llamador expone la función deprecada a través de un wrapper, y no se hace un tercer Grep por el nombre del wrapper?
> Se pierden los tests que cubren la función transitivamente a través de ese wrapper — quedarían sin detectar, porque ni el Grep original ni el Glob de tests hermanos por sí solos los encuentran.

## Síntesis

> [!question]- Un desarrollador necesita encontrar todos los callers de `processOrder()` y sus tests, pero decide: primero Glob por `**/*processOrder*` para "acotar la búsqueda", y luego lee cada archivo completo para confirmar si realmente llama a la función. ¿Qué dos errores hay aquí?
> Primero, usar Glob para una búsqueda de contenido (encontrar callers) — Glob compara rutas, no puede confirmar quién llama a la función. Segundo, leer archivos completos para "confirmar" en vez de usar Grep, que ya habría dado la respuesta de forma directa y barata en tokens — es un uso invertido e ineficiente de las herramientas frente al patrón Grep → Glob → Grep.
