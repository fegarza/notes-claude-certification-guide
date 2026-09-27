Cuestionario de repaso completo de [[1 resumen]]. Respuestas ocultas — intenta responder mentalmente antes de revisar.

## Argumento central

> [!question]- ¿Qué evalúa el examen sobre iterative refinement, más allá de conocer las técnicas?
> Saber **cuál técnica usar primero** en cada situación — cada una resuelve un problema distinto.

> [!question]- ¿Cuáles son las tres técnicas de la jerarquía y para qué problema es cada una?
> Concrete input/output examples → interpretación inconsistente. Test-driven iteration → transformaciones complejas. Interview pattern → dominios desconocidos.

> [!question]- Además de la técnica, ¿qué otra decisión importa al dar feedback?
> Cómo se entrega: en batch (un mensaje) o secuencial (una iteración por problema).

## Concrete input/output examples

> [!question]- ¿Por qué más prosa no arregla una interpretación inconsistente?
> Porque una prosa más precisa sigue dependiendo de interpretación; solo cambia qué ambigüedad queda. Los ejemplos eliminan la interpretación.

> [!question]- ¿Cuántos ejemplos recomienda la guía y por qué no más?
> 2-3. El modelo generaliza el patrón a partir de ellos; no hace falta darle cada caso posible.

> [!question]- ¿Qué deben cubrir esos 2-3 ejemplos?
> El caso estándar y un edge case clave.

> [!question]- ¿Por qué el modelo aprende mejor de ejemplos que de descripciones?
> Porque generaliza a partir de ejemplos concretos de forma más confiable que a partir de cualquier descripción en prosa.

> [!question]- ¿Qué pasaría si respondes a la inconsistencia reescribiendo la prosa "con terminología más técnica"?
> Seguiría habiendo interpretación y probablemente inconsistencia; es la trampa clásica del examen. La respuesta correcta es ejemplos primero.

## El proceso de comunicación basada en ejemplos

> [!question]- ¿Cuáles son los 4 pasos del proceso?
> Observar la inconsistencia → cambiar a 2-3 ejemplos before/after → verificar la generalización con un caso nuevo → agregar ejemplos de edge cases si hace falta.

> [!question]- ¿Por qué verificar con un caso nuevo y no con uno de los ejemplos?
> Porque lo que interesa es si el modelo generalizó el patrón, no si copia los ejemplos que ya vio.

> [!question]- Si el modelo maneja bien el caso estándar pero falla un edge case, ¿qué haces?
> Agregar un ejemplo que muestre específicamente el manejo de ese edge case — no apilar ejemplos genéricos.

## Test-driven iteration

> [!question]- ¿Para qué tipo de problema es test-driven iteration la técnica más efectiva?
> Transformaciones complejas, con muchos edge cases.

> [!question]- ¿Qué tres categorías deben cubrir los tests escritos antes de implementar?
> Happy path, edge cases (nulos, inputs vacíos, fronteras) y performance requirements si aplican.

> [!question]- ¿Por qué un test failure es mejor feedback que una explicación en prosa?
> Porque "Expected X, got Y" es concreto e inequívoco: no deja espacio a interpretación sobre qué hay que arreglar.

> [!question]- ¿En qué orden van los tests y la implementación?
> Tests primero, luego implementación, luego iterar compartiendo los fallos.

> [!question]- ¿Cómo se relaciona test-driven iteration con la técnica de ejemplos?
> Un test case es un ejemplo concreto de input y output esperado, pero ejecutable y repetible — por eso sirve para arreglar edge cases como nulos en una migración.

> [!question]- Un script de migración convierte `null` en `""`. ¿Qué feedback le das a Claude Code?
> Un test case específico con ese input y el output esperado (null preservado), y compartir su fallo — no una descripción vaga como "hazlo más robusto".

## Interview pattern

> [!question]- ¿Qué es el interview pattern?
> Pedirle a Claude que haga preguntas sobre requisitos, edge cases y restricciones antes de implementar.

> [!question]- ¿Cuándo se usa?
> Cuando trabajas en un dominio donde te falta experiencia y podrías pasar por alto consideraciones importantes.

> [!question]- ¿Qué problema tiene prescribir "Build me a caching layer" en un dominio desconocido?
> Que las decisiones críticas quedan resueltas por defecto sin que sepas que existían; no salen a la luz las consideraciones que un experto abordaría.

> [!question]- ¿Qué tipo de consideraciones suele sacar a la luz en el caso de una caché?
> Cache invalidation strategies, TTL policies, consistency requirements y failure modes.

> [!question]- ¿Cuál es la diferencia entre interview pattern y concrete examples?
> Interview: el hueco de conocimiento está en el developer (dominio desconocido). Examples: el developer sabe la transformación exacta, pero el modelo la interpreta de forma inconsistente.

> [!question]- ¿Por qué no sirve dar ejemplos de input/output en un dominio que no dominas?
> Porque no sabes cuál es el output correcto — justo ese es el conocimiento que te falta.

> [!question]- ¿Por qué no tiene sentido usar interview pattern cuando ya sabes exactamente qué transformación quieres?
> Porque no hay consideraciones que descubrir; el problema es la interpretación del modelo, que se resuelve con ejemplos.

## Batch vs sequential feedback

> [!question]- ¿Cuándo das todo el feedback en un solo mensaje?
> Cuando los arreglos interactúan entre sí (arreglar A afecta B).

> [!question]- ¿Por qué hacer batch con problemas que interactúan?
> Porque el modelo necesita ver todas las restricciones a la vez para producir un arreglo coherente.

> [!question]- ¿Qué pasa si arreglas en secuencia problemas que interactúan?
> El modelo arregla uno de una forma que choca con los demás; cada arreglo puede entrar en conflicto con el siguiente.

> [!question]- ¿Cuándo iteras de forma secuencial?
> Cuando los problemas son independientes (ej. naming e indentación).

> [!question]- ¿Qué pasa si juntas en un mensaje problemas independientes?
> Puede confundir al modelo sobre qué feedback aplica a qué parte del código.

> [!question]- ¿Cuál es la pregunta que decide batch vs secuencial?
> ¿Arreglar un problema cambia o restringe cómo se arregla otro?

> [!question]- ¿Por qué error code + logging + tipos del SDK van en batch?
> Porque la forma del campo `error_code` determina cómo se loguea y cómo se tipa en el SDK — son restricciones acopladas.

## Elegir la técnica

> [!question]- Prosa interpretada distinto cada vez → ¿qué técnica?
> Concrete input/output examples.

> [!question]- Transformación compleja con muchos edge cases → ¿qué técnica?
> Test-driven iteration.

> [!question]- Dominio desconocido → ¿qué técnica?
> Interview pattern.

> [!question]- ¿Cuándo usarías test-driven iteration en vez de solo ejemplos?
> Cuando la transformación es compleja, con muchos edge cases y quizás requisitos de performance: los tests cubren todo y dan feedback verificable en cada iteración.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], aplícalo en [[2 example]] y evalúate con [[4 test]].
