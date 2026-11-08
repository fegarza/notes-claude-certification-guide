# Resultados — Tutor socrático: Agentic loop

> [!info] Sesión evaluada el 2026-10-26
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 88/100**

De 10 preguntas, 8 fueron correctas y 2 parcialmente correctas. Hay una mejora clara respecto a la sesión anterior (75/100): los dos puntos débiles de entonces, los valores adicionales de `stop_reason` y el distractor de límites de iteración ante una terminación prematura, ahora se respondieron completos y sin pistas. Lo que queda por pulir es conceptual y de vocabulario de examen: por qué se prefiere el enfoque model-driven (flexibilidad) y qué distingue al agentic loop de un loop de reintentos.

## Conceptos con buen dominio

- **`stop_reason` como señal de control:** identificó de inmediato que `"end_turn"` termina el loop y que la señal viene en la respuesta de la Messages API.
- **Ciclo con `tool_use`:** describió bien cómo se ejecuta la herramienta y se agrega el resultado al historial antes de reenviar, y además entiende por qué ese paso es indispensable.
- **Valores adicionales de `stop_reason`:** detectó el supuesto erróneo de que "todo lo que no sea `end_turn` es `tool_use`" y nombró `max_tokens`, `refusal` y `pause_turn`. Este punto era débil en la sesión anterior.
- **Trampa 1 (`content[0].type == "text"`):** explicó con precisión que puede llegar texto junto con un `tool_use` en la misma respuesta.
- **Distractor de límites de iteración:** distinguió con claridad entre loops descontrolados y terminaciones prematuras. También era débil en la sesión anterior.
- **Trampa 2 (límite de iteraciones como control principal):** nombró los dos problemas opuestos: corta trabajo útil o fuerza iteraciones innecesarias.
- **Trampa 3 (parsear lenguaje natural):** explicó la ambigüedad del texto y el riesgo de que una frase de cierre aparezca con un `tool_use` pendiente.
- **Trampa 4 (`tool_choice: "any"`):** conectó directamente forzar herramientas con el riesgo de un loop infinito.

## Áreas que necesitan revisión

### Por qué se prefiere model-driven decision-making

- **Qué pasó:** Reconoció bien el enfoque model-driven en el escenario de la secuencia fija de herramientas. Pero al preguntarle por qué se prefiere, respondió cuándo conviene el enfoque contrario (código determinista) y no mencionó la razón central: la **flexibilidad**. Además formuló la excepción como "riesgo de error crítico irreversible", mientras que la guía usa el criterio de "cumplimiento determinista" en temas financieros, de seguridad o regulatorios.
- **Contenido del resumen:**
  > En un agentic loop, es Claude quien decide qué herramienta llamar según el contexto actual — lee la tarea, evalúa las herramientas disponibles, y elige una. Esto contrasta con árboles de decisión preconfigurados o secuencias fijas de herramientas donde el desarrollador programa a mano el orden de ejecución.
  >
  > El enfoque dirigido por el modelo es preferido porque es flexible — **excepto** cuando la lógica de negocio exige cumplimiento determinista (temas financieros, de seguridad o regulatorios), donde la aplicación programática de reglas debe imponerse sobre la flexibilidad del modelo.
- **Sección:** [[1 resumen]] → "3. Toma de decisiones dirigida por el modelo (model-driven decision-making)"
- **Analogía para reforzarlo:** Un GPS que recalcula la ruta según el tráfico (model-driven) llega mejor que un mapa impreso con una ruta fija, porque se adapta a lo que encuentra. Pero si la ley prohíbe pasar por cierta calle, eso no se deja a criterio del GPS: es una regla que se impone desde fuera.

### El agentic loop no es un loop de reintentos

- **Qué pasó:** Rebatió correctamente que el loop sea "un buen system prompt": está definido en código y se controla con `stop_reason`. Pero no supo explicar por qué tampoco es "un loop de reintentos", que es una de las tres cosas que el argumento central dice explícitamente que el agentic loop **no** es.
- **Contenido del resumen:**
  > El agentic loop es el ciclo determinista de control de flujo (definido en código) detrás de todo agente basado en Claude — no es un truco de prompt, ni un loop de reintentos, ni simplemente un turno de chatbot. El único mecanismo confiable para controlarlo es el campo `stop_reason` de la respuesta de la Messages API.
- **Sección:** [[1 resumen]] → "Argumento central"
- **Analogía para reforzarlo:** Volver a marcar porque la llamada se cortó es un reintento: algo falló y repites lo mismo. Un cocinero que vuelve a la cocina porque el cliente pidió otro platillo no está reintentando nada. Es el flujo normal del trabajo, impulsado por una petición nueva. El agentic loop es lo segundo: vuelve a llamar a Claude porque Claude **pidió** una herramienta (`tool_use`), no porque algo haya fallado.

## Siguiente paso recomendado

Repasa el "Argumento central" (qué **no** es un agentic loop) y la sección 3 sobre model-driven decision-making, con atención al criterio exacto de la excepción (cumplimiento determinista: financiero, seguridad, regulatorio). Con eso estás listo para [[4 test]].
