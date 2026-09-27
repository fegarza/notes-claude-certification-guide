## La limitación del self-review en la misma sesión

> [!question]- ¿Por qué Claude tiende a confirmar en vez de cuestionar su propio trabajo cuando lo revisa en la misma sesión donde lo generó?
> Porque retiene el razonamiento que lo llevó a cada decisión — ya "sabe" por qué eligió cada clasificación, valor o línea de código, así que al revisar tiende a validar esas elecciones en vez de examinarlas críticamente.

> [!question]- ¿Qué hace diferente a una instancia independiente al revisar una salida, comparado con self-review en la misma sesión?
> No tiene acceso al razonamiento previo, así que evalúa la salida solo por lo que observa directamente — sin el sesgo de anclaje de "elegí este enfoque porque...". Esto la hace más efectiva para detectar defectos sutiles.

> [!question]- Si el examen presenta "mejorar las instrucciones de revisión dentro de la misma sesión" y "usar una instancia separada" como opciones, ¿por qué la segunda es casi siempre la respuesta correcta?
> Porque el problema es estructural, no de instrucción: ninguna instrucción elimina el hecho de que el modelo retiene el razonamiento original dentro de la misma sesión. Solo una instancia sin ese contexto previo rompe el sesgo de confirmación.

> [!question]- ¿Por qué usar extended thinking durante la generación tampoco resuelve la limitación del self-review?
> Porque extended thinking ocurre dentro de la misma invocación que genera la salida — no introduce una perspectiva independiente ni elimina el razonamiento retenido; el problema es la ausencia de una instancia separada, no la falta de más "pensamiento" en la misma sesión.

## Arquitectura de multi-pass review

> [!question]- ¿Qué es attention dilution y en qué contexto aparece?
> Es la pérdida de calidad y consistencia de atención que ocurre cuando se procesa una revisión grande (muchos archivos) en un solo pase — el modelo reparte su atención de forma desigual sobre todo el contenido en vez de examinarlo con profundidad uniforme.

> [!question]- Menciona los tres síntomas observables de attention dilution en una revisión single-pass de múltiples archivos.
> Profundidad desigual entre archivos (algunos detallados, otros superficiales), bugs perdidos por fatiga de atención (sobre todo a la mitad de la revisión), y contradicciones internas (el mismo patrón marcado como crítico en un archivo y aprobado en otro).

> [!question]- ¿Qué revisa específicamente el Pass 1 (análisis local por archivo) y qué NO puede detectar por diseño?
> Revisa cada archivo de forma independiente y enfocada, buscando bugs, problemas de seguridad y errores de lógica dentro de ese archivo. Por diseño, no puede detectar problemas que solo se ven al comparar dos o más archivos entre sí (flujo de datos entre módulos, contratos de API rotos).

> [!question]- ¿Qué revisa específicamente el Pass 2 (integración cross-file) que el Pass 1 no puede cubrir?
> Datos que fluyen entre módulos en formatos incompatibles, patrones que se contradicen entre archivos, violaciones de contrato de API en los límites entre servicios, e inconsistencias entre los propios hallazgos reportados por cada pasada del Pass 1.

> [!question]- ¿Por qué las invocaciones del Pass 1 se pueden paralelizar pero la del Pass 2 no puede empezar hasta que terminen todas las del Pass 1?
> Porque cada invocación del Pass 1 revisa un archivo de forma completamente independiente de las demás, sin depender de su resultado. El Pass 2 necesita los hallazgos de *todos* los archivos como insumo para poder comparar entre ellos, así que estructuralmente depende de que el Pass 1 haya terminado.

## Por qué una ventana de contexto más grande no resuelve esto

> [!question]- ¿Por qué "usar un modelo con ventana de contexto más grande" es un distractor plausible pero incorrecto para el problema de attention dilution?
> Porque suena lógico (más capacidad debería manejar más archivos), pero el problema no es cuánto texto cabe en el contexto — es cómo se distribuye la atención sobre ese texto. Un modelo con más contexto sigue pudiendo repartir su atención de forma desigual entre archivos.

> [!question]- ¿Cuál es la diferencia entre "capacidad de contexto" y "calidad de atención", y por qué esa distinción es clave para el examen?
> Capacidad de contexto es cuánto texto el modelo puede contener a la vez; calidad de atención es qué tan uniforme y profundamente lo procesa. Aumentar la primera no mejora la segunda — solo la separación estructural en pasadas enfocadas garantiza profundidad consistente. El examen prueba específicamente si se confunden ambos conceptos.

## Ruteo por confianza calibrada

> [!question]- ¿Qué es la confianza "cruda" reportada por el modelo, y por qué no es segura para decisiones automatizadas?
> Es el puntaje de certeza que el propio modelo reporta junto a un hallazgo, basado solo en su propia sensación — no está validado contra precisión real, así que usarlo directamente para automatizar decisiones (como enviar algo sin revisión humana) es poco confiable.

> [!question]- ¿Cómo se calibra un umbral de confianza antes de usarlo para ruteo automático?
> Corriendo un dataset de validación etiquetado (donde ya se conoce la respuesta correcta) a través del sistema, y midiendo qué tan bien la confianza reportada por el modelo se correlaciona con la precisión real verificada en ese dataset.

> [!question]- En un sistema de ruteo por confianza, ¿qué pasa con los hallazgos de alta confianza calibrada versus los de baja confianza?
> Los de alta confianza (por encima del umbral calibrado) se envían directo a los desarrolladores sin pasar por revisión humana. Los de baja confianza se envían a una cola de revisión humana para validación.

> [!question]- ¿Por qué "confianza calibrada" y "confianza cruda" no son lo mismo aunque ambas sean un número entre 0 y 1?
> Porque la cruda es solo la certeza que el modelo dice tener, sin verificación externa; la calibrada es esa misma señal ya contrastada contra un dataset etiquetado, de modo que se sabe con qué tanta fiabilidad ese número predice la precisión real — solo la segunda es segura para automatizar decisiones.

## Arquitectura de producción completa

> [!question]- ¿Cuáles son las cinco fases de la arquitectura de producción completa que combina todos los conceptos de este tema, en orden?
> 1) Generación (primera instancia crea el output). 2) Revisión por archivo (instancias independientes, Pass 1). 3) Revisión de integración (instancia separada, Pass 2). 4) Ruteo por confianza (hallazgos bajo el umbral calibrado van a cola humana, el resto a desarrollo). 5) Loop de calibración continua con datasets etiquetados.

> [!question]- ¿Por qué se acepta el costo adicional de esta arquitectura de múltiples instancias y múltiples pasadas en sistemas de producción?
> Porque en sistemas donde la calidad de la revisión impacta directamente la confiabilidad (CI/CD, extracción de datos financieros, compliance), un problema no detectado se propaga a procesos posteriores — el costo de más invocaciones es menor que el costo de dejar pasar un defecto real.

---

> [!tip] Repasa la teoría en [[1 resumen]], aplícala en [[2 example]] y evalúate en [[4 test]]
