Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Argumento central

> [!question]- ¿Qué determina el `error propagation` en un sistema multi-agente?
> Si el sistema se recupera con elegancia o falla en silencio.

> [!question]- ¿Quién toma la decisión de recuperación: el subagente o el coordinador?
> El coordinador decide la recuperación; el subagente solo resuelve localmente lo transitorio y le entrega la información para decidir.

## `Structured error context`

> [!question]- ¿Cuáles son los 4 elementos del `structured error context`?
> `failureType`, `attemptedAction` (lo que se intentó), `partialResults` y `alternativeApproaches`.

> [!question]- ¿Cuáles son las 4 categorías de `failureType`?
> `transient` (timeout, rate limit), `validation` (input inválido), `business` (violación de regla) y `permission` (acceso denegado).

> [!question]- ¿Por qué importa el `failureType` para el coordinador?
> Porque le dice si reintentar tiene sentido: un transitorio puede reintentarse; un permiso o validación no se arregla repitiendo lo mismo.

> [!question]- ¿Para qué sirve incluir el `attemptedAction`?
> Para que el coordinador pueda reintentar con una query o parámetros modificados en vez de a ciegas.

> [!question]- ¿Por qué incluir `partialResults` en vez de descartarlos?
> Porque son trabajo útil ya hecho; el coordinador puede seguir con ellos aunque la operación no haya terminado.

> [!question]- ¿Por qué las `alternativeApproaches` las sugiere el subagente?
> Porque el subagente conoce su dominio (otras bases, rangos, cachés) mejor que el coordinador.

> [!question]- ¿Qué cuatro decisiones puede tomar el coordinador con un error estructurado?
> Reintentar con query modificada, probar una alternativa, seguir con resultados parciales, o escalar.

> [!question]- ¿Por qué un error genérico como "search unavailable" es un problema?
> Porque oculta qué se intentó, qué se obtuvo y qué alternativas hay; el coordinador no puede tomar una decisión informada.

## Los dos antipatrones

> [!question]- ¿Qué es el `silent suppression`?
> Atrapar una falla y devolver resultados vacíos marcados como éxito.

> [!question]- ¿Qué concluye el coordinador ante un `silent suppression`?
> Que la búsqueda corrió y no encontró nada, así que no reintenta ni prueba alternativas.

> [!question]- ¿Por qué el `silent suppression` es el antipatrón más peligroso?
> Porque es invisible: el output final parece completo aunque le falta contenido.

> [!question]- ¿Qué es el `workflow termination`?
> Matar todo el pipeline por la falla de un solo subagente.

> [!question]- ¿Por qué el `workflow termination` es un antipatrón?
> Porque es desproporcionado: tira a la basura el trabajo de los subagentes que sí terminaron bien.

> [!question]- ¿Cuál es el punto medio correcto entre ambos antipatrones?
> `Structured error propagation`: la falla se hace visible pero acotada, permitiendo recuperación dirigida y preservando resultados parciales.

> [!question]- ¿En qué se parecen `silent suppression` y error genérico?
> Ambos le quitan información al coordinador: uno oculta que hubo falla, el otro oculta los detalles de la falla.

## `Access failure` vs. `valid empty result`

> [!question]- ¿Qué es un `access failure`?
> La herramienta no pudo llegar a la fuente (timeout, conexión, permiso): la query no se ejecutó.

> [!question]- ¿Qué es un `valid empty result`?
> La query se ejecutó correctamente y no hubo matches.

> [!question]- ¿Cuál es la diferencia clave entre ambos?
> Si la query se ejecutó o no.

> [!question]- ¿Qué acción corresponde a un `access failure`?
> Considerar retry, con los mismos o con parámetros modificados.

> [!question]- ¿Por qué no se reintenta un `valid empty result`?
> Porque ese vacío ES la respuesta; reintentar desperdicia recursos en una query que siempre devolverá nada.

> [!question]- ¿Qué pasa si tratas un `access failure` como vacío válido?
> Nunca reintentas cuando debías — es justamente el `silent suppression`.

> [!question]- ¿Qué pasa si tratas un vacío válido como `access failure`?
> Pierdes tiempo y recursos reintentando una query que siempre devolverá cero resultados.

> [!question]- ¿Qué campo/señal ayuda a distinguirlos en la respuesta del subagente?
> `status` (`error` vs. `success`) junto con `shouldRetry` (`true` vs. `false`).

## `Local recovery`

> [!question]- ¿Qué fallas debe resolver el subagente localmente?
> Las transitorias, con retry y fuentes de fallback.

> [!question]- ¿Qué errores debe propagar el subagente al coordinador?
> Solo los que no pudo resolver localmente.

> [!question]- ¿Qué debe incluir siempre un error propagado tras el `local recovery`?
> Lo que se intentó y los resultados parciales (es decir, el `structured error context`).

> [!question]- ¿Qué ventaja da el `local recovery` al coordinador?
> Reduce su complejidad: no tiene que gestionar los reintentos de cada falla transitoria de cada subagente.

> [!question]- ¿Hacer retry con exponential backoff es incorrecto?
> No; lo incorrecto es terminar el retry devolviendo un error genérico en vez de contexto estructurado.

## `Coverage annotations`

> [!question]- ¿Qué indican las `coverage annotations` en la síntesis?
> Qué hallazgos están bien respaldados y qué áreas tienen huecos por fuentes no disponibles.

> [!question]- ¿Qué pasa si una síntesis omite un tema sin anotarlo?
> El lector cree que el tema no era relevante, cuando en realidad la fuente no estuvo disponible.

> [!question]- ¿Cómo se relacionan las `coverage annotations` con los `partialResults`?
> Permiten seguir con resultados parciales de forma honesta: se usa lo que hay y se declara la limitación.

> [!question]- ¿Por qué la transparencia es mejor que la omisión silenciosa?
> Porque convierte un hueco invisible en una limitación conocida que el consumidor del reporte puede evaluar.

## Trampas de examen

> [!question]- Un subagente atrapa un timeout y devuelve `results: []` con `status: success`. ¿Qué antipatrón es?
> `Silent suppression`.

> [!question]- Un timeout de un subagente hace que se aborte toda la investigación. ¿Qué antipatrón es?
> `Workflow termination`.

> [!question]- ¿Por qué la opción "retry con backoff + 'search unavailable'" es tan tentadora y aun así incorrecta?
> Porque el retry suena a buena práctica, pero el status genérico final oculta query, parciales y alternativas.

> [!question]- Una query exitosa devuelve 0 resultados y el sistema la reintenta 3 veces. ¿Cuál es el error?
> Tratar un `valid empty result` como `access failure`.

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], aplícalo en [[2 example]] y evalúate con [[4 test]].
