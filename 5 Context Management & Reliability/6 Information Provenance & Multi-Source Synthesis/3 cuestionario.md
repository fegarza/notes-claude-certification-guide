---
tags:
  - claude-cert/dominio-5
  - task-statement/5.6
---

# 3 cuestionario — Information Provenance & Multi-Source Synthesis

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## Argumento central

> [!question]- ¿Qué es information provenance?
> Saber de dónde viene cada afirmación y cuánta confianza merece.

> [!question]- ¿Por qué la provenance es "la diferencia" en un sistema de investigación?
> Porque sin ella el output puede sonar plausible pero no se puede verificar: es "plausible-sounding fiction" en vez de un resultado confiable.

> [!question]- ¿Cuáles son los tres ejes que el examen evalúa en este tema?
> Cómo sobrevive (o muere) la atribución a través del pipeline de síntesis, cómo manejar fuentes en conflicto, y cómo el contexto temporal evita falsas contradicciones.

## Structured claim-source mappings

> [!question]- ¿Cuáles son los 5 campos de un claim-source mapping?
> Claim, source URL, document name, relevant excerpt y publication date (fecha de publicación o de recolección de datos).

> [!question]- ¿Por qué el claim-source mapping no es "metadata opcional"?
> Porque es la garantía estructural de que el output final se puede rastrear hasta fuentes específicas; sin él, no hay forma de verificar nada.

> [!question]- ¿Qué aporta el relevant excerpt que no aporta la source URL?
> Señala el pasaje exacto que sostiene el claim, permitiendo verificarlo sin releer todo el documento y detectar si el claim tergiversa la fuente.

> [!question]- ¿Por qué los subagentes deben producir mappings desde el inicio y no "agregarlos después"?
> Porque si devuelven prosa no estructurada, la atribución ya se perdió antes de la síntesis y ningún paso posterior puede reconstruirla.

## La atribución muere en la summarisation

> [!question]- ¿Por qué la atribución se pierde al sintetizar?
> Porque el synthesis agent, de forma natural, comprime y parafrasea; sin instrucciones explícitas, descarta montos, fuentes y fechas.

> [!question]- ¿Qué tipo de frase produce una síntesis que perdió la atribución?
> Algo genérico como "la inversión creció significativamente": sin cifra, sin fuente, sin fecha.

> [!question]- ¿Cuáles son los 4 pasos de un pipeline por los que debe sobrevivir la atribución?
> Research (recolecta con mappings) → analysis (evalúa, preserva mappings) → synthesis (combina y mergea mappings) → report generation (output con inline citations).

> [!question]- ¿Cuál es el punto de falla más común y por qué?
> El paso de synthesis, porque es donde se combinan y parafrasean hallazgos de varios agentes sin llevar los mappings hacia adelante.

> [!question]- ¿Qué debe exigir explícitamente el prompt del synthesis agent?
> Que cada claim de su output sea rastreable a una fuente específica, preservando y mergeando los claim-source mappings.

> [!question]- ¿De qué dos formas puede el output final mostrar la procedencia?
> Con inline citations o con una sección de referencias estructurada que liga cada claim a su fuente.

> [!question]- ¿Cómo se relaciona este tema con la progressive summarisation trap de 5.1?
> Es el mismo fenómeno: comprimir destruye lo preciso. En 5.1 se pierden montos y fechas; aquí se pierde de dónde salió cada dato.

## Conflict handling

> [!question]- Dos fuentes creíbles dan 12% y 8% para la misma medida. ¿Qué hace el synthesis agent?
> Anota ambos valores con atribución completa (fuente, fecha, periodo/método) y deja que el consumidor decida.

> [!question]- ¿Por qué no se debe elegir la fuente más reciente?
> Porque es una selección arbitraria: destruye información y presenta falsa certeza.

> [!question]- ¿Por qué promediar los dos valores también es incorrecto?
> Porque produce una cifra que ninguna fuente reportó y oculta que existía un desacuerdo.

> [!question]- ¿Y elegir la del publisher "más autoritativo"?
> Sigue siendo elegir arbitrariamente un valor entre fuentes creíbles; se pierde la otra perspectiva y el consumidor no puede juzgar.

> [!question]- ¿Qué gana el consumidor cuando se anotan ambos valores?
> El panorama completo: ve los dos valores, entiende las fuentes y decide cuál es más relevante para su necesidad.

> [!question]- ¿Qué contexto conviene agregar a la anotación de un conflicto?
> La posible explicación de la diferencia: periodos de reporte distintos, diferencias metodológicas, cifras auditadas vs preliminares.

## Completar el análisis con los conflictos intactos

> [!question]- ¿Qué debe hacer el analysis agent cuando encuentra valores en conflicto?
> Terminar su trabajo con los valores en conflicto incluidos y anotados explícitamente, sin resolverlos.

> [!question]- ¿Quién decide cómo reconciliar un conflicto y cuándo?
> El coordinator (o el consumidor), antes de pasar a síntesis.

> [!question]- ¿Cuáles son las tres opciones del coordinator ante un conflicto?
> Presentar ambos valores, investigar más, o escalar a un analista humano.

> [!question]- ¿Qué campos tiene el registro de conflicto del ejemplo de la guía?
> `field`, `conflictDetected`, `values` (cada uno con value, source y context) y `possibleExplanation`.

> [!question]- ¿Qué pasaría si el analysis agent resolviera el conflicto por su cuenta?
> El coordinator nunca se enteraría de que existía; perdería la opción de investigar o escalar y el reporte final tendría falsa certeza.

## Temporal awareness

> [!question]- ¿Por qué dos números distintos no son necesariamente una contradicción?
> Porque pueden venir de fechas o periodos distintos: la diferencia puede ser una tendencia, no un conflicto.

> [!question]- Fuente de 2023 dice 8% y fuente de 2024 dice 12%. ¿Cómo se lee con fechas?
> Como una aceleración del crecimiento de 8% a 12% en el periodo medido.

> [!question]- ¿Qué pasa si los structured outputs no incluyen fechas?
> Tendencias válidas se leen como problemas de calidad de datos, y el synthesis agent puede marcar o suprimir incorrectamente hallazgos consistentes.

> [!question]- ¿Por qué exigir fechas no es simple "housekeeping"?
> Porque es lo que hace posible interpretar correctamente los datos; sin ellas la interpretación temporal es imposible.

> [!question]- ¿Quién es responsable de las fechas en cada etapa del pipeline?
> Los subagentes las incluyen en su output, el synthesis agent las preserva al mergear, y el output final las presenta junto a los datos que describen.

> [!question]- ¿Cuál es la diferencia entre publication date y data collection date, y por qué importa?
> Una indica cuándo se publicó la fuente, la otra qué periodo cubren los datos; dos fuentes publicadas casi al mismo tiempo pueden medir periodos distintos (año calendario vs año fiscal) y eso explica diferencias.

## Well-established vs contested

> [!question]- ¿Por qué un reporte debe separar findings well-established de contested?
> Para que el lector no trate un dato disputado o de una sola fuente con la misma certeza que un consenso.

> [!question]- ¿Cuál es la diferencia entre un hallazgo con tres fuentes independientes y uno con un solo reporte?
> El primero es mucho más sólido, aunque en el texto ambos puedan presentarse con la misma confianza si no se separan explícitamente.

> [!question]- Además de separar secciones, ¿qué se debe preservar de las fuentes?
> Sus caracterizaciones originales (cómo la fuente describe el hallazgo) y su contexto metodológico.

## Content-appropriate rendering

> [!question]- ¿Qué formato corresponde a financial data y por qué?
> Tablas: números, comparaciones y tendencias se comparan mejor en columnas.

> [!question]- ¿Qué formato corresponde a news y por qué?
> Prosa: el contexto narrativo, la causa-efecto y la cronología fluyen naturalmente en párrafos.

> [!question]- ¿Qué formato corresponde a technical findings y por qué?
> Listas estructuradas: patrones de arquitectura, specs de API y opciones de configuración se entienden mejor con jerarquía.

> [!question]- ¿Qué pasa si la síntesis fuerza todo a un solo formato?
> Se degradan la legibilidad y la comprensión: los números en prosa son difíciles de comparar y la narrativa en bullets pierde su hilo.

> [!question]- ¿Cuándo usarías una tabla en vez de prosa dentro de un mismo reporte?
> Cuando la sección contiene datos financieros o numéricos que el lector necesita comparar entre periodos o fuentes.

## Trampas de examen

> [!question]- ¿Cuál es el error de "usar la fuente más reciente" ante un conflicto y cuál es la forma correcta?
> Es una selección arbitraria que destruye información; lo correcto es anotar ambos valores con atribución y fechas y dejar decidir al consumidor.

> [!question]- ¿Cuál es el error de tratar números distintos como contradicción automática?
> Ignora que fechas distintas suelen explicar la diferencia; lo correcto es exigir fechas en los structured outputs para interpretarlos temporalmente.

> [!question]- ¿Cuál es el error de dejar que el synthesis agent parafrasee libremente?
> La atribución muere y el output es intrazable; lo correcto es exigir que preserve y mergee explícitamente los claim-source mappings.

> [!question]- ¿Cuál es el error de renderizar todo igual?
> Aplana contenidos que se leen mejor en formatos distintos; lo correcto es tablas para finanzas, prosa para noticias y listas para hallazgos técnicos.

> [!question]- ¿Cuál es el error de que el analysis agent "resuelva" un conflicto?
> Oculta información y usurpa una decisión que le toca al coordinator; lo correcto es completar el análisis con los conflictos anotados.

---

> [!tip] Vuelve a [[1 resumen]], aplícalo en [[2 example]] y evalúate en [[4 test]]
