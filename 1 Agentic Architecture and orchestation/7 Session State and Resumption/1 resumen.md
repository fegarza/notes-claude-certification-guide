> [!note] En una frase
> Elegir entre `--resume`, `fork_session` y un inicio limpio con resumen inyectado nunca depende de "qué tan importante" es la sesión, sino de una sola pregunta diagnóstica — ¿el contexto previo sigue siendo válido, quiero explorar en paralelo, o los resultados de herramientas ya están obsoletos? — y cuando la respuesta es "obsoletos", releer los archivos modificados dentro de la misma sesión resumida **no alcanza**: el historial contaminado sigue ahí.

## Explícamelo como si tuviera 5 años

Imagina que estás armando un rompecabezas con un amigo y se van a la cama a mitad de camino. Al día siguiente tienen tres formas de seguir:

1. **Seguir donde quedaron** (`--resume`): se sientan exactamente frente a las mismas piezas, en el mismo orden, y continúan. Perfecto si nadie tocó el rompecabezas de noche.
2. **Hacer una copia y probar dos caminos distintos** (`fork_session`): fotocopian el rompecabezas tal como está y cada quien prueba una estrategia distinta a partir de ahí — sin que lo que hace uno afecte al otro.
3. **Empezar de cero con un resumen en una hoja** (fresh start + summary injection): si durante la noche alguien reorganizó varias piezas, sentarse a "seguir donde quedaron" es peligroso — van a seguir recordando piezas que ya no están donde las dejaron. Mejor escriben en una hoja "ya armamos el cielo y el pasto, faltan las montañas" y empiezan una sesión nueva con esa hoja como única guía, sin las piezas viejas mal ubicadas en la cabeza.

La trampa es pensar que "seguir donde quedaron" siempre es lo más eficiente. Si las piezas se movieron (los archivos cambiaron), seguir con el recuerdo viejo produce confusión — vas a "corregir" piezas que ya estaban bien, o vas a buscar una pieza que ya no existe donde la dejaste.

## Argumento central

> **Selección de estrategia de sesión**: las tres opciones — reanudar, bifurcar (fork) o empezar de cero con resumen — sirven propósitos mutuamente excluyentes: `--resume` es para **continuación** (el contexto previo sigue siendo válido), `fork_session` es para **exploración divergente** (partir de una base compartida hacia caminos distintos), y el inicio limpio con resumen inyectado es para cuando **los resultados de herramientas previos ya están obsoletos**. Ninguna es "mejor" en abstracto; la pregunta correcta decide.
>
> **El problema del contexto obsoleto (stale context)**: reanudar una sesión después de modificar archivos no borra los resultados de herramientas viejos — el historial completo se restaura, incluyendo lecturas de archivos que ya no reflejan el estado actual. El modelo termina razonando con una mezcla de datos viejos y nuevos, y la corrección de "simplemente pedir que relea los archivos modificados" **no es suficiente**, porque los resultados obsoletos siguen presentes en el historial de conversación.

## Ideas clave (Conclusiones)

### 1. Opción 1 — `--resume <session-name>`: continuación

- Reanuda una sesión nombrada específica, **restaurando todo el historial de conversación**, incluyendo resultados de herramientas y análisis previos.
- Los nombres se asignan con `--name` / `-n` al iniciar, o a mitad de sesión con `/rename`. `claude --resume <name>` retoma esa sesión. `-c` / `--continue` retoma la conversación más reciente en el directorio actual (sin necesidad de nombre).
- **Úsalo cuando**: el contexto previo sigue siendo mayormente válido y los archivos no han cambiado de forma significativa desde la última sesión.
- **Evítalo cuando**: los archivos se modificaron desde la última sesión — los resultados de herramientas en el historial ya no reflejan el estado actual del código.

### 2. Opción 2 — `fork_session`: exploración divergente

- Crea **ramas independientes** desde una base de análisis compartida. Los cambios en una rama no afectan a las otras, y las ramas no pueden ver los resultados de las demás.
- En el SDK, `fork_session` opera **junto con** `resume`, no en su lugar: `resume` selecciona la sesión base, y `fork_session: true` crea una sesión nueva a partir de una **copia** del historial en vez de seguir escribiendo sobre el mismo. En la CLI, `--fork-session` se combina con `--resume` de forma equivalente.
- **Úsalo cuando**: ya completaste un análisis inicial y quieres explorar enfoques divergentes desde ese punto de partida compartido — por ejemplo, comparar dos estrategias de refactorización a partir del mismo análisis de codebase.
- **No lo uses cuando**: solo quieres continuar la misma línea de investigación — fork es para divergencia, no para continuación.

> [!note] Analogía rápida
> `--resume` = seguir escribiendo en el mismo cuaderno. `fork_session` = fotocopiar el cuaderno y que cada quien escriba en su copia. Ninguna fotocopia "limpia" el cuaderno — si el cuaderno original tenía datos obsoletos, la copia también los hereda.

### 3. Opción 3 — inicio limpio con resumen inyectado (fresh start + summary injection)

- Empieza una sesión **completamente nueva**, inyectando en el contexto inicial un **resumen estructurado** de los hallazgos previos. ~={red}No contiene resultados de herramientas obsoletos — solo resúmenes curados=~.
- **Úsalo cuando**: los resultados de herramientas de la sesión previa están obsoletos (archivos modificados, APIs actualizadas, dependencias cambiadas) o el contexto se degradó tras una sesión muy larga.
- **No lo uses cuando**: el contexto previo sigue siendo válido y quieres mantener el historial completo de conversación — en ese caso, `--resume` es más eficiente.

### 4. El problema del contexto obsoleto, en detalle

> [!warning] Definición exacta
> Ocurre cuando un agente reanuda una sesión después de que el código fue modificado y razona a partir de resultados de herramientas cacheados que ya no reflejan el estado actual de los archivos.

- **Manifestación**: consejos contradictorios — recomendaciones para cambios que ya se hicieron, o referencias a código que ya no existe — porque el razonamiento se basa en resultados de herramientas viejos dentro del historial de conversación.
- **Causa raíz**: reanudar restaura *todo* el historial, incluyendo resultados de herramientas previos. Los archivos modificados siguen apareciendo con su contenido viejo como salida de herramienta cacheada, y el modelo mezcla eso con datos actuales.
- **Arreglo insuficiente**: reanudar y pedirle al agente que relea los archivos modificados. Ayuda, pero los resultados obsoletos **siguen en el historial** — el modelo puede seguir refiriéndose a información vieja de más atrás en el contexto.
- **Arreglo correcto**: iniciar una sesión nueva con un resumen estructurado de los hallazgos previos, especificando qué archivos cambiaron, para que el agente haga un **re-análisis dirigido** solo de esos archivos.

> [!warning] Trampa de examen central
> El examen evalúa si reconoces que reanudar después de cambios en archivos puede producir contexto obsoleto — y que simplemente reanudar y pedir que se relean los archivos modificados **no es la mejor respuesta**.

### 5. Re-análisis dirigido vs. re-exploración completa

En vez de re-analizar todo el codebase cuando cambian algunos archivos, se informa al agente específicamente cuáles cambiaron para que re-analice solo esos.

**Flujo del enfoque correcto:**
1. Iniciar una sesión nueva.
2. Inyectar un resumen estructurado: "El análisis previo encontró X, Y y Z en el codebase. Los siguientes 3 archivos se modificaron desde entonces: `auth.ts`, `database.ts` y `api-routes.ts`."
3. El agente relee y re-analiza únicamente esos 3 archivos.
4. Combina el análisis fresco de los archivos cambiados con el resumen preservado de los archivos sin cambios.

**Beneficio**: más rápido que una re-exploración completa, y más confiable que reanudar con contexto obsoleto.

### 6. Matriz de decisión

| Escenario | Mejor opción | Razonamiento |
|---|---|---|
| Continuar trabajo de ayer, sin archivos modificados | `--resume` | El contexto previo es válido, el historial completo es útil |
| Comparar dos estrategias de refactorización | `fork_session` | Exploración divergente desde una base compartida |
| Reanudar tras modificar 3 de 50 archivos | Inicio limpio + resumen | Resultados obsoletos en los 3 archivos modificados producirían contradicciones |
| Sesión larga con historial saturado | Inicio limpio + resumen | El contexto degradado se beneficia de una base limpia |
| Explorar estrategia de testing vs. estrategia de documentación | `fork_session` | Dos enfoques independientes desde el mismo análisis |
| Reanudar tras actualizar dependencias | Inicio limpio + resumen | Múltiples archivos pudieron cambiar indirectamente |

> [!warning] Por qué la tabla no es "memoriza estos 6 ejemplos"
> El examen va a describir un escenario nuevo, no repetir literalmente "actualizar dependencias" o "comparar refactorizaciones". Memoriza el criterio detrás de cada fila — ¿el contexto sigue siendo válido, hay divergencia, o hay obsolescencia? — no la lista.

### 7. Caso guía: el bug de los consejos contradictorios

Un desarrollador analiza un codebase de 50 archivos durante dos días. El día 1 identifica tres problemas de autenticación. Durante la noche, corrige los tres modificando `auth.ts`, `session.ts` y `middleware.ts`.

Al reanudar el día 2 con `--resume`, el agente recomienda arreglar problemas que ya fueron corregidos, porque los resultados de herramientas viejos (mostrando el código sin arreglar) siguen en el historial. Las respuestas se vuelven contradictorias — a veces se refieren al código viejo (resultados obsoletos), a veces al nuevo (lecturas frescas).

**Solución aplicada**: iniciar una sesión nueva con un resumen — "El análisis previo identificó tres problemas de autenticación en `auth.ts`, `session.ts` y `middleware.ts`. Los tres fueron corregidos. Por favor re-analiza estos tres archivos para verificar las correcciones y revisar si se introdujeron problemas nuevos." El agente lee los archivos actuales sin ningún resultado obsoleto de por medio, verifica las correcciones y da consejos consistentes basados en el estado real actual.

## Trampas de examen

> [!warning] Trampa 1 — sugerir re-exploración completa cuando solo cambiaron unos pocos archivos
> Re-explorar un codebase de 50 archivos cuando solo 3 cambiaron es un desperdicio. La respuesta correcta es informar al agente sobre esos 3 archivos específicos para un re-análisis dirigido.

> [!warning] Trampa 2 — recomendar `--resume` después de que los archivos fueron modificados
> Reanudar preserva resultados de herramientas obsoletos en el historial de conversación. El agente puede razonar con contenidos de archivo desactualizados, produciendo consejos contradictorios. Un inicio limpio con resumen inyectado evita esto.

> [!warning] Trampa 3 — confundir `fork_session` con `--resume`
> `fork_session` inicia una sesión nueva a partir de una **copia** del historial existente, dejando el original intacto. `--resume` sigue escribiendo sobre la misma sesión. Fork es para divergencia, resume es para continuación.

> [!warning] Trampa 4 — usar `fork_session` para manejar contexto obsoleto tras cambios en archivos
> `fork_session` bifurca desde la sesión existente, que **todavía contiene** los resultados de herramientas obsoletos. La bifurcación hereda ese contexto obsoleto. La respuesta correcta ante archivos modificados es un inicio limpio con resumen inyectado, no un fork.

## Fuentes citadas por la guía
- Claude Certification Guide — módulo "Session State and Resumption" (`/learn/1-agentic-architecture/1-7-session-state-resumption`)
- `examguide.pdf` — Task Statement 1.7: "Manage session state, resumption, and forking"
- Claude Code Documentation — Anthropic
- Claude Code in Action (Skilljar) — Anthropic
- Claude Agent SDK Overview — Anthropic
- Agent SDK: Work with sessions — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
