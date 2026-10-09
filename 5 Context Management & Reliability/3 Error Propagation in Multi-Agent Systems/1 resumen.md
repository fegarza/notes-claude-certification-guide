> [!note] En una frase
> Cuando un subagente falla, debe devolverle al coordinador un **error estructurado** (qué tipo de falla, qué intentó, qué alcanzó a obtener y qué alternativas hay) — nunca ocultar la falla como si fuera "éxito sin resultados", ni tumbar todo el pipeline por una sola falla.

## Explícamelo como si tuviera 5 años

Imagina que un jefe de cocina (el **coordinador**) manda a cinco ayudantes (los **subagentes**) al mercado, cada uno por un ingrediente distinto.

Uno de ellos vuelve con las manos vacías. Hay tres formas de reportarlo:

- **Mal (mentir):** "Todo bien, jefe, no había nada." — pero en realidad el puesto estaba cerrado. El jefe cree que ese ingrediente *no existe* y cocina sin él, sin saber que faltó algo. Nadie se entera.
- **Mal (pánico):** el ayudante grita "¡fracasamos!" y el jefe tira a la basura lo que trajeron los otros cuatro, que sí cumplieron.
- **Bien (reporte útil):** "El puesto de tomates estaba cerrado (tipo de problema). Fui al puesto de la calle 5 (lo que intenté). Alcancé a comprar 2 de los 6 que pediste (resultados parciales). Hay otro mercado a 10 minutos, o podemos usar tomate en lata (alternativas)."

Con el tercer reporte, el jefe puede **decidir con inteligencia**: mandar a alguien al otro mercado, usar la lata, o cocinar con 2 tomates y avisar en el menú que el plato lleva menos salsa.

Y ojo con una diferencia: "el puesto estaba cerrado" (no pude buscar) no es lo mismo que "el puesto estaba abierto y no venden tomates" (busqué y la respuesta es *no hay*). Lo primero vale la pena reintentarlo; lo segundo **ya es la respuesta**.

## Argumento central

> El `error propagation` determina si un sistema multi-agente **se recupera con elegancia o falla en silencio**. El subagente debe resolver localmente lo que pueda (fallas transitorias) y propagar al coordinador solo lo que no pudo resolver, en forma de **`structured error context`** — `failure type`, `attempted action`, `partial results` y `alternative approaches` — para que el coordinador tome decisiones de recuperación informadas. Los dos extremos (**`silent suppression`** y **`workflow termination`**) son antipatrones, y es crítico distinguir un **`access failure`** de un **`valid empty result`**.

## Ideas clave (Conclusiones)

### 1. `Structured error context` — los 4 elementos obligatorios

Cuando un subagente falla, su respuesta al coordinador debe incluir:

| Elemento | Qué contiene | Para qué le sirve al coordinador |
|---|---|---|
| **`failureType`** | Categoría: `transient` (timeout, rate limit), `validation` (input inválido), `business` (violación de regla), `permission` (acceso denegado) | Saber si tiene sentido reintentar o no |
| **`attemptedAction`** | Query, parámetros y sistema destino específicos | Poder reintentar con una query modificada |
| **`partialResults`** | Datos que sí se obtuvieron antes de la falla | No descartar trabajo útil |
| **`alternativeApproaches`** | Sugerencias de recuperación del propio subagente (que conoce su dominio) | Elegir un plan B concreto |

Con esto el coordinador puede decidir entre: **reintentar con query modificada**, **probar una alternativa**, **seguir con resultados parciales**, o **escalar**.

> [!example] Evidencia — snippet de la guía
> ```json
> {
>   "status": "partial_failure",
>   "failureType": "transient",
>   "attemptedAction": {
>     "tool": "search_academic_db",
>     "query": "renewable energy policy",
>     "dateRange": "2022-2024"
>   },
>   "partialResults": [
>     { "title": "EU Renewable Energy Directive 2023", "source": "EUR-Lex", "retrieved": true }
>   ],
>   "alternativeApproaches": [
>     "Retry with narrower date range (2023-2024)",
>     "Search alternative database: government_publications",
>     "Use cached results from previous research session"
>   ]
> }
> ```

> [!note] Analogía mental
> Un error genérico tipo "search unavailable" es como un parte médico que solo dice "el paciente está mal". El `structured error context` es la historia clínica completa: diagnóstico, qué tratamiento se probó, qué funcionó parcialmente y qué opciones quedan.

### 2. Los dos antipatrones (los extremos opuestos)

**Antipatrón 1 — `Silent suppression`** (suprimir en silencio):
- El subagente atrapa un timeout y devuelve `{ "results": [], "status": "success" }`.
- El coordinador cree que la búsqueda corrió y **no encontró nada** → no reintenta, no prueba alternativas.
- Es el **más peligroso porque es invisible**: el output final *parece* completo aunque le falta contenido.
- Ejemplo de la guía: el sistema de pedidos está caído y el coordinador le dice al cliente "no se encontraron pedidos".

**Antipatrón 2 — `Workflow termination`** (matar el pipeline):
- Un timeout de un subagente tumba todo el pipeline de investigación.
- Los otros cuatro subagentes terminaron bien, pero **su trabajo se tira a la basura**.
- Es una respuesta **desproporcionada** que desperdicia trabajo completado.

> [!note] El punto medio correcto
> `Structured error propagation`: la falla se hace **visible** (a diferencia de suppression) pero **acotada** (a diferencia de termination), permitiendo recuperación dirigida y preservando resultados parciales. El coordinador es quien toma la decisión inteligente de recuperación, en vez de aplicar políticas en bloque.

### 3. `Access failure` vs. `valid empty result`

Distinción **explícitamente evaluada en el examen**:

| | `Access failure` | `Valid empty result` |
|---|---|---|
| ¿Qué pasó? | La herramienta **no pudo llegar** a la fuente de datos | La herramienta llegó, **ejecutó la query** y no hubo matches |
| Ejemplos | timeout, error de conexión, permiso denegado | búsqueda correcta con 0 resultados |
| ¿Se ejecutó la query? | **No** | **Sí** |
| Acción | Considerar retry (mismos o distintos parámetros) | **No reintentar — eso ES la respuesta** |
| Señal típica | `status: "error"`, `shouldRetry: true` | `status: "success"`, `results: []`, `shouldRetry: false` |

> [!example] Evidencia — snippet de la guía
> ```python
> # Access failure — consider retry
> {
>     "status": "error",
>     "failureType": "transient",
>     "message": "Connection timeout after 30s",
>     "shouldRetry": True
> }
>
> # Valid empty result — no retry needed
> {
>     "status": "success",
>     "results": [],
>     "message": "Query executed successfully. No matching records found.",
>     "shouldRetry": False
> }
> ```

**Consecuencias de confundirlos:**
- `Access failure` tratado como vacío válido → **nunca reintentas cuando debías** (es justo el `silent suppression`).
- Vacío válido tratado como `access failure` → **pierdes tiempo reintentando** una query que siempre devolverá nada.

### 4. `Local recovery` para fallas transitorias

- Cada subagente implementa su propia **lógica de retry** y **fuentes de fallback** antes de escalar.
- Solo se **propagan los errores que el subagente no pudo resolver** localmente.
- Al propagar, **siempre** se incluye qué se intentó y los resultados parciales.
- Ventaja: **reduce la complejidad del coordinador** — no tiene que gestionar reintentos de cada falla transitoria de cada subagente.

> [!warning] Retry local ≠ error genérico al final
> Hacer retry con backoff está bien; lo que está mal es que, al agotarse los reintentos, se devuelva un genérico "search unavailable". El retry local debe **terminar** en un `structured error context`, no en un mensaje vacío de contexto.

### 5. `Coverage annotations` en la síntesis

- El output de síntesis debe indicar qué hallazgos están **bien respaldados** y qué áreas temáticas tienen **huecos por fuentes no disponibles**.
- Ejemplo de la guía: *"Section on geothermal energy is limited due to unavailable journal access during research."*
- Beneficio: el hueco aparece como **limitación conocida**, no como si el tema fuera irrelevante. Transparencia > omisión silenciosa.

> [!note] Analogía mental
> Es como un mapa que marca una zona como "sin explorar" en vez de dibujarla como océano vacío. Quien lo lee sabe que ahí *puede* haber algo — no que no hay nada.

## Trampas de examen

> [!warning] Trampa 1 — Atrapar el timeout y devolver resultados vacíos marcados como éxito
> **El error:** `try/except` que, ante un timeout, retorna `{"results": [], "status": "success"}` "para que el workflow siga". **Por qué falla:** el `silent suppression` impide toda recuperación — el coordinador cree que la búsqueda tuvo éxito y no encontró nada, así que nunca intenta alternativas, y el reporte final parece completo sin estarlo. **Lo correcto:** propagar `structured error context`. *(Es la opción C de la Sample Question 8 oficial.)*

> [!warning] Trampa 2 — Terminar todo el pipeline por el timeout de un solo subagente
> **El error:** propagar la excepción a un handler de nivel superior que mata el workflow completo. **Por qué falla:** desperdicia los resultados de los demás subagentes que sí terminaron bien; es desproporcionado cuando había estrategias de recuperación viables. **Lo correcto:** el coordinador evalúa la falla y decide una recuperación dirigida. *(Opción D de la Sample Question 8 oficial.)*

> [!warning] Trampa 3 — Retry con backoff y luego un genérico "search unavailable"
> **El error:** parece la opción "madura" porque incluye exponential backoff, pero al agotar los reintentos devuelve solo un status genérico. **Por qué falla:** el error genérico **oculta** la query intentada, los resultados parciales y las alternativas, impidiendo que el coordinador decida informadamente. **Lo correcto:** el retry local está bien, pero lo que se propaga al final debe ser `structured error context`. *(Opción B de la Sample Question 8 oficial — el distractor más tentador.)*

> [!warning] Trampa 4 — Reintentar un `valid empty result` porque "parece" falla
> **El error:** tratar `results: []` de una query exitosa como si fuera un error y reintentar. **Por qué falla:** un vacío válido significa que la query se ejecutó y no hubo matches — **esa es la respuesta**; reintentar desperdicia recursos en una query que siempre devolverá nada. **Lo correcto:** distinguir explícitamente `access failure` (`shouldRetry: true`) de `valid empty result` (`shouldRetry: false`).

> [!info] Pregunta de muestra oficial
> La **Sample Question 8** de `examguide.pdf` (escenario Multi-Agent Research System) evalúa exactamente este tema: un subagente de web search hace timeout y hay que elegir cómo fluye la falla al coordinador. La respuesta correcta es **`structured error context`**; los tres distractores son las Trampas 1, 2 y 3 de arriba. Está incluida en [[4 test]].

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
