# Resultados — Tutor socrático: Subagent Invocation and Context Passing

> [!info] Sesión evaluada el 2026-09-18
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 97/100**

Dominio sólido y consistente en las 7 preguntas, cubriendo las 5 Conclusiones clave y las 4 Trampas de examen del tema sin un solo error. Las respuestas no fueron solo correctas sino completas: en la pregunta sobre la falla de atribución no solo identificaste la causa raíz, sino que recordaste de memoria los campos exactos del formato JSON de metadata (claim, url, document_name, page_number, confidence). No se detectó ninguna confusión conceptual durante la sesión.

## Conceptos con buen dominio

- **Task tool como compuerta binaria** — identificaste de inmediato que sin `"Task"`/`"Agent"` en `allowedTools` la invocación falla por completo, sin matices.
- **AgentDefinition y tool restrictions** — entendiste que dar herramientas fuera del rol rompe la separación de responsabilidades y abre riesgo de acciones no deseadas.
- **`fork_session` vs `--resume`** — distinguiste correctamente bifurcar (explorar enfoques divergentes) de continuar (agregar a la misma sesión), en dos preguntas distintas (una directa, otra sobre nomenclatura del examen).
- **Invocación paralela vs. secuencial** — reconociste el problema de latencia y la solución (múltiples llamadas al Task tool en una sola respuesta) sin ayuda.
- **Paso de contexto y metadata estructurada** — el patrón de examen más específico del tema (falla de atribución por pérdida de metadata) lo respondiste con precisión completa, incluyendo qué NO se debe hacer (tocar el prompt del agente de síntesis).
- **Aislamiento total de contexto** — confirmaste que no hay herencia automática del historial del coordinador ni de otros subagentes.
- **Task vs. Agent para el examen** — supiste que la respuesta correcta en el examen es "Task", aunque el código actual use "Agent".

## Áreas que necesitan revisión

Ninguna. No se evidenció ningún error ni respuesta incompleta durante la sesión.

## Siguiente paso recomendado

Con este nivel de dominio, estás listo para pasar directamente a [[4 test]] y validar el entendimiento bajo el formato real de examen (casos de producción, distractores plausibles).
