---
tags:
  - claude-cert/dominio-2
  - task-statement/2.3
---

# 2.3 — Tool Distribution & Tool Choice

## Explícamelo como si tuviera 5 años

Imagina que le das a un solo empleado 18 herramientas distintas para su trabajo — un martillo, tres tipos de destornillador, dos llaves inglesas, una sierra, etc. — y le pides que arregle una silla. Va a tardar más en decidir cuál herramienta usar que en usarla, y es más probable que tome la equivocada. Ahora imagina que a ese mismo empleado, en vez de darle solo lo necesario para arreglar sillas, también le das el martillo del carpintero de al lado "por si acaso" — empieza a meterse en trabajos que no le tocan, y probablemente lo hace peor que el especialista.

Eso es exactamente lo que pasa con los agentes: cuántas herramientas les das, y cuáles, no es un detalle de implementación — es una decisión de arquitectura. Demasiadas herramientas confunden la selección; herramientas fuera de su rol invitan a que las use mal.

## Argumento central

> [!note] Idea central
> El número de tools que le das a un agente afecta directamente qué tan confiable es al elegir la correcta. Esto no es un detalle de implementación, es una decisión arquitectónica que determina si tu sistema multi-agente funciona en producción. La solución no es solo "menos tools", sino **tools bien acotadas al rol** (*scoped*) más una configuración correcta de `tool_choice` para controlar cuándo el modelo debe usarlas.

## Conclusiones

### 1. El problema de la sobrecarga de tools

- Darle a un solo agente 18 tools degrada su confiabilidad de selección — no es un límite arbitrario, es un patrón observado.
- Cada tool adicional agrega complejidad de decisión, y la tasa de error sube conforme crece el toolkit.
- El rango óptimo es **4-5 tools por agente**, acotadas al rol específico de ese agente.

> [!tip] Analogía mental
> Es como un menú de restaurante: uno con 5 platos bien definidos hace que decidir sea rápido y que rara vez pidas algo que no querías. Un menú de 18 páginas te hace dudar, comparar de más, y a veces terminas pidiendo lo que no era.

### 2. Agentes con tools fuera de su especialización tienden a mal-usarlas

Si un agente de síntesis tiene acceso a búsqueda web, es probable que la use para repetir investigación en vez de trabajar con los resultados que ya le dieron — duplicando trabajo que otro agente ya hizo. La causa no es que el modelo sea torpe: es que la tool está disponible, así que a veces la usa, aunque no sea su rol.

### 3. Consolidar tools casi-duplicadas en vez de solo dividir por rol

Dividir por rol es la respuesta obvia a la sobrecarga de tools — pero es la incorrecta cuando todas las tools hacen básicamente el mismo tipo de trabajo.

> [!note] Caso: 22 tools de una plataforma de datos
> Una plataforma tiene 3 tools de consulta (una por fuente de datos) y 19 operaciones de transformación (`pivot_table`, `calculate_percentile`, `normalise_currency`, etc.). Dividir por rol dejaría a un "agente de transformación" cargando con 19 tools — mueve el problema, no lo resuelve.
>
> La solución: reconocer que las 19 transformaciones comparten el mismo patrón (reciben datos, aplican una operación, devuelven datos) y consolidarlas en **una sola tool parametrizada**, `transform_data`, con un parámetro enum (`transform_type`) que selecciona la operación. Esto reduce 22 tools a 4 en total. Cada transformación sigue siendo alcanzable — ahora como un valor de enum que el modelo elige dentro de una sola llamada, en vez de una tool que tiene que encontrar entre diecinueve descripciones casi idénticas.

### 4. Configuración de `tool_choice`: tres modos, tres trabajos distintos

| Modo | Estructura | Comportamiento | Cuándo usarlo |
|---|---|---|---|
| **`auto`** (default) | `{"type": "auto"}` | El modelo decide si llama una tool o responde en texto conversacional. | Cuando se necesita flexibilidad y la llamada a una tool no debe ser obligatoria. |
| **`any`** | `{"type": "any"}` | El modelo **debe** llamar alguna tool, pero elige cuál. Garantiza salida estructurada. | Pipelines de extracción con varios schemas posibles (factura, recibo, contrato) donde no se sabe de antemano el tipo de documento, y se necesita que el modelo produzca datos estructurados en vez de conversación. |
| **Selección forzada** | `{"type": "tool", "name": "extract_metadata"}` | El modelo **debe** llamar la tool específica nombrada. | Forzar un paso obligatorio del workflow que no puede saltarse (ej. exigir `extract_metadata` antes de que corran las tools de enriquecimiento). Tras esa llamada forzada, los siguientes turnos pueden volver a `auto`. |

> [!tip] Analogía mental
> `auto` es dejar que alguien decida libremente si necesita una herramienta o no. `any` es decirle "usa alguna herramienta, la que sea, pero no me contestes solo de palabra". Selección forzada es ponerle en la mano la herramienta específica y decirle "usa exactamente esta ahora".

### 5. Tools acotadas cruzando roles (*scoped cross-role tools*)

A veces un agente necesita, de forma ocasional, una capacidad que pertenece a otro rol. El enfoque ingenuo es rutear siempre esa necesidad a través del coordinador — pero si la necesidad es frecuente y casi siempre simple, eso desperdicia recursos en idas y vueltas innecesarias.

> [!note] Caso: `verify_fact` en el agente de síntesis
> Un agente de síntesis, al combinar hallazgos para un reporte, necesita frecuentemente verificar afirmaciones puntuales. El enfoque ingenuo: cada verificación regresa al coordinador, que delega al agente de búsqueda especializado, y vuelve a invocar síntesis con el resultado — esto agrega 2-3 idas y vueltas por tarea. El análisis muestra que el 85% de esas verificaciones son búsquedas simples de un solo dato (fechas, nombres, estadísticas), y solo el 15% requiere investigación más profunda.
>
> La solución: darle al agente de síntesis una tool `verify_fact` **acotada**, que maneja directamente los casos simples de una sola fuente. Las verificaciones complejas (que requieren múltiples fuentes, cruce de datos, o juicio sustancial) siguen escalando al coordinador. Esto divide la carga: lo rutinario se procesa localmente, lo genuinamente complejo usa el pipeline completo — reduciendo la latencia hasta en un 40%.

### 6. Reemplazar tools genéricas por alternativas acotadas (principio de mínimo privilegio)

En vez de darle a un subagente `fetch_url` (que puede traer cualquier cosa de cualquier lugar), se le da `load_document`, que valida que la URL sea específicamente de un documento.

- **Previene mal uso**: el agente no puede recuperar URLs o recursos arbitrarios.
- **Clarifica la intención**: una descripción específica hace transparente el propósito de la tool, en vez de uno genérico.
- **Limita efectos secundarios**: solo se puede acceder a los recursos apropiados para ese rol.

> [!tip] Analogía mental
> Es la diferencia entre darle a alguien una llave maestra que abre todo el edificio ("por si acaso la necesita") y darle solo la llave de su propia oficina. La llave maestra no lo hace mejor en su trabajo — solo aumenta el riesgo de que abra puertas que no debía.

### 7. Distribución de tools en la práctica: un sistema multi-agente de investigación

Un sistema multi-agente bien diseñado distribuye las tools así, cada rol con exactamente lo que necesita:

- **Agente de búsqueda web** (4 tools): `search_web`, `fetch_page`, `extract_links`, `save_snippet`.
- **Agente de análisis de documentos** (4 tools): `extract_metadata`, `extract_data_points`, `summarize_content`, `verify_claim`.
- **Agente de síntesis** (4 tools): `compile_report`, `verify_fact` (acotada, ver Conclusión 5), `format_citation`, `assess_coverage`.
- **Agente coordinador** (sin tools de dominio): `Agent` (invoca subagentes), `review_output`, `request_revision`.

El coordinador orquesta el flujo sin tener capacidades específicas de dominio — su trabajo es delegar y revisar, no ejecutar tareas de investigación él mismo.

## En una frase

> Dar a un agente las tools correctas no es "cuantas más, mejor" ni "una por cada capacidad imaginable" — es 4-5 tools acotadas a su rol, consolidando duplicados en tools parametrizadas, con tools acotadas cruzando roles solo para el 80-85% de casos frecuentes y simples, y `tool_choice` configurado según si se necesita flexibilidad (`auto`), salida estructurada garantizada (`any`), o un paso obligatorio (selección forzada).

## Trampas de examen

> [!warning] Trampa 1 — Rutear todo lo simple a través del coordinador
> Cuando ~85% de las verificaciones (u otra tarea similar) son búsquedas simples, seguir ruteando cada una a través del coordinador agrega 2-3 saltos innecesarios por solicitud. **Por qué es un error:** desperdicia latencia (hasta 40%) en casos que una tool acotada directamente en el agente resolvería en milisegundos. La forma correcta es dar una tool `verify_fact`-style acotada al agente que la necesita con frecuencia, y solo escalar al coordinador los casos genuinamente complejos.

> [!warning] Trampa 2 — Usar `auto` cuando se requiere salida estructurada
> Con `tool_choice: auto`, el modelo puede decidir responder con texto conversacional en vez de llamar una tool. **Por qué es un error:** si el sistema necesita garantizar una llamada a tool (ej. un pipeline de extracción que siempre debe producir datos estructurados), `auto` no lo garantiza. La forma correcta es usar `any` (el modelo debe llamar alguna tool, pero elige cuál) o selección forzada (debe llamar una tool específica).

> [!warning] Trampa 3 — Sobrecarga de tools (18+ en un solo agente)
> Asignar 18 o más tools a un solo agente socava drásticamente la confiabilidad de selección. **Por qué es un error:** cada tool adicional agrega complejidad de decisión y multiplica los errores de selección — el rango óptimo es 4-5 tools por agente, acotadas a su rol.

> [!warning] Trampa 4 — Dar tools genéricas en vez de versiones acotadas
> Darle a un subagente una tool de acceso amplio como `fetch_url` en vez de una alternativa construida a propósito. **Por qué es un error:** viola el principio de mínimo privilegio — una tool genérica permite mal uso, oscurece la intención real de la tool, y aumenta el riesgo de efectos secundarios no deseados. La forma correcta es una tool acotada como `load_document`, que solo valida y acepta URLs de documentos.

> [!info] Pregunta de muestra oficial (examguide.pdf, Sample Question 9)
> El escenario del agente de síntesis que regresa control al coordinador para verificaciones simples, agregando 2-3 idas y vueltas y 40% de latencia, con 85% de los casos siendo verificaciones simples, es literalmente la Question 9 de la sección "9. Sample Questions" del Exam Guide oficial. La respuesta correcta es dar al agente de síntesis una tool `verify_fact` acotada para los casos simples, mientras las verificaciones complejas siguen delegándose al agente de búsqueda a través del coordinador — exactamente la Conclusión 5 de este resumen.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
