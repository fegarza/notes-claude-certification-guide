# Resultados — Tutor socrático: Structured Output with Tool Use

> [!info] Sesión evaluada el 2026-10-08
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 86/100**

Dominio sólido del tema. Respondiste correctamente 9 de 11 preguntas, y las otras 2 fueron parciales. Entiendes muy bien los tres modos de `tool_choice`, cuándo usar cada uno y que se definen por request. También dominas el diseño del schema contra la fabricación (`required` frente a nullable, `"unclear"` frente a `"other"`). Los dos puntos flojos fueron de precisión, no de fondo: confundiste `tool_choice` con `tool_use` al explicar qué elimina los errores de sintaxis, y propusiste ejemplos de los formatos de entrada en lugar de instrucciones de normalización de formato.

## Conceptos con buen dominio

- **Jerarquía de confiabilidad (`tool_use` sobre JSON basado en prompt):** explicaste que el prompt depende de que el modelo obedezca, mientras que el schema impone la forma desde el propio mecanismo.
- **`tool_choice: "auto"`:** identificaste que permite devolver texto en vez de invocar un tool y que por eso no garantiza salida estructurada.
- **`tool_choice: "any"`:** lo elegiste correctamente para documentos de tipo desconocido con varios schemas, y descartaste con argumentos tanto `"auto"` como el modo forzado.
- **`tool_choice` forzado y su alcance por request:** forzaste `extract_metadata` en el primer paso y notaste por tu cuenta que la siguiente llamada lleva su propia configuración.
- **Trampa 3 (`required` provoca fabricación):** explicaste la causa sin ayuda y propusiste correctamente el campo nullable u opcional.
- **Enum con `"unclear"`:** entendiste que sin esa salida de escape el modelo se ve obligado a elegir una categoría "real".
- **Patrón `"other"` + campo de detalle:** lo distinguiste correctamente de `"unclear"` y lo aplicaste a la categorización extensible.
- **Nota sobre vigencia:** para el examen respondes con la jerarquía tool_use, aunque existan `strict: true` y `output_config.format`.

## Áreas que necesitan revisión

### Errores semánticos frente a errores de sintaxis (Trampa 1)

- **Qué pasó:** identificaste bien que las sumas que no cuadran y los campos mal ubicados no se evitan. Pero atribuiste la eliminación de los errores de sintaxis a `tool_choice` en lugar de `tool_use`, y no nombraste la categoría (errores *semánticos*) ni el principio de fondo.
- **Contenido del resumen:**
  > - `tool_use` elimina errores de **sintaxis** — pero **no** previene errores **semánticos**:
  >   - Discrepancias de suma (ej. líneas de un invoice que no suman el total).
  >   - Errores de ubicación de campo (un valor correcto puesto en el campo incorrecto).
  >   - Fabricación de valores para campos requeridos cuando la fuente no tiene esa información.
  > - **"El schema garantiza estructura. No garantiza corrección."** — la validez sintáctica de un JSON no dice nada sobre si los datos dentro de ese JSON son ciertos.

  > - El parámetro `tool_choice` es el que obliga al modelo a invocar el tool (ver Conclusión 2); el schema del tool es el que define la forma exacta de esa invocación.
- **Sección:** [[1 resumen]] → "3. Lo que `tool_use` NO previene: errores semánticos"
- **Analogía para reforzarlo:** un corrector ortográfico te garantiza que todas las palabras de tu carta están bien escritas (sintaxis), pero no que la carta diga la verdad ni que sus cuentas cuadren (semántica). Además, el corrector (`tool_use` + schema) y el botón que te obliga a pasarlo (`tool_choice`) son dos piezas distintas.

### Normalización de formato

- **Qué pasó:** atribuiste las fechas en formatos mezclados a que "el schema no valida la semántica", cuando la causa exacta es que el schema restringe el *tipo* y no el *formato interno*. Como solución propusiste darle al modelo ejemplos de los formatos de entrada posibles. La guía, en cambio, recomienda instrucciones de normalización en el prompt que digan a qué formato de salida convertir.
- **Contenido del resumen:**
  > - **Normalización de formato**: además del schema estricto, conviene incluir en el prompt instrucciones de normalización de formato (ej. cómo formatear fechas o montos), ya que el schema por sí solo restringe el *tipo* de dato, no su *formato* interno.
- **Sección:** [[1 resumen]] → "4. Diseño de schema para producción"
- **Analogía para reforzarlo:** un formulario que pide "fecha de nacimiento" en una casilla de texto acepta "3 de abril", "03/04" o "2026-04-03", porque todo cabe en la casilla. Para que todos escriban igual no basta la casilla: hay que poner la instrucción "escribe la fecha como AAAA-MM-DD" justo encima.

## Siguiente paso recomendado

Repasa la sección 3 del resumen (la distinción entre sintaxis y semántica, y qué pieza hace qué: `tool_use` frente a `tool_choice`) y el bullet de normalización de formato. Con eso estás listo para [[4 test]].
