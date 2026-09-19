---
tags:
  - claude-cert/dominio-5
  - task-statement/5.1
---

# 3 cuestionario — Context Window Management

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## La trampa de la progressive summarisation

> [!question]- ¿Por qué la progressive summarisation es una trampa y no simplemente una técnica de ahorro de tokens?
> Porque destruye sistemáticamente los datos más críticos en sistemas transaccionales — valores numéricos, fechas, porcentajes, expectativas explícitas del cliente. No es un riesgo ocasional: es lo que la summarisation le hace por defecto a ese tipo de datos.

> [!question]- ¿Qué pasaría si un agente resumiera "quiero un reembolso de $247.83 para la orden #8891" como "el cliente quiere un reembolso reciente"?
> El agente perdería los tres datos que necesita para procesar el reembolso (monto, número de orden, fecha) y no podría actuar sobre la solicitud sin volver a pedírselos al cliente.

> [!question]- ¿Qué es un persistent case facts block y dónde vive dentro del prompt?
> Un bloque estructurado con los hechos transaccionales (montos, fechas, order IDs, estados) que se incluye en cada prompt, fuera del historial que se resume. Nunca se resume y persiste sin importar qué le pase al resto de la conversación.

> [!question]- ¿Cómo se maneja una sesión donde el cliente plantea varios problemas distintos en la misma conversación?
> Cada issue se extrae y persiste como una entrada separada dentro de la capa de case facts, con su propio order ID, monto y estado — esto evita que la summarisation mezcle los datos de un issue con los de otro.

> [!question]- ¿Por qué se dice que el persistent case facts block es "el patrón más importante" del tema?
> Porque es la solución directa a la trampa de mayor impacto (progressive summarisation) y, además, es la pieza que resuelve la tensión entre necesitar el historial completo y tener presupuesto de tokens limitado — es la base sobre la que se apoyan las demás conclusiones.

## El efecto "lost in the middle"

> [!question]- ¿Qué es el efecto "lost in the middle"?
> Los modelos procesan de forma confiable la información al inicio y al final de un input largo, pero pueden pasar por alto o darle menos peso a lo que queda enterrado en el medio.

> [!question]- ¿Por qué no basta con decirle al modelo "presta atención a todo el contenido, incluido el del medio"?
> Porque el fix es estructural, no un asunto de instrucción — es un efecto de posición del procesamiento del modelo, y los recordatorios dentro del prompt no son confiables para corregirlo.

> [!question]- ¿Cuál es la forma correcta de estructurar el input cuando se agregan los resultados de varios subagentes?
> Colocar primero una sección de resumen de hallazgos clave (ej. "Key Findings Summary"), y después el detalle completo de cada fuente organizado con encabezados de sección explícitos.

> [!question]- ¿Cómo se relaciona este efecto con dónde se coloca la información más importante en un documento largo que se le pasa al modelo?
> La información más importante debe ir al principio (o, en su defecto, al final), nunca depender de que el modelo la encuentre en medio de un bloque largo de texto sin estructura.

## Tool result trimming

> [!question]- ¿Por qué se dice que los resultados de tools sin recortar son un "asesino silencioso" del presupuesto de contexto?
> Porque un resultado verboso (ej. 40+ campos de un lookup de orden) queda en el historial de la conversación y consume tokens en cada turno subsecuente, aunque solo unos pocos campos sean relevantes para la tarea.

> [!question]- ¿En qué momento del flujo debe ocurrir el recorte de un resultado de tool?
> Antes de que el resultado entre al historial de la conversación — típicamente en un hook `PostToolUse` o dentro de la propia implementación de la tool, nunca después de que el dato ya quedó registrado.

> [!question]- ¿Qué pasaría si el recorte de un resultado de tool se aplicara solo al mostrárselo al usuario, pero el dato completo ya estuviera en el historial?
> No se ahorraría nada — como cada request debe llevar el historial completo, el dato verboso seguiría reenviándose en cada turno subsecuente sin importar cómo se le presente al usuario en la interfaz.

## La API es stateless

> [!question]- ¿Qué significa que la API de Claude sea stateless?
> Que no guarda estado del lado del servidor entre requests — no existe una sesión persistida, así que cada request debe incluir el historial completo de la conversación para que el modelo mantenga coherencia.

> [!question]- ¿Qué pasaría si un sistema truncara selectivamente turnos antiguos de la conversación en vez de resumirlos?
> Rompería la coherencia conversacional, porque no hay ningún estado del lado del servidor que compense la información que falta — la API solo sabe lo que llega en la request actual.

> [!question]- ¿Cómo resuelve el persistent case facts block la tensión entre "se necesita el historial completo" y "el historial crece con cada turno"?
> Separando los hechos críticos (que nunca se resumen ni se pierden) de la narrativa conversacional (que sí puede comprimirse) — así se puede resumir el flujo de la conversación sin sacrificar ningún dato transaccional.

> [!question]- ¿Cuál es la diferencia entre "resumir" y "truncar" el historial, y por qué solo una de las dos es aceptable?
> Resumir comprime la narrativa preservando aparte los hechos críticos (aceptable, si se hace con un case facts block). Truncar elimina turnos completos sin ese resguardo, rompiendo la coherencia — no es aceptable como estrategia de manejo de contexto.

## Upstream agent optimisation

> [!question]- ¿Qué problema ocurre cuando un subagente de investigación le manda al agente de síntesis toda su cadena de razonamiento?
> El agente de síntesis, que tiene presupuesto de contexto limitado, desperdicia tokens procesando razonamiento que no puede usar directamente para su tarea de síntesis.

> [!question]- ¿Qué deberían devolver los agentes upstream en vez de contenido y razonamiento verbosos?
> Datos estructurados: hechos clave, citas, puntajes de relevancia — información ya procesada y lista para que el agente downstream la use sin reinterpretarla.

> [!question]- ¿Qué metadata deben incluir los subagentes en sus salidas estructuradas, y para qué sirve?
> Fechas, ubicación de la fuente, y contexto metodológico — sirve para que el agente downstream pueda hacer una síntesis precisa sin tener que volver a la fuente original a buscar ese contexto.

> [!question]- ¿Cuál es el beneficio de las salidas estructuradas más allá del ahorro de tokens?
> Le permiten al agente downstream procesar los hallazgos directamente, sin tener que re-parsear prosa verbosa para extraer la información que necesita.

> [!question]- ¿Cómo se relaciona esto con el principio de separación de responsabilidades en arquitecturas multi-agente?
> Cada agente debe producir la salida en la forma que su consumidor necesita, no en la forma que le resultó más natural producirla — el subagente de investigación optimiza su salida para el agente de síntesis, no para sí mismo.

## Prompt caching

> [!question]- ¿Qué problema resuelve el prompt caching, y en qué se diferencia de las demás técnicas del tema?
> Evita pagar por reprocesar las partes del prompt que no cambian entre requests, reutilizando un prefijo ya procesado. A diferencia de las demás técnicas (que recortan qué ve el modelo), el caching no cambia el contenido — cambia el costo de procesarlo.

> [!question]- ¿Por qué el orden del contenido dentro del prompt determina si hay un cache hit?
> Porque el caching hace match de prefijo desde el inicio del prompt — si el contenido volátil aparece antes del contenido estático, el prefijo cambia en cada request y nunca hay coincidencia con lo cacheado previamente.

> [!question]- ¿Dónde debe colocarse el contenido estático (instrucciones, definiciones de tools, documentos de referencia) y dónde el volátil?
> El contenido estático va primero, con el breakpoint `cache_control` al final de ese bloque; el contenido volátil (el mensaje actual del usuario) va después del breakpoint.

> [!question]- ¿Por qué el bloque estático debe ir en el parámetro `system` y no dentro de `messages`?
> Porque la Messages API no tiene un rol `"system"` para los mensajes de input — `messages` solo acepta turnos `"user"` y `"assistant"`; el contenido de sistema tiene su propio parámetro de nivel superior.

> [!question]- ¿Qué pasaría si el contenido dinámico del usuario se colocara antes del bloque estático en el prompt?
> El prefijo cambiaría en cada request, nunca habría coincidencia con lo cacheado, y se perdería el beneficio del caching por completo — cada llamada pagaría el precio completo de procesamiento.

> [!question]- ¿Cuánto dura un breakpoint `ephemeral` por defecto, y qué alternativa existe para una duración mayor?
> Dura unos 5 minutos desde el último uso. Existe la variante `{"type": "ephemeral", "ttl": "1h"}`, que dura una hora a un costo de escritura más alto.

> [!question]- ¿Cuántos breakpoints de `cache_control` puede llevar como máximo una sola request?
> Cuatro.

> [!question]- ¿Qué tan a fondo evalúa el examen los detalles de implementación de prompt caching?
> Poco — la guía excluye explícitamente los detalles de implementación más allá de saber que existe, para qué sirve, y la lógica básica de orden estático-antes-que-dinámico.

## Síntesis

> [!question]- Un agente de soporte multi-issue empieza a decir "tu reembolso reciente" en vez de mencionar el monto exacto tras varios turnos. ¿Cuál es la causa más probable y cuál la solución?
> La causa es que la summarisation entre turnos está destruyendo los datos transaccionales (progressive summarisation trap). La solución es extraer esos hechos hacia un persistent case facts block que se incluya en cada prompt fuera del historial resumido.

> [!question]- Un sistema de investigación multi-agente agrega los hallazgos de tres fuentes, pero el hallazgo de la fuente del medio nunca aparece en el reporte final de síntesis. ¿Qué patrón del tema aplica y cuál es el fix?
> Es el efecto "lost in the middle". El fix es estructural: colocar un resumen de hallazgos clave al inicio del input agregado y organizar el detalle completo debajo con encabezados de sección explícitos — no basta con instruir al modelo a "no ignorar nada".

> [!question]- ¿Por qué un sistema que no recorta los resultados de sus tools, no separa hechos transaccionales, y confía solo en la summarisation, eventualmente se vuelve poco confiable en conversaciones largas — aunque cada componente individual "funcione"?
> Porque cada uno de esos huecos (tool results sin recortar, ausencia de case facts block, dependencia total en summarisation) se combina: el contexto crece sin control, los datos críticos se degradan con cada resumen, y como la API es stateless, todo ese deterioro se reenvía y se agrava en cada turno subsecuente.
