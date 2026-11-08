# Progreso: Task Decomposition Strategies (CCAR-F Task Statement 1.6)

- **Objetivo**: examen CCAR-F — quiere entender a fondo fixed pipeline vs dynamic decomposition
- **Nivel inicial**: sesión socrática previa 78/100 (2026-09-19, ver `5 results.md`). Conoce el criterio (pasos conocidos vs emergentes) pero le cuesta aplicarlo y justificarlo en escenarios.
- **Inicio**: 2026-10-25 · **Última sesión**: 2026-10-25
- **Fase actual**: 3 Repaso

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Fixed sequential pipeline (prompt chaining): definición exacta + caso límite del salto condicional | dominado | 2 | Analogía tren sobre rieles; datos varían ≠ pasos varían. Caso spam-skip OK (corregido: saltar pasos no rompe runtime acotado) |
| 2 | Dynamic adaptive decomposition: definición, planes que generan subtareas según hallazgos | dominado | 2 | Analogía GPS; médico; MasterChef caja misteriosa. Falló 500s, acertó ventas, falló 'ramas predefinidas' (lo llamó fixed disfrazado y luego combinación) |
| 3 | Criterio de selección + leer señales del enunciado (restricciones de tiempo, diffs, consistencia) e ignorar propuestas-distractor | dominado | 2 | Analogía inspector de fábrica + tabla de señales. Casos tickets/fuga y contratos/auditores OK. "Complejo ≠ pasos desconocidos" |
| 4 | Patrón híbrido: pipeline fijo marca → workflow dynamic separado investiga | dominado | 2 | Casos tickets/fuga y PR cripto OK |
| 5 | Dynamic en práctica: mapear estructura → identificar alto impacto → plan priorizado que se adapta | en curso (combinación con 8) | 0 | Falló sept y caso 200 módulos alfabético (vago). Analogía bodega + tabla de consecuencias + mnemotecnia pirata. Caso migración TS OK |
| 6 | Attention dilution: definición, 3 síntomas, inconsistencia = atención no capacidad | dominado | 2 | Analogía cajero + tabla consistente vs a veces. Caso race condition OK |
| 7 | Arreglos falsos: modelo más grande / más contexto / mejor prompt | dominado | – | MCQ 18 archivos OK |
| 8 | Multi-pass: pasadas locales por ítem + pasada de integración cruzada (para qué sirve cada capa) | dominado | – | Caso calculateTax OK; matiz: resúmenes locales deben incluir interfaces/firmas |
| 9 | Batching: qué sí logra (menos dilución dentro del lote) y qué le falta (cruce entre lotes) | dominado | 2 | Falló mitad positiva en sept |

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-09-19 | Definición de pipeline fijo | Un salto condicional ("si error, salta al Paso 4") sigue siendo pipeline fijo porque no se agregan pasos | Si la secuencia cambia según resultados intermedios, ya no es fija | Sí (2026-10-25) |
| 2026-09-19 | Fases de decomposición dinámica | Saltarse el mapeo = "no sabe por dónde empezar" (vago) | Sin mapear, el plan priorizado se basa en suposiciones no verificadas sobre dependencias/impacto | Sí (2026-10-25) |
| 2026-09-19 | Dilución de atención | Solo describió degradación general | La inconsistencia (mismo patrón marcado y aprobado) prueba que el modelo SÍ sabe — es atención, no comprensión | Sí (2026-10-25) |
| 2026-09-19 | Batching | Solo dijo que falta integración cruzada | Batching SÍ reduce dilución dentro de cada lote, pero sin pasada cruzada pierde problemas entre lotes | Sí (2026-10-25) |
| 2026-10-25 | Diagnóstico 1 (facturas) | Dio el criterio en abstracto pero no supo aplicarlo/justificarlo | Pasos fijos (12 campos → esquema → CSV) ⇒ fixed pipeline; consistencia > adaptabilidad | Sí |
| 2026-10-25 | Attention dilution (diagnóstico 2) | Dos veces describió el síntoma ("a veces sí, a veces no", "menos enfoque") sin la inferencia | Detectarlo en un archivo prueba que el modelo SÍ sabe; fallo consistente = capacidad, fallo intermitente = atención | Sí |
| 2026-10-25 | Dynamic decomposition (caso errores 500) | Diseñó un flujo de reintentos con backoff (tratar el 500 como obstáculo para consumir el servicio) con ramas todas predefinidas | Dynamic = investigar la causa; el agente genera subtareas NUEVAS a partir de hallazgos que no podían preverse. Un árbol de decisión escrito de antemano no es un plan adaptativo | Sí |
| 2026-10-25 | Dynamic en práctica (legacy 200 módulos alfabético) | "No reconocemos el problema / no sabemos por dónde empezar / no hay prioridad" — vago otra vez, repite el error de sept | Sin mapear: plan basado en suposiciones → dependencias rompen el orden (módulo A depende de uno que va después), trabajo en código muerto. Sin alto impacto: esfuerzo parejo, lo crítico (pagos) espera semanas. Y el plan rígido no se adapta | Sí (TS + API 80 endpoints) |
| 2026-10-25 | Dynamic vs ramas predefinidas | Ramas A/B preparadas por el diseñador = 'fixed disfrazado', luego 'combinación donde dynamic decide la rama' | No es fixed (el flujo depende de un resultado intermedio) NI tiene la esencia de dynamic (nadie genera pasos nuevos). Prueba: ¿quién escribió el siguiente paso? | Sí (2/2: factura, Feynman profesor) |
| 2026-10-25 | Eval final P9 (auditoría legacy 1,200 archivos) — combinación 5+8 | Solo detectó dilution → multi-pass + modelo/prompt; no vio que "legacy que nadie conoce" es tarea abierta | Primero dynamic (mapear → alto impacto → plan priorizado), y multi-pass DENTRO de cada área para revisar. Además, no todos los 1,200 archivos pesan igual | En curso |
| 2026-10-25 | Selección de patrón (mock test: testbench nightly) | La propuesta del ingeniero ("investiga lo raro antes de seguir") convierte la tarea en dinámica → eligió dynamic | La naturaleza de la tarea la define el trabajo principal + restricciones (checklist fijo, ventana 2h, diff nocturno), no una propuesta en el enunciado. Lo raro se **marca** y alimenta un workflow de investigación **separado** (ahí sí dynamic). Una cadena que "agrega pasos si algo se ve serio" es dynamic disfrazado | Sí |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| Attention dilution vs capacidad | "Si lo hizo bien una vez, lo sabe. Si falla a veces, no lo atendió." (cajero) | Frase de contraste | Sí — la reprodujo en caso contratos |
| Dynamic vs ramas predefinidas | "¿Quién escribió el siguiente paso?" Diseñador antes = no es dynamic; agente con lo que encontró = dynamic (menú vs caja misteriosa) | Pregunta-prueba / contraste | Sí — la aplicó en caso soporte/factura |
| Dynamic en práctica | "Mapa → Tesoro → Ruta" (pirata; río inesperado = dependencia → cambia ruta) | Historia encadenada | Sin confirmar explícitamente |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| | | | |

| 2026-10-25 | Evaluación final | 9/10 | P9: combinación dynamic (5) + multi-pass (8) en escenario mixto |

## Resumen final y plan de repaso


> Pendiente para el resumen final: incluir alias **prompt chaining** y el distractor "complejo/importante ≠ dynamic". Nota: routing (ramas predefinidas) mencionado como patrón fuera del 1.6.
