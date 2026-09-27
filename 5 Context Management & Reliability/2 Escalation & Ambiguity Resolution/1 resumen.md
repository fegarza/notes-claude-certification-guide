> [!note] En una frase
> Un agente solo debe escalar a un humano por tres razones válidas y verificables —se lo piden explícitamente, la política no cubre el caso, o genuinamente no puede avanzar—, nunca porque el cliente "suena molesto" o porque el propio modelo "no se siente seguro".

## Explícamelo como si tuviera 5 años

Imagina un mesero nuevo en un restaurante. Su jefe le dio tres reglas claras para llamar al gerente: (1) si el cliente pide hablar con el gerente directamente, (2) si el cliente pide algo que no está en el menú de excepciones que el mesero conoce, o (3) si algo se rompió y el mesero no tiene forma de arreglarlo (la cocina no responde, por ejemplo).

Lo que el mesero **no** debe hacer es llamar al gerente solo porque el cliente parece de mal humor —a lo mejor solo quiere que le cambien un plato frío por uno caliente, algo que el mesero puede resolver solo—. Tampoco debe decidir cuándo llamar al gerente basándose en qué tan seguro se "siente" él mismo de poder resolverlo, porque esa sensación de seguridad no es confiable: a veces se siente muy seguro en un caso difícil y se equivoca, y a veces duda en un caso fácil que sí podía resolver.

## Argumento central

> `Escalation` y `ambiguity resolution` son decisiones de diseño explícitas, no un efecto secundario de "qué tan difícil se ve el caso". Solo tres condiciones justifican escalar a un humano, y dos señales aparentemente intuitivas —el tono emocional del cliente y la confianza que el propio modelo reporta sobre sí mismo— son proxies no confiables de la complejidad real del caso. La forma más efectiva de calibrar esto es agregar criterios explícitos de escalación con ejemplos `few-shot` al `system prompt`, antes de construir infraestructura adicional (clasificadores, análisis de sentimiento, etc.).

## Ideas clave (Conclusiones)

### 1. Los tres triggers válidos de escalación

1. **Solicitud explícita de un humano**: cuando el cliente dice literalmente "quiero hablar con una persona", se escala de inmediato — sin investigar antes, sin intentar resolver primero.
2. **Excepción o vacío de política**: la política documentada no cubre el caso específico del cliente. Esto es distinto de una violación de política (que sí tiene una respuesta documentada y clara: se aplica la política, no se escala).
3. **Incapacidad genuina de progresar**: después de intentos reales de resolución, las herramientas devuelven errores, falta acceso a un sistema requerido, o hay un bug técnico que necesita intervención de ingeniería.

> [!note] Analogía mental
> Piensa en los tres triggers como tres "puertas de salida" distintas del flujo normal: pedida por el cliente (puerta 1), no cubierta por el mapa que el agente tiene (puerta 2), y bloqueada por un obstáculo técnico real (puerta 3). Si no estás parado frente a ninguna de esas tres puertas, no escalas — sigues resolviendo.

### 2. Los dos antipatrones de escalación

La guía marca explícitamente dos señales como **no confiables** para decidir si escalar:

- **`Sentiment-based escalation`**: escalar porque el cliente "suena frustrado" o "escribe en mayúsculas". La frustración **no correlaciona** con la complejidad del caso — un cliente furioso por un envío tarde puede tener un problema trivial de resolver.
- **`Self-reported confidence scores`**: pedirle al modelo que reporte qué tan seguro está y escalar cuando esa confianza es baja. Estos scores están mal calibrados: el modelo suele estar **confiado incorrectamente en casos difíciles** e **inseguro en casos fáciles** — es decir, la señal apunta justo al revés de lo que se necesitaría.

> [!warning] Por qué importa
> Ambas señales se sienten intuitivas ("si suena difícil o el modelo duda, mejor que lo vea un humano"), pero ninguna mide la variable real que importa: si el caso cae dentro de uno de los tres triggers válidos.

### 3. Frustración vs. solicitud explícita — la distinción crítica

Un cliente frustrado **no** es automáticamente un cliente que hay que escalar:

- Si el problema está dentro de la capacidad del agente, la respuesta correcta es **reconocer la frustración y ofrecer la resolución** directamente (empatía + acción), no escalar.
- Solo se escala si, después de esa oferta, el cliente **reitera** su preferencia por hablar con un humano — en ese punto se convierte en el trigger 1 (solicitud explícita).

> [!note] Analogía mental
> Es la diferencia entre un cliente que grita "¡esto es un desastre, arréglenlo YA!" (frustrado, pero pidiendo una solución) y uno que grita "¡esto es un desastre, quiero hablar con una persona YA!" (frustrado y pidiendo explícitamente escalar). El primero se resuelve; el segundo se escala de inmediato.

### 4. Múltiples matches de cliente → pedir identificadores, nunca heurística

Cuando una herramienta devuelve **varios registros posibles** para un mismo cliente (ej. dos cuentas con nombre similar), el agente **no** debe elegir por su cuenta usando una heurística (el registro más reciente, el más activo, el primero de la lista). La acción correcta es pedirle al cliente **identificadores adicionales** — email, teléfono, número de orden — para desambiguar con certeza.

> [!warning] Por qué importa
> Elegir heurísticamente entre varios matches puede significar actuar sobre la cuenta equivocada: exponer datos de otra persona o tomar una acción (reembolso, cambio) sobre el pedido incorrecto. La ambigüedad de identidad se resuelve pidiendo más información, no adivinando.

### 5. Mejor práctica de implementación: criterios explícitos + few-shot en el system prompt

La forma más efectiva de calibrar cuándo escalar es agregar **criterios explícitos de escalación**, acompañados de **ejemplos `few-shot`** dentro del `system prompt`, que demuestren casos concretos de "esto se escala" vs. "esto se resuelve de forma autónoma". Esto se recomienda **antes** de invertir en infraestructura más compleja (modelos clasificadores dedicados, análisis de sentimiento, scores de confianza), porque esas alternativas son precisamente los antipatrones que ya se descartaron.

## Trampas de examen

> [!warning] Trampa 1 — Escalar por tono/sentimiento del cliente
> Diseñar el agente para que escale cuando detecta frustración, enojo o lenguaje en mayúsculas. **Por qué falla:** la frustración no correlaciona con la complejidad real del caso; un cliente furioso puede tener un problema simple, y uno tranquilo puede tener uno genuinamente irresoluble. **La forma correcta:** escalar solo por los tres triggers válidos; ante frustración con un caso resoluble, reconocer la emoción y ofrecer la solución.

> [!warning] Trampa 2 — Usar la confianza autorreportada del modelo como criterio
> Pedirle a Claude que emita un "confidence score" sobre el caso y escalar cuando es bajo. **Por qué falla:** estos scores están mal calibrados — el modelo suele estar confiado de más en casos difíciles e inseguro en casos fáciles, así que la señal es poco fiable y puede ir en la dirección contraria a la deseada.

> [!warning] Trampa 3 — Investigar primero cuando el cliente ya pidió un humano explícitamente
> Intentar resolver el caso ("déjame revisar esto primero...") antes de escalar, aunque el cliente ya haya pedido explícitamente hablar con una persona. **Por qué falla:** una solicitud explícita se honra de inmediato, sin investigación previa — retrasarla contradice directamente el trigger 1.

> [!warning] Trampa 4 — Elegir heurísticamente entre múltiples matches de cliente
> Cuando una búsqueda devuelve varios registros posibles, seleccionar automáticamente el más reciente o el más activo en vez de preguntar. **Por qué falla:** puede llevar a actuar sobre la cuenta o el pedido equivocado, exponiendo datos ajenos o tomando una acción incorrecta. **La forma correcta:** pedir identificadores adicionales (email, teléfono, número de orden) para desambiguar con certeza.

> [!warning] Trampa 5 — Confundir vacío de política con violación de política
> Tratar cualquier caso "raro" como si la política ya lo resolviera, o al revés, escalar violaciones de política claras que ya tienen una respuesta documentada. **Por qué falla:** una violación de política tiene respuesta clara (se aplica, no se escala); un vacío o excepción de política (la política simplemente no dice nada sobre ese caso específico) sí requiere juicio humano y se escala.

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
