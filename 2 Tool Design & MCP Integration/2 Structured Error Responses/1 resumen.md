---
tags:
  - claude-cert/dominio-2
  - task-statement/2.2
---

# 2.2 — Structured Error Responses

## Explícamelo como si tuviera 5 años

Imagina que le pides a un amigo que busque un libro en la biblioteca y te diga qué pasó. Si vuelve y solo dice "no funcionó", no tienes ni idea de qué hacer después: ¿el libro no existe? ¿la biblioteca estaba cerrada? ¿se le perdió el catálogo? No puedes decidir nada con esa respuesta.

Ahora imagina que en vez de eso vuelve y dice: "la biblioteca estaba cerrada por mantenimiento, vuelve a intentar en una hora" — o "busqué bien, y ese libro simplemente no existe en el catálogo, no hace falta que vuelvas a intentar". Son dos fracasos distintos, y cada uno te dice qué hacer después. Eso es exactamente lo que necesita un agente cuando una tool falla: no un "no funcionó" genérico, sino información que le permita decidir el siguiente paso.

## Argumento central

> [!note] Idea central
> Un mensaje de error genérico ("Operation failed") es inútil para un LLM porque no le da contexto para decidir cómo recuperarse. El protocolo MCP resuelve esto con el flag **`isError`** más metadata estructurada — y la distinción de mayor peso dentro de esa metadata es diferenciar **una tool que falló** de **una tool que tuvo éxito pero no encontró nada**.

## Conclusiones

### 1. Dos capas de error en MCP

MCP distingue dos niveles de fallo, y solo uno de ellos lo ve el modelo:

- **Errores de protocolo** — ocurren a nivel JSON-RPC cuando la request está mal formada (tool desconocida, schema inválido). **El modelo nunca ve estos** — se resuelven antes de llegar a la conversación.
- **Errores de ejecución de la tool** — la tool sí corrió, pero encontró un problema (rate limit de una API, datos inválidos). Estos regresan con `isError: true` y metadata recuperable para que el modelo decida qué hacer.

> [!tip] Analogía mental
> Es la diferencia entre marcar mal un número de teléfono (nunca conectas, error de protocolo) y que sí conteste alguien pero te diga "no puedo ayudarte con eso ahora mismo" (la llamada sí ocurrió, error de ejecución). Solo la segunda le da al modelo algo con qué trabajar.

### 2. Las cuatro categorías de error de ejecución

Cada categoría implica una estrategia de recuperación distinta — mezclarlas es el error más común del tema:

| Categoría | Ejemplo | `isRetryable` | Qué hacer |
|---|---|---|---|
| **Transient** (transitorio) | timeout, servicio no disponible | `true` | reintentar tras una espera |
| **Validation** (validación) | input malformado | `false` | corregir el input y reenviar |
| **Business** (regla de negocio) | violación de política | `false` | escalar o buscar un flujo alterno |
| **Permission** (permisos) | acceso denegado | `false` | usar otras credenciales |

> [!note] Los errores de negocio necesitan algo más que el booleano
> Para errores de tipo *business*, no basta con `isRetryable: false` — hay que incluir una explicación legible y orientada al cliente (*customer-friendly*) para que el agente pueda comunicar por qué falló la solicitud, no solo que falló.

### 3. La forma estructurada de la respuesta

La metadata recuperable vive en campos concretos de la respuesta de la tool, no en un string libre:

- `isError` (boolean) — marca si hubo un error de ejecución.
- `content` — el mensaje descriptivo del error.
- `structuredContent` — objeto contenedor con los campos que le dan al modelo poder de decisión: `errorCategory` (`transient` / `validation` / `business` / `permission`), `isRetryable` (boolean), y una `description` legible con guía de recuperación.

> [!tip] Analogía mental
> Es la diferencia entre un semáforo que solo dice "rojo" o "verde" (¿por qué está rojo? ¿por cuánto tiempo?) y uno con un letrero al lado: "rojo — obra en la vía, reabre en 10 minutos". El booleano de error es el color; `structuredContent` es el letrero.

### 4. La distinción más evaluada: fallo de acceso vs. resultado vacío válido

> [!warning] Este es el concepto de mayor peso del tema
> Una query exitosa que devuelve cero resultados (`isError: false`, con algo como `resultCount: 0`) es fundamentalmente distinta de una conexión a base de datos que falló (`isError: true`). **Confundir estos dos casos causa reintentos innecesarios** — reintentar una query que ya tuvo éxito y legítimamente no encontró nada solo repite el mismo resultado vacío, una y otra vez.

### 5. Recuperación local en subagentes y propagación selectiva

En arquitecturas coordinador-subagente (ver Task Statement 1.2/1.3), el manejo de errores no debe subir automáticamente al coordinador entero:

- Un subagente debe **resolver localmente** los errores transitorios que le competen (ej. reintentar un timeout dentro de su propio scope).
- Al coordinador solo se **propagan los errores que no se pudieron resolver localmente** — y se propagan junto con los resultados parciales obtenidos y una descripción de qué se intentó, no solo el hecho de que algo falló.

> [!tip] Analogía mental
> Es como un equipo de reporteros: cada uno resuelve sus propios problemas menores de logística sin llamar al editor en jefe. Solo cuando algo de verdad bloquea la nota, llaman — y cuando llaman, no dicen solo "no pude", sino "esto es lo que ya tengo, esto es lo que intenté, y esto es lo que falta".

## En una frase

> Un error sin estructura es un callejón sin salida para el modelo; un error estructurado (`isError` + `errorCategory` + `isRetryable` + descripción) es información que le permite decidir el siguiente paso — y la distinción que más se evalúa es no confundir "fallé" con "tuve éxito y no encontré nada".

## Trampas de examen

> [!warning] Trampa 1 — Reintentar un resultado vacío exitoso
> Tratar un `isError: false` con cero resultados como si fuera un fallo y reintentar la query. **Por qué es un error:** la query ya tuvo éxito — no hay nada que un reintento vaya a cambiar, porque el resultado vacío es legítimo, no accidental. El reintento desperdicia esfuerzo repitiendo el mismo resultado.

> [!warning] Trampa 2 — Ocultar el error tras agotar reintentos con un status genérico
> Implementar reintentos con backoff exponencial dentro del subagente, y solo al agotarlos devolver un status genérico tipo "search unavailable". **Por qué es un error:** ese status genérico oculta al coordinador el contexto valioso (tipo de fallo, qué se intentó, resultados parciales) que necesitaría para decidir una recuperación inteligente — vuelve al mismo problema del Argumento central, un "Operation failed" con otro nombre.

> [!warning] Trampa 3 — Marcar un fallo real como éxito para "resolverlo"
> Capturar un timeout dentro del subagente y devolver un resultado vacío marcado como exitoso (`isError: false`). **Por qué es un error:** esto suprime el error en vez de resolverlo — invierte exactamente la distinción de la Conclusión 4 (un fallo de acceso real se disfraza de resultado vacío legítimo), y le quita al coordinador cualquier posibilidad de recuperación, arriesgando una salida incompleta sin que nadie lo note.

> [!warning] Trampa 4 — Propagar la excepción cruda y terminar todo el flujo
> Ante un timeout de un subagente, dejar que la excepción suba directo a un handler de nivel superior que termina todo el workflow de investigación. **Por qué es un error:** termina el flujo completo de forma innecesaria cuando estrategias de recuperación (reintentar con query modificada, alternativa, o continuar con resultados parciales) podrían funcionar — es una respuesta desproporcionada al tamaño real del fallo.

> [!info] Pregunta de muestra oficial (examguide.pdf, Sample Question 8)
> El escenario de un subagente de búsqueda web que hace timeout, evaluando cuatro formas de propagar esa falla al coordinador, es literalmente la Question 8 de la sección "9. Sample Questions" del Exam Guide oficial. La respuesta correcta es devolver contexto estructurado (tipo de fallo, query intentada, resultados parciales, posibles alternativas) — no un status genérico tras reintentos silenciosos (Trampa 2), no marcar el fallo como éxito (Trampa 3), y no terminar todo el workflow (Trampa 4).

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
