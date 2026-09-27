## Retry-with-error-feedback: los tres ingredientes

> [!question]- ¿Cuáles son las tres piezas de información que debe incluir un mensaje de retry para que sea efectivo?
> El documento original, la extracción fallida, y el error de validación específico — no basta con decirle al modelo "vuelve a intentarlo".

> [!question]- ¿Qué tiende a pasar cuando se reintenta una extracción sin incluir el error específico de validación?
> El modelo típicamente reproduce la misma extracción fallida, porque no recibe ninguna señal nueva sobre qué corregir.

> [!question]- ¿Por qué incluir el documento original en el mensaje de retry, y no solo la extracción fallida y el error?
> Porque le permite al modelo volver a examinar la fuente directamente, en vez de intentar "adivinar" la corrección basándose solo en su propio error anterior.

## La frontera de efectividad del retry

> [!question]- ¿Qué tipo de errores SÍ se corrigen reintentando con retroalimentación de error?
> Discrepancias de formato (fechas, notación de moneda), errores estructurales (valor en campo equivocado, anidación incorrecta), valores mal ubicados, y errores matemáticos (líneas faltantes que afectan un total).

> [!question]- ¿Por qué reintentar no sirve cuando la información simplemente no está en el documento fuente?
> Porque el reintento solo le da al modelo la oportunidad de re-leer y re-razonar sobre datos que ya tiene disponibles — no puede generar información verídica que no existe en ningún lado del input, y forzarlo a "producir algo" solo empuja hacia la fabricación.

> [!question]- Un campo `department` viene vacío en la extracción y el documento fuente no lo menciona en ningún lado. ¿Cuál es la acción correcta?
> Marcarlo como `null` (si el schema lo permite) y/o enrutar a revisión humana — no reintentar la extracción, porque la información ausente no se resuelve con más intentos.

> [!question]- ¿Por qué el examen presenta escenarios corregibles y no corregibles juntos, en vez de solo uno de los dos tipos?
> Porque la habilidad que evalúa es precisamente distinguir cuál es cuál — reconocer que no todo fallo de extracción amerita el mismo tratamiento es el punto central de este tema.

## Self-correction dentro del schema

> [!question]- ¿Qué logra extraer `calculated_total` y `stated_total` como dos campos separados, en vez de solo un `total`?
> Permite detectar automáticamente una discrepancia entre lo que dicen las líneas individuales y lo que declara el documento, sin necesidad de lógica de validación externa — la discrepancia queda marcada por el simple hecho de comparar ambos campos.

> [!question]- ¿Para qué sirve un campo booleano como `conflict_detected` cuando el documento fuente tiene información contradictoria?
> Para extraer ambos valores contradictorios y señalar el conflicto explícitamente, en vez de que el modelo elija silenciosamente uno de los dos — evita que una inconsistencia de la fuente se pierda sin que nadie la note.

> [!question]- ¿Cómo se relaciona el diseño de self-correction en el schema con la idea de "no depender solo de validación externa"?
> Al incluir campos que se comparan entre sí (como `calculated_total` vs `stated_total`) directamente en la salida estructurada, la inconsistencia queda visible con solo leer el JSON resultante, sin tener que ejecutar lógica de validación aparte para detectarla.

## detected_pattern y loops de mejora sistemática

> [!question]- ¿Qué información captura el campo `detected_pattern` en un finding de revisión de código?
> Qué construcción específica de código disparó ese finding — no solo el mensaje del finding en sí, sino el patrón subyacente que lo generó.

> [!question]- ¿Cómo se usa `detected_pattern` para decidir qué parte del prompt de un reviewer refinar primero?
> Analizando la tasa de descarte (dismissal) de findings agrupada por `detected_pattern` — los patrones que los desarrolladores descartan consistentemente son los candidatos prioritarios para refinamiento del prompt.

> [!question]- ¿Cuál es el ciclo completo que describe un "loop de mejora sistemática" en este contexto?
> Extraer → validar → recolectar datos de descarte (por `detected_pattern`) → refinar el prompt → repetir.

## Errores de sintaxis de schema vs. errores semánticos de validación

> [!question]- ¿Qué tipo de error elimina `tool_use` con JSON schema, y cuál sigue sin cubrir?
> Elimina errores de sintaxis (JSON malformado, campos requeridos faltantes, tipo de dato incorrecto). No cubre errores semánticos (valores que no cuadran entre sí, mal ubicados) — esos requieren validación aparte.

> [!question]- ¿Por qué el examen puede evaluar el mismo escenario de extracción tanto desde `tool_use`/JSON schema como desde validación-retry?
> Porque son dos capas distintas y complementarias del mismo pipeline: `tool_use` resuelve la capa de sintaxis, la validación-retry resuelve la capa semántica — entender el límite entre ambas es justo lo que se evalúa.

> [!question]- ¿Qué pasaría si un pipeline asume que, como el JSON pasó la validación de schema, la extracción ya es correcta?
> Pasaría por alto errores semánticos como totales que no cuadran o valores en el campo equivocado — la validación de schema por sí sola nunca los detecta, porque no verifica relaciones entre campos ni veracidad del contenido.

## Pydantic como capa de validación

> [!question]- ¿Qué dos funciones cumple un modelo de Pydantic en un loop de validación-retry, y a qué corresponde cada una?
> Parsing (estructura: tipos, campos requeridos, enums — equivalente al rol de un JSON schema) y validators (semántica: reglas entre campos, como aritmética o ejemplo de orden de fechas, que un JSON schema no puede expresar).

> [!question]- ¿Por qué un `ValidationError` de Pydantic es especialmente útil para construir el mensaje de retry?
> Porque incluye errores legibles por máquina que nombran el campo específico y la regla incumplida — es exactamente el tercer ingrediente (el error específico) del patrón retry-with-error-feedback, ya en un formato directamente incorporable al prompt de reintento.

> [!question]- Si `tool_use` ya garantiza que el JSON cumple el schema, ¿por qué seguir usando Pydantic con validators adicionales?
> Porque el schema del tool no puede expresar reglas semánticas entre campos (sumas que deben coincidir, fechas que deben ordenarse) — los validators de Pydantic sí, y son los que producen el error específico y por campo que el loop de retry necesita.

> [!question]- ¿Qué garantizan `client.messages.parse(..., output_format=...)` y `strict: true` en una definición de tool, y qué NO garantizan?
> Garantizan conformidad estructural con el schema (parsing/validación a nivel de SDK o de servidor). No eliminan la necesidad de la capa semántica: las reglas de negocio (validators) y el loop de retry siguen siendo necesarios para los errores que el schema no puede expresar.

---

> [!tip] Repasa la teoría en [[1 resumen]], aplícala en [[2 example]] y evalúate en [[4 test]]
