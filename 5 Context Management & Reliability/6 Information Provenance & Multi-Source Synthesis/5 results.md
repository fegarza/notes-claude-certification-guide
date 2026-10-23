# Resultados — Tutor socrático: Information Provenance & Multi-Source Synthesis

> [!info] Sesión evaluada el 2026-10-20
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 82/100**

De 15 preguntas, 11 fueron correctas y 4 parcialmente correctas, sin respuestas incorrectas. Dominas bien la idea central (sin trazabilidad el reporte no vale), los 5 campos del claim-source mapping, y las cinco trampas de examen: no promediar ni elegir la fuente más reciente, no dejar que el analista resuelva el conflicto, y el formato según el contenido. Los huecos son de matiz, no de concepto: los verbos y mecanismos exactos de la guía ("mantener y mergear", inline citations o sección de referencias, contexto metodológico) y, sobre todo, reconocer que números distintos con fechas distintas suelen ser una **tendencia** y no una contradicción.

## Conceptos con buen dominio

- **Argumento central / information provenance**: identificaste de inmediato que sin trazabilidad el reporte es "ficción plausible".
- **Los 5 campos del structured claim-source mapping**: los listaste completos (claim, source URL, document name, relevant excerpt, publication date).
- **Producir el formato desde el inicio**: entendiste que si los subagentes devuelven prosa, la atribución ya se perdió antes de la síntesis.
- **Synthesis como punto de falla más común**: lo identificaste y explicaste por qué (comprime y parafrasea).
- **Conflict handling (Trampa 1)**: rechazaste promediar y elegir la más reciente, y explicaste bien por qué destruyen información.
- **El analista no resuelve el conflicto (Trampa 5)** y **las 3 opciones del coordinator**: presentar ambos, investigar más o escalar a un humano.
- **Responsabilidad compartida de las fechas**: subagentes las incluyen, synthesis las preserva, output las presenta.
- **Well-established vs. contested findings**: secciones explícitas según el respaldo de las fuentes.
- **Content-appropriate rendering (Trampa 4)**: tablas para financial data, prosa para news y listas para technical findings.

## Áreas que necesitan revisión

### Temporal awareness: el "conflicto" que en realidad es una tendencia

- **Qué pasó:** Ante 8% vs. 12% sin fechas, propusiste correctamente exigir fechas, pero describiste la situación como "una discrepancia que no sabemos tratar". No identificaste que lo más probable es que **no exista contradicción**, sino una tendencia, ni que el error del synthesis agent fue suprimir hallazgos consistentes.
- **Contenido del resumen:**

| Sin fechas | Con fechas |
|---|---|
| Fuente A: 8% · Fuente B: 12% → "se contradicen" | Fuente A (2023): 8% · Fuente B (2024): 12% → **el crecimiento se aceleró** |

> - El "conflicto" en realidad es una **tendencia**.
> - Por eso se **exigen fechas de publicación / recolección de datos en todos los structured outputs**. No es "housekeeping": es lo que hace posible la interpretación correcta.
> - Sin contexto temporal, tendencias válidas se leen como problemas de calidad de datos, y el synthesis agent puede **marcar o suprimir incorrectamente** hallazgos que en realidad son consistentes.

- **Sección:** [[1 resumen]] → "5. Temporal awareness: fechas distintas explican números distintos"
- **Analogía para reforzarlo:** Si una foto dice que tu sobrino mide 1.10 m y otra dice 1.30 m, no son fotos "contradictorias": son de años distintos y muestran que creció. Sin la fecha en la foto, parecería que una de las dos miente.

### Synthesis agent: "mantener *y mergear*", y cómo se cita en el output final

- **Qué pasó:** Dijiste que el synthesis agent debe "conservar" los mappings, pero no mencionaste que debe **mergearlos** al combinar hallazgos de varios subagentes. Al preguntar por el output final respondiste con el *contenido* de la cita (fecha, documento, fuente), no con las dos *formas* que da la guía: inline citations o sección de referencias estructurada.
- **Contenido del resumen:**

> - El prompt del synthesis agent debe **exigir explícitamente** que cada claim de su output sea rastreable a una fuente específica.
> - Tres requisitos concretos:
>   1. Los subagentes devuelven hallazgos en el formato claim-source estructurado.
>   2. El synthesis agent tiene instrucción de **mantener y mergear** esos mappings al combinar.
>   3. El output final incluye **inline citations** o una **sección de referencias estructurada** que liga cada claim a su fuente.

- **Sección:** [[1 resumen]] → "2. La atribución muere en la summarisation (y hay que cargarla por todo el pipeline)"
- **Analogía para reforzarlo:** Es como juntar las listas de compras de tres compañeros de piso en una sola. No basta con no tirarlas: al fusionarlas tienes que anotar junto a cada artículo quién lo pidió, ya sea al lado ("leche — Ana") o en una lista aparte al final. Si no, nadie sabe a quién cobrarle.

### Contexto metodológico en la anotación de conflictos

- **Qué pasó:** Mencionaste el periodo de reporte, pero no el **método** de cada fuente como parte del contexto que explica la diferencia entre dos valores.
- **Contenido del resumen:**

> - Anotar ambos preserva el panorama completo: el consumidor ve los dos valores, entiende las fuentes y juzga cuál es más relevante para su necesidad.
> - La anotación incluye el **contexto metodológico** (periodo de reporte, método) que puede explicar la diferencia.

- **Sección:** [[1 resumen]] → "3. Conflict handling: anotar, nunca elegir arbitrariamente"
- **Analogía para reforzarlo:** Dos básculas dan pesos distintos para la misma persona: una la pesó con ropa y en la tarde, otra sin ropa y en la mañana. Ninguna está rota. Para entender la diferencia necesitas saber *cuándo* (el periodo) y *cómo* (el método) pesó cada una.

### Quién decide el conflicto: el coordinator, antes de la síntesis

- **Qué pasó:** En el caso del document analysis agent dijiste que hay que "dejar que el lector decida". En ese punto del pipeline quien decide es el **coordinator**, y lo hace antes de pasar a síntesis. (Después sí lo nombraste bien al listar sus 3 opciones.)
- **Contenido del resumen:**

> Cuando el agente de document analysis encuentra valores en conflicto, debe **terminar su trabajo con los conflictos incluidos y anotados explícitamente**. **No debe resolver el conflicto** — esa decisión le pertenece al **coordinator** (o al consumidor).

> El coordinator decide qué hacer con el conflicto **antes de pasar a síntesis**:
>
> - presentar ambos valores,
> - investigar más, o
> - escalar a un analista humano.

- **Sección:** [[1 resumen]] → "4. Completar el análisis con los conflictos intactos"
- **Analogía para reforzarlo:** El mesero que nota que la cocina tiene dos recetas distintas para el mismo platillo no elige una por su cuenta ni le pregunta al comensal: le avisa al jefe de cocina, que decide si sirve ambas, averigua más o le pregunta al dueño.

## Siguiente paso recomendado

Repasa en [[1 resumen]] las secciones 5 (temporal awareness) y 2 (los tres requisitos concretos del pipeline). Ya cubriste todo el checklist del tema, así que estás listo para [[4 test]]. Pon atención especial a preguntas donde dos números "contradictorios" tengan fechas distintas.
