# Resultados — Tutor socrático: Tool Distribution & Tool Choice

> [!info] Sesión evaluada el 2026-10-09
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 62/100**

La base conceptual está sólida: entiendes por qué la sobrecarga de tools degrada la selección, el rango óptimo de 4-5 tools acotadas, la consolidación con enum y la diferencia entre `auto` y `any` (justificaste bien por qué descartar los otros dos modos). El desempeño baja en la segunda mitad del tema. Hubo respuestas parciales en selección forzada, en el caso `verify_fact` y en mínimo privilegio, y tres preguntas sin respuesta (escalamiento de casos complejos y el 40% de latencia, los beneficios de `load_document`, y las tools del coordinador). Aquí faltan justo los detalles concretos que el examen suele evaluar.

## Conceptos con buen dominio

- **Argumento central** — identificaste que el número de tools afecta la confiabilidad de selección, por eso es una decisión arquitectónica.
- **Rango óptimo (4-5 tools acotadas al rol)** — respuesta exacta.
- **Consolidar tools casi-duplicadas** — explicaste por qué dividir por rol solo mueve el problema y propusiste por tu cuenta la tool parametrizada con enum.
- **`tool_choice: auto`** — sabes que el modelo decide entre llamar una tool o responder con texto.
- **`tool_choice: any` en pipelines de extracción** — elegiste bien y argumentaste por qué no sirven `auto` ni la selección forzada (Trampa 2 dominada).
- **Agentes con tools fuera de su rol** — reconociste el retrabajo/duplicación de investigación.

## Áreas que necesitan revisión

### Selección forzada y qué pasa después

- **Qué pasó:** acertaste en forzar `extract_metadata` en el primer turno, pero no diste la estructura exacta. Para los turnos siguientes propusiste cambiar a `any` y quitar la tool, cuando la guía indica volver a `auto`.
- **Contenido del resumen:**

| Modo | Estructura | Comportamiento | Cuándo usarlo |
|---|---|---|---|
| **Selección forzada** | `{"type": "tool", "name": "extract_metadata"}` | El modelo **debe** llamar la tool específica nombrada. | Forzar un paso obligatorio del workflow que no puede saltarse (ej. exigir `extract_metadata` antes de que corran las tools de enriquecimiento). Tras esa llamada forzada, los siguientes turnos pueden volver a `auto`. |

- **Sección:** [[1 resumen]] → "4. Configuración de `tool_choice`: tres modos, tres trabajos distintos"
- **Analogía para reforzarlo:** es como el check-in de un vuelo: es obligatorio pasar por el mostrador primero (forzado), pero ya con tu pase de abordar eres libre de ir al café, a la tienda o directo a la sala (`auto`). Nadie te obliga a "entrar a alguna tienda" (`any`).

### Caso `verify_fact`: costo, división 85/15 y beneficio

- **Qué pasó:** identificaste la latencia como costo, pero sin el dato de 2-3 idas y vueltas. Propusiste la tool acotada sin nombrarla y sin decir qué pasa con el 15% complejo, y no recordaste que las verificaciones complejas siguen escalando al coordinador ni la mejora de hasta 40% en latencia. Es la Sample Question 9 oficial.
- **Contenido del resumen:**

> Un agente de síntesis, al combinar hallazgos para un reporte, necesita frecuentemente verificar afirmaciones puntuales. El enfoque ingenuo: cada verificación regresa al coordinador, que delega al agente de búsqueda especializado, y vuelve a invocar síntesis con el resultado — esto agrega 2-3 idas y vueltas por tarea. El análisis muestra que el 85% de esas verificaciones son búsquedas simples de un solo dato (fechas, nombres, estadísticas), y solo el 15% requiere investigación más profunda.
>
> La solución: darle al agente de síntesis una tool `verify_fact` **acotada**, que maneja directamente los casos simples de una sola fuente. Las verificaciones complejas (que requieren múltiples fuentes, cruce de datos, o juicio sustancial) siguen escalando al coordinador. Esto divide la carga: lo rutinario se procesa localmente, lo genuinamente complejo usa el pipeline completo — reduciendo la latencia hasta en un 40%.

- **Sección:** [[1 resumen]] → "5. Tools acotadas cruzando roles (*scoped cross-role tools*)"
- **Analogía para reforzarlo:** en un hospital, la enfermera puede tomar la presión y la temperatura ella misma (85% de los casos, rápido) sin llamar al médico cada vez. Pero si algo sale raro o necesita un diagnóstico, sí escala al médico especialista (15%). Si llamara al médico para cada termómetro, la sala de espera se atascaría.

### Mínimo privilegio: `fetch_url` vs. `load_document`

- **Qué pasó:** hablaste de "responsabilidades acotadas al rol", pero no nombraste el principio de mínimo privilegio, no mencionaste que `load_document` valida que la URL sea de un documento y no recordaste dos de los tres beneficios (clarificar la intención y limitar efectos secundarios).
- **Contenido del resumen:**

> En vez de darle a un subagente `fetch_url` (que puede traer cualquier cosa de cualquier lugar), se le da `load_document`, que valida que la URL sea específicamente de un documento.
>
> - **Previene mal uso**: el agente no puede recuperar URLs o recursos arbitrarios.
> - **Clarifica la intención**: una descripción específica hace transparente el propósito de la tool, en vez de uno genérico.
> - **Limita efectos secundarios**: solo se puede acceder a los recursos apropiados para ese rol.

- **Sección:** [[1 resumen]] → "6. Reemplazar tools genéricas por alternativas acotadas (principio de mínimo privilegio)"
- **Analogía para reforzarlo:** es como darle a un repartidor una tarjeta de crédito que solo acepta cargos en gasolineras, en vez de una tarjeta abierta. No puede gastar en otra cosa (previene mal uso), todos entienden para qué es (clarifica la intención) y si la pierde, el daño es limitado (efectos secundarios).

### Tools del coordinador

- **Qué pasó:** no recordaste qué tools tiene el coordinador ni por qué no tiene tools de dominio.
- **Contenido del resumen:**

> - **Agente coordinador** (sin tools de dominio): `Agent` (invoca subagentes), `review_output`, `request_revision`.
>
> El coordinador orquesta el flujo sin tener capacidades específicas de dominio — su trabajo es delegar y revisar, no ejecutar tareas de investigación él mismo.

- **Sección:** [[1 resumen]] → "7. Distribución de tools en la práctica: un sistema multi-agente de investigación"
- **Analogía para reforzarlo:** el director de orquesta no toca ningún instrumento. Su "kit" es la batuta para indicar quién entra, el oído para revisar cómo suena y la señal de "otra vez desde el compás 20". Si se pusiera a tocar el violín, dejaría de dirigir.

## Siguiente paso recomendado

Antes de ir a [[4 test]], repasa las Conclusiones 5, 6 y 7 del resumen: el caso `verify_fact` completo (2-3 round trips, 85/15, escalamiento de los casos complejos, 40%), los tres beneficios de mínimo privilegio y las tools del coordinador. También fija que tras una selección forzada se vuelve a `auto`.
