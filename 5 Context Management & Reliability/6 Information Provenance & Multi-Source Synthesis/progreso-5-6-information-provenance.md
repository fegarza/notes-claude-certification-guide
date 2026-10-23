# Progreso: 5.6 Information Provenance & Multi-Source Synthesis

- **Objetivo**: examen CCAR-F (Task Statement 5.6), tema completo
- **Nivel inicial**: leyó el resumen, no sabe cuánto recuerda. Diagnóstico: conflicto entre fuentes → "no elegir" correcto, pero omitió atribución completa (fuente, fecha, contexto). Punto de falla del pipeline → dijo "analysis" (incorrecto: synthesis). Analysis agent que elige $4.2M → detectó que no debía elegir, pero omitió "completar con conflicto anotado" y que decide el coordinator.
- **Inicio**: 2026-10-20 · **Última sesión**: 2026-10-20
- **Fase actual**: Terminado (criterio de dominio cumplido 2026-10-20)

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Information provenance: qué es y por qué sin ella el output es "plausible-sounding fiction" | dominado | – | Analogía chisme de oficina. Acertó "73% sin fuente" (precisado: no verificable = indistinguible de alucinación) |
| 2 | Structured claim-source mappings: 5 campos, producidos desde el inicio | dominado | – | Etiqueta de frasco. Acertó campos faltantes + excerpt (no sacar de contexto); con pista detectó claim vago ("bastante") |
| 3 | Atribución muere en summarisation: synthesis = punto de falla; 3 requisitos del pipeline | dominado | 2 | Falló diagnóstico (dijo analysis). F1: teléfono descompuesto + conexión 5.1. Acierto 1: aun con mappings completos, falla si el prompt de synthesis no exige preservarlos. Acierto 2: reporte en prosa sin citas rompe requisito 3 |
| 4 | Conflict handling: anotar ambos con atribución; no elegir/promediar/más reciente/más autoritativo | dominado | 2 | Acierto 2: detecta el error en prompt "report only the newer value" (más reciente + publicación ≠ periodo de datos). Diagnóstico: idea central bien, omitió atribución completa. Tabla ❌/✅. Acierto 1: promedio 10% (necesitó replanteo de la pregunta); dio atribución completa. F2: propuso como opción del coordinator "elegir uno según nivel de confianza" (= trampa más autoritativo). Acierto 1 tras F2: opción múltiple Gartner vs blog → C, explicó A/B/D (necesitó pedirle el porqué) |
| 5 | Análisis completa con conflictos anotados; coordinator decide (ambos / investigar / escalar) | dominado | 2 | Acierto 1 tras F: dosis medicamento → escalar o presentar ambos (descartó investigar). Acierto 2: opción múltiple fecha fundación → B, explicó A/C/D (dijo "revisor" en vez de coordinator). Diagnóstico incompleto. Analogía laboratorio vs médico. Acierto 1: agente que se detiene con error (con pista). Caso empleados 1,200 vs 950: dio "presentar ambos" (válido) pero no recordó las otras 2 opciones aun con pista (dijo revisar fechas / elegir por confianza). Mnemotecnia PIE |
| 6 | Temporal awareness: fechas convierten falsas contradicciones en tendencias | dominado | 2 | Acierto 1 tras F2: autos eléctricos 2021 vs 2023, misma publicación → tendencia. Acierto 2: remote work 28% vs 41% → falta periodo de datos. Analogía estatura mamá vs doctor. F1: respuesta vaga ("seleccionado mal"); reexplicado paso a paso. Acierto 1: paneles $0.45→$0.30 (dijo 2025 en vez de 2024). F2: churn 15% vs 9% mismo periodo 2023 → dijo "tendencia, 1 mes de diferencia" (confundió fecha de publicación con periodo de datos). Analogía básculas mismo día |
| 7 | Well-established vs contested findings | dominado | – | Pronóstico del clima. Feynman bien; precisado: well-established = corroborado por fuentes independientes (no "certeza"), agrupar ≠ rankear. Acertó fuente única → no well-established |
| 8 | Content-appropriate rendering (financial → tablas, news → prosa, technical → listas) | dominado | – | Periódico. Acertó adquisición/ingresos/API |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-10-20 | 3 | La atribución se pierde en el paso de analysis; arreglo = forzar al analysis agent a devolver fuente y fecha | El punto de falla más común es synthesis (comprime y parafrasea); su prompt debe exigir mantener y mergear los claim-source mappings | Sí (2 seguidos) |
| 2026-10-20 | 6 | Sin fechas el agente "selecciona mal" (vago) | Sin fechas lo trata como conflicto/problema de calidad → marca o suprime hallazgos consistentes; se pierde la tendencia | Parcial |
| 2026-10-20 | 2 | Eval final P1: llamó "cita" al claim y omitió que relevantExcerpt verifica contexto | Campo = claim (afirmación específica); relevantExcerpt comprueba que no se sacó de contexto | No (acertó P8 sobre mappings desde el inicio) |
| 2026-10-20 | 6 | Eval final P9: respondió responsabilidad de atribución en general en vez de fechas | Subagentes incluyen fechas → synthesis las preserva en el merge → output las presenta junto a los datos (incluir → preservar → presentar) | No |
| 2026-10-20 | 5 | Opciones del coordinator = revisar fechas / elegir el de mayor confianza | PIE: Presentar ambos, Investigar más, Escalar a humano; ninguna elige un valor | No |
| 2026-10-20 | 4 | Elegir según nivel de confianza de la fuente es válido | Es la trampa del publisher "más autoritativo": sigue siendo selección arbitraria | No |
| 2026-10-20 | 6 | Distinta fecha de publicación (1 mes) = tendencia | Lo que importa es el periodo de los datos; mismo periodo con valores distintos = conflicto real (concepto 4) | No |
| 2026-10-20 | 4 | Basta con remarcar la discrepancia | Ambos valores con atribución completa: fuente, fecha, contexto metodológico | No |
| 2026-10-20 | 5 | Decide "quien lo solicitó" | El analysis agent completa con el conflicto anotado; decide el coordinator (presentar ambos / investigar / escalar a humano) antes de síntesis | No |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| 6 | "La fecha del periódico no es la fecha de la foto" (periodo de datos > fecha de publicación) | Contraste / imagen | Sí |
| 5 | El coordinator se pone de PIE: Presentar ambos · Investigar más · Escalar a humano | Acrónimo | Por confirmar |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| 2026-10-20 | Evaluación final | 8/10 | 2 (P1: nombre claim + relevantExcerpt), 6 (P9: incluir → preservar → presentar fechas) |
| 2026-10-20 | Repaso 1 | 7 preguntas; 2 necesitaron segunda pasada (R2 fechas en output, R4 por qué confidenceScore no sustituye) | Recuperados: 2 (R1 relevantExcerpt, R6 no opcional/desde el inicio) y 6 (R5 opción múltiple Gartner/IDC, R7 incluir→preservar→presentar) |

## Resumen final y plan de repaso

**Resumen del estudiante validado** (tras completar: values con value + source + context, 4 trampas incl. más autoritativo, periodo de datos > fecha de publicación, prosa = news/current events).

- **Conceptos dominados**: los 8 del mapa.
- **En qué batallé**:
  - Punto de falla del pipeline: creía analysis → es synthesis (teléfono descompuesto).
  - Fecha de publicación vs periodo de datos (2 fallos) → mnemotecnia periódico/foto.
  - Opciones del coordinator: propuse "elegir por confianza" (= trampa más autoritativo) → mnemotecnia PIE.
  - Hábitos: en opción múltiple doy la letra sin descartes; digo "revisor" en vez de coordinator; "6%" vs 6 puntos porcentuales.
- **Mnemotecnias útiles**: "La fecha del periódico no es la fecha de la foto"; "El coordinator se pone de PIE"; incluir → preservar → presentar (fechas).
- **Plan de repaso**:
  - 2026-10-21: `3 cuestionario.md` (foco conflictos + PIE)
  - 2026-10-23: `4 test.md` completo, explicando descartes
  - 2026-10-27: inventar un caso de tendencia y uno de conflicto real
  - 2026-11-03 y 2026-11-19: 5 trampas de `1 resumen.md` + test rápido

