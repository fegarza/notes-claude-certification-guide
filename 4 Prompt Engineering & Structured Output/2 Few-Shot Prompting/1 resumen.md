## Explícamelo como si tuviera 5 años

Imagina que le enseñas a un niño a separar la ropa para lavar. Le dices "separa la ropa oscura de la clara, usa tu criterio con las dudosas". El niño ve una camiseta blanca con una raya roja y no sabe qué hacer — la mete con las claras hoy, con las oscuras mañana. Tu instrucción sonaba clara, pero no le diste ninguna regla que cubra el caso dudoso.

Ahora en vez de eso le muestras tres prendas ya separadas, y por cada una le explicas el porqué: "esta playera blanca con una raya roja delgada va con las claras, porque la raya es pequeña y no destiñe mucho; esta camisa azul oscuro con estampado blanco va con las oscuras, porque el azul domina y sí puede destiñir". El niño no memoriza esas tres prendas exactas — aprende el **principio** ("¿qué color domina y qué tan probable es que destiña?") y lo aplica a prendas que nunca había visto.

Con Claude pasa igual: cuando las instrucciones detalladas ya no bastan para lograr consistencia, mostrarle **2-4 ejemplos con el razonamiento incluido** (no solo el resultado final) es lo que le permite generalizar el criterio a casos nuevos, en vez de solo memorizar los ejemplos exactos que le diste.

## Argumento central

> [!note] Idea central
> Cuando instrucciones detalladas por sí solas producen resultados inconsistentes, los **ejemplos few-shot** (2-4, con razonamiento explícito de por qué se eligió esa acción/clasificación) son la técnica más efectiva para lograr salida consistente y accionable. El razonamiento dentro del ejemplo es lo que enseña el **principio de decisión** al modelo, no solo el caso puntual — por eso generaliza a patrones nuevos que nunca vio.

## Conclusiones clave

### 1. Tres disparadores para usar few-shot

No cualquier problema se arregla con few-shot — la guía identifica tres escenarios concretos donde es la herramienta correcta:

1. **Inconsistencia de formato** a pesar de instrucciones detalladas y exhaustivas.
2. **Juicios ambiguos** que producen clasificaciones distintas para entradas similares (ej. qué herramienta elegir ante una solicitud ambigua, o si una brecha de cobertura de tests a nivel de rama es significativa).
3. **Campos de extracción vacíos** cuando la información sí existe en el texto, pero en un formato inesperado.

**Analogía:** son los tres tipos de "dudas de la ropa" del ejemplo — no saber dónde va una prenda mixta (formato/juicio ambiguo), o directamente no reconocer que una prenda manchada de tinta *sí* cuenta como "ropa oscura para lavar por separado" (campo vacío por formato inesperado).

### 2. Cómo construir buenos ejemplos few-shot

- **Cantidad:** 2-4 ejemplos dirigidos. Menos de 2 no establece un patrón reconocible; más de 4 gasta tokens sin aportar señal adicional.
- **Contenido:** cada ejemplo debe incluir el **razonamiento** de por qué se tomó esa decisión, no solo el par entrada-salida.
- **Foco:** los ejemplos deben construirse específicamente sobre los **escenarios que están fallando**, no sobre casos genéricos que ya funcionan bien.
- Para tareas de revisión o clasificación, esto significa mostrar el formato de salida deseado completo (ej. ubicación, problema, severidad, corrección sugerida) para lograr consistencia real.

> [!note] Ejemplo mental
> No es lo mismo mostrar "Input: X → Output: Y" que mostrar "Input: X → Como X tiene la propiedad Z, y Z siempre implica Y en este contexto, entonces → Output: Y". La segunda forma es la que enseña el criterio.

### 3. El razonamiento enseña generalización, no memorización

- El componente de razonamiento es lo que hace la diferencia entre que el modelo **memorice casos literales** y que **aprenda el principio de decisión** detrás de ellos.
- Esto es lo que permite manejar **patrones nuevos** que no aparecían en los ejemplos — no solo repetir el caso exacto que se mostró.
- Dos dominios típicos donde esto se aplica: elegir la herramienta correcta ante una solicitud ambigua, y decidir si una brecha de cobertura de tests a nivel de rama merece ser señalada.

### 4. Reducir alucinación en extracción con documentos variados

- Cuando una tarea de extracción devuelve campos vacíos o nulos aunque la información exista, pero en un formato distinto al esperado (ej. mediciones informales, estructuras de documento poco convencionales), la causa suele ser falta de ejemplos que cubran esa variedad.
- Mostrar ejemplos few-shot con **estructuras de documento variadas** (citas en línea vs. bibliografías, secciones de metodología vs. detalles incrustados en el texto) enseña al modelo a reconocer la información aunque no venga en el formato "canónico".
- Esto también reduce falsos positivos: mostrar ejemplos que distinguen **patrones de código aceptables** de **problemas genuinos** ayuda a generalizar el criterio de "esto sí es un hallazgo" sin sacrificar precisión.

### 5. Distinción clave: few-shot no es la solución universal

> [!warning] Cada síntoma tiene su técnica — no todas se arreglan con más ejemplos
> Few-shot arregla la **falta de un patrón demostrado** para casos ambiguos o formatos inconsistentes. Pero otros síntomas parecidos tienen causas raíz distintas, y por tanto otra técnica correcta:
> - **Valores fabricados/alucinados** en un schema → campos opcionales/nullable en el schema (tema de [[3 Structured Output with Tool Use/1 resumen|Structured Output with Tool Use]]), no más ejemplos.
> - **Discrepancias de cálculo** (el modelo "hace mal las cuentas") → bucles de validación y reintento (tema de [[4 Validation, Retry, and Feedback Loops/1 resumen|Validation, Retry, and Feedback Loops]]), no few-shot.
> - **Problema de enrutamiento inicial** (el modelo elige la herramienta equivocada porque su descripción es pobre) → mejorar la **descripción de la herramienta** primero, no agregar ejemplos few-shot encima de una descripción insuficiente.

**Por qué importa esta distinción:** el examen prueba que se entienda que few-shot ataca causas raíz específicas (inconsistencia de formato, juicio ambiguo, extracción con formato inesperado) — no es un parche genérico para "el modelo se porta raro".

## Trampas de examen

> [!warning] Trampa 1 — Agregar más instrucciones detalladas cuando ya no bastan
> Si instrucciones detalladas y exhaustivas ya se probaron y el resultado sigue siendo inconsistente, seguir añadiendo *más* instrucciones no es la respuesta correcta — es exactamente el escenario que dispara el uso de few-shot. La guía es explícita: pasado ese punto, demostrar con ejemplos supera a seguir instruyendo.

> [!warning] Trampa 2 — Ejemplos sin razonamiento, solo pares entrada-salida
> Un ejemplo few-shot que solo muestra "esto entra, esto sale" sin explicar el porqué no enseña el principio de decisión — el modelo puede memorizar el caso literal en vez de generalizar. La forma correcta siempre incluye el razonamiento de por qué se eligió esa acción o clasificación sobre las alternativas plausibles.

> [!warning] Trampa 3 — Usar umbrales de confianza para arreglar juicios inconsistentes
> Ante clasificaciones o decisiones ambiguas inconsistentes, la opción tentadora es filtrar por confianza del modelo. No es la solución: la confianza auto-reportada está mal calibrada (mismo problema visto en [[1 System Prompts with Explicit Criteria/1 resumen|System Prompts with Explicit Criteria]]). El arreglo correcto ante juicios ambiguos es mostrar ejemplos few-shot con razonamiento, no imponer un umbral de confianza.

> [!warning] Trampa 4 — Usar few-shot cuando el problema real es la descripción de la herramienta
> Si el modelo elige la herramienta equivocada porque dos herramientas tienen descripciones mínimas y similares, agregar ejemplos few-shot de enrutamiento *funciona parcialmente* pero no ataca la causa raíz y suma tokens de forma innecesaria. El primer paso correcto en ese caso es enriquecer la descripción de cada herramienta (formatos de entrada, casos límite, cuándo usar una vs. la otra) — few-shot es una técnica secundaria, no el primer paso, cuando el síntoma es de enrutamiento por descripciones pobres.

## En una frase

> Cuando las instrucciones detalladas ya no logran consistencia, 2-4 ejemplos few-shot con razonamiento explícito —enfocados en los casos que fallan— son lo que enseña al modelo a generalizar el criterio correcto, en vez de solo memorizar casos puntuales.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
