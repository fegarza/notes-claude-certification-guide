# Progreso: Codebase Exploration & Context Degradation (Task Statement 5.4)

- **Objetivo**: examen CCAR-F (Claude Certified Architect – Foundations), dominio 5
- **Nivel inicial**: intuición general sólida (sesión socrática 2026-10-15: 69/100). Diagnóstico: cold start/summary injection bien con causa raíz; mecanismo de degradación impreciso ("nueva información" en vez de output verboso); confunde roles de scratchpad vs manifest y cree que el manifest "se carga en automático".
- **Inicio**: 2026-11-08 · **Última sesión**: 2026-11-08
- **Fase actual**: 1 Enseñanza

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Context degradation: síntoma ("typical patterns") y mecanismo (output verboso → atención a lo reciente) | dominado | 2 | Fallado en diagnóstico. Enfoques usados: chat de grupo con memes, aguja en pajar (proporción aguja/paja). Acierto 1: agente cat vs grep. (Postmortem: necesitó empujón, no contó.) Acierto 2: opción múltiple 200 turnos al 70% vs 25 turnos verbosos al 40% |
| 2 | No es token limit: ventana más grande o precargar el codebase no lo arreglan | dominado | – | Acertó lo del 45% en diagnóstico + ventana 1M vs precargar repo. Ojo: dijo "context dilution" en vez de "context degradation" |
| 3 | Scratchpad files: qué contiene (específico), desde el inicio, por qué es inmune, quién lo usa | dominado | 2 | Fallado en diagnóstico ("nadie lo usa"). Enfoque: sacar la aguja del pajar a una caja. Aciertos: escribir notas a la hora 2 (necesitó empujón en el porqué) + reescribir scratchpad genérico |
| 4 | Subagent delegation: context isolation > paralelización, delegar preguntas específicas | en curso | 0 | Enfoque: director de obra con plomero/electricista; "la paja nunca cae en tu granero". Falló: tarea vaga "explora todo" |
| 5 | Summary injection entre fases y cold start (subagentes no heredan contexto) | pendiente (verificar rápido) | – | Acertado con causa raíz en diagnóstico |
| 6 | /compact proactivo: resume preservando lo clave; calidad, no solo cantidad | pendiente | – | Hueco en sesión socrática |
| 7 | Crash recovery con state manifests: contenido, ubicación conocida, el coordinador lo carga e inyecta on resume | pendiente | 0 | Fallado en diagnóstico ("se carga en automático") |
| 8 | Scratchpad vs manifest + trampa de reiniciar sin guardar estado | pendiente | – | |
| 9 | Integración: elegir la mitigación correcta según el síntoma | pendiente | – | |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-11-08 | 1 Mecanismo | "La nueva información hace que la antigua pierda el foco" | Lo que entierra los hallazgos es el **output verboso** acumulado (archivos, búsquedas, listados), no nuevos hallazgos; la atención se desplaza a lo reciente | Sí |
| 2026-11-08 | 3 Scratchpad | "notes.md nadie lo usa" | El agente lo escribe y lo consulta **durante** la sesión para no depender del contexto degradado | Sí |
| 2026-11-08 | 4 Subagent delegation | Ante "explora todo el codebase" solo vio que era vago; no dijo que el subagente se degrada él mismo; propuso "otro subagent exploration" | Una tarea vaga traslada la degradación al subagente (se llena de output verboso → resumen genérico). Solución: preguntas acotadas (un flujo, un servicio, un tipo de integración) | No |
| 2026-11-08 | 7 Manifest | "El manifest se carga en automático" | El **coordinador** carga el manifest on resume y lo **inyecta** en los prompts de los agentes | No |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| | | | |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| | | | |

## Resumen final y plan de repaso

<Se llena al cumplir el criterio de dominio.>
