# Progreso: 2.5 Built-in Tools

- **Objetivo**: examen CCAR-F (Task Statement 2.5)
- **Nivel inicial**: leyó el resumen hace tiempo; pidió empezar desde cero. Diagnóstico: Grep para contenido correcto; Edit → Read + Write correcto, pero razón imprecisa ("solo reemplaza 1 a la vez" en vez de "requiere texto único para saber cuál"); alias de barril correcto, pero omitió el paso de Read para descubrir el alias.
- **Inicio**: 2026-10-19 · **Última sesión**: 2026-10-19
- **Fase actual**: 2 Evaluación final

## Mapa del tema

| # | Concepto | Estado | Aciertos seguidos tras fallo | Notas |
|---|---|---|---|---|
| 1 | Las 6 herramientas nativas (Read, Write, Edit, Bash, Grep, Glob) y por qué elegir mal cuesta tiempo/contexto | dominado | – | Taller/pinzas. Acertó caso Read de 80 archivos (tokens + tiempo) |
| 2 | Grep (contenido) vs Glob (rutas) + trampa "funciona técnicamente pero es la herramienta equivocada" | dominado | – | Linterna vs catálogo. Acertó YAML → Glob; omitió falsos positivos y creyó que Grep "solo busca 1 palabra" (aclarado: soporta regex; el problema es que busca en contenido, no en nombres) |
| 3 | Edit: coincidencia de texto única (por qué falla = mecanismo de seguridad) | dominado | 2 | Diagnóstico: razón imprecisa ("1 a la vez") — se trata como fallo. F1: analogía "tacha a Juan" (3 Juanes). Acierto 1: `return None` ×4 → reemplazar el primero = cambio silencioso equivocado; fallo = alarma. Acierto 2: `MAX_RETRIES = 3` (único, funciona) vs `}` (falla) |
| 4 | Fallback cuando Edit falla: examen (Read + Write) vs trabajo real (ampliar `old_string` / `replace_all`); nunca Read + Write por defecto | dominado | – | Diagnóstico: acertó Read + Write. Bisturí vs reconstrucción. Acertó "detecta el error" (Read + Write siempre) y opción múltiple B (no A porque exam guide v1.0 documenta Read + Write) — al inicio solo dio la letra, se le pidió el porqué |
| 5 | Exploración incremental: nunca leer todo de entrada; Grep → Read → Grep → Read | dominado | – | Analogía detective. Acertó bug "Session expired unexpectedly": Grep del mensaje literal → Read del archivo + imports |
| 6 | Trazar uso a través de wrappers y archivos barril (Read para descubrir el alias) | dominado | 2 | Acierto 2: `formatPrice` → barril `lib/index.ts` → Grep `toCurrency` + Grep de imports del barril (necesitó pista para el Grep del módulo barril; dijo "lib/index.ts" → precisado: buscar `from '../lib'`). Diagnóstico: acertó Grep del alias, omitió Read para descubrirlo — se trata como incompleto. F1: analogía Roberto/"Beto" (leer el perfil para descubrir el apodo). Acierto 1: `sendEmail` → Read de los archivos → Grep de wrappers (genérico; se precisó notifications.ts y repetir hasta no hallar nombres nuevos) |
| 7 | Patrón de deprecación: Grep → Glob → Grep otra vez (no "Glob primero") | dominado | – | Acertó por qué Grep no ve el test hermano. Malinterpretó "patrón" (nombre del patrón vs expresión Glob). Escribió `**/LoginService.test.**` → corregido a `.*` (`*` dentro de nombre, `**` = carpetas) |

> "Aciertos seguidos tras fallo" solo aplica a conceptos que se fallaron alguna vez: se necesitan 2 seguidos, con preguntas distintas, para cumplir el criterio de dominio. Un fallo lo regresa a 0.

## Errores cometidos

| Fecha | Concepto | Lo que pensé | La idea correcta | ¿Resuelto? |
|---|---|---|---|---|
| 2026-10-19 | 3 | Edit falla porque "solo puede reemplazar 1 a la vez" | Edit exige que `old_string` sea único en el archivo para saber cuál ocurrencia cambiar; es un mecanismo de seguridad, no un límite de cantidad | Sí (2 seguidos) |
| 2026-10-19 | 6 | Basta con hacer Grep también del alias | Primero hay que descubrir el alias: Read del archivo que define/re-exporta (barril) para ver los nombres exportados, y luego Grep de cada nombre | Sí (2 seguidos) |
| 2026-10-19 | 7 | Patrón Glob `**/LoginService.test.**` | `**/LoginService.test.*`: `**` solo como segmento de carpetas; `*` para la extensión | Sí (aclarado) |

## Mnemotecnias

| Concepto | Mnemotecnia | Tipo | ¿Funciona? |
|---|---|---|---|
| | | | |

## Evaluaciones

| Fecha | Ronda | Resultado | Conceptos fallados |
|---|---|---|---|
| | | | |

## Resumen final y plan de repaso

