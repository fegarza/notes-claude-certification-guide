---
tags:
  - claude-cert/dominio-2
  - task-statement/2.1
---

# 3 cuestionario — Tool Interface Design

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## El rol de las descripciones

> [!question]- ¿Por qué se dice que las descripciones de tools son el mecanismo "primario" y no "una ayuda más" para la selección?
> Porque es la **única** señal real que el modelo tiene para decidir qué tool llamar. No hay otra fuente de verdad — si la descripción no diferencia el propósito, el modelo no tiene cómo diferenciarlo.

> [!question]- ¿Qué le falta a una descripción como "Retrieves customer information" para ser production-grade?
> Le faltan 4 de los 5 elementos: formatos de input, ejemplos de queries que resuelve, edge cases/límites, y fronteras explícitas frente a tools similares. Solo cubre (a medias) el "qué hace".

> [!question]- ¿Cuáles son los 5 elementos de una descripción production-grade?
> Propósito sin ambigüedad, inputs esperados (tipos/formatos/restricciones), ejemplos de queries típicas, edge cases y límites, y fronteras explícitas frente a tools similares.

## El problema del misrouting

> [!question]- ¿Qué pasaría si `get_customer` y `lookup_order` tuvieran nombres perfectos pero descripciones mínimas de una sola frase cada una?
> El misrouting seguiría ocurriendo. El nombre ayuda, pero la decisión de selección la hace principalmente el modelo leyendo la descripción — un nombre claro con una descripción pobre no basta para diferenciar cuándo usar una u otra.

> [!question]- ¿Cómo se relaciona la frontera explícita ("Do NOT use for...") con la eliminación del misrouting, más allá de simplemente describir mejor la tool?
> La frontera le dice al modelo explícitamente cuándo preferir la tool vecina en vez de esta. No es solo "más información sobre la tool", es información comparativa que resuelve la ambigüedad entre dos opciones parecidas.

## Diagnóstico: cuántas tools hay

> [!question]- ¿Cuándo expandir descripciones deja de ser el arreglo correcto?
> Cuando el toolkit es grande (aprox. más de 4-5 tools por agente): ahí la selección se degrada por la complejidad de la decisión misma, no por descripciones ambiguas, y reescribir muchas descripciones no resuelve ese problema de fondo.

> [!question]- Un agente tiene 22 tools y selección poco confiable. ¿Por qué reescribir las 22 descripciones probablemente no arregla el problema?
> Porque con un toolkit tan grande el problema no es que dos tools se confundan entre sí por descripciones pobres, sino la sobrecarga de decisión que implica elegir entre tantas opciones. Ese es un problema de tamaño de toolkit (Task Statement 2.3), no de calidad de descripción.

> [!question]- ¿Cuándo usarías expandir descripciones en vez de dividir/consolidar tools por rol?
> Cuando el número de tools es manejable y el síntoma es que el modelo confunde dos (o pocas) tools concretas entre sí, no que esté abrumado por la cantidad total de opciones.

> [!question]- ¿Son los few-shot examples un remedio válido para misselection en algún escenario, según la guía?
> No. La guía los descarta como remedio válido para misselection en cualquier escenario — siempre se consideran una forma de tratar el síntoma (overhead de tokens) sin atacar la causa raíz.

## Tool splitting

> [!question]- ¿Por qué dividir `analyze_document` en tres tools resuelve el problema mejor que solo mejorar su descripción?
> Porque el problema no es que falte texto descriptivo, sino que una sola tool carga varias responsabilidades distintas (extraer, resumir, verificar). Ninguna descripción, por buena que sea, elimina la ambigüedad de qué operación se quiere si la tool sigue haciendo varias cosas a la vez.

> [!question]- ¿Qué característica deben tener las tools resultantes de un splitting para que funcione?
> Cada una debe tener un contrato de input/output definido y un propósito narrow (estrecho) y claramente descrito — no basta con dividir el nombre, hay que dividir la responsabilidad.

## Renombrar tools

> [!question]- ¿Cuál es la diferencia entre arreglar el misrouting con splitting y arreglarlo con renombrar?
> Splitting se usa cuando una tool hace varias cosas distintas y hay que separar responsabilidades. Renombrar se usa cuando dos tools ya tienen responsabilidades distintas pero sus **nombres** son confusamente parecidos — el problema está en la etiqueta, no en el alcance de la tool.

> [!question]- Al renombrar `analyze_content` a `extract_web_results`, ¿qué se modifica y qué se deja igual?
> Se modifica el nombre y la descripción (ahora específica a resultados web). La implementación de la tool no cambia — el arreglo ocurre completamente a nivel de interfaz.

## Interacciones con el system prompt

> [!question]- ¿Cómo se relaciona el wording del system prompt con las descripciones de las tools, incluso después de mejorarlas?
> Una instrucción sensible a palabras clave en el system prompt puede seguir generando asociaciones de tool no intencionadas, "por encima" de lo que dicen las descripciones — pueden competir con la descripción en cada turno, no quedar por debajo de ella.

> [!question]- Ya mejoraste las descripciones de `get_customer` y `lookup_order`, pero el misrouting persiste. ¿Qué deberías revisar antes de seguir tocando las descripciones?
> El system prompt — buscar frases sensibles a palabras clave del tipo "always check customer details before proceeding" que puedan estar creando una asociación no intencionada con una tool específica en cada turno.

> [!question]- ¿Por qué no tiene sentido pensar que el modelo "cacheó" las descripciones anteriores y por eso sigue enrutando mal?
> Porque las descripciones de las tools no se cachean entre llamadas a la API — cada request se evalúa contra las definiciones de tools que se envían con esa misma request. Si sigue fallando, la causa está en otra parte (típicamente el system prompt).

## Síntesis

> [!question]- ¿Cuál es el criterio del examen para elegir entre "expandir descripciones", "few-shot examples", "routing classifier" y "consolidar tools" cuando hay misrouting?
> El examen favorece consistentemente el arreglo de menor esfuerzo y mayor impacto que ataque la causa raíz: mejores descripciones antes que routing classifiers, acceso con alcance definido antes que acceso completo, servidores de la comunidad antes que builds propios. Expandir descripciones es casi siempre el primer paso correcto cuando el toolkit es manejable.
