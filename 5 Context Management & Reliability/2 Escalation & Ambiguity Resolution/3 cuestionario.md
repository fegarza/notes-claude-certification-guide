Cuestionario completo de repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta.

## Los tres triggers válidos de escalación

> [!question]- ¿Cuáles son los tres triggers válidos para escalar a un humano?
> Solicitud explícita del cliente, excepción/vacío de política, e incapacidad genuina de progresar.

> [!question]- Si el cliente dice "quiero hablar con una persona", ¿qué debe hacer el agente antes de escalar?
> Nada — escala de inmediato, sin investigar ni intentar resolver primero.

> [!question]- ¿Cuál es la diferencia entre una violación de política y un vacío de política?
> Una violación tiene una respuesta clara y documentada (se aplica, no se escala); un vacío significa que la política no dice nada sobre ese caso específico, y por eso sí requiere juicio humano.

> [!question]- ¿Qué tipo de fallas técnicas justifican el trigger de "incapacidad de progresar"?
> Herramientas que devuelven errores, falta de acceso a un sistema requerido, o un bug técnico que necesita intervención de ingeniería — después de intentos reales de resolución.

## Los dos antipatrones de escalación

> [!question]- ¿Por qué el `sentiment-based escalation` es un antipatrón?
> Porque la frustración del cliente no correlaciona con la complejidad real del caso — un cliente furioso puede tener un problema trivial.

> [!question]- ¿Por qué son poco confiables los `self-reported confidence scores` del modelo?
> Porque están mal calibrados: el modelo suele estar confiado de más en casos difíciles e inseguro en casos fáciles, justo al revés de lo que se necesitaría.

> [!question]- ¿Qué tienen en común los dos antipatrones de escalación?
> Ambos se sienten intuitivos pero no miden si el caso realmente cae en uno de los tres triggers válidos.

## Frustración vs. solicitud explícita

> [!question]- Un cliente frustrado pero con un problema resoluble, ¿se debe escalar de inmediato?
> No — se debe reconocer la frustración y ofrecer la resolución directamente, dentro de la capacidad del agente.

> [!question]- ¿Cuándo se convierte la frustración de un cliente en un caso de escalación válido?
> Cuando, después de ofrecerle ayuda, el cliente reitera su preferencia por hablar con un humano — en ese punto pasa a ser el trigger de solicitud explícita.

> [!question]- ¿Qué diferencia a "arréglenlo YA" de "quiero hablar con una persona YA" en términos de la respuesta del agente?
> El primero pide una solución (se resuelve); el segundo pide explícitamente escalar (se escala de inmediato).

## Ambigüedad en matches de cliente

> [!question]- ¿Qué debe hacer el agente cuando una búsqueda devuelve varios registros posibles para un mismo cliente?
> Pedir identificadores adicionales (email, teléfono, número de orden) para desambiguar, en vez de elegir por su cuenta.

> [!question]- ¿Por qué es riesgoso elegir heurísticamente (ej. el registro más reciente) entre varios matches?
> Porque puede llevar a actuar sobre la cuenta o el pedido equivocado, exponiendo datos de otra persona o tomando una acción incorrecta.

> [!question]- ¿Qué tienen en común la ambigüedad de identidad y los dos antipatrones de escalación?
> Las tres son situaciones donde una heurística "intuitiva" (adivinar, medir sentimiento, confiar en la autoevaluación del modelo) reemplaza incorrectamente a un criterio explícito y verificable.

## Mejor práctica de implementación

> [!question]- ¿Cuál es la forma más efectiva de calibrar cuándo escalar, según la guía?
> Agregar criterios explícitos de escalación con ejemplos `few-shot` al `system prompt`.

> [!question]- ¿Por qué se recomienda esta práctica antes que construir infraestructura como clasificadores o análisis de sentimiento?
> Porque esas alternativas más complejas tienden a caer en los mismos antipatrones (sentimiento, confianza autorreportada) que ya se identificaron como no confiables.

> [!question]- ¿Qué tipo de ejemplos deben incluir los `few-shot examples` del `system prompt` sobre escalación?
> Casos concretos que demuestren cuándo escalar y cuándo resolver de forma autónoma.
