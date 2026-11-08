# Progreso: Agent SDK Hooks (CCAR-F Task Statement 1.5)

- **Objetivo**: examen CCAR-F
- **Nivel inicial**: sesión socrática previa 68/100 (2026-09-19, ver `5 results.md`). Diagnóstico 2026-10-25: conceptual sólido (dirección Pre/Post, hooks vs prompts, normalización); flojo en nombres exactos de API (`permissionDecision` y sus valores, `updatedToolOutput`).
- **Inicio**: 2026-10-25 · **Última sesión**: 2026-10-25
- **Fase actual**: 2 Evaluación final

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Dirección Pre vs Post y sus poderes (Post detiene loop, no deshace efecto) | dominado | – | Diag 2 OK (refund $800 ya salió). Matiz: es `decision: "block"`, no "excepción" |
| 2 | Retorno `PreToolUse`: `permissionDecision` (allow/deny/ask/defer) + `updatedInput` | dominado | 2 | Diag 1 falló. Tabla aduana + A-D-A-D. Aciertos: descuento 25% → ask; AML → deny |
| 3 | Retorno `PostToolUse`: `updatedToolOutput` (reemplaza obsoleto `updatedMCPToolOutput`) | dominado | 2 | Diag 3 `updatedOutput`; en resumen del estudiante `outputToolResult` (regresó a 0). Mnemotecnia "se cayó el MCP" NO funcionó → tabla `updated`+qué cambias + **UTO** (propia). Aciertos tras reset: charge_card (Input+ToolOutput); fechas en output no input (integración 1) |
| 4 | Normalización de datos heterogéneos con PostToolUse (por qué no el modelo) | dominado | – | Diag 3 OK; distractor herramienta `convert_date` OK (decidir llamarla sigue siendo probabilístico) |
| 5 | 3 patrones de enforcement PreToolUse + `deny` accionable (`permissionDecisionReason`: motivo + herramienta exacta + qué NO hacer) | dominado | 2 | Confundió con structured error response (dominio 2). Analogía letrero "calle cerrada + desvío". Falló reembolso, parcial disputa; aciertos MCQ send_email + predecir comportamientos (reintentar, partir monto, bajar monto) |
| 6 | Hooks vs prompts: consecuencia de un solo fallo; sobre-ingeniería en formato | dominado | – | Caso clínica (tono formal vs PII a analytics) OK; tuvo que precisar criterio (no "suma importancia" sino "un solo fallo = riesgo legal") |
| 7 | PreCompact/PostCompact: Pre puede bloquear (exit 2 / `decision: "block"`), Post solo observa | dominado | 2 | Falló sept; hoy caso archivar olvidó el bloqueo (con empujón lo dijo). Analogía editor/imprenta. Aciertos: PostCompact exit 2 no revierte; MCQ múltiple opción C |
| 8 | Integración: escenarios tipo examen y distractores | dominado | – | MCQ refund 3% + fechas OK (precisó C); multi-respuesta A,C OK |

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-09-19 | Retorno PreToolUse | No supo `ask`/`defer` ni `updatedInput` | `permissionDecision` allow/deny/ask/defer + `updatedInput` | Sí |
| 2026-09-19 | Retorno PostToolUse | No recordó predecesor obsoleto | `updatedToolOutput` reemplaza `updatedMCPToolOutput` | Sí |
| 2026-09-19 | PreCompact | No mencionó que puede bloquear | PreCompact puede prevenir compactación (exit 2 / block); PostCompact solo recibe el resumen | Sí |
| 2026-10-25 | Retorno PreToolUse (diag 1) | Solo `updatedInput`; no supo el campo de veredicto ni con pista | `permissionDecision: "allow"` + `updatedInput` | Sí |
| 2026-10-25 | Retorno PostToolUse (diag 3) | `updatedOutput` | `updatedToolOutput` | Sí |
| 2026-10-25 | Razón del deny | Pensó que era structured error response (`isError`, `errorCategory`) | Eso es de herramientas que SÍ corrieron (dominio 2); el hook devuelve `deny` + `permissionDecisionReason` | Sí |
| 2026-10-25 | Razón del deny (reembolso $1,200) | "No se permiten reembolsos > $500" — motivo sin siguiente paso; luego "abra un ticket" (vago) | Motivo + herramienta exacta (`escalate_to_human`) + qué NO hacer (no reintentar / no partir el monto) | Sí |
| 2026-10-25 | Retorno PostToolUse (resumen estudiante) | `outputToolResult` | `updatedToolOutput` (UTO) | Sí |
| 2026-10-25 | PreCompact (caso archivar) | Solo archivar; olvidó que puede bloquear | exit 2 / `decision: "block"` | Sí |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| 2 | **A-D-A-D**: Allow, Deny, Ask, Defer (pasa / no pasa / pregunta / delega) | Acrónimo | por confirmar |
| 3 | "Se le cayó el MCP": `updatedMCPToolOutput` → `updatedToolOutput` (el Tool se queda) | Contraste | No |
| 3 | **UTO** = Updated Tool Output; "si cambias algo, empieza con `updated`" (`updatedInput` / `updatedToolOutput`) | Acrónimo (propio) | Sí |
| 5 | Letrero: "calle cerrada" vs "calle cerrada, desvío por X" → motivo + herramienta + qué no hacer | Imagen | por confirmar |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|

## Resumen final y plan de repaso

