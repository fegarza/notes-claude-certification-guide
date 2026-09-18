## Cuándo usar few-shot

> [!question]- ¿Cuál es el punto de partida correcto para decidir usar few-shot: probar directamente con ejemplos, o agotar primero las instrucciones detalladas?
> Agotar primero las instrucciones detalladas. Few-shot entra en juego específicamente cuando instrucciones detalladas y exhaustivas ya se probaron y el resultado sigue siendo inconsistente — no es el primer recurso por defecto.

> [!question]- ¿Cuáles son los tres escenarios (disparadores) donde few-shot es la técnica correcta?
> Inconsistencia de formato a pesar de instrucciones detalladas, juicios ambiguos que producen clasificaciones distintas para entradas similares, y campos de extracción vacíos cuando la información existe pero en un formato inesperado.

> [!question]- Si un modelo deja campos de extracción vacíos aunque la información sí aparece en el texto fuente, ¿qué indica esto sobre la causa probable?
> Que la información viene en un formato distinto al que el modelo espera (ej. mediciones informales, estructura de documento poco convencional) — no que falte la información. Es uno de los tres disparadores clásicos para agregar ejemplos few-shot.

> [!question]- ¿Cómo se relaciona la elección de herramienta ante una solicitud ambigua con el tema de few-shot?
> Es uno de los dominios típicos de "juicio ambiguo" que few-shot ayuda a resolver: mostrar ejemplos con razonamiento de por qué se elige una herramienta sobre otra en casos ambiguos enseña el criterio de decisión, en vez de dejar la elección a la interpretación libre del modelo.

## Construcción de ejemplos few-shot

> [!question]- ¿Por qué el rango recomendado es 2-4 ejemplos, y no "cuantos más, mejor"?
> Menos de 2 ejemplos no alcanza a establecer un patrón reconocible; más de 4 gasta tokens sin aportar señal adicional relevante. El rango 2-4 es el punto donde el patrón queda claro sin desperdicio.

> [!question]- ¿Qué diferencia hay entre un ejemplo few-shot que solo muestra "input → output" y uno que incluye razonamiento?
> El primero muestra el resultado sin explicar el porqué, por lo que el modelo puede memorizar el caso literal. El segundo explica por qué se eligió esa acción o clasificación sobre otras alternativas plausibles, lo que enseña el principio de decisión detrás del ejemplo.

> [!question]- Al construir ejemplos few-shot para un problema de inconsistencia, ¿sobre qué casos deberían enfocarse: los que ya funcionan bien o los que están fallando?
> Sobre los casos que están fallando. Los ejemplos deben construirse específicamente sobre los escenarios problemáticos, no sobre casos genéricos donde el modelo ya se comporta correctamente.

> [!question]- Para una tarea de revisión o clasificación, ¿qué debería demostrar un ejemplo few-shot además de la decisión correcta?
> El formato de salida deseado completo (por ejemplo: ubicación, problema, severidad, corrección sugerida), no solo la etiqueta o decisión final — esto es necesario para lograr consistencia real en el formato de la respuesta.

## Generalización vs. memorización

> [!question]- ¿Por qué el componente de razonamiento en un ejemplo few-shot es lo que permite generalizar a casos nuevos?
> Porque enseña el principio de decisión subyacente (el "por qué"), no solo el resultado del caso mostrado. Un modelo que aprende el principio puede aplicarlo a patrones que nunca vio; uno que solo memoriza pares input-output solo repite lo que ya vio.

> [!question]- ¿Qué pasaría si se usaran ejemplos few-shot sin razonamiento para enseñar a distinguir patrones de código aceptables de problemas genuinos?
> El modelo tendría dificultad para generalizar ese criterio a patrones de código que no se parezcan exactamente a los ejemplos mostrados, arriesgando tanto falsos positivos como falsos negativos en código nuevo no cubierto por los ejemplos literales.

> [!question]- ¿Cómo ayuda mostrar ejemplos con estructuras de documento variadas (citas en línea vs. bibliografías) a una tarea de extracción?
> Enseña al modelo a reconocer la misma información aunque aparezca en un formato de documento distinto al "canónico", reduciendo tanto la alucinación de valores como los campos vacíos causados por variedad estructural no cubierta.

## Distinción clave: few-shot vs. otras técnicas

> [!question]- Si un modelo fabrica/alucina valores en un campo de un schema de salida, ¿es few-shot la técnica correcta para arreglarlo?
> No. La fabricación de valores en un schema se arregla con campos opcionales/nullable en el schema, no con más ejemplos few-shot — son causas raíz distintas.

> [!question]- Si el modelo comete errores de cálculo o discrepancias numéricas, ¿qué técnica ataca esa causa raíz en vez de few-shot?
> Bucles de validación y reintento (validation and retry loops), no ejemplos few-shot. Few-shot enseña patrones de decisión, no corrige aritmética.

> [!question]- El modelo elige la herramienta equivocada porque dos herramientas tienen descripciones mínimas y casi idénticas. ¿Cuál debería ser el primer paso: enriquecer las descripciones de las herramientas o agregar ejemplos few-shot de enrutamiento?
> Enriquecer primero las descripciones de las herramientas (formatos de entrada, casos límite, cuándo usar una vs. otra). Es el arreglo de mayor apalancamiento y menor costo; few-shot puede ayudar pero no ataca la causa raíz cuando el problema es descripción insuficiente.

> [!question]- ¿Cuál es la regla general para decidir si few-shot es la técnica correcta ante un síntoma de inconsistencia?
> Preguntar primero cuál es la causa raíz del síntoma: si es falta de un patrón demostrado para casos ambiguos o formatos inconsistentes, few-shot es correcto; si es fabricación de valores, error de cálculo, o descripción de herramienta pobre, la técnica correcta es otra (schema, validación, o mejorar la descripción), no few-shot.

## Trampas comunes

> [!question]- ¿Por qué "seguir agregando instrucciones más detalladas" es una trampa cuando el problema ya persiste tras instrucciones exhaustivas?
> Porque ese es exactamente el punto donde las instrucciones dejaron de ser suficientes — seguir por ese camino no resuelve la inconsistencia. La señal de que hay que pasar a few-shot es justamente que instrucciones detalladas ya se probaron y fallaron.

> [!question]- ¿Por qué usar un umbral de confianza no es la forma correcta de arreglar juicios ambiguos inconsistentes?
> Porque la confianza auto-reportada por el modelo está mal calibrada — puede estar muy seguro de una decisión incorrecta. Ante juicios ambiguos, mostrar ejemplos few-shot con razonamiento es la técnica correcta, no filtrar por confianza.

> [!question]- ¿Qué pasaría si se usan ejemplos few-shot como primer paso para arreglar un problema de enrutamiento causado por descripciones de herramienta pobres?
> Se gastarían tokens adicionales sin atacar la causa raíz; las descripciones seguirían siendo insuficientes para casos no cubiertos por los ejemplos. El primer paso de mayor apalancamiento es mejorar las descripciones de las herramientas.

---

> [!tip] Repasa la teoría en [[1 resumen]], aplícala en [[2 example]] y evalúate en [[4 test]]
