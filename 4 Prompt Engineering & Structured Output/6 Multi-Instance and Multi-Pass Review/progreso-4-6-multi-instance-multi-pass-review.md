# Progreso: 4.6 Multi-Instance and Multi-Pass Review

- **Objetivo**: examen CCAR-F (Task Statement 4.6)
- **Nivel inicial**: leyó el resumen una vez. Diagnóstico: intuición correcta en ventana de contexto y calibración, pero respuestas vagas (sin nombrar *attention dilution* ni cómo se calibra). Confundió self-review con instrucciones vagas (tema 4.1).
- **Inicio**: 2026-10-18 · **Última sesión**: 2026-10-18
- **Fase actual**: ✅ Terminado — criterio de dominio cumplido (2026-10-18)

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Limitación del self-review en la misma sesión (razonamiento retenido → confirma en vez de cuestionar) | dominado | 2 | Falló en diagnóstico. 2º acierto: 200 tickets/2 cambios. Enfoques usados: (diag) razonamiento retenido; (F1) analogía detective + tabla contraste 4.1 vs 4.6; (error 2) tabla "¿escribió lo que revisa?" vs "¿ve demasiado?" + juez de su propio examen vs juez de 40 ajenos. Fallo 2: aplicó self-review a una invocación que no generó el código → reset. Acertó clasificación A-D (1/2) |
| 2 | Instancia independiente como solución (y por qué no "mejores instrucciones" ni extended thinking) | dominado | – | Analogía editor de periódico. Acertó: nueva llamada con historial completo ≠ independiente (contexto no limpio) |
| 3 | Attention dilution: los 3 síntomas del single-pass | dominado | 2 | 2º acierto: opción múltiple timeout 30s (contradicción → Pass 2). Analogía maestro calificando 40 exámenes. Acertó: reordenar solo mueve el problema. Fallo en recuperación de la lista de 3 síntomas (mezcló 1 y 2, inventó "observaciones genéricas", omitió contradicciones) → mnemotecnia "Lupa, Bostezo, Doble vara" |
| 4 | Por qué una ventana de contexto más grande NO lo resuelve (Sample Q12) | dominado | 2 | Diagnóstico: dirección correcta, respuesta vaga (se trata como fallo). F1: analogía escritorio grande ≠ más ojos. Feynman completado bien al pedir precisión; repaso "detecta el error" (1M tokens, eliminar Pass 1) acertado |
| 5 | Pass 1 — análisis local por archivo (profundidad uniforme, paralelizable) | dominado | – | Analogía 14 inspectores. Aclarado: no es "secuencial", es aislado (paralelizable). Acertó: pasar los otros 13 archivos "de referencia" reintroduce dilution |
| 6 | Pass 2 — integración cross-file (qué revisa y por qué va después) | dominado | 2 | Mini-cuestionario M2: F/H/C/P 4/4. Fase 3: tras invertir P/H, acertó 3 casos nuevos P/H (1/2). Falló en eval final P6 (lista de tipos incompleta, 3ª vez) → mnemotecnia "Fui Por Café Helado". Analogía inspector jefe / tuberías entre pisos. Acertó caso orders/billing (formatos incompatibles entre módulos) |
| 7 | Ruteo por confianza: confianza cruda vs calibrada | en curso | 1 | Fase 3: checklist de calibración real + restaurante "certificado"; acertó A/B/C (1/2). Falló en eval final P7 (no detectó calibración falsa: 5 casos + promedio). Acierto 2: opción múltiple contratos (C: set verificado + precisión por rango). Acierto 1: "detecta el error" prompt "solo da 90+ si estás seguro" no reemplaza calibración. Diagnóstico: correcto pero incompleto (se trata como fallo). F1: analogía meteoróloga 70%. Caso presupuesto 20%: riesgo bien, pero "qué medir" vago/circular ("histórico para recalibrarlo") |
| 8 | Calibración con dataset etiquetado y loop de calibración | en curso | 1 | Fase 3: acertó cuál equipo sí calibró (1/2). Falló en eval final P7 (calibración falsa). Aciertos tras reset: recalibración 90→95 (82% vs 96%); opción múltiple contratos. Fallo: caso "calibraron hace 6 meses, cambiaron prompt y modelo" → respondió "hay que calibrar" como si nunca se hubiera hecho (repitió el mantra estructural sin leer el caso) → reset; enfoque nuevo: receta de lentes. Antes: acertó entrada/salida del loop (dataset etiquetado + scores del modelo → precisión real por rango); faltó mencionar umbral actualizado y usó "confidencialidad" por "confianza". Enfoques: báscula + pesa conocida (no funcionó); ejemplo de 10 hallazgos etiquetados (funcionó); aritmética 71% → 3/10 falsos (acertó); umbral con tolerancia 95% (acertó 90, falló destino del 80–89); aeropuerto escáner vs prueba mensual |
| 9 | Arquitectura de producción completa y trade-off de costo | dominado | – | No entendió la 1ª pregunta (posts con revisión humana) → analogía seguro caro vs paraguas + pista "¿qué le pasa al error?". Acertó posts (no se justifica) y facturas → contabilidad (sí: sin humano + pérdida económica = propagación) |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-10-18 | 1 | La revisión en la misma sesión falla porque la instrucción ("revisa como auditor estricto") es vaga / no accionable | Falla porque el modelo retiene el razonamiento de la generación y tiende a confirmar sus decisiones; es estructural, no de redacción — ninguna instrucción lo arregla | Sí (2 aciertos seguidos) |
| 2026-10-18 | 4 | "Es cómo manejamos lo que entra al contexto" (vago); en Feynman usó "problema de atención" sin explicarlo | Capacidad ≠ calidad de atención: más contexto no evita que la atención se reparta desigual (profundidad dispareja, bugs a la mitad); solución = pasadas enfocadas por archivo | Sí (2 aciertos seguidos) |
| 2026-10-18 | 1 | Una sola invocación que revisa 14 archivos ajenos (local + cross-file) tiene sesgo de self-review "porque verifica sus revisiones en la misma sesión" | Self-review = el revisor tiene en contexto el razonamiento con el que GENERÓ lo revisado. Una pasada única sobre archivos ajenos es attention dilution (hallazgos contradictorios en la misma salida = síntoma 3). Matiz válido: se pierde la verificación independiente de hallazgos que hace el Pass 2 | Sí (2 seguidos) |
| 2026-10-18 | 3 | Síntomas: "precisos al inicio y después no", "inconsistente, deja pasar errores fáciles en unos sí y en otros no", "observaciones muy genéricas" | 1) Profundidad desigual entre archivos, 2) bugs dependientes de la posición (obvios perdidos a la mitad), 3) contradicciones internas (mismo patrón crítico en un archivo, aprobado en otro). "Genéricas" no es un síntoma de la guía | Sí |
| 2026-10-18 | 3 | Clasificó "23 comentarios vs 1" como posición y "archivo 8/14 con None obvio sin mencionar" como profundidad (invirtió Lupa y Bostezo) — RESUELTO con mnemotecnia "pista en la palabra" (1 acierto) | Lupa = desigualdad en CUÁNTO análisis recibe cada archivo; Bostezo = error OBVIO perdido + posición a la MITAD (las dos pistas juntas) | Sí (resuelto con "pista en la palabra") |
| 2026-10-18 | 3 | 2ª vez invertido: archivos 5-7 con credenciales obvias sin reportar → "lupa"; análisis de 2 páginas vs "se ve bien" → "bostezo" | Igual que arriba. Sospecha: la imagen "bostezo" se asocia a "revisión floja/superficial" (= profundidad), así que la mnemotecnia es ambigua | Sí (2 seguidos) |
| 2026-10-18 | 7/8 | Para calibrar: "medir cuáles marcadas como confiables lo fueron y ver el histórico para recalibrarlo" | Se usa un dataset de validación ETIQUETADO (respuesta correcta conocida); se mide, por rango de score, qué % de hallazgos fue realmente correcto (correlación confianza reportada ↔ precisión real) — en todos los rangos, no solo los "confiables"; con eso se fija el UMBRAL (no se recalibra el modelo); loop continuo | Sí (2 seguidos) |
| 2026-10-18 | 8 | Dijo "sigo sin entender" tras la báscula; luego preguntó qué % es "confiable" (no hay número universal: decisión de negocio según costo del error). Umbral 90 correcto, pero mandó el rango 80–89 "al loop de recalibración" | Ruteo ≠ calibración. Los hallazgos de baja confianza van a la COLA DE REVISIÓN HUMANA. El loop de calibración es un proceso periódico del equipo (dataset etiquetado → medir → ajustar umbral); los hallazgos en vivo no "van" ahí | Sí |
| 2026-10-18 | 8 | Ante un sistema YA calibrado hace 6 meses (luego cambió prompt + modelo) respondió "hay que calibrar con dataset etiquetado" como si no existiera calibración | La calibración caduca: si cambia el modelo, el prompt o el tipo de datos, la relación score ↔ acierto puede moverse; el umbral de 90 ya no está validado. Hay que volver a correr el dataset etiquetado (loop de calibración) y reajustar el umbral | Sí (2 seguidos) |
| 2026-10-18 | 1 | (resuelto) 200 tickets, cambió 2 → "99% correcto" | Acertó: self-review; bajo % de cambios mide tendencia a confirmar, no precisión | Sí (2 seguidos) |
| 2026-10-18 | 6 | (Eval final P6) Tipos de problemas del Pass 2: "contradicciones, errores de relaciones entre archivos, gaps" — 3ª vez incompleta | 1) formatos incompatibles entre módulos, 2) patrones contradictorios entre archivos, 3) violaciones de contrato de API en límites entre servicios, 4) inconsistencias en los hallazgos del Pass 1. "Gaps" no es de la guía | No |
| 2026-10-18 | 7 | (Eval final P7) Ante "0.95 promedio + 5 pruebas manuales correctas = calibrado" solo señaló falta de loop | El error principal: 5 casos no es calibración; un promedio no mide correlación score↔acierto POR RANGO; hace falta un dataset etiquetado representativo. El loop es secundario | No |
| 2026-10-18 | 6 | (Fase 3) Invirtió P y H: veredictos contradictorios del Pass 1 sobre el mismo código → "P"; dos archivos con estilos de logging distintos → "H" | P = el CÓDIGO choca entre archivos; H = las OPINIONES del revisor chocan (mismo código, veredicto distinto = síntoma 3 "contradicción" de dilution). Pregunta clave: ¿lo que choca es el código o el veredicto? | No |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| 3 | **Lupa, Bostezo, Doble vara** — Lupa: a unos archivos con lupa y a otros de reojo (profundidad desigual); Bostezo: a la mitad se le pasan errores obvios (posición); Doble vara: el mismo error castigado en uno y perdonado en otro (contradicciones) | Frase de imágenes en orden (basada en la analogía del maestro) | ❌ No — imágenes ambiguas ("bostezo" se lee como revisión floja), invirtió 2 veces |
| 3 | **Pista en la palabra**: cada síntoma se llama igual que el dato que trae el caso — ¿te da una *posición* (archivo 8/14, los del medio)? → Posición. ¿Te da *cuánto* análisis recibió cada archivo? → Profundidad. ¿Mismo patrón, veredicto distinto? → Contradicción | Pista en la palabra | ✅ Sí — 3/3 a la primera |
| 6 | **Fui Por Café Helado** — Formatos incompatibles entre módulos, Patrones contradictorios entre archivos, Contratos de API violados en los límites, Hallazgos del Pass 1 inconsistentes entre sí | Frase de iniciales | ✅ Sí (con la pregunta "¿choca el código o el veredicto?" para P vs H) |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| 2026-10-18 | Diagnóstico | 0 correctas, 2 parciales, 1 incorrecta | 1 (incorrecta), 4 y 7 (parciales) |
| 2026-10-18 | Evaluación final | 9/11 | 6 (P6 tipos de problemas del Pass 2, parcial), 7/8 (P7 calibración falsa) |
| 2026-10-18 | Repaso 1 (mini-cuestionario) | 5/5 | — |

## Resumen final y plan de repaso

**Criterio de dominio cumplido** (2026-10-18): 5/5 en el mini-cuestionario final, los conceptos fallados alguna vez (1, 3, 4, 6, 7, 8) con 2+ aciertos seguidos, y resumen final validado.

**Conceptos dominados**: los 9 del mapa.

**En qué batallé**
- Confundir self-review con instrucciones vagas (tema 4.1) y aplicarlo donde el revisor no generó el código.
- Los 3 síntomas de attention dilution (invertía profundidad ↔ posición).
- Los 4 tipos de problemas del Pass 2 (P vs H).
- Calibración: qué medir, ruteo ≠ loop, detectar calibración falsa, calibración que caduca al cambiar modelo/prompt/tipo de datos.
- Tendencia a responder con el mantra "es estructural" sin leer el detalle del caso.

**Mnemotecnias que funcionaron**
- *Pista en la palabra* para los síntomas: ¿posición? / ¿cuánto análisis? / ¿mismo patrón, veredicto distinto?
- *Fui Por Café Helado* (Formatos, Patrones, Contratos, Hallazgos) + "¿choca el código o el veredicto?".
- Checklist de calibración real: dataset etiquetado · tamaño representativo · por rango (no promedio) · de ahí sale el umbral · se repite cuando algo cambia.

**Plan de repaso**
- **2026-10-19 (1 día)**: Pass 2 (F/P/C/H) y checklist de calibración falsa.
- **2026-10-21 (3 días)**: síntomas de dilution + ¿cuándo SÍ es self-review? (el revisor generó lo que revisa).
- **2026-10-25 (7 días)**: `4 test.md` completo de este tema.
- **2026-11-08 – 2026-11-17 (3–4 semanas)**: `3 cuestionario.md` completo, mezclado con el test de 4.4 y 4.5.
