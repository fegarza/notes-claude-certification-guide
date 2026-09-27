## Explícamelo como si tuviera 5 años

Imagina que un niño hace la tarea de matemáticas y se equivoca en un problema. Hay dos formas de corregirlo. La mala: tacharle el ejercicio y decirle "está mal, hazlo de nuevo" — el niño probablemente vuelva a cometer el mismo error, porque no sabe *qué* estuvo mal. La buena: decirle exactamente "sumaste 4+5 y te dio 8, revisa esa cuenta" — ahí sí puede corregirse solo.

Pero hay un tercer caso: si le pides al niño la capital de un país que nunca le enseñaron, no importa cuántas veces le digas "vuelve a intentarlo" — no tiene esa información, y no puede inventarla honestamente. Ahí la solución no es "reintentar de nuevo", es preguntarle a alguien que sí sepa (revisión humana).

Ese es exactamente el tema: cuando Claude extrae datos y algo sale mal, la corrección eficaz depende de decirle *exactamente* qué salió mal (no solo "está mal") — y de saber distinguir cuándo el error es corregible reintentando, de cuándo el dato simplemente no existe en la fuente y reintentar es inútil.

## Argumento central

> [!note] Idea central
> El patrón **retry-with-error-feedback** convierte fallos de extracción en flujos auto-correctivos, pero solo funciona si el reintento incluye el documento original, la extracción fallida y el error de validación específico. Ese patrón tiene una **frontera de efectividad** clara: corrige errores de formato/estructura/matemática, pero **no puede crear información que simplemente no está en el documento fuente** — ahí la respuesta correcta es revisión humana, no un reintento más.

## Conclusiones clave

### 1. Retry-with-error-feedback: los tres ingredientes

El reintento correcto le comunica al modelo tres piezas de información, no solo "vuelve a intentarlo":

1. **El documento original** — para que el modelo pueda volver a examinar la fuente.
2. **La extracción fallida** — lo que el modelo produjo.
3. **El error de validación específico** — qué salió mal exactamente (ej. "las líneas suman £450 pero `stated_total` es £500").

> [!note] Por qué importa el orden de los tres
> Un reintento ingenuo (sin el error específico) típicamente reproduce el mismo error, porque el modelo no tiene ninguna señal nueva para corregirse. Con retroalimentación precisa, el modelo puede apuntar la corrección: revisar líneas faltantes, verificar ubicación de campos, recalcular totales.

### 2. La frontera de efectividad del retry

Esta es la idea más examinada de este tema — hay que poder distinguir un caso del otro con precisión.

| Los retries SÍ funcionan para | Los retries NO funcionan para |
|---|---|
| Discrepancias de formato (fechas, notación de moneda) | Información genuinamente ausente del documento fuente |
| Errores estructurales (valor en el campo equivocado, anidación incorrecta) | Datos que solo existen en un documento externo no provisto al modelo |
| Valores mal ubicados (el dato existe en el documento pero se extrajo al campo incorrecto) | Campos que requieren conocimiento que el modelo no tiene |
| Errores matemáticos (líneas faltantes que afectan el total) | — |

Si el documento no menciona el nombre de un departamento, reintentar no produce un valor correcto — solo hay dos opciones honestas: marcar el campo como `null` (si el schema lo permite) o enviar la extracción a revisión humana.

### 3. Diseño de self-correction dentro del schema

En vez de depender solo de lógica de validación externa, el schema mismo puede incluir campos que se auto-delaten cuando hay un problema:

- **`calculated_total` vs. `stated_total`**: se extraen *ambos* — la suma calculada de las líneas individuales y el total que dice el documento. Si difieren, la discrepancia queda marcada automáticamente, sin lógica externa.
- **`conflict_detected` (booleano)**: cuando la fuente contiene información contradictoria (ej. "pago a 30 días" en una sección y "net 60" en otra), se extraen ambos valores y se marca `conflict_detected: true`, en vez de que el modelo elija uno silenciosamente.

> [!note] Analogía
> Es como pedirle al niño que muestre su trabajo *y* el resultado final por separado, en vez de solo el resultado — si ambos no cuadran, el error salta a la vista sin que un profesor tenga que revisar cuenta por cuenta.

### 4. `detected_pattern`: loops de mejora sistemática

En pipelines de revisión de código o análisis, cada finding estructurado incluye un campo `detected_pattern` que registra qué construcción específica de código disparó ese finding.

- Cuando los desarrolladores descartan (dismiss) findings, se puede analizar el patrón de descarte agrupado por `detected_pattern`.
- Si un patrón específico (ej. "variable shadowing en scope anidado") se descarta consistentemente, probablemente el prompt necesita refinamiento para ese patrón.
- Esto crea un ciclo: extraer → validar → recolectar datos de descarte → refinar el prompt → repetir.

### 5. Errores de sintaxis de schema vs. errores semánticos de validación

Esta distinción conecta directamente con [[3 Structured Output with Tool Use/1 resumen|Structured Output with Tool Use]]:

- **Errores de sintaxis de schema**: JSON malformado, campos requeridos faltantes, tipo de dato incorrecto — **eliminados por completo** por `tool_use` con JSON schema.
- **Errores semánticos de validación**: estructura JSON correcta, pero valores incorrectos — líneas que no suman, fechas mal ordenadas, valores en el campo equivocado. Requieren **lógica de validación** aparte del schema, y son el foco de los loops de retry de este tema.

> [!note] El solapamiento es intencional
> El examen evalúa si entiendes que `tool_use` resuelve la primera categoría de error, pero no la segunda — son capas distintas del mismo pipeline, no soluciones intercambiables.

### 6. Pydantic como capa de validación

La guía nombra explícitamente **Pydantic** junto a JSON Schema para implementar el loop de validación-retry en pipelines de Python. Un modelo de Pydantic cumple dos funciones a la vez:

- **Parsing**: aplica estructura — tipos, campos requeridos, enums (equivalente al rol de `tool_use`/JSON schema).
- **Validators**: aplican semántica — reglas que un JSON schema no puede expresar, como aritmética entre campos u orden de fechas (ej. un `model_validator` que compara `calculated_total` contra `stated_total`).

Ambos tipos de fallo salen a través de una sola `ValidationError`, con errores legibles por máquina que nombran el campo y la regla incumplida — exactamente el tercer ingrediente (el error específico) que necesita el patrón retry-with-error-feedback, ya en formato directamente incorporable al prompt de reintento.

> [!note] Vigencia: parsing a nivel de SDK y Structured Outputs
> La guía menciona que, a la fecha, el SDK de Python permite `client.messages.parse(..., output_format=Invoice)` (devuelve instancias de Pydantic ya validadas vía `parsed_output`), y que `strict: true` en una definición de tool garantiza inputs conformes al schema del lado del servidor (Structured Outputs). Ninguno de los dos elimina la capa semántica: la plataforma refuerza el schema, los validators refuerzan las reglas de negocio, y el loop de retry consume el que falle. Para efectos del examen, el criterio a aplicar sigue siendo la distinción sintaxis/semántica de la Conclusión 5, no estos mecanismos más nuevos.

## Trampas de examen

> [!warning] Trampa 1 — Asumir que los retries siempre funcionan para fallos de extracción
> Los retries corrigen discrepancias de formato, errores estructurales y valores mal ubicados. **No pueden** producir información genuinamente ausente del documento fuente. El examen presenta ambos escenarios (corregible vs. no corregible) y espera que se distingan correctamente.

> [!warning] Trampa 2 — Implementar retries sin incluir el error de validación específico
> Un retry ingenuo sin retroalimentación de error produce el mismo error de nuevo. El modelo necesita ver exactamente qué salió mal (ej. "las líneas suman £450 pero el total declarado es £500") para poder auto-corregirse con eficacia — no basta con decir "vuelve a intentarlo".

> [!warning] Trampa 3 — Confiar solo en la validación de schema, sin checks semánticos
> La validación de schema (vía `tool_use`) atrapa errores de sintaxis. Los errores semánticos — sumas incorrectas, valores mal ubicados, datos fabricados — requieren lógica de validación adicional y loops de retry; el schema por sí solo no los detecta.

> [!warning] Trampa 4 — Tratar Pydantic como redundante una vez que `tool_use` ya impone un JSON schema
> El schema elimina errores de sintaxis, pero no puede expresar reglas semánticas entre campos — sumas que deben coincidir, fechas que deben estar ordenadas. Los validators de Pydantic codifican esas reglas y producen los mensajes de error específicos, por campo, que el loop de retry necesita devolver al modelo.

## En una frase

> Retry-with-error-feedback funciona enviando el documento original, la extracción fallida y el error de validación específico — pero corrige errores de formato y estructura, no puede crear información ausente de la fuente, así que siempre hay que identificar primero si un fallo es corregible antes de reintentar.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
