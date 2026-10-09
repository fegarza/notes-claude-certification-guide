# Resultados — Tutor socrático: Error Propagation in Multi-Agent Systems

> [!info] Sesión evaluada el 2026-10-06
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 84/100**

De 17 preguntas, 13 fueron correctas y 4 parcialmente correctas, sin respuestas incorrectas. Dominas bien la idea central: los dos antipatrones, por qué importan `attemptedAction` y `partialResults`, y la distinción `access failure` vs. `valid empty result`, incluida la trampa del retry con backoff que termina en un error genérico. Lo que falla es la precisión en los detalles que el examen usa para separar opciones parecidas. Confundiste qué campo señala explícitamente el retry (`shouldRetry`, no `failureType`) y no tenías completas las 4 opciones de recuperación del coordinador. En un par de respuestas también agregaste ideas que no vienen en la guía, como "cambiar credenciales" o "cambiar permisos".

## Conceptos con buen dominio

- **Argumento central** — identificaste que la clave es propagar la falla como `structured error context`.
- **`attemptedAction`** — entendiste que sin él el coordinador no puede reintentar con una query modificada.
- **`partialResults`** — lo relacionaste correctamente con no tirar a la basura el trabajo útil.
- **`alternativeApproaches`** — justificaste bien que las proponga el subagente porque es el especialista en su dominio.
- **`Silent suppression`** — lo reconociste en el código y entendiste por qué es el más peligroso.
- **`Workflow termination`** — explicaste correctamente que desperdicia el trabajo de los demás subagentes.
- **`Access failure` vs. `valid empty result`** — distinguiste ambos casos e integraste por tu cuenta la `local recovery` en la respuesta.
- **Confundir `access failure` con vacío válido = `silent suppression`** — hiciste la conexión sin ayuda.
- **`Local recovery`** — entendiste que reduce la complejidad del coordinador.
- **Trampa 3 (retry con backoff + genérico "search unavailable")** — detectaste que el problema es el error genérico del final.
- **Trampa 4 (reintentar un `valid empty result`)** — la resolviste correctamente.
- **`Coverage annotations`** — sabes qué son y para qué sirven en la síntesis.

## Áreas que necesitan revisión

### Para qué sirve `failureType`

- **Qué pasó:** respondiste con la intuición correcta (`transient` es temporal y `permission` no se arregla insistiendo), pero agregaste "cambiar credenciales", que no viene en la guía. La función concreta que da la guía es más simple.
- **Contenido del resumen:**

| Elemento | Qué contiene | Para qué le sirve al coordinador |
|---|---|---|
| **`failureType`** | Categoría: `transient` (timeout, rate limit), `validation` (input inválido), `business` (violación de regla), `permission` (acceso denegado) | Saber si tiene sentido reintentar o no |

- **Sección:** [[1 resumen]] → "1. `Structured error context` — los 4 elementos obligatorios"
- **Analogía para reforzarlo:** es como el letrero de una tienda cerrada. "Vuelvo en 10 minutos" (`transient`) te dice que vale la pena esperar y volver a tocar. "Solo socios" (`permission`) te dice que tocar de nuevo no sirve de nada. El letrero no te dice cómo hacerte socio, solo si conviene volver a intentar.

### `shouldRetry` como señal explícita entre `access failure` y `valid empty result`

- **Qué pasó:** la consecuencia de tratar un vacío válido como falla la tenías clara (reintentos innecesarios), pero dijiste que el campo que los distingue es `failureType`. En el ejemplo de la guía el `valid empty result` ni siquiera lleva `failureType`, porque no es una falla. La señal explícita es `shouldRetry`, junto con `status`.
- **Contenido del resumen:**

| | `Access failure` | `Valid empty result` |
|---|---|---|
| Acción | Considerar retry (mismos o distintos parámetros) | **No reintentar — eso ES la respuesta** |
| Señal típica | `status: "error"`, `shouldRetry: true` | `status: "success"`, `results: []`, `shouldRetry: false` |

> **Consecuencias de confundirlos:**
> - `Access failure` tratado como vacío válido → **nunca reintentas cuando debías** (es justo el `silent suppression`).
> - Vacío válido tratado como `access failure` → **pierdes tiempo reintentando** una query que siempre devolverá nada.

- **Sección:** [[1 resumen]] → "3. `Access failure` vs. `valid empty result`"
- **Analogía para reforzarlo:** buscas un contacto en tu teléfono. Si sale "sin señal, no se pudo sincronizar", vuelves a intentar (`shouldRetry: true`). Si la agenda cargó bien y dice "0 contactos con ese nombre", buscarlo otra vez no lo va a hacer aparecer (`shouldRetry: false`). La pregunta que decide no es "¿qué tipo de error es?" sino "¿vale la pena volver a buscar?".

### El punto medio: falla "visible" y "acotada"

- **Qué pasó:** la primera formulación de la pregunta no fue clara. Al reformularla identificaste bien la parte "visible" (vs. `silent suppression`). Para la parte de `workflow termination` respondiste con una consecuencia ("no desperdicia resultados parciales") en vez de la propiedad: la falla queda **acotada**, y eso permite una recuperación dirigida decidida por el coordinador.
- **Contenido del resumen:**

> [!note] El punto medio correcto
> `Structured error propagation`: la falla se hace **visible** (a diferencia de suppression) pero **acotada** (a diferencia de termination), permitiendo recuperación dirigida y preservando resultados parciales. El coordinador es quien toma la decisión inteligente de recuperación, en vez de aplicar políticas en bloque.

- **Sección:** [[1 resumen]] → "2. Los dos antipatrones (los extremos opuestos)"
- **Analogía para reforzarlo:** piensa en los fusibles de una casa. Una casa sin fusibles esconde el corto hasta que algo se quema (suppression). Una casa con un solo interruptor general deja a oscuras toda la casa por un enchufe malo (termination). Con fusibles por circuito, la falla se ve (se apaga ese foco) y queda acotada (el resto de la casa sigue con luz).

### Las 4 opciones de recuperación del coordinador

- **Qué pasó:** acertaste 2 de 4 (reintentar con la solicitud ajustada y escalar). Te faltaron "probar una alternativa" y "seguir con resultados parciales", y agregaste opciones que no vienen en la guía ("reintentar sin cambiar nada", "cambiar permisos").
- **Contenido del resumen:**

> Con esto el coordinador puede decidir entre: **reintentar con query modificada**, **probar una alternativa**, **seguir con resultados parciales**, o **escalar**.

- **Sección:** [[1 resumen]] → "1. `Structured error context` — los 4 elementos obligatorios"
- **Analogía para reforzarlo:** un GPS que encuentra la calle cerrada te ofrece cuatro cosas: recalcular por la misma zona con otra ruta (query modificada), tomar la autopista (alternativa), dejarte a dos cuadras para que llegues caminando (resultados parciales) o decirte "no puedo llevarte, llama al lugar" (escalar). Tres de las opciones salen de un campo del error: `attemptedAction` → query modificada, `alternativeApproaches` → alternativa, `partialResults` → seguir con parciales. La cuarta es escalar.

## Siguiente paso recomendado

Repasa la tabla de `access failure` vs. `valid empty result` (fíjate en `shouldRetry`) y las 4 opciones de recuperación del coordinador, relacionando cada una con su campo del `structured error context`. Después de eso ya estás listo para [[4 test]], con atención especial a la Sample Question 8.
