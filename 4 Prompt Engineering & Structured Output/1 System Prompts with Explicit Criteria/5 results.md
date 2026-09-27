# Resultados — Tutor socrático: System Prompts with Explicit Criteria

> [!info] Sesión evaluada el 2026-09-20
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 90/100**

Dominio sólido del tema: identificaste con precisión el argumento central, la lógica detrás de la contaminación global de confianza, la solución contraintuitiva de deshabilitar temporalmente una categoría problemática, por qué la prosa es insuficiente para calibrar severidad, y por qué la confianza auto-reportada de un LLM no sirve como filtro primario. Las únicas fricciones fueron identificar el *contenido específico* de un disparador explícito en vez de solo su categoría, y completar el razonamiento del "por qué" en dos preguntas donde ya tenías la mitad correcta.

## Conceptos con buen dominio

- Vago vs. explícito (por qué falla lo vago) — identificaste correctamente que instrucciones como "sé conservador" no son accionables y dependen del contexto.
- Analogía del letrero de tránsito — reconociste que la verificabilidad es la característica clave que distingue un criterio útil de uno vago.
- Contaminación global de la confianza — explicaste con precisión que los desarrolladores no compartimentan su desconfianza por categoría, sino que juzgan el reporte completo.
- Primer paso de la solución contraintuitiva — identificaste correctamente deshabilitar temporalmente la categoría problemática.
- Por qué la prosa es insuficiente para calibrar severidad — articulaste bien que deja la interpretación abierta al modelo, mientras que el código da un criterio concreto.
- Por qué la confianza auto-reportada no sirve como filtro — identificaste la mala calibración del LLM como causa raíz.
- Trampa 1 (por qué son distractores tentadores) — explicaste correctamente que suenan a buena práctica profesional aunque no sean accionables.

## Áreas que necesitan revisión

### Conclusión 1 — El disparador explícito del ejemplo correcto

- **Qué pasó:** Al pedir cuál era el disparador explícito y verificable del ejemplo de "enfoque correcto", respondiste describiendo la categoría de la respuesta ("un disparador explícito categórico") en vez de identificar el contenido real del disparador.
- **Contenido del resumen:**
  > **Enfoque correcto** (categorías concretas y accionables):
  > ```
  > Flag comments only when claimed behaviour contradicts actual code behaviour.
  > Report bugs and security vulnerabilities.
  > Skip minor style preferences and local patterns.
  > ```
  > Aquí hay tres cosas concretas: qué reportar (bugs, seguridad), qué excluir (estilo, patrones locales) y un disparador explícito (contradicción entre comportamiento declarado y comportamiento real).
- **Sección:** [[1 resumen]] → "1. Vago vs. explícito: la diferencia real"
- **Analogía para reforzarlo:** Es como la diferencia entre decirle a un inspector de calidad "reporta cuando algo esté mal etiquetado" (categoría, sin contenido) y decirle "reporta cuando la etiqueta del producto diga 'sin gluten' pero la lista de ingredientes incluya trigo" (el disparador real y verificable). Nombrar que existe un disparador no es lo mismo que decir cuál es.

### Conclusión 4 — Orden entre criterios explícitos y enrutamiento por confianza

- **Qué pasó:** Identificaste correctamente que la confianza sirve como mecanismo de enrutamiento (ej. a revisión humana), pero no mencionaste el orden en el que debe aplicarse respecto a los criterios explícitos.
- **Contenido del resumen:**
  > La confianza sí tiene un uso legítimo, pero como **mecanismo de enrutamiento** (ej. mandar hallazgos de baja confianza a revisión humana...), nunca como filtro primario de validez.
  > **Secuencia correcta:** primero criterios explícitos, después (opcionalmente) enrutamiento por confianza. Nunca al revés, y nunca saltarse el primer paso.
- **Sección:** [[1 resumen]] → "4. Por qué el filtrado por confianza no resuelve el problema"
- **Analogía para reforzarlo:** Es como triage en una sala de urgencias: primero necesitas criterios médicos claros para decidir qué es una emergencia (criterios explícitos), y *después* puedes usar una señal secundaria como "el paciente dice que le duele mucho" para priorizar dentro de esa lista (enrutamiento). Usar solo el dolor auto-reportado del paciente para decidir qué es una emergencia, sin criterios médicos previos, es el orden invertido que el resumen descarta.

### Trampa 3 — Por qué mantener la categoría activa "mientras se arregla" es la opción incorrecta

- **Qué pasó:** Identificaste correctamente la opción tentadora del escenario de CI/CD (actualizar la categoría problemática sobre la marcha, en producción), pero no llegaste a articular explícitamente por qué esa opción sigue siendo dañina mientras se itera.
- **Contenido del resumen:**
  > **Solución (contraintuitiva):** no es "seguir puliendo la categoría problemática mientras sigue activa" — eso sigue erosionando la confianza en todo el sistema mientras iteras.
  > (Este es el escenario del pipeline de CI/CD con 40% de falsos positivos en "documentation mismatch" que aparece como pregunta de práctica en la guía.)
- **Sección:** [[1 resumen]] → "Trampa 3 — Mantener todas las categorías activas mientras se arregla la problemática"
- **Analogía para reforzarlo:** Es como seguir sirviendo un plato con receta defectuosa mientras el chef "la va ajustando poco a poco" en vez de retirarlo del menú de inmediato — cada plato servido durante el ajuste sigue dañando la reputación del restaurante completo, no solo la de ese plato.

## Siguiente paso recomendado

El dominio conceptual es fuerte (90%); antes de intentar [[4 test]], repasa específicamente el contenido exacto del "disparador explícito" en el ejemplo de la Conclusión 1 y el orden criterios-antes-que-confianza de la Conclusión 4 — son los dos puntos donde tuviste la idea general pero no el detalle preciso que el examen suele pedir.
