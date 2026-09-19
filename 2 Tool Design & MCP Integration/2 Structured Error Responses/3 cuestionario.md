---
tags:
  - claude-cert/dominio-2
  - task-statement/2.2
---

# 3 cuestionario — Structured Error Responses

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## Dos capas de error

> [!question]- ¿Cuál es la diferencia entre un error de protocolo y un error de ejecución de una tool en MCP?
> El error de protocolo ocurre a nivel JSON-RPC (request mal formada, tool desconocida, schema inválido) y el modelo nunca lo ve — se resuelve antes. El error de ejecución ocurre cuando la tool sí corrió pero encontró un problema, y sí llega al modelo con `isError: true` y metadata recuperable.

> [!question]- ¿Por qué importa que el modelo nunca vea los errores de protocolo?
> Porque esos errores no son algo sobre lo que el modelo pueda decidir nada útil (la request ni siquiera llegó a ejecutarse correctamente) — solo los errores de ejecución traen información procesable para una decisión de recuperación.

## Las cuatro categorías de error

> [!question]- ¿Cuáles son las cuatro categorías de error de ejecución y cuál es retryable?
> Transient (retryable: sí, reintentar tras espera), validation (no, corregir input y reenviar), business (no, escalar o buscar flujo alterno), permission (no, usar otras credenciales). Solo transient es retryable.

> [!question]- ¿Qué pasaría si un agente tratara un error de validación como si fuera transient y simplemente reintentara la misma llamada?
> El reintento fallaría exactamente igual, porque el input sigue siendo inválido — nada cambió entre el primer intento y el reintento. Reintentar solo tiene sentido cuando la causa del fallo es externa y puede desaparecer sola (transient), no cuando la causa está en el input mismo.

> [!question]- ¿Qué le falta a un error de tipo business si solo se devuelve `isRetryable: false` sin nada más?
> Le falta una descripción legible y orientada al cliente que explique por qué falló — el booleano le dice al agente que no debe reintentar, pero no le da con qué comunicarle al usuario la razón de la violación de política.

> [!question]- ¿Cuándo usarías `isRetryable: true` en vez de escalar a un flujo alterno?
> Cuando el error es transitorio (timeout, servicio no disponible temporalmente) — algo externo que puede resolverse solo con el tiempo. Escalar o buscar un flujo alterno es para errores donde reintentar la misma llamada nunca va a cambiar el resultado (business, permission, validation).

## La forma estructurada de la respuesta

> [!question]- ¿Qué campos componen `structuredContent` en una respuesta de error de MCP?
> `errorCategory` (transient/validation/business/permission), `isRetryable` (boolean), y `description` (explicación legible con guía de recuperación).

> [!question]- ¿Cómo se relaciona `isError` con `structuredContent`?
> `isError` es el flag que marca que hubo un fallo de ejecución; `structuredContent` es la metadata que explica ese fallo con suficiente detalle para que el agente decida qué hacer después. Uno marca que algo pasó, el otro dice qué y qué hacer.

## Fallo de acceso vs. resultado vacío válido

> [!question]- ¿Por qué es el concepto de mayor peso del tema la distinción entre fallo de acceso y resultado vacío válido?
> Porque confundirlos lleva directamente al comportamiento que el examen más penaliza: reintentar algo que no necesita reintento (o al revés, dar por válido algo que en realidad falló). Es la trampa central sobre la que se construyen varias de las otras.

> [!question]- Una query a una base de datos devuelve `isError: false` y `resultCount: 0`. ¿Qué significa esto y qué se debería hacer?
> Significa que la query tuvo éxito y legítimamente no encontró coincidencias. No se debe reintentar — reintentar solo repetiría el mismo resultado vacío, porque no hubo ningún fallo que corregir.

> [!question]- ¿Qué pasaría si un agente reintentara automáticamente cualquier respuesta con `resultCount: 0`, sin revisar `isError`?
> Desperdiciaría esfuerzo en reintentos inútiles cada vez que una búsqueda legítima no encuentra nada, porque `resultCount: 0` con `isError: false` no es un fallo — es exactamente el resultado correcto de esa consulta.

## Recuperación local en subagentes y propagación

> [!question]- ¿Qué errores debería resolver un subagente localmente, y cuáles debería propagar al coordinador?
> Debería resolver localmente los errores transitorios dentro de su propio scope (ej. reintentar un timeout). Solo debería propagar al coordinador los errores que no logró resolver localmente.

> [!question]- Cuando un subagente propaga un error al coordinador, ¿qué más debe incluir además del hecho de que algo falló?
> Los resultados parciales que ya obtuvo y una descripción de qué se intentó — no solo la notificación del fallo. Esto es lo que le permite al coordinador tomar una decisión de recuperación inteligente en vez de empezar desde cero.

> [!question]- ¿Cómo se relaciona esto con la arquitectura hub-and-spoke de coordinador-subagentes (Task Statement 1.2)?
> Es consistente con que toda la comunicación entre subagentes pase por el coordinador para mantener observabilidad y manejo de errores consistente — pero no significa que el coordinador deba enterarse de cada fallo menor; solo de los que el subagente no pudo resolver por sí mismo.

> [!question]- ¿Por qué reintentar con backoff exponencial dentro del subagente y devolver un status genérico ("search unavailable") tras agotar los reintentos sigue siendo un error, aunque sí hubo reintentos?
> Porque el status genérico, aunque llega después de reintentos legítimos, oculta al coordinador el contexto que necesitaría (tipo de fallo, qué se intentó, resultados parciales) para decidir una recuperación distinta — el problema no es que falten reintentos, es que la información que llega al final sigue siendo opaca.

> [!question]- ¿Por qué marcar un timeout como resultado vacío exitoso es peor que simplemente propagar el error sin resolverlo?
> Porque suprime el fallo en vez de exponerlo: invierte la distinción entre fallo de acceso y resultado vacío legítimo, y le quita al coordinador toda posibilidad de recuperación — un error propagado sin resolver al menos permite que alguien más intente algo; un error disfrazado de éxito no.

## Síntesis

> [!question]- Un timeout de un subagente de búsqueda llega al coordinador. ¿Qué opción de propagación evalúa mejor el examen: reintentos silenciosos con status genérico, resultado vacío marcado como éxito, terminar todo el workflow, o contexto estructurado con tipo de fallo + query intentada + resultados parciales + alternativas?
> Contexto estructurado con tipo de fallo, query intentada, resultados parciales y alternativas posibles — es la única opción que le da al coordinador información suficiente para decidir de forma inteligente entre reintentar con una query modificada, probar un enfoque alterno, o continuar con los resultados parciales que ya existen.
