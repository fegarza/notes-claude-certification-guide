# Resultados — Tutor socrático: Few-Shot Prompting

> [!info] Sesión evaluada el 2026-09-21
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 74/100**

Dominio sólido del núcleo del tema: sabes cuándo usar few-shot, cuántos ejemplos, por qué el razonamiento es el ingrediente clave, y —especialmente bien— distingues few-shot de otras técnicas (schema nullable, validation/retry loops) cuando la causa raíz es distinta. Las fallas fueron puntuales: dos datos concretos de la fuente (el segundo dominio de "juicio ambiguo", y el detalle de extracción con documentos variados) y una explicación incompleta del mecanismo por el cual few-shot no ataca la causa raíz en el caso de descripciones de herramienta pobres.

## Conceptos con buen dominio

- Los tres disparadores de few-shot — los identificaste completos y en tus propias palabras.
- El rango 2-4 ejemplos y su justificación (menos no establece patrón, más gasta tokens sin señal extra).
- El razonamiento como ingrediente clave y el mecanismo generalización vs. memorización — explicado con precisión ("el modelo memoriza en vez de aprender el patrón y aplicarlo de forma genérica").
- Enfocar ejemplos en los casos que fallan, no en los que ya funcionan.
- Distinción few-shot vs. schema nullable (alucinación de valores) — correcta.
- Distinción few-shot vs. validation/retry loops (errores de cálculo) — correcta.
- Trampa del umbral de confianza mal calibrado — correcta y completa.

## Áreas que necesitan revisión

### El segundo dominio de "juicio ambiguo" (branch coverage)

- **Qué pasó:** No recordaste el segundo ejemplo de dominio típico donde aplica el disparador de juicio ambiguo (el primero, elección de herramienta, sí lo mencionaste en la Pregunta 2).
- **Contenido del resumen:** 
> Dos dominios típicos donde esto se aplica: elegir la herramienta correcta ante una solicitud ambigua, y decidir si una brecha de cobertura de tests a nivel de rama merece ser señalada.
- **Sección:** [[1 resumen]] → "3. El razonamiento enseña generalización, no memorización"
- **Analogía para reforzarlo:** es como decidir si una mancha pequeña en una prenda "cuenta" como sucia o no — no hay una regla binaria, depende del criterio aprendido de casos similares. La cobertura de tests por rama es igual: un 62% de cobertura puede ser aceptable en un módulo y preocupante en otro, y solo ejemplos con razonamiento enseñan ese criterio.

### Por qué few-shot no ataca la causa raíz cuando el problema es descripción de herramienta pobre

- **Qué pasó:** Identificaste correctamente el primer paso (mejorar la descripción), pero al explicar el *porqué* few-shot no es suficiente, respondiste apelando a un "orden general" de agotar instrucciones primero — que es el razonamiento de otro trigger (inconsistencia de formato), no el mecanismo específico de este caso.
- **Contenido del resumen:**
> Si el modelo elige la herramienta equivocada porque dos herramientas tienen descripciones mínimas y similares, agregar ejemplos few-shot de enrutamiento *funciona parcialmente* pero no ataca la causa raíz y suma tokens de forma innecesaria. El primer paso correcto en ese caso es enriquecer la descripción de cada herramienta (formatos de entrada, casos límite, cuándo usar una vs. la otra) — few-shot es una técnica secundaria, no el primer paso, cuando el síntoma es de enrutamiento por descripciones pobres.
- **Sección:** [[1 resumen]] → "Trampas de examen" (Trampa 4)
- **Analogía para reforzarlo:** es como poner letreros con ejemplos de "aquí sí, aquí no" en dos puertas casi idénticas sin nombre — ayuda con los casos que anticipaste, pero cualquier visitante con una duda nueva sigue perdido, porque el problema real es que las puertas nunca tuvieron un letrero claro que las distinga.

### Extracción con documentos variados — el "qué" y el segundo beneficio

- **Qué pasó:** No recordaste que la solución específica es mostrar ejemplos con estructuras de documento variadas, ni el beneficio secundario (reducción de falsos positivos).
- **Contenido del resumen:**
> Mostrar ejemplos few-shot con **estructuras de documento variadas** (citas en línea vs. bibliografías, secciones de metodología vs. detalles incrustados en el texto) enseña al modelo a reconocer la información aunque no venga en el formato "canónico".
> Esto también reduce falsos positivos: mostrar ejemplos que distinguen **patrones de código aceptables** de **problemas genuinos** ayuda a generalizar el criterio de "esto sí es un hallazgo" sin sacrificar precisión.
- **Sección:** [[1 resumen]] → "4. Reducir alucinación en extracción con documentos variados"
- **Analogía para reforzarlo:** es como enseñarle a alguien a reconocer "efectivo" en un ticket sin importar si dice "$50", "cincuenta pesos" o "50.00 MXN" — si solo le muestras un formato, fallará con los otros dos; mostrarle los tres formatos variados enseña a reconocer el concepto detrás de la notación.

## Siguiente paso recomendado

Repasa específicamente la sección "3. El razonamiento enseña generalización, no memorización" (el segundo dominio de juicio ambiguo) y "4. Reducir alucinación en extracción con documentos variados" en [[1 resumen]], y vuelve a leer la Trampa 4 completa. Con esos dos puntos reforzados, estás listo para intentar [[4 test]].
