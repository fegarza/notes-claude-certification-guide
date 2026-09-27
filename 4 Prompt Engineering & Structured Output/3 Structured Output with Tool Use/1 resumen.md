## Explícamelo como si tuviera 5 años

Imagina que le pides a un niño que llene un formulario en papel escribiendo libremente: "pon aquí tu nombre, tu edad y tu color favorito". El niño puede escribir en cualquier orden, saltarse una línea, poner la edad donde va el nombre, o simplemente no entregar el formulario completo. Ahora imagina que en vez de eso le das un formulario con casillas: una casilla marcada "Nombre", otra "Edad" (solo números), otra "Color favorito" (con opciones para marcar). El niño ya no puede "olvidarse" del formato — la estructura del papel se lo impone.

Pedirle a Claude que "responda en JSON" dentro del texto de una instrucción es como el papel en blanco: puede funcionar casi siempre, pero de vez en cuando el niño (el modelo) se sale del formato — olvida una comilla, deja una llave sin cerrar. Usar **`tool_use` con un JSON schema** es como el formulario con casillas: la estructura de salida queda garantizada por el mecanismo mismo, no por que el modelo "se acuerde" de seguir instrucciones.

Pero ojo: el formulario con casillas garantiza que el niño llenó *algo* en cada casilla — no garantiza que la edad que puso sea la verdadera. Esa es la segunda mitad del tema.

## Argumento central

> [!note] Idea central
> `tool_use` con JSON schemas es el mecanismo más confiable para obtener salida estructurada garantizada de Claude — elimina por completo los errores de **sintaxis** JSON. Pero esa garantía es solo estructural: no elimina los errores **semánticos** (valores inventados, mal ubicados, o que no cuadran entre sí). Diseñar bien el schema (campos opcionales/nullable, enums con salidas de escape) y elegir bien el modo de `tool_choice` son lo que cierra esa brecha.

## Conclusiones clave

### 1. La jerarquía de confiabilidad: tool_use sobre JSON basado en prompt

- **Extracción basada en prompt** (pedir "responde en JSON" como texto): no da ninguna garantía estructural y **periódicamente producirá salida JSON no parseable** en producción (llaves sin cerrar, comillas faltantes, claves sin comillas).
- **`tool_use` con JSON schema**: el schema del tool restringe la *forma* de la salida a nivel de mecanismo, no de instrucción — elimina por completo los errores de sintaxis JSON.
- El parámetro `tool_choice` es el que obliga al modelo a invocar el tool (ver Conclusión 2); el schema del tool es el que define la forma exacta de esa invocación.

> [!note] Nota sobre vigencia
> La guía menciona que, a la fecha, la API de Claude ya incorpora `strict: true` en las definiciones de tool y `output_config.format`, funcionalidades posteriores a la redacción original del exam guide. Aun así, para efectos del examen la respuesta correcta sigue siendo razonar con la jerarquía **tool_use sobre extracción basada en prompt** — es el criterio que evalúa el examen, independientemente de mecanismos más nuevos.

### 2. `tool_choice`: los tres modos

| Modo                                                     | Comportamiento                                      | Cuándo usarlo                                                                                                                 |
| -------------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `"auto"` (default)                                       | El modelo decide si invoca un tool o devuelve texto | Cuando una respuesta conversacional también es aceptable                                                                      |
| `"any"`                                                  | El modelo **debe** invocar un tool, pero elige cuál | Garantizar salida estructurada cuando hay varios schemas de extracción posibles y no se sabe de antemano el tipo de documento |
| `{"type": "tool", "name": "extract_metadata"}` (forzado) | El modelo **debe** invocar ese tool específico      | Garantizar que un paso de extracción obligatorio ocurra antes de pasos de enriquecimiento posteriores                         |

```typescript
// Salida estructurada garantizada cuando el tipo de documento es desconocido
const response = await client.messages.create({
  model: "claude-sonnet-5",
  max_tokens: 4096,
  tool_choice: { type: "any" },
  tools: [extractInvoiceTool, extractReceiptTool, extractContractTool],
  messages: [{ role: "user", content: documentText }]
});

// Forzar un paso de extracción específico
const response = await client.messages.create({
  model: "claude-sonnet-5",
  max_tokens: 4096,
  tool_choice: { type: "tool", name: "extract_metadata" },
  tools: [extractMetadataTool],
  messages: [{ role: "user", content: documentText }]
});
```

> [!note] `tool_choice` es por request, no por conversación
> Cada llamada a la API lleva su propio `tool_choice`. No es una configuración que persista automáticamente a lo largo de una conversación de varios turnos — si el segundo turno necesita forzar un tool distinto (o volver a `"auto"`), hay que especificarlo de nuevo en esa request.

### 3. Lo que `tool_use` NO previene: errores semánticos

- `tool_use` elimina errores de **sintaxis** — pero **no** previene errores **semánticos**:
  - Discrepancias de suma (ej. líneas de un invoice que no suman el total).
  - Errores de ubicación de campo (un valor correcto puesto en el campo incorrecto).
  - Fabricación de valores para campos requeridos cuando la fuente no tiene esa información.
- **"El schema garantiza estructura. No garantiza corrección."** — la validez sintáctica de un JSON no dice nada sobre si los datos dentro de ese JSON son ciertos.

### 4. Diseño de schema para producción

- **Campos opcionales / nullable — la defensa principal contra la fabricación.** Cuando el documento fuente puede no contener cierta información, ese campo debe ser opcional o nullable en el schema. Un campo marcado como `required` presiona al modelo a inventar un valor con tal de cumplir el schema; un campo nullable le permite responder `null` honestamente.

  ```json
  {
    "type": "object",
    "properties": {
      "invoice_number": { "type": "string" },
      "vendor_name": { "type": "string" },
      "payment_terms": { "type": ["string", "null"] },
      "purchase_order": { "type": ["string", "null"] }
    },
    "required": ["invoice_number", "vendor_name"]
  }
  ```

- **Valor enum `"unclear"`**: para casos ambiguos donde la fuente genuinamente no permite determinar la categoría, se agrega un valor explícito `"unclear"` al enum, en vez de forzar al modelo a elegir entre las categorías "reales".
- **Patrón `"other"` + campo de detalle libre**: para categorización extensible (categorías que no se pueden enumerar por completo de antemano), se incluye un valor `"other"` en el enum, acompañado de un campo de texto libre que capture el detalle.

  ```json
  {
    "category": {
      "type": "string",
      "enum": ["invoice", "receipt", "contract", "unclear", "other"]
    },
    "category_detail": {
      "type": ["string", "null"],
      "description": "Freeform detail when category is 'other'"
    }
  }
  ```

- **Normalización de formato**: además del schema estricto, conviene incluir en el prompt instrucciones de normalización de formato (ej. cómo formatear fechas o montos), ya que el schema por sí solo restringe el *tipo* de dato, no su *formato* interno.

## Trampas de examen

> [!warning] Trampa 1 — Creer que `tool_use` previene todos los errores de extracción
> `tool_use` solo elimina errores de **sintaxis** JSON, no errores **semánticos**. Un JSON perfectamente válido puede tener el monto en el campo equivocado o un total que no cuadra con las líneas — eso requiere validación adicional, separada del mecanismo de `tool_use` (ver [[4 Validation, Retry, and Feedback Loops/1 resumen|Validation, Retry, and Feedback Loops]]).

> [!warning] Trampa 2 — Confundir `tool_choice: "auto"` con `tool_choice: "any"`
> `"auto"` permite que el modelo devuelva texto en vez de invocar un tool — **no** da ninguna garantía de salida estructurada. `"any"` sí garantiza que se invoque un tool (aunque el modelo elija cuál). Si el objetivo es garantizar salida estructurada, `"auto"` es la opción equivocada aunque sea la configuración por default.

> [!warning] Trampa 3 — Marcar todos los campos del schema como `required`
> Forzar como `required` un campo que el documento fuente puede no contener presiona al modelo a **fabricar** un valor con tal de satisfacer el schema, en vez de responder honestamente `null`. La forma correcta es marcar como opcional/nullable cualquier campo cuya presencia en la fuente no esté garantizada.

## En una frase

> `tool_use` con JSON schemas garantiza la *forma* de la salida de Claude (cero errores de sintaxis), pero la *corrección* de esa salida —evitar fabricación y errores semánticos— depende del diseño del schema (campos nullable, enums con `"unclear"`/`"other"`) y de elegir el modo correcto de `tool_choice`.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
