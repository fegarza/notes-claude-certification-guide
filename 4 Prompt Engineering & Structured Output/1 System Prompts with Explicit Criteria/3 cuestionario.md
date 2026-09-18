## Vago vs. explícito

> [!question]- ¿Por qué instrucciones como "sé conservador" o "usa tu buen juicio" no mejoran la precisión de un system prompt?
> Porque no establecen un límite de decisión accionable: términos como "conservador" no tienen un significado consistente entre contextos, así que el modelo termina interpretándolos de forma distinta en cada corrida, produciendo clasificaciones inconsistentes.

> [!question]- ¿Cuál es la diferencia entre un criterio vago y un criterio categórico explícito?
> El criterio vago describe un estado deseado con adjetivos subjetivos ("conservador", "alta confianza") que requieren interpretación. El criterio explícito define categorías concretas de qué reportar y qué omitir, con un disparador verificable en el propio contenido (ej. "contradicción entre comportamiento declarado y comportamiento real").

> [!question]- Si tuvieras que reescribir "revisa el código con cuidado" como criterio explícito, ¿qué le tendrías que agregar?
> Categorías concretas de qué marcar (ej. bugs, vulnerabilidades de seguridad) y qué omitir (ej. estilo, patrones locales), más un disparador objetivo y verificable en el código, no una palabra que dependa de interpretación subjetiva.

## El problema de la confianza y los falsos positivos

> [!question]- ¿Por qué un alto índice de falsos positivos en una sola categoría de revisión afecta a las demás categorías, aunque estas tengan alta precisión?
> Porque la confianza del usuario no se compartimenta por categoría: al ver falsos positivos repetidos en una categoría, el desarrollador deja de confiar en el reporte completo y empieza a ignorar también los hallazgos precisos de otras categorías.

> [!question]- ¿Qué pasaría si, ante una categoría con 40% de falsos positivos, decides dejarla activa mientras la vas puliendo poco a poco?
> La confianza en todo el sistema seguiría erosionándose mientras dure la iteración, porque los desarrolladores siguen viendo (y descartando) los falsos positivos de esa categoría en cada corrida, arrastrando consigo la credibilidad de las categorías que sí funcionan bien.

> [!question]- ¿Cuál es la estrategia correcta para recuperar la confianza cuando una categoría tiene un falso positivo alto, y por qué es "contraintuitiva"?
> Deshabilitarla temporalmente mientras se refina su prompt, en vez de mantenerla activa "por si acaso". Es contraintuitiva porque parece que estás perdiendo cobertura, pero en realidad recuperas de inmediato la confianza en el resto del sistema, y luego reactivas esa categoría solo cuando su precisión mejora.

> [!question]- ¿Cuándo es correcto reactivar una categoría que fue deshabilitada por falsos positivos?
> Solo después de iterar su prompt con ejemplos de código concretos y verificar que su tasa de falsos positivos bajó a un nivel aceptable — no por tiempo transcurrido ni por presión de cobertura.

## Calibración de severidad

> [!question]- ¿Por qué una descripción de severidad en prosa (ej. "Critical: issues that could cause system failures") es insuficiente?
> Porque obliga al modelo a interpretar qué cuenta como "podría causar fallas del sistema", lo cual es subjetivo y varía entre invocaciones — el mismo problema de fondo que las instrucciones vagas tipo "sé conservador".

> [!question]- ¿Cómo se relaciona la calibración de severidad con ejemplos de código con el argumento central del tema?
> Es una aplicación directa del mismo principio: reemplazar descripciones interpretables por criterios concretos y verificables. Un patrón de código real por nivel de severidad elimina la ambigüedad de la misma forma en que las categorías explícitas eliminan la ambigüedad de qué reportar.

> [!question]- ¿Qué efecto tiene usar ejemplos de código concretos por nivel de severidad, en vez de prosa, sobre la consistencia de clasificación entre corridas?
> Produce clasificaciones consistentes: al comparar cada hallazgo contra un patrón de código concreto en vez de interpretar una descripción abstracta, el modelo llega al mismo nivel de severidad de forma repetible.

## Filtrado por confianza vs. criterios explícitos

> [!question]- ¿Por qué "solo reporta hallazgos de alta confianza" es una trampa frecuente en el examen, si suena a buena práctica de ingeniería?
> Porque suena razonable — filtrar por confianza parece prudente — pero en la práctica la confianza auto-reportada de un LLM está mal calibrada: el modelo puede estar muy seguro de algo incorrecto y dudar de algo correcto, así que ese filtro no mejora la precisión real.

> [!question]- ¿Cuál es la diferencia entre usar la confianza como filtro primario y usarla como mecanismo de enrutamiento?
> Como filtro primario, la confianza decide qué hallazgos existen o no en el reporte final — y falla porque está mal calibrada. Como mecanismo de enrutamiento, la confianza decide únicamente a dónde va un hallazgo ya válido (ej. a revisión humana si es baja), después de que los criterios explícitos ya determinaron que es un hallazgo real.

> [!question]- ¿Cuándo usarías confianza-como-enrutamiento en vez de criterios explícitos?
> Nunca en lugar de — solo después. Los criterios explícitos siempre van primero para definir qué es un hallazgo válido; la confianza-como-enrutamiento se usa después, sobre hallazgos ya filtrados por esos criterios, para decidir cuáles requieren revisión humana adicional.

> [!question]- ¿Qué pasaría si un pipeline de revisión aplicara enrutamiento por confianza sin haber definido primero criterios categóricos explícitos?
> El enrutamiento heredaría la mala calibración del modelo: hallazgos incorrectos con confianza alta pasarían el filtro igual, y hallazgos correctos con confianza baja se descartarían o se mandarían a revisión innecesariamente — el problema de fondo (falta de límites de decisión) seguiría sin resolverse.

---

> [!tip] Repasa la teoría en [[1 resumen]], aplícala en [[2 example]] y evalúate en [[4 test]]
