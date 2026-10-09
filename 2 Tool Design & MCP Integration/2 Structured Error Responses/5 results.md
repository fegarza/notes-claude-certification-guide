# Resultados — Tutor socrático: Structured Error Responses

> [!info] Sesión evaluada el 2026-10-06
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 78/100**

De 15 preguntas: 9 correctas, 5 parcialmente correctas y 1 sin responder. La base conceptual es sólida: entiende el problema de los errores genéricos, las dos capas de error de MCP y las cuatro categorías de error con su `isRetryable`. El patrón débil es la **precisión al cerrar las respuestas**: identifica bien *qué* está mal, pero suele omitir la consecuencia concreta (reintentos innecesarios, pérdida de recuperación), los nombres exactos de los campos y las listas que la guía enumera (estrategias de recuperación, contenido de la respuesta correcta en la Sample Question 8). Esos detalles son justo los que diferencian las opciones en el examen.

## Conceptos con buen dominio

- **Argumento central (error genérico inútil)**: explicó con claridad que "Operation failed" no le da al modelo contexto para decidir el siguiente paso.
- **Errores de protocolo vs. de ejecución**: ubicó correctamente una tool inexistente como error JSON-RPC que el modelo nunca ve, y el rate limit como error de ejecución con `isError`.
- **Categorías transient y validation**: dio la categoría, el `isRetryable` y la estrategia correctos, con buen razonamiento (no hay que cambiar la request vs. hay que corregir el input).
- **Business requiere descripción customer-friendly**: identificó que se necesita una explicación en lenguaje natural para dar feedback al usuario.
- **Permission**: categoría y estrategia (cambiar credenciales/permisos) correctas.
- **Recuperación local en subagentes**: explicó que el subagente maneja primero los errores transitorios y solo propaga lo no resuelto, junto con resultados parciales y la causa.
- **Trampa 2 (status genérico tras reintentos)**: la reconoció de inmediato como el mismo problema del Argumento central.

## Áreas que necesitan revisión

### Estrategias de recuperación ante un error business

- **Qué pasó:** solo mencionó escalar. Omitió la alternativa de buscar un flujo alterno.
- **Contenido del resumen:**

| Categoría | Ejemplo | `isRetryable` | Qué hacer |
|---|---|---|---|
| **Business** (regla de negocio) | violación de política | `false` | escalar o buscar un flujo alterno |

- **Sección:** [[1 resumen]] → "2. Las cuatro categorías de error de ejecución"
- **Analogía para reforzarlo:** si el banco no te deja retirar más de cierta cantidad en el cajero, puedes pedir autorización a tu ejecutivo (escalar) o hacer una transferencia en lugar del retiro (flujo alterno). Insistir en el cajero no sirve de nada.

### Campos concretos de `structuredContent`

- **Qué pasó:** entendió el propósito (metadata que le da al agente poder de decisión), pero no nombró los campos que contiene.
- **Contenido del resumen:**
> - `isError` (boolean) — marca si hubo un error de ejecución.
> - `content` — el mensaje descriptivo del error.
> - `structuredContent` — objeto contenedor con los campos que le dan al modelo poder de decisión: `errorCategory` (`transient` / `validation` / `business` / `permission`), `isRetryable` (boolean), y una `description` legible con guía de recuperación.
- **Sección:** [[1 resumen]] → "3. La forma estructurada de la respuesta"
- **Analogía para reforzarlo:** es como la etiqueta de un paquete devuelto por paquetería. No solo dice "devuelto": tiene casillas marcadas con el motivo (dirección incorrecta, destinatario ausente…), si conviene reenviarlo, y una nota que explica qué hacer.

### Consecuencia de confundir resultado vacío con fallo

- **Qué pasó:** diagnosticó correctamente que cero resultados no es un error, pero describió la consecuencia de forma vaga ("el modelo cree que hay un error") en lugar de la consecuencia concreta: reintentos innecesarios.
- **Contenido del resumen:**
> Una query exitosa que devuelve cero resultados (`isError: false`, con algo como `resultCount: 0`) es fundamentalmente distinta de una conexión a base de datos que falló (`isError: true`). **Confundir estos dos casos causa reintentos innecesarios** — reintentar una query que ya tuvo éxito y legítimamente no encontró nada solo repite el mismo resultado vacío, una y otra vez.
- **Sección:** [[1 resumen]] → "4. La distinción más evaluada: fallo de acceso vs. resultado vacío válido"
- **Analogía para reforzarlo:** buscas tus llaves en un cajón, lo revisas completo y no están. Volver a abrir el mismo cajón diez veces no las hará aparecer: el cajón no "falló", simplemente ahí no están.

### Trampa 3 — consecuencia de disfrazar un fallo como éxito

- **Qué pasó:** identificó que se engaña al coordinador, pero no mencionó la consecuencia clave: se pierde toda posibilidad de recuperación y la salida queda incompleta sin que nadie lo note.
- **Contenido del resumen:**
> **Por qué es un error:** esto suprime el error en vez de resolverlo — invierte exactamente la distinción de la Conclusión 4 (un fallo de acceso real se disfraza de resultado vacío legítimo), y le quita al coordinador cualquier posibilidad de recuperación, arriesgando una salida incompleta sin que nadie lo note.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** es como un termómetro roto que siempre marca 36.5 °C. No te dice que esté roto, así que nunca vas al doctor aunque tengas fiebre.

### Trampa 4 — estrategias de recuperación descartadas

- **Qué pasó:** reconoció que terminar el workflow pierde todo el trabajo, pero no enumeró las estrategias alternativas que quedan descartadas.
- **Contenido del resumen:**
> **Por qué es un error:** termina el flujo completo de forma innecesaria cuando estrategias de recuperación (reintentar con query modificada, alternativa, o continuar con resultados parciales) podrían funcionar — es una respuesta desproporcionada al tamaño real del fallo.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** si en una receta te falta un ingrediente, no tiras toda la comida. Lo buscas con otro nombre en la tienda (query modificada), lo sustituyes (alternativa) o sirves el platillo sin ese toque (resultados parciales).

### Sample Question 8 — contenido de la respuesta correcta

- **Qué pasó:** no pudo enumerar los cuatro elementos que debe devolver el subagente al coordinador, aunque en una respuesta anterior ya había mencionado dos (resultados parciales y causa del fallo).
- **Contenido del resumen:**
> La respuesta correcta es devolver contexto estructurado (tipo de fallo, query intentada, resultados parciales, posibles alternativas) — no un status genérico tras reintentos silenciosos (Trampa 2), no marcar el fallo como éxito (Trampa 3), y no terminar todo el workflow (Trampa 4).
- **Sección:** [[1 resumen]] → "Trampas de examen" (callout "Pregunta de muestra oficial")
- **Analogía para reforzarlo:** un buen reporte de un repartidor que no pudo entregar: "nadie abrió (tipo de fallo), toqué en el depto 4B (qué intenté), dejé 2 de los 3 paquetes con el portero (parciales), puedo volver mañana o dejarlo en sucursal (alternativas)".

## Siguiente paso recomendado

Se cubrió todo el checklist del resumen. Antes de [[4 test]], repasa las **Trampas de examen** y la Sample Question 8 enfocándote en memorizar las listas exactas (las 3 estrategias de recuperación y los 4 elementos de la respuesta estructurada) y las consecuencias concretas de cada trampa: en el examen, los distractores se distinguen justo por esos detalles.
