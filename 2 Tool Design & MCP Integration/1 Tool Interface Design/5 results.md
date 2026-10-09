# Resultados — Tutor socrático: 2.1 Tool Interface Design

> [!info] Sesión evaluada el 2026-10-07
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 76/100**

Se respondieron 16 preguntas: 10 correctas (varias con matices faltantes), 5 parcialmente correctas y 1 incorrecta. Las reglas "qué hacer" del tema están claras: las descripciones son la señal primaria, el misrouting nace de descripciones pobres, se usa tool splitting o renombrado según el caso, y hay que revisar el system prompt después de actualizar descripciones. Lo débil es el **porqué** detrás de algunas reglas: la razón exacta por la que se descarta cada distractor, y que el criterio para elegir el remedio es el **número de tools** y no la similitud funcional. También faltaron algunos detalles concretos de la fuente, como los identificadores de input y el contrato de input/output de cada tool tras un split.

## Conceptos con buen dominio

- **Argumento central** — Identificó sin dudar que la descripción de la tool es el único mecanismo que tiene el modelo para decidir qué tool llamar.
- **Misrouting y su causa raíz** — Nombró el problema y lo atribuyó correctamente a descripciones mínimas o casi idénticas.
- **Los 5 elementos production-grade** — Enumeró los cinco: qué hace, inputs, ejemplos, edge cases y fronteras.
- **Remedio para toolkits grandes** — Con 22 tools, descartó reescribir descripciones y propuso dividir por rol o consolidar en una tool parametrizada.
- **Routing classifier** — Lo reconoció como complejidad innecesaria para un primer paso.
- **Tool splitting** — Diagnosticó correctamente `analyze_document` como tool genérica que hay que dividir.
- **Renombrar tools** — Eligió el renombrado como técnica ante nombres confusos.
- **Interacción con el system prompt** — Entendió que las instrucciones sensibles a palabras clave pueden anular descripciones bien escritas, y que hay que revisar el prompt después de actualizar las descripciones.
- **Sample Question 2** — Eligió B (expandir descripciones) con el principio de bajo esfuerzo y causa raíz.

## Áreas que necesitan revisión

### Trampa 3 — Consolidar como primer paso, y el diagnóstico por número de tools

- **Qué pasó:** Rechazó consolidar `get_customer` / `lookup_order` argumentando que crearía una tool genérica ambigua. Esa razón pertenece a tool splitting, no a esta trampa. Luego, al comparar este caso con el de 22 tools, explicó la diferencia por la similitud funcional de las tools, no por su número.
- **Contenido del resumen:**
  > [!warning] Trampa 3 — Consolidar tools como primer paso
  > Fusionar tools similares en una sola de inmediato. **Por qué es un error:** consolidar es una decisión arquitectónica válida a largo plazo, pero cuesta mucho más esfuerzo que expandir descripciones. El examen favorece arreglos de bajo esfuerzo y alto impacto como primer paso.

  > Expandir descripciones **solo es el arreglo correcto cuando el agente tiene un número manejable de tools** y simplemente no logra distinguir dos de ellas entre sí.
  > - Por encima de aproximadamente **4-5 tools por agente**, la selección se degrada solo por la complejidad de la decisión — reescribir 22 descripciones no soluciona ese problema de fondo.

  > [!warning] Diagnóstico antes que receta
  > El examen espera que primero diagnostiques **cuál enfermedad** estás viendo (¿pocas tools ambiguas, o demasiadas tools?) antes de recetar el remedio.
- **Sección:** [[1 resumen]] → "3. Cuenta las tools antes de elegir el remedio" y "Trampas de examen"
- **Analogía para reforzarlo:** Si tus dos llaves de casa se confunden, les pones etiquetas de colores (barato, inmediato). Remodelar la puerta para que use una sola llave es válido, pero no es lo primero que haces. Ahora bien, si tienes 22 llaves en el llavero, ya no basta con etiquetarlas: tienes que repartirlas en llaveros distintos o cambiar a una llave maestra. Lo que decide qué hacer es **cuántas llaves hay**, no lo parecidas que sean.

### Por qué fallan los few-shot examples

- **Qué pasó:** Afirmó correctamente que los few-shot nunca son la opción, pero no explicó por qué.
- **Contenido del resumen:**
  > [!warning] Trampa 1 — Few-shot examples para arreglar misrouting
  > Elegir agregar 5-8 ejemplos few-shot al system prompt para corregir el enrutamiento en vez de arreglar las descripciones. **Por qué es un error:** los few-shot examples añaden overhead de tokens en cada request sin atacar la causa raíz — el modelo sigue sin tener forma de diferenciar las tools. Tratas el síntoma, no la enfermedad.
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** Es como pegar en la entrada de una oficina una lista de "si vienes por X, ve a la puerta 3". Tienes que leérsela a cada visitante (costo cada vez), y las puertas siguen sin letrero. El arreglo de verdad es ponerle a cada puerta un letrero claro.

### Fronteras explícitas: qué elimina la ambigüedad

- **Qué pasó:** Dijo que la versión production-grade funciona porque da "información necesaria", sin identificar que lo decisivo es la frontera explícita frente a la tool vecina. Antes había interpretado "fronteras" como límites del input.
- **Contenido del resumen:**
  > 5. **Fronteras explícitas** — cuándo usar ESTA tool en vez de otra similar del mismo toolkit.

  > [!note] Por qué funciona
  > La versión larga no solo describe mejor la tool — le da al modelo la frontera explícita frente a la tool vecina. Eso es lo que elimina la ambigüedad, no simplemente "más texto".
- **Sección:** [[1 resumen]] → "1. Los 5 elementos de una descripción production-grade" y "2. El problema del misrouting"
- **Analogía para reforzarlo:** Dos ventanillas de un banco: "Depósitos" y "Retiros". Si un letrero dice además "¿Vienes a sacar dinero? Ve a la ventanilla de al lado", nadie se equivoca. Lo que ayuda no es un letrero más largo: es que el letrero señale a la otra ventanilla.

### Identificadores de input que diferencian las tools

- **Qué pasó:** Entendió que los inputs distintos ayudan a diferenciar, pero no nombró los identificadores concretos de cada tool.
- **Contenido del resumen:**
  > Descripción production-grade (selección confiable): cada una indica qué identificadores acepta (email, teléfono, ID vs. número de orden #NNNNN o tracking ID), qué devuelve, cuándo usarla, y explícitamente cuándo NO usarla ("Do NOT use for order-specific queries — use lookup_order for those").
- **Sección:** [[1 resumen]] → "2. El problema del misrouting"
- **Analogía para reforzarlo:** En el correo, un paquete con código postal va a un buzón y uno con número de guía a otro. El cartero no necesita leer el contenido: la forma del identificador ya dice adónde va. "#12345" tiene forma de número de orden.

### Contrato de input/output al hacer tool splitting

- **Qué pasó:** Respondió que cada tool resultante necesita una descripción production-grade, pero omitió el propósito único y el contrato propio de input/output.
- **Contenido del resumen:**
  > - **Después:** se divide en tools de propósito único, cada una con su propio contrato de input/output:
  >   - `extract_data_points` — extrae campos estructurados (fechas, montos, nombres).
  >   - `summarize_content` — produce un resumen conciso.
  >   - `verify_claim_against_source` — verifica si una afirmación está respaldada por el documento.
  >
  > Cada tool resultante hace un trabajo estrecho y claramente descrito, así el modelo elige según lo que el usuario realmente necesita.
- **Sección:** [[1 resumen]] → "4. Tool splitting (dividir tools genéricas)"
- **Analogía para reforzarlo:** En una cocina, la licuadora, el horno y la báscula reciben cosas distintas y entregan cosas distintas (jugo, pan horneado, un número). No basta con ponerle un buen nombre a cada aparato: cada uno tiene su propia entrada y su propia salida.

## Siguiente paso recomendado

Repasa en [[1 resumen]] la sección "3. Cuenta las tools antes de elegir el remedio" y las Trampas 1 y 3, centrándote en el **porqué** de cada distractor y no solo en la regla. Después ya estás listo para [[4 test]].
