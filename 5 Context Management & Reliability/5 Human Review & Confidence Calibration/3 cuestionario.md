Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Argumento central

> [!question]- ¿Qué papel cumple el `human review` en un sistema de extracción?
> Es la red de seguridad: atrapa los errores que el modelo comete antes de que lleguen a los procesos downstream.

> [!question]- ¿Cuál es el reto de fondo del tema?
> Asignar una capacidad de revisión humana limitada para maximizar la accuracy al menor costo.

> [!question]- ¿Cuáles son los tres conceptos que hay que dominar para lograrlo?
> Que las métricas agregadas engañan, que los `confidence scores` crudos no están calibrados, y que hay que seguir midiendo con `stratified random sampling`.

## `The Aggregate Metrics Trap`

> [!question]- ¿Por qué un 97% de accuracy global puede ser engañoso?
> Porque el tipo de documento dominante en volumen infla el promedio y esconde segmentos con accuracy muy baja.

> [!question]- ¿Por qué los errores de alto impacto suelen quedar escondidos en el agregado?
> Porque suelen estar en tipos minoritarios (manuscritos, internacionales, escaneados) que pesan poco en el promedio.

> [!question]- ¿Por qué ejes hay que desglosar la accuracy?
> Por tipo de documento **y** por campo.

> [!question]- ¿Por qué no basta con desglosar solo por tipo de documento?
> Porque dentro de un mismo tipo, un campo puede ir bien y otro mal (ej. nombres bien, montos mal).

> [!question]- ¿Qué pasaría si automatizas con base en el 97% global?
> Automatizarías también segmentos con accuracy pésima, y sus errores pasarían sin revisión a downstream.

> [!question]- ¿Cuál es la regla de la guía sobre métricas agregadas?
> Nunca tomar decisiones de automatización solo con métricas agregadas; validar por tipo de documento y segmento de campo antes.

## `Stratified Random Sampling`

> [!question]- ¿Qué es un "estrato" en este contexto?
> Un segmento definido por tipo de documento, banda de confianza y/o tipo de campo.

> [!question]- ¿Por qué hay que muestrear las extracciones de alta confianza?
> Porque son las que se automatizan y nadie ve; sin muestreo, sus errores nunca se detectan.

> [!question]- ¿Por qué no sirve muestrear solo las de baja confianza?
> Porque esas ya van a `human review`; el muestreo no agregaría información nueva.

> [!question]- ¿Cuáles son los dos propósitos del `stratified random sampling`?
> Medir continuamente que cada segmento mantiene su accuracy validada y detectar `novel error patterns`.

> [!question]- ¿Qué es un `novel error pattern`?
> Un modo de falla nuevo que no aparecía en el validation set original (ej. un formato de documento nuevo).

> [!question]- ¿Qué pasa si no haces sampling continuo?
> El sistema puede acumular errores sistemáticos en formatos nuevos sin detectarlos hasta que fallan los procesos de negocio downstream.

> [!question]- ¿Por qué estratificado y no simplemente aleatorio sobre todo?
> Porque un muestreo aleatorio simple casi solo tomaría del tipo dominante y dejaría sin muestra a los segmentos pequeños.

## `Field-Level Confidence Calibration`

> [!question]- ¿Por qué pedir confianza por campo y no por documento?
> Porque la accuracy varía por campo; necesitas enrutar y calibrar cada campo por separado.

> [!question]- ¿Qué significa que un `confidence score` crudo "no está calibrado"?
> Que da información relativa, no absoluta: un 0.95 no equivale a 95% de probabilidad de acierto.

> [!question]- ¿Puede el mismo score significar cosas distintas?
> Sí: 0.90 puede ser 94% de accuracy real en fechas y 82% en montos.

> [!question]- ¿Qué se necesita para calibrar?
> Un `labeled validation set`: documentos con la extracción correcta conocida (ground truth).

> [!question]- ¿Qué es una calibration curve?
> La relación entre la confianza que reporta el modelo y la accuracy real observada en el validation set.

> [!question]- ¿Para qué sirve la calibración en la práctica?
> Para fijar thresholds por campo (y tipo de documento) que de verdad correspondan a la accuracy deseada.

> [!question]- ¿Por qué subir el threshold de 0.95 a 0.99 no resuelve el problema?
> Porque sigue usando scores crudos sin calibrar y un solo threshold para segmentos que se comportan distinto.

> [!question]- ¿Cuáles son las tres zonas de routing con thresholds calibrados?
> Arriba del threshold → automatizado (con sampling); abajo → `human review`; zona ambigua → `human review` priorizado.

> [!question]- ¿Lo automatizado deja de verificarse?
> No: lo que pasa el threshold se automatiza **con** `stratified sampling` continuo.

> [!question]- ¿Cómo se relaciona esto con la self-reported confidence de la Sample Question 3 (5.2)?
> Es el mismo principio: la confianza autodeclarada del LLM está mal calibrada y no sirve como criterio sin validarla.

## `Reviewer Capacity Prioritisation`

> [!question]- ¿Cuál es el principio de priorización de revisores?
> Mandar primero los ítems de mayor incertidumbre, no repartir la capacidad parejo.

> [!question]- ¿Qué cuatro tipos de ítems se priorizan?
> Campos con baja confianza, documentos fuente ambiguos o contradictorios, tipos con mala accuracy histórica y campos con varias interpretaciones posibles.

> [!question]- ¿Por qué repartir la revisión de forma pareja es un anti-pattern?
> Porque gasta tiempo en ítems que el modelo ya hace bien y deja sin capacidad a los inciertos.

> [!question]- ¿Cuál es la diferencia entre priorización estática y dinámica?
> La dinámica reordena la cola por incertidumbre conforme llegan documentos; la estática fija un orden de antemano.

> [!question]- ¿Qué ítem recibe un revisor al terminar uno?
> El ítem restante más incierto, no el siguiente en orden cronológico.

> [!question]- ¿Por qué una fuente contradictoria merece revisión aunque la confianza sea alta?
> Porque la contradicción en el documento es incertidumbre real que requiere juicio humano, independientemente del score.

## `Validation Before Automation`

> [!question]- ¿Cuáles son los 5 pasos de la secuencia en orden?
> 1) Medir accuracy por tipo y campo; 2) calibrar con `labeled validation sets`; 3) fijar thresholds calibrados; 4) implementar `stratified random sampling`; 5) solo entonces reducir `human review` en segmentos consistentemente precisos.

> [!question]- ¿Cuál es la trampa principal respecto a esta secuencia?
> Saltar directo al paso 5 basándose en métricas agregadas.

> [!question]- ¿Por qué el sampling va antes de reducir el `human review`?
> Porque al reducir revisión necesitas ya tener el mecanismo que detecte si lo automatizado empieza a fallar.

> [!question]- ¿Se reduce el `human review` en todos los segmentos a la vez?
> No: solo en los segmentos que demostraron ser consistentemente precisos.

## Trampas de examen

> [!question]- Si un equipo propone automatizar todo lo de confianza > 95% porque el sistema tiene 97% global, ¿cuál es el riesgo crítico?
> Que el agregado esconda mal desempeño en ciertos tipos/campos y que los scores no estén calibrados contra `labeled validation sets`.

> [!question]- ¿Qué está mal en muestrear solo extracciones de baja confianza?
> Que deja sin verificar justo lo automatizado, donde los `novel error patterns` pasan inadvertidos.

> [!question]- ¿Qué está mal en usar un threshold único de 0.90 para todos los campos?
> Que 0.90 significa accuracies reales distintas por campo; hay que calibrar thresholds por campo.

> [!question]- ¿Qué está mal en revisar "el 10% de todo" en orden de llegada?
> Que reparte la capacidad parejo y desperdicia revisores en ítems fáciles en vez de los más inciertos.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], aplícalo en código en [[2 example]] y evalúate con [[4 test]].
