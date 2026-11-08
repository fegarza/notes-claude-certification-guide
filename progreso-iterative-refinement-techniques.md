# Progreso: Iterative Refinement Techniques (Task Statement 3.5)

- **Objetivo**: examen CCAR-F — preguntas tipo escenario en inglés. En 5 mocks (60 preguntas c/u, ~53+ correctas) este fue el tema con más fallos (5).
- **Nivel inicial**: resultados socráticos 2026-09-29: 88/100. Mocks: 3 de 5 fallos en batch vs sequential, 2 en alcance de test-driven iteration (pegar failures crudos, performance como test).
- **Inicio**: 2026-11-01 · **Última sesión**: 2026-11-01
- **Fase actual**: Terminado (dominado 2026-11-02)
- **Resumen del estudiante (bloque 1-6)**: bien en general; se corrigió la condición "tú sabes lo que quieres" (examples), "dominio desconocido" ≠ "no sé implementar" (interview) y la señal de "no interview" ≠ "ya existe infraestructura"
- **Diagnóstico 2026-11-01**: D1 (formatPrice + plantilla + typo en el mismo archivo) ✗ — juntó todo "porque está en el mismo scope". D2 (pegar el failure crudo frente a la paráfrasis) ✓

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Diagnosticar qué le falta a Claude (hueco en ti / interpretación / verificación objetiva / entrega del feedback) | dominado | – | tabla de 4 huecos + analogía del mecánico |
| 2 | Concrete I/O examples: 2-3 pares, 4 pasos, edge case → agregar ejemplo | dominado | – | caso de recursos anidados ✓ |
| 3 | Test-driven iteration: tests primero + pegar failures tal cual | dominado | – | D2 ✓ |
| 4 | Test case = ejemplo concreto ejecutable y repetible | dominado | 2 | ✓ migración null/empty ("fix and protect", con precisión pedida); | analogía pizarrón vs máquina de calificar (en sep se usó la del termómetro); ✓ "detecta el error"; ✗ Feynman (se quedó solo con la cara de ejemplo). Mnemotecnia ER + foto vs monitor cardíaco |
| 5 | Performance requirement → test que falla | dominado | 2 | ✓ repaso ETL (opción B frente a la A sin árbitro); | ✗ evaluación final P6 (le faltó "junto con los funcionales"); ✓ repaso: thumbnails en blanco; | analogía de la báscula; ✓ "¿qué pasa si solo corre el timing?"; ✓ p95 del SLA (hueco nombrado impreciso, corregido) |
| 6 | Interview vs examples; no usar interview si los problemas son conocidos | dominado | 2 | falló en mock y en sesión (pagos); ✓ export de datos de pacientes (interview correcto); analogías del médico con laboratorio y del GPS; mnemotecnia A-A; ✓ inventarios (a)/(b) + experto → no interview (aclarado: plan mode es otra decisión, 3.4) |
| 7 | Criterio de interacción frente a falsas señales | dominado | 2 | falló en mock + D1; prueba "¿arreglar A solo deja mal a B?"; analogía pared/toma vs pintar/llave; ✓ API/Greeting/CSS/429; ✓ DATABASE_URL |
| 8 | Batch: un mensaje detallado que señale la dependencia | dominado | 2 | ✓ repaso getUserAge (3C); ✓ Order.total en cents (opción C); | ✗ evaluación final P5 (redacción del mensaje); | falló en mock; analogía del sastre; ✓ detecta el error de "ordenar" (con precisión pedida); ✓ DATABASE_URL |
| 9 | Secuencial: uno por mensaje, aun con presión de tiempo o round trips | dominado | 2 | ✓ detecta el error "tests + mandar juntos"; ✓ /orders mismo endpoint; falló en mock; analogía de la nota con 4 mandados; ✓ DATABASE_URL; ✗ CLI (eligió tests + batch). Mnemotecnia I-I; ángulo nuevo: "how should they send" = fila de entrega |
| 10 | Distractores típicos: prosa precisa, plan mode, renombrar tests, tests locales | dominado | 2 | ✓ P7, P8 y P10 de la evaluación final; ✓ p95 del SLA; ✗ CLI (tests + batch); ✓ migración null (precisó el descarte de D: Trampa 4). Truco: nombrar el hueco antes de leer las opciones |

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-11-01 | 4 Test case = ejemplo | "Es programático/determinístico"; con 2 pistas, "ni idea" | Un test case ES un concrete example, pero ejecutable (se verifica solo: Expected X, got Y) y repetible (corre en cada iteración y detecta regresiones). Explicado con la analogía pizarrón vs máquina de calificar | No |
| 2026-11-01 | 4 Test case = ejemplo (2.º fallo) | Feynman: "los test cases son solo ejemplos para darnos a entender, no iteramos sobre ellos" | Un test tiene dos caras: ejemplo (comunica) + ejecutable y repetible (corre en cada iteración, es el motor del ciclo). Mnemotecnia ER | No |
| 2026-11-01 | 9 Secuencial / 10 Distractores | CLI con 3 fixes independientes + "Claude dijo all done y no era cierto" → "falta verificación" → tests para los 3 y mandar el output en un solo mensaje (opción D) | El enunciado pregunta cómo *enviar* → hueco de entrega. Independientes → uno por mensaje verificando cada uno. D sigue siendo batch; un test para un typo no vale el tiempo. "Claude dijo listo y no era cierto" = evidencia de que el batch falla. Sobrecorrección: después de varias respuestas de tests, aplicó tests a todo | No |
| 2026-11-02 | 5 Performance → test | Job de 25 min vs límite de 5: dijo "timing test" pero no mencionó correrlo junto con los tests funcionales, aun cuando se le preguntó directamente | Timing test + functional tests en cada intento; si no, Claude puede cambiar correctitud por velocidad | No |
| 2026-11-02 | 8 Batch (redactar el mensaje) | Juntó bien los fixes 2 y 3, pero invirtió el fix (pidió redondeo por línea) y dejó la dependencia ambigua ("considerando este cambio dentro de…") | Un mensaje batch = cada fix claro y en la dirección correcta + la dependencia dicha explícitamente + qué debe quedar coherente al final | No |
| 2026-11-01 | 6 Interview | Dominio desconocido (pagos) + bug ya reportado con failure exacto → interview "para no adivinar" | Interview es ANTES de implementar, para descubrir consideraciones. Con un failure concreto en la mano → test-driven: pegarlo tal cual. Analogía del GPS (planear antes vs corregir la salida 14) | No |
| 2026-11-01 (mock) | Batch vs sequential | Fixes que interactúan (schema renombra campos que usan date/currency) → arreglar el schema primero y luego los otros uno por uno | Interactúan → **un solo mensaje detallado** indicando la dependencia. Ordenarlos sigue siendo tratarlos como trabajos separados | No |
| 2026-11-01 (mock) | Batch vs sequential | 4 problemas independientes → un mensaje numerado con "confirma cada uno" (para ahorrar round trips) | Independientes → **secuencial**, un problema por mensaje, verificando cada uno. Un checklist no arregla un batch que se aplica a medias | No |
| 2026-11-01 (mock) | Criterio de interacción | "Mismo layout" o "toca todas las filas" = interactúa | Interactuar = arreglar A cambia aquello sobre lo que actúa B (A deja obsoleto o roto el fix de B). Estar cerca, ser pequeño o afectar muchas filas no cuenta | No |
| 2026-11-01 (mock) | Test-driven iteration | Feedback parafraseado que falla 2 veces → interview pattern | Pegar los failing assertions **tal cual** (expected/actual/step). Funciona aunque Claude no pueda correr los tests | No |
| 2026-11-01 (mock) | Test-driven iteration (performance) | Un requisito de velocidad que falla 2 veces → plan mode | Un test de timing que falle si excede el límite, corriendo junto con los tests funcionales. Los performance requirements son parte de test-driven iteration | No |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| 4 Test case = ejemplo | "El test es un ejemplo en la sala de ER: Ejecutable y Repetible" (foto vs monitor cardíaco) | Acrónimo + imagen | por confirmar |
| 5 Performance | "Cronómetro + Árbitro": timing test + functional tests, siempre juntos (carrera sin árbitro = cortar por el pasto) | Imagen | ✓ thumbnails en blanco |
| 8 Batch (redacción) | "3C": Cada fix claro (dirección correcta), Conexión dicha, Coherencia final (orden de trabajo para un contratista que no puede llamarte) | Acrónimo | por confirmar |
| 9 Secuencial | "I-I: Independientes → Individuales"; si la pregunta es "how should they send", aplicar primero I-I | Acrónimo (pareja de A-A) | por confirmar |
| 6 Interview | "A-A": Antes de implementar y A ciegas en el dominio. Si ya tienes el failure en la mano → pégalo | Acrónimo | por confirmar |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| 2026-11-02 | Repaso 1 (mini-cuestionario) | 4/4 | — (✓ batch con 3C, ✓ idioma vs memory leak, ✓ ETL Cronómetro+Árbitro, ✓ geoespacial → interview antes que tests) |
| 2026-11-02 | Evaluación final | 8/10 | P5 (8: redacción del mensaje batch), P6 (5: faltó correr el timing test con los funcionales) |

## Resumen final y plan de repaso

**Estado: DOMINADO (2026-11-02).** Se cumplieron los 3 criterios: mini-cuestionario final 4/4; cada concepto fallado con 2 aciertos seguidos; resumen final validado (necesitó 3 rondas de precisión en la condición de interview).

**Conceptos dominados**: los 10 del mapa.

**En qué batalló y cómo se resolvió**:
- **Criterio de interacción (7-9)**: era su hueco principal en los mocks (cercanía = interacción; ordenar en vez de batch; batch con checklist). Se resolvió con la prueba "¿arreglar A solo deja mal a B?" y con las falsas señales (mismo archivo, endpoint, área o scope; presión de tiempo).
- **Redacción del mensaje batch (8)**: invirtió el fix y dejó la dependencia ambigua → mnemotecnia 3C.
- **Performance (5)**: olvidaba correr los functional tests junto al timing test → mnemotecnia Cronómetro + Árbitro.
- **Test case = ejemplo (4)**: se quedaba con una sola cara → mnemotecnia ER.
- **Interview (6)**: tiende a definirlo como "no sé cómo implementarlo" en vez de "desconozco el dominio". Sigue siendo su punto más frágil al explicarlo, aunque en escenarios lo elige bien.
- **Sobrecorrección**: después de varias respuestas de "tests", eligió tests + batch para fixes independientes (CLI) → regla: "how should they *send*" = fila de entrega, aplicar primero I-I.

**Mnemotecnias útiles**: A-A (Interview: Antes de implementar y A ciegas en el dominio), I-I (Independientes → Individuales), ER (el test es un ejemplo Ejecutable y Repetible), Cronómetro + Árbitro, 3C (Cada fix claro, Conexión dicha, Coherencia final). Truco general: **nombrar el hueco antes de leer las opciones**.

**Plan de repaso espaciado**:
| Fecha | Qué repasar |
|---|---|
| 2026-11-03 (1 día) | Escribir de memoria A-A, I-I, ER, Cronómetro + Árbitro y 3C. Explicar interview en una frase con sus dos condiciones |
| 2026-11-05 (3 días) | Hacer `4 test.md` de 3.5. En cada pregunta, nombrar el hueco antes de leer las opciones |
| 2026-11-09 (7 días) | Redactar 2 mensajes batch nuevos con 3C + 3 ejercicios de "¿interactúan?" con falsas señales |
| 2026-11-16 (14 días) | Mock completo y revisar específicamente las preguntas de 3.5 |
| 2026-12-02 (30 días) | Repaso rápido de `1 resumen.md` (Trampas de examen) + retomar este archivo si hubo fallos |

