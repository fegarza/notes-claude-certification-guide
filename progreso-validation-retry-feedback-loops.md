# Progreso: Validation, Retry, and Feedback Loops (CCAR-F Task Statement 4.4)

- **Objetivo**: examen CCAR-F
- **Nivel inicial**: sesión socrática previa 84/100 (2026-10-11, ver `4 Prompt Engineering & Structured Output/4 Validation, Retry, and Feedback Loops/5 results.md`). Diagnóstico 2026-11-04: sólido en sintaxis vs semántica (clasificó 4/4 y ubicó tool_use vs validators); reconoce que la info ausente no es retryable pero lo resolvió con structured errors MCP (`errorCategory: permission`, `isRetryable`), confundiendo dominios y sin plantear null/revisión humana; falló self-correction en schema (propuso enum en vez de extraer ambos valores + `conflict_detected`).
- **Inicio**: 2026-11-04 · **Última sesión**: 2026-11-04
- **Fase actual**: 1 Enseñanza

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Schema syntax errors (eliminados por `tool_use`/JSON schema) vs semantic validation errors (requieren validación aparte) | dominado | 2 | Diag 2 OK. Verif "detecta error" falló (dijo "strict no existe", no vio cumple schema ≠ correcto). Analogía formulario de aduana. Aciertos: £450 vs £500 pasa schema; MCQ qué llega al retry con strict (C) |
| 2 | Retry-with-error-feedback: 3 ingredientes (documento original, extracción fallida, error específico); retry ingenuo repite el error | en curso | 1 | Analogía mecánico. Falló: en código con doc+extracción dijo que faltaban los 3 (solo faltaba error). Tabla mensaje→ingrediente. Luego feedback vago (quitó números) → corregido con valores exactos |
| 3 | Frontera de efectividad: retry sirve para formato/estructura/valor mal ubicado/matemática; NO para info ausente o en documento externo → clasificar antes de reintentar; `null` o revisión humana; forzar "obligatorio" → fabricación | en curso | 1 | Diag 1: lo trató como structured error MCP (`permission`, `isRetryable`) — Dominio 2, no aplica. ⭐ Oct también le faltó la alternativa. Analogía receta sin tiempo de horno + contraste con MCP. Clasificó 3/3 OK pero de nuevo omitió qué hacer (null) hasta que se le pidió; con empujón: null + "nunca vacío" → inventa. Falta: que proponga null/revisión humana SIN que se le pida |
| 4 | Pydantic: parsing (estructura) + validators (semántica); `ValidationError` = error específico listo para el retry; `strict: true`/`messages.parse` no eliminan la capa semántica | pendiente | – | |
| 5 | Self-correction en el schema: `calculated_total` vs `stated_total`; `conflict_detected` + extraer AMBOS valores; se marca sola sin lógica externa | pendiente | 0 | Diag 3 falló: propuso enum (oculta el conflicto, el modelo sigue eligiendo uno). ⭐ Oct también impreciso |
| 6 | `detected_pattern`: loop de mejora (extraer → validar → recolectar descartes → refinar prompt por patrón) | pendiente | – | |
| 7 | Integración: escenarios tipo examen y distractores | pendiente | – | |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-11-04 | Frontera de efectividad (diag 1) | PO ausente → respuesta estructurada con `errorCategory: permission`, `isRetryable: false` | No es fallo de tool ni de permisos: el dato no existe en la fuente → no reintentar; `null` (si schema lo permite) o revisión humana | No |
| 2026-11-04 | Self-correction schema (diag 3) | Contradicción 30 vs 60 días → cambiar `payment_terms` a enum | El enum solo restringe formato; el modelo sigue eligiendo uno en silencio. Extraer ambos valores + `conflict_detected: true` | No |
| 2026-11-04 | Sintaxis vs semántica | "`strict` no existe" (no detectó el error real) | `strict: true` existe (Structured Outputs) y garantiza schema, pero cumplir el schema ≠ datos correctos | Parcial (1/2) |
| 2026-11-04 | Retry-with-error-feedback | Código con `invoice_text` + `failed_extraction` + "con más cuidado" → "faltan los 3" | Solo falta el error específico; los otros dos están en los mensajes | Parcial (1/2) |
| 2026-11-04 | Retry-with-error-feedback | Feedback genérico "la suma no corresponde" | Conservar valores exactos (1,240 vs 1,390; mes 13) → corrección dirigida | Sí (en la misma pregunta) |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|

## Resumen final y plan de repaso

