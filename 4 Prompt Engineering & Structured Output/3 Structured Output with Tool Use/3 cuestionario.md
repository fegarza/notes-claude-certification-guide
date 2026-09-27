## La jerarquía de confiabilidad

> [!question]- ¿Por qué pedirle a Claude "responde en JSON" como parte del texto de un prompt no es confiable para producción?
> Porque es extracción basada en prompt, que no da ninguna garantía estructural — el modelo puede producir salida periódicamente no parseable (llaves sin cerrar, texto conversacional antes del JSON), y ese fallo no es raro, es esperable a escala.

> [!question]- ¿Qué elimina exactamente `tool_use` con JSON schema, y qué es lo que NO elimina?
> Elimina por completo los errores de sintaxis JSON, porque la forma de la salida queda restringida por el mecanismo del tool, no por que el modelo "recuerde" seguir una instrucción. No elimina los errores semánticos: valores fabricados, mal ubicados, o que no cuadran entre sí.

> [!question]- Si el equipo de certificación actualiza la API con nuevas funcionalidades (ej. `strict: true`, `output_config.format`) posteriores al exam guide, ¿qué criterio debe seguir usando para responder en el examen?
> La jerarquía de que `tool_use` con JSON schema es más confiable que la extracción basada en prompt — ese sigue siendo el criterio que evalúa el examen, independientemente de que existan mecanismos más nuevos en la API real.

## tool_choice: los tres modos

> [!question]- ¿Cuál es la diferencia entre `tool_choice: "auto"` y `tool_choice: "any"`?
> Con `"auto"` el modelo decide si invoca un tool o devuelve texto — no hay garantía de salida estructurada. Con `"any"` el modelo está obligado a invocar algún tool, aunque elige cuál, lo que sí garantiza salida estructurada.

> [!question]- ¿Cuándo usarías `tool_choice: "any"` en vez de forzar un tool específico?
> Cuando hay varios schemas de extracción posibles y no se sabe de antemano qué tipo de documento se está procesando — `"any"` deja que el modelo elija el tool correcto entre varias opciones, mientras que forzar un tool específico solo tiene sentido cuando ya se sabe cuál se necesita.

> [!question]- ¿Qué pasaría si un pipeline de extracción usa `tool_choice: "auto"` y asume que la respuesta siempre trae un bloque `tool_use`?
> El código fallaría en los casos donde el modelo decide responder con texto en vez de invocar el tool (stop_reason `"end_turn"`), porque `"auto"` no garantiza que el tool se invoque — es exactamente el escenario que ese modo permite.

> [!question]- ¿Por qué forzarías un tool específico (`{"type": "tool", "name": "..."}`) en vez de dejarlo en `"any"` cuando hay un solo paso obligatorio de extracción?
> Porque se necesita garantizar que ese paso exacto ocurra antes de continuar con pasos posteriores (ej. enriquecimiento) — forzar el nombre específico elimina cualquier posibilidad de que el modelo elija un tool distinto o ninguno.

> [!question]- ¿`tool_choice` se configura una vez por conversación o en cada llamada a la API?
> En cada llamada (request). No persiste automáticamente entre turnos de una conversación de varios pasos — si un turno posterior necesita otro modo, hay que especificarlo de nuevo en esa llamada.

## Errores sintácticos vs. errores semánticos

> [!question]- Un JSON extraído con `tool_use` parsea sin errores y cumple el schema exactamente. ¿Eso significa que los datos son correctos?
> No necesariamente. Cumplir el schema solo garantiza corrección sintáctica (forma). Puede seguir teniendo errores semánticos: un total que no cuadra con la suma de líneas, o un valor correcto puesto en el campo equivocado.

> [!question]- Da dos ejemplos de errores semánticos que `tool_use` no previene por sí solo.
> Discrepancias de suma (ej. line items que no suman el total declarado) y errores de ubicación de campo (un valor válido colocado en el campo incorrecto del schema).

> [!question]- ¿Por qué es un error decir que "el schema garantiza que los datos son correctos"?
> Porque el schema solo restringe la forma/tipo de cada campo, no verifica relaciones entre campos ni la veracidad del contenido — para eso se necesita una validación semántica separada, aplicada después de la extracción.

## Diseño de schema para producción

> [!question]- ¿Por qué marcar un campo como `required` cuando el documento fuente puede no contener esa información es una mala práctica?
> Porque presiona al modelo a fabricar un valor con tal de satisfacer el schema, en vez de poder responder honestamente que el dato no está presente.

> [!question]- ¿Cuál es "la defensa principal contra la fabricación" en el diseño de un schema de extracción?
> Marcar como opcional/nullable cualquier campo cuya presencia en el documento fuente no esté garantizada, en vez de marcarlo como required.

> [!question]- ¿Qué problema resuelve agregar un valor `"unclear"` a un campo enum, en vez de solo las categorías "reales"?
> Permite que el modelo reconozca honestamente un caso genuinamente ambiguo, en vez de verse forzado a elegir la categoría "real" que menos mal le quede — evitando una clasificación falsa por falta de una opción de escape.

> [!question]- ¿Cómo se relaciona el patrón `"other"` + campo de detalle libre con la extensibilidad del schema?
> Permite capturar categorías no anticipadas al diseñar el enum sin tener que rediseñar el schema cada vez que aparece un caso nuevo: el modelo marca `"other"` y describe el caso en un campo de texto libre asociado.

> [!question]- ¿Por qué no basta con un JSON schema estricto para garantizar que los datos extraídos tengan el formato correcto (ej. fechas, montos)?
> Porque el schema restringe el *tipo* de dato (string, number), no necesariamente su *formato* interno — por eso conviene incluir además instrucciones de normalización de formato en el prompt, junto al schema.

---

> [!tip] Repasa la teoría en [[1 resumen]], aplícala en [[2 example]] y evalúate en [[4 test]]
