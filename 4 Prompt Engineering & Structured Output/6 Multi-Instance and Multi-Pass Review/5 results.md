# Resultados — Tutor socrático: Multi-Instance and Multi-Pass Review

> [!info] Sesión evaluada el 2026-10-18
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 87/100**

Dominio sólido del tema: de 14 preguntas, 11 fueron correctas y 3 parcialmente correctas, sin respuestas incorrectas. Identificaste bien que el self-review y el single-pass son problemas **arquitectónicos**, explicaste attention dilution con sus tres síntomas, describiste correctamente Pass 1 y Pass 2, y resolviste la trampa de la ventana de contexto con una analogía propia muy buena. Los puntos parciales siguen un mismo patrón: diste la conclusión correcta ("es arquitectónico") pero sin nombrar el **mecanismo concreto** que la guía usa para justificarla (razonamiento retenido, fallas sistemáticas, mapeo exacto del ruteo).

## Conceptos con buen dominio

- **Sesgo del self-review en la misma sesión**: identificaste que el razonamiento en el historial lleva al modelo a confirmar su salida.
- **Instancia independiente**: describiste una sesión aislada, sin historial de razonamiento, que recibe solo el output y criterios explícitos.
- **Attention dilution y sus 3 síntomas**: nombraste profundidad desigual, bugs que dependen de la posición y contradicciones.
- **Pass 1 (análisis local por archivo)**: un archivo por invocación con criterios enfocados, lo que da profundidad uniforme.
- **Paralelización de Pass 1**: las invocaciones son independientes entre sí.
- **Pass 2 (integración cross-file)**: se ejecuta después de Pass 1, recibe los hallazgos y busca problemas de formato, contrato, patrón e inconsistencias en los hallazgos.
- **Por qué no basta solo con Pass 2 sobre todos los archivos**: reconociste que vuelve el attention dilution.
- **Ventana de contexto más grande (Trampa 3 / Pregunta 12 oficial)**: distinguiste capacidad de almacenamiento de calidad de atención con la analogía del escritorio grande.
- **Calibración**: dataset etiquetado, medir confianza reportada contra precisión real, y loop continuo.
- **Trade-off de costo**: se justifica en sistemas de alto impacto (financieros, pipelines).
- **Arquitectura de producción completa**: ordenaste correctamente las 5 etapas.

## Áreas que necesitan revisión

### Por qué "mejores instrucciones" o extended thinking no arreglan el self-review

- **Qué pasó:** Respondiste que es un problema arquitectónico y no de prompt ni de capacidad, lo cual es correcto, pero no explicaste el mecanismo: ninguna de las dos alternativas saca al revisor del contexto donde está el razonamiento de la generación.
- **Contenido del resumen:**
  > [!note] Lo que evalúa el examen aquí
  > Cuando se presentan opciones para mejorar la calidad de una revisión, la respuesta correcta casi siempre es desplegar una **instancia de modelo separada** — nunca "mejorar las instrucciones dentro de la misma sesión" ni "usar extended thinking durante la generación". Ninguna de esas dos alternativas elimina el sesgo de razonamiento retenido.
- **Sección:** [[1 resumen]] → "1. La limitación del self-review en la misma sesión"
- **Analogía para reforzarlo:** Es como pedirle a un juez que "sea imparcial" en un caso donde él mismo es una de las partes. Puedes darle la instrucción más estricta del mundo, pero sigue sabiendo lo que sabe. La única forma real de tener imparcialidad es cambiar de juez.

### Mapeo exacto del ruteo por confianza

- **Qué pasó:** Mencionaste las dos rutas (revisión humana y desarrollador) y te adelantaste bien al tema de calibración, pero no dijiste qué nivel de confianza va a cada ruta.
- **Contenido del resumen:**
  > - **Hallazgos de alta confianza** → se envían directo a los desarrolladores, sin pasar por revisión humana.
  > - **Hallazgos de baja confianza** → se envían a una cola de revisión humana para validación.
  > - **Calibración del umbral** → se usa un dataset de validación etiquetado (donde ya se conoce la respuesta correcta) para medir qué tan bien la confianza reportada se correlaciona con la precisión real.
- **Sección:** [[1 resumen]] → "4. Ruteo por confianza calibrada"
- **Analogía para reforzarlo:** Es como el triage en urgencias, pero al revés: lo que el sistema tiene claro sale directo a quien lo resuelve, y lo dudoso pasa primero por un médico que lo confirme. El humano se queda con los casos en los que su criterio aporta más.

### Las fallas del single-pass son sistemáticas, no aleatorias

- **Qué pasó:** En la pregunta trampa diste la solución correcta (multi-pass, no un mejor prompt), pero no señalaste lo que el colega malinterpretaba: esas fallas no son ruido aleatorio, son patrones predecibles que salen de la estructura misma del single-pass.
- **Contenido del resumen:**
  > [!warning] Trampa 2 — Un solo pase para revisiones grandes multi-archivo
  > Un single-pass sobre muchos archivos produce tres fallas sistemáticas, no errores aleatorios: profundidad desigual, bugs perdidos (sobre todo a la mitad), y hallazgos contradictorios sobre el mismo patrón en archivos distintos. La solución es arquitectónica (dividir en pasadas), no "usar mejores prompts" ni "un modelo más grande".
- **Sección:** [[1 resumen]] → "Trampas de examen"
- **Analogía para reforzarlo:** Si una báscula siempre marca 2 kg de más, no es mala suerte: está mal calibrada, y pesarte con más cuidado no sirve, hay que arreglar la báscula. Lo aleatorio se resuelve repitiendo; lo sistemático, cambiando el instrumento.

### Matiz menor: Pass 1 no garantiza consistencia entre archivos

- **Qué pasó:** Dijiste que con Pass 1 "las contradicciones ya no serán un problema". Pass 1 elimina la fatiga por posición, pero como cada invocación ve un solo archivo, nadie compara los hallazgos entre sí. Esa revisión de consistencia le toca a Pass 2.
- **Contenido del resumen:**
  > - **Pass 2 — Integración cross-file**: después de que terminan todos los análisis por archivo, una invocación separada recibe todos los hallazgos y revisa específicamente problemas *entre* archivos: datos que fluyen entre módulos en formatos incompatibles, patrones que se contradicen entre archivos, violaciones de contrato de API en los límites entre servicios, e inconsistencias en los propios hallazgos por archivo.
- **Sección:** [[1 resumen]] → "2. Arquitectura de multi-pass review"
- **Analogía para reforzarlo:** Catorce maestros califican cada uno un examen distinto con toda su atención. Nadie se cansa, pero sin un coordinador que compare las calificaciones al final, un maestro podría ser más estricto que otro con el mismo error.

## Siguiente paso recomendado

Ya estás listo para [[4 test]]. Antes, practica justificar cada respuesta con el **mecanismo concreto** y no solo con "es arquitectónico": razonamiento retenido, fallas sistemáticas, y alta confianza → desarrolladores / baja → humanos. En el examen, los distractores también suelen decir "cambia la arquitectura", y lo que los separa de la respuesta correcta es justamente ese porqué.
