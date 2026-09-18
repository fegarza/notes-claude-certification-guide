---
tags:
  - claude-cert/dominio-2
  - task-statement/2.1
---

# 2.1 — Tool Interface Design

## Explícamelo como si tuviera 5 años

Imagina una caja de herramientas con dos cajones. Uno dice "cosas" y el otro dice "otras cosas". Si le pides a alguien que nunca ha visto la caja "tráeme el destornillador", esa persona no tiene forma de saber en qué cajón está — las etiquetas no le dicen nada.

Ahora imagina que los cajones dicen: "Herramientas de apriete: destornilladores y llaves, para tornillos y tuercas visibles. NO uses este cajón para cortar" y "Herramientas de corte: tijeras y cutters, para cortar cable o cartón. NO uses este cajón para apretar tornillos". Ahora esa misma persona, sin haber visto la caja antes, elige bien a la primera.

Un modelo como Claude es exactamente esa persona que "nunca ha visto la caja": lo único que tiene para decidir qué herramienta (tool) llamar es el texto de la descripción. Si la descripción es pobre, el modelo adivina — y adivina mal.

## Argumento central

> [!note] Idea central
> Las **descripciones de las tools son el mecanismo primario** que un LLM usa para decidir qué herramienta invocar. No son metadata secundaria ni un detalle cosmético: son la única señal de selección que tiene el modelo. Descripciones mínimas ("Retrieves customer information") son la causa raíz de que dos tools con propósitos parecidos se confundan entre sí (*misrouting*).

## Conclusiones

### 1. Los 5 elementos de una descripción production-grade

Una descripción de calidad para producción debe incluir:

1. **Qué hace la tool** — su propósito, dicho sin ambigüedad.
2. **Qué inputs espera** — tipos de datos, formatos, restricciones, y qué campos son obligatorios vs. opcionales.
3. **Ejemplos de queries que resuelve bien** — casos de uso concretos que anclan el entendimiento del modelo.
4. **Edge cases y límites** — qué NO hace la tool, y qué pasa cuando el input se sale del rango esperado.
5. **Fronteras explícitas** — cuándo usar ESTA tool en vez de otra similar del mismo toolkit.

> [!tip] Analogía mental
> Son los mismos 5 datos que le darías a alguien nuevo en el trabajo para que use una máquina sin supervisión: para qué sirve, qué le metes, un ejemplo de uso típico, qué no debe intentar hacer con ella, y cuándo usar la máquina de al lado en su lugar.

### 2. El problema del misrouting

Dos tools con descripciones mínimas o casi idénticas generan confusión de selección. El caso clásico de la guía: `get_customer` ("Retrieves customer information") y `lookup_order` ("Retrieves order details"). Con descripciones así de pobres, el agente enruta mal una consulta como "check my order #12345" — la manda a `get_customer` en vez de `lookup_order`.

**Evidencia — antes y después:**

Descripción mínima (causa misrouting):
- `get_customer`: "Retrieves customer information"
- `lookup_order`: "Retrieves order details"

Descripción production-grade (selección confiable): cada una indica qué identificadores acepta (email, teléfono, ID vs. número de orden #NNNNN o tracking ID), qué devuelve, cuándo usarla, y explícitamente cuándo NO usarla ("Do NOT use for order-specific queries — use lookup_order for those").

> [!note] Por qué funciona
> La versión larga no solo describe mejor la tool — le da al modelo la frontera explícita frente a la tool vecina. Eso es lo que elimina la ambigüedad, no simplemente "más texto".

### 3. Cuenta las tools antes de elegir el remedio

Expandir descripciones **solo es el arreglo correcto cuando el agente tiene un número manejable de tools** y simplemente no logra distinguir dos de ellas entre sí.

- Por encima de aproximadamente **4-5 tools por agente**, la selección se degrada solo por la complejidad de la decisión — reescribir 22 descripciones no soluciona ese problema de fondo.
- Ahí el remedio correcto es otro: dividir por rol o consolidar variantes de un mismo trabajo en una tool parametrizada (ver Task Statement 2.3 sobre sobrecarga de tools).
- Los few-shot examples **no son un remedio válido** para misselection en ningún escenario, y un routing classifier está mal calibrado para toolkits grandes.

> [!warning] Diagnóstico antes que receta
> El examen espera que primero diagnostiques **cuál enfermedad** estás viendo (¿pocas tools ambiguas, o demasiadas tools?) antes de recetar el remedio. Aplicar el arreglo de "pocas tools" a un problema de "demasiadas tools" dejará el problema intacto.

### 4. Tool splitting (dividir tools genéricas)

Una tool genérica con responsabilidades amplias crea ambigüedad porque obliga al modelo a adivinar qué operación se quiere.

- **Antes:** `analyze_document`: "Analyses a document and returns results" — una sola tool para todo.
- **Después:** se divide en tools de propósito único, cada una con su propio contrato de input/output:
  - `extract_data_points` — extrae campos estructurados (fechas, montos, nombres).
  - `summarize_content` — produce un resumen conciso.
  - `verify_claim_against_source` — verifica si una afirmación está respaldada por el documento.

Cada tool resultante hace un trabajo estrecho y claramente descrito, así el modelo elige según lo que el usuario realmente necesita.

### 5. Renombrar tools para dar claridad

Cuando dos tools tienen nombres confusamente parecidos, **renombrar** resuelve el solapamiento a nivel de interfaz — sin tocar la implementación. Ejemplo: renombrar `analyze_content` a `extract_web_results` y darle una descripción específica de resultados web hace que el propósito sea inequívoco.

### 6. Interacciones con el system prompt

Instrucciones sensibles a palabras clave en el system prompt pueden crear asociaciones de tool no intencionadas que **anulan silenciosamente** descripciones bien escritas.

- Ejemplo: si el system prompt dice "always check customer details before proceeding", el modelo puede enrutar cualquier consulta relacionada con clientes hacia `get_customer`, sin importar lo bien escritas que estén las descripciones.
- Por eso, **después de actualizar las descripciones de las tools, hay que releer el system prompt buscando conflictos**. Es un modo de falla sutil, y el examen lo evalúa explícitamente.

## En una frase

> Las descripciones de las tools son la única señal real que el modelo usa para elegir entre ellas — y el arreglo de más bajo esfuerzo y más alto impacto casi siempre es escribirlas mejor, no añadir infraestructura alrededor.

## Trampas de examen

> [!warning] Trampa 1 — Few-shot examples para arreglar misrouting
> Elegir agregar 5-8 ejemplos few-shot al system prompt para corregir el enrutamiento en vez de arreglar las descripciones. **Por qué es un error:** los few-shot examples añaden overhead de tokens en cada request sin atacar la causa raíz — el modelo sigue sin tener forma de diferenciar las tools. Tratas el síntoma, no la enfermedad.

> [!warning] Trampa 2 — Routing classifier como primer paso
> Implementar una capa de routing que analiza el input y preselecciona la tool antes de dejar que el modelo decida. **Por qué es un error:** está sobre-ingenierizado como primera respuesta — evita la comprensión de lenguaje natural del LLM y añade infraestructura que el examen no considera proporcionada para un primer paso.

> [!warning] Trampa 3 — Consolidar tools como primer paso
> Fusionar tools similares en una sola de inmediato. **Por qué es un error:** consolidar es una decisión arquitectónica válida a largo plazo, pero cuesta mucho más esfuerzo que expandir descripciones. El examen favorece arreglos de bajo esfuerzo y alto impacto como primer paso.

> [!warning] Trampa 4 — Ignorar el wording del system prompt
> Dar por resuelto el misrouting solo con mejorar las descripciones, sin revisar el system prompt. **Por qué es un error:** instrucciones sensibles a palabras clave pueden seguir generando asociaciones de tool no intencionadas incluso después de que las descripciones ya están bien escritas.

> [!info] Pregunta de muestra oficial (examguide.pdf, Sample Question 2)
> El escenario `get_customer` / `lookup_order` con descripciones mínimas, donde el agente enruta mal "check my order #12345", es literalmente la Question 2 de la sección "9. Sample Questions" del Exam Guide oficial. La respuesta correcta es expandir las descripciones — no few-shot examples, no routing layer, no consolidación.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
