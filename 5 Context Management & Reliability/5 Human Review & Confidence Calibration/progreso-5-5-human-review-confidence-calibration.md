# Progreso: 5.5 Human Review & Confidence Calibration

- **Objetivo**: examen CCAR-F (Task Statement 5.5), tema completo
- **Nivel inicial**: leyó el resumen hace >1 mes. Diagnóstico: aggregate trap → correcto "por categoría", omitió eje por campo. Threshold único → dijo "no calibrado / confianza errónea", omitió que el mismo score significa distinto por campo, labeled validation set y thresholds por campo. Sampling solo baja confianza → correcto (muestrear alta confianza para novel errors), omitió que baja confianza ya va a review y que es estratificado.
- **Inicio**: 2026-10-21 · **Última sesión**: 2026-10-22
- **Fase actual**: 2 Evaluación final

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Aggregate Metrics Trap: accuracy por tipo de documento Y por campo | dominado | – | Diagnóstico: acertó tipo, omitió campo. Analogía hospital 37°C. Acertó: promedio por tipo (85% manuscritos) también es agregado → desglose por campo |
| 2 | Field-level confidence + calibración: scores crudos relativos no absolutos; labeled validation set → calibration curves | dominado | 2 | Diagnóstico parcial. Analogía 2 meteorólogos. Pidió explicar labeled validation set → tabla modelo vs ground truth + agrupar por campo/banda. Acierto 1: validation set solo estándar → internacionales se automatizan en silencio (necesitó pedir consecuencia concreta; dijo "escalada" en vez de human review). Acierto 2: jefa "0.95 = 95%" → relativo no absoluto, labeled validation set; omitió "por campo/tipo" |
| 3 | Thresholds calibrados y routing: por campo y tipo; arriba/abajo/zona ambigua; subir threshold sin calibrar no arregla | dominado | – | Analogía aduana/semáforo. Opción múltiple → B con descartes (A arbitrario→arbitrario). Precisado: C diagnostica mal "sobreconfiado/reentrenar" |
| 4 | Stratified random sampling: estratos (tipo, banda de confianza, campo), incluir alta confianza, doble propósito | dominado | – | Diagnóstico: idea central bien. Analogía fábrica (sacar de cada línea). No entendía "estratificar" → ejemplo frasco de dulces (fresa/limón/menta). Acertó: aleatorio simple 1% → casi sin internacionales |
| 5 | Reviewer capacity prioritisation: cola dinámica por incertidumbre, qué priorizar, anti-pattern reparto parejo | dominado | 2 | Analogía triage. Cola cronológica 200 ítems: diagnóstico y mecánica (reordenar, tomar el más incierto) bien tras pedir precisión. F: señales del recibo manuscrito → dio zonas del routing / "threshold bajo el promedio", luego mezcló invoiceDate de otro ítem. Ángulo nuevo: tabla dato→señal + síntomas vs diagnóstico. Acierto 1: X (contradictorio) antes que Y (bajo threshold); omitió señal tipo internacional (OJO: el dato de accuracy histórica no estaba en la pregunta → error del tutor, no contar). Aclarado: la guía NO define ranking entre señales; solo "mayor incertidumbre primero" + cola dinámica. Resumen 2026-10-22: invirtió prioridad ("no están por debajo del umbral") → contador a 0. Acierto 1: corrigió resumen (entra debajo de threshold + zona ambigua, orden dinámico por incertidumbre). Acierto 2: 4 revisores/1,000 ítems → anti-pattern reparto al azar; necesitó empujón para incluir contradicciones además de baja confianza (omitió manuscritos). Confirmar en eval final |
| 6 | Validation before automation: secuencia de 5 pasos; trampa = saltar al paso 5 con agregados | en curso (falló eval P6) | 1 | Analogía checklist de piloto. Detecta el error: plan sin monitoreo → faltó paso 4, doble propósito + fallas silenciosas. Eval P6 F: metió priorización de cola como paso 4, sampling como 5, omitió paso 5 (reducir review solo en segmentos consistentemente precisos), dijo "porcentajes de confianza" en paso 1. Historia aprender a manejar: Mide→Calibra→Umbrales→Muestrea→Suelta. Tras F: dijo "nunca se quita review" y luego "donde la confianza es alta" (confianza≠accuracy, usó R/A). Acierto 1: tabla 3 meses → solo estándar·fecha; monto no por caída a 94.1% (consistente), internacionales < 98% |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-10-21 | 2 | La confianza del modelo es "errónea" | Es relativa, no absoluta: mismo 0.90 = 94% real en fechas, 82% en montos; se calibra con labeled validation set → thresholds por campo | Sí (2 seguidos) |
| 2026-10-21 | 5 | Señales de prioridad = "zona ambigua" / "threshold por debajo del promedio"; mezcló datos de otro ítem | Señales concretas: baja confianza calibrada, fuente ambigua/contradictoria, tipo de documento con mala accuracy histórica, varias interpretaciones. Zona ambigua es el resultado, no la señal; lo que baja es la confianza, no el threshold | No |
| 2026-10-22 | 5 | Resumen parcial: "se priorizan los que NO están por debajo del umbral" | Se priorizan los que SÍ están por debajo del threshold calibrado (baja confianza), fuentes contradictorias, tipos con mala accuracy histórica; cola dinámica | No |
| 2026-10-22 | 2 | Resumen + Eval P2: "no calibrado al 100% / el modelo no es bueno dando score" (3 veces), el modelo no es bueno" (repite vaguedad) | Score relativo, no absoluto; mismo valor = distinta accuracy por campo | No |
| 2026-10-22 | 6 | Secuencia: 4=priorizar cola, 5=sampling; sin paso de reducir review; paso 1 "confianza" | 1 accuracy por tipo×campo, 2 calibrar (labeled set), 3 thresholds, 4 stratified sampling, 5 solo entonces reducir review en segmentos consistentemente precisos. Cola ≠ paso | No |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| 2 | "Un 9 del profe barco ≠ un 9 del profe estricto" | Contraste | No |
| 2 | "Regla sin números: ORDENA, no MIDE" | Imagen | No (quería relativo/absoluto) |
| 2 | R = Ranking (Relativo), A = Acierto real (Absoluto): "el score crudo es R, no A; calibrar lo convierte a A, campo por campo" | Acrónimo/iniciales | Sí. Aclarado: ranking solo vale dentro del mismo campo |
| 6 | Aprender a manejar: Mide → Calibra → Umbrales → Muestrea → Suelta (solo calles donde siempre vas bien) | Historia encadenada | Por confirmar |

## Evaluaciones

> Resumen parcial (conceptos 1–5), 2026-10-22: omitió concepto 1 (aggregate trap), invirtió prioridad de la cola, vago en calibración, faltaron 3 zonas de routing. Pedido corregir puntos 1 y 2 → corregidos bien (resumen parcial validado).

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| 2026-10-22 | Evaluación final (en curso) | P1 ✅ (B; D: volumen≠accuracy) · P2 ✅ (ejemplo numérico; vaguedad recurrente) · P3 ✅ zona ambigua · P4 ✅ no excluir banda >0.98 · P5 ✅ C con descartes · P6 ❌ secuencia | |

## Resumen final y plan de repaso

