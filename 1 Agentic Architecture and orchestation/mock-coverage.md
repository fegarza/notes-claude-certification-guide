---
dominio: 1
confianza_modulo: 26
sesiones: 1
ultima_sesion: 2026-11-04
tags: [mock-coverage, ccar-f]
---

# Mock coverage — Dominio 1: Agentic Architecture & Orchestration

> [!info] Confianza del módulo: **26/100**
> Registro de [[tutor-mock]]. Cada respuesta actualiza este archivo; la siguiente sesión prioriza los TS débiles y las trampas activas.

## Confianza por Task Statement

| TS | Tema | Preguntas | Correctas | Historial (últimos 8) | Confianza | Nivel | Última vez |
|---|---|---|---|---|---|---|---|
| 1.1 | [[1 Agentic Architecture and orchestation/1 Agentic loop/1 resumen\|Agentic loop]] | 1 | 1 | ✅ | 20 | Débil | 2026-11-04 |
| 1.2 | [[1 Agentic Architecture and orchestation/2 Multi-Agent Orchestration/1 resumen\|Multi-Agent Orchestration]] | 1 | 1 | ✅ | 20 | Débil | 2026-11-04 |
| 1.3 | [[1 Agentic Architecture and orchestation/3 Subagent Invocation and Context Passing/1 resumen\|Subagent Invocation and Context Passing]] | 2 | 1 | ✅❌ | 20 | Débil | 2026-11-04 |
| 1.4 | [[1 Agentic Architecture and orchestation/4 Workflow Enforcement and Handoff/1 resumen\|Workflow Enforcement and Handoff]] | 2 | 2 | ✅✅ | 40 | En progreso | 2026-11-04 |
| 1.5 | [[1 Agentic Architecture and orchestation/5 Agent SDK Hooks/1 resumen\|Agent SDK Hooks]] | 2 | 2 | ✅✅ | 40 | En progreso | 2026-11-04 |
| 1.6 | [[1 Agentic Architecture and orchestation/6 Task Decomposition Strategies/1 resumen\|Task Decomposition Strategies]] | 1 | 1 | ✅ | 20 | Débil | 2026-11-04 |
| 1.7 | [[1 Agentic Architecture and orchestation/7 Session State and Resumption/1 resumen\|Session State and Resumption]] | 1 | 1 | ✅ | 20 | Débil | 2026-11-04 |

## Trampas

| Trampa | TS | Fallos | Aciertos seguidos | Estado |
|---|---|---|---|---|
| Culpar al agente de síntesis por citas faltantes cuando el problema real es la metadata | 1.3 | 1 | 0 | Activa |

## Bitácora de preguntas

| Fecha | TS | Objetivo evaluado | Contexto / industria | Trampa(s) en distractores | Resultado |
|---|---|---|---|---|---|
| 2026-11-04 | 1.1 | Control de flujo con `stop_reason` vs chequeo de texto | Customer Support / seguros de hogar | Revisar el tipo de contenido en vez de `stop_reason`; límites de iteración como mecanismo principal; parsear frases de lenguaje natural; fix por prompt | ✅ |

## Historial de sesiones

| Fecha | Preguntas | Score | Confianza del módulo | Foco de la sesión |
|---|---|---|---|---|
| 2026-11-04 | 10 | 9/10 (90%) | 0 → 26 | Primera sesión: diagnóstico de los 7 TS (1.3, 1.4 y 1.5 con 2 preguntas cada uno) |
