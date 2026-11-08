# Progreso: Agentic loop (CCAR-F Task Statement 1.1)

- **Objetivo**: examen CCAR-F
- **Nivel inicial**: sesión socrática previa 75/100 (2026-09-18, ver `1 Agentic Architecture and orchestation/1 Agentic loop/5 results.md`). Lo trae frío. Diagnóstico 2026-10-26: sólido en append de resultados (stateless) y en iteration cap como red de seguridad; falló en `stop_reason` distintos a `tool_use`/`end_turn` (mismo hueco que septiembre).
- **Inicio**: 2026-10-26 · **Última sesión**: 2026-10-26
- **Fase actual**: 2 Evaluación final

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Ciclo de 4 pasos + historial **completo** en cada iteración (Claude es stateless); ahorro de tokens correcto = recortar tool results a campos relevantes, NO resumir/cortar historial | dominado | 2 | Diag 1 OK, pero mock test falló (eligió resumen regenerado). ⭐ prioridad alta. Analogía expediente médico. Acierto caso "últimos 4 mensajes"; luego falló roles (reset). Tabla 2 mensajes + ping-pong 2 jugadores. Aciertos: detecta error en código (role user + 1 solo mensaje); falta assistant msg → tool_use_id huérfano |
| 2 | `stop_reason` como señal autoritativa: `tool_use` → continuar, `end_turn` → terminar | dominado | – | Verif: end_turn + texto "voy a buscar" → termina OK |
| 3 | Otros valores (`pause_turn`, `max_tokens`, `stop_sequence`, `refusal`, `model_context_window_exceeded`): manejar cada uno, nunca asumir que todo lo no-`end_turn` es `tool_use` | dominado | 2 | Diag 2 falló (sept también). Sept: semáforo amarillo. Hoy: analogía paquetería (estados de envío) + tabla. Aciertos: refusal con else→tools; MCQ pause_turn (B) |
| 4 | Señales no confiables: tipo de contenido (`content[0].type == "text"`; texto + `tool_use` juntos) y parsing de lenguaje natural | dominado | 2 | Falló por qué el bug no aparece en preguntas simples (creyó que horario usa tool). Analogía reloj descompuesto + pista de enunciado "simples OK, complejas se cortan". Aciertos: detecta error frase fija "¿algo más?"; MCQ keyword "revisar" (C) |
| 5 | Iteration caps: red de seguridad, no control principal; runaway vs terminación prematura | dominado | – | Diag 3 OK (sept falló). Verif: web_search x80 → cap sí. Matiz corregido: al llegar al cap no "mostrar última respuesta" ni "devolver end_turn" (el código no produce stop_reason) → salir y manejar como incompleto + alertar |
| 6 | `tool_choice: "any"` para evitar texto → loop infinito | dominado | – | OK a la primera (nunca end_turn → loop infinito) |
| 7 | Model-driven decision-making vs árboles/secuencias fijas; regla de prompt sobre secuencia fija no cambia el control flow; paso scripted = árbol de decisión; excepción determinista solo si la regla debe cumplirse en TODOS los casos (financiero, seguridad, regulatorio) | dominado | 2 | Sept genérico + mock test falló. ⭐ prioridad alta. GPS y call center NO funcionaron; SÍ funcionó: "¿quién elige la siguiente tool: tu código (if) o Claude?" + mock test traducido. Confusión: "hay riesgo financiero → secuencia fija" → aclarado: costo a optimizar ≠ regla no negociable (¿hay acción correcta para TODOS los casos?). Aciertos: (a)/(b) + por qué prompt no basta; multi-respuesta aseguradora B,D |
| 8 | Integración: escenarios tipo examen y distractores | dominado | – | Detectar bugs en loop: encontró any, content type, falta end_turn, falta assistant; con empujón: range(5) como control principal + messages sobrescrito |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-09-18 | Otros stop_reason | "Ninguno es motivo para terminar" → continuar igual | Revisar por qué; no tratar como `tool_use` | Sí |
| 2026-09-18 | Model-driven | "Cuando algo debe pasar 100%" (genérico) | Dominios: financiero, seguridad, regulatorio | No |
| 2026-10-26 | Otros stop_reason (diag 2) | Con `max_tokens` "continuar y preguntarse por qué"; luego "loop infinito" | Bug: el loop trata todo no-`end_turn` como `tool_use` y busca herramientas que no existen; manejar `max_tokens` como respuesta truncada | Sí |
| 2026-10-26 | Historial (mock test) | Reemplazar historial por resumen regenerado del último tool result + recortar output | Enviar historial completo + recortar output de la herramienta; lo que dijo el cliente (tormenta) no está en ningún tool result | Sí |
| 2026-10-26 | Model-driven (mock test) | Secuencia fija + regla en system prompt "sugerir reinicio antes de book_engineer" | El prompt cambia lo que dice, no qué herramienta llama la secuencia; dejar que Claude elija con payload + conversación | No |
| 2026-10-26 | Historial: roles | Rol "agent"; 4 mensajes (uno por tool_use y uno por tool_result) | Solo `user`/`assistant`; 1 msg `assistant` con ambos `tool_use` + 1 msg `user` con ambos `tool_result` (`tool_use_id`) | Sí |
| 2026-10-26 | Señales no confiables | Pregunta simple (horario) también trae `tool_use` | Simple = solo texto + `end_turn` → check de contenido acierta por casualidad; falla solo cuando hay texto + `tool_use` | Sí |
| 2026-10-26 | Model-driven | "Flujo repetible/condición conocida → código"; "riesgo financiero → secuencia fija" | Default model-driven; código solo si la regla debe cumplirse en TODOS los casos y un fallo = riesgo financiero/seguridad/regulatorio. Si no hay acción única correcta, no hay regla que imponer | Sí |

> Pendiente para sesión de **1.2 Multi-Agent Orchestration** (no es de 1.1): mock test regulador/hepático — eligió solo C; faltó A (coordinator invoca solo los subagentes que pueden responder, scoped).

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| | | | |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| | | | |

## Resumen final y plan de repaso

