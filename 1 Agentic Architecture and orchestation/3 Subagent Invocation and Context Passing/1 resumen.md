> [!note] En una frase
> El coordinador solo puede invocar subagentes si tiene el "Task tool" (hoy llamado `Agent`) habilitado en `allowedTools` — y una vez invocados, cada subagente solo sabe lo que el coordinador le escribió explícitamente en el prompt, así que perder la metadata (fuente, documento, página) al pasar contexto es la causa número uno de reportes sin citas.

## Explícamelo como si tuviera 5 años

Piensa en el jefe de cocina del tema anterior, pero ahora fíjate en los detalles de cómo *físicamente* le pasa la comanda a un cocinero. Primero, el jefe necesita tener colgado en su cinturón el silbato correcto — si no lo tiene, no puede llamar a ningún cocinero, punto, sin importar cuántas ganas tenga. Segundo, cuando por fin llama a un cocinero, no le dice solo "haz el plato" — tiene que escribirle en un papelito *exactamente* qué ingredientes usar y de dónde vienen (no solo "usa camarones", sino "camarones del proveedor X, lote 12, revisado el lunes"). Si el jefe le pasa el papelito sin el origen de los ingredientes, el cocinero no tiene manera de decir después "estos camarones son del proveedor X" — simplemente no lo sabe.

Ese "silbato obligatorio" es el Task tool en `allowedTools`. Y ese "papelito con origen de cada ingrediente" es pasar contexto con metadata estructurada, no solo el contenido pelado.

## Argumento central

> El examen evalúa la **mecánica concreta** de cómo un coordinador invoca subagentes (Task tool como requisito binario en `allowedTools`) y cómo les pasa contexto (aislamiento total salvo transmisión explícita, con metadata estructurada para preservar atribución). Si el tema 1.2 fue la arquitectura, el 1.3 es el cableado: la API real, la configuración exacta, y el patrón de falla más probado del examen — un agente de síntesis sin citas por culpa de metadata perdida, no por su propio prompt.

## Ideas clave (Conclusiones)

### 1. El Task tool: el requisito binario para invocar subagentes

- El **Task tool** es el mecanismo real de la API con el que un coordinador genera (spawns) subagentes — no es solo una convención de nombres, es la pieza de la que depende toda la orquestación multi-agente.
- **Regla dura**: `allowedTools` del coordinador debe incluir `"Task"` (o `"Agent"`, su nombre actual). Sin esto, el coordinador **no puede invocar ningún subagente**, sin importar cómo estén definidos.
- Es una compuerta binaria, no una preferencia suave: está o no está.

> [!note] Task vs. Agent — nombres, misma cosa
> Claude Code actual (v2.1.63, febrero 2026) renombró la herramienta de `Task` a `Agent`; `Task` sigue funcionando como alias, y el Agent SDK emite `Agent` en los bloques de tool-use. **Para el examen, la respuesta correcta dice "Task tool"** — es lo que fija la guía oficial (v1.0) — aunque el código actual muestre "Agent".

> [!note] Cómo funciona esto en la práctica actual (septiembre 2026)
> La documentación actual del Agent SDK describe `allowedTools` como una lista de auto-aprobación: no incluir `Agent` ahí no elimina la herramienta, sino que manda cada invocación por el callback de permisos — que en una ejecución desatendida la deniega. El resultado práctico es el mismo que dice el examen (el coordinador no puede invocar subagentes), solo que el mecanismo interno cambió ligeramente.

### 2. AgentDefinition: la configuración de cada subagente

Cada subagente se define con tres componentes:

1. **Description** — qué hace el subagente (el coordinador la usa para decidir cuándo invocarlo).
2. **System prompt** — las instrucciones que el subagente sigue.
3. **Tool restrictions** — qué herramientas puede usar, limitadas a su rol específico.

> [!note] Analogía
> Es como la ficha de un cocinero especializado: su título (qué cocina), su receta base (cómo cocina), y qué utensilios tiene permitido usar (para que el de postres no termine con acceso al horno de pizzas).

### 3. Paso de contexto: el detalle que hace o deshace el sistema

El principio de aislamiento de contexto del tema [[2 Multi-Agent Orchestration/1 resumen|Multi-Agent Orchestration]] aplica sin excepción: los subagentes **solo** reciben lo que el coordinador escribe en su prompt. Nada más. De ahí se derivan tres reglas prácticas:

- **Regla 1 — Incluir hallazgos completos de agentes previos.** Si el agente de síntesis necesita los resultados de búsqueda web y de análisis de documentos, el coordinador debe pasarle **ambos, completos**, en su prompt. No se puede asumir que el agente de síntesis "va a buscarlos" en ningún lado — no puede.
- **Regla 2 — Usar formatos de datos estructurados que separen contenido de metadata.** Al pasar hallazgos de investigación entre agentes, los datos deben incluir tanto el **contenido** (la afirmación, el hecho, el análisis) como la **metadata** (URL de la fuente, nombre del documento, número de página). Pasar contenido sin metadata deja al agente receptor sin forma de atribuir sus afirmaciones a una fuente.
- **Regla 3 — Diseñar prompts de coordinador orientados a objetivos, no a procedimientos.** El prompt del coordinador debe decirle al subagente **qué lograr** y **qué criterios de calidad cumplir**, no una secuencia paso a paso de cómo hacerlo. Los prompts orientados a objetivos permiten que el subagente se adapte; las instrucciones procedimentales lo encasillan y le impiden ajustar su enfoque ante situaciones inesperadas.

> [!warning] El patrón de examen más específico de este tema
> Un agente de síntesis produce un reporte con afirmaciones **sin fuente**. Los agentes de búsqueda web y de análisis de documentos están funcionando correctamente. La causa raíz es que **el coordinador pasó contenido sin metadata estructurada** — el agente de síntesis literalmente no tenía información de fuente que incluir.

## Evidencia — Formato de metadata estructurada

La guía muestra un formato concreto para separar contenido de metadata al pasar hallazgos entre agentes:

```json
{
  "findings": [
    {
      "claim": "Solar panel efficiency has increased 25% in the last decade",
      "source_url": "https://example.com/solar-report",
      "document_name": "Annual Solar Industry Report 2024",
      "page_number": 14,
      "confidence": "high",
      "retrieved_by": "web_search_agent"
    }
  ]
}
```

Cada hallazgo lleva su atribución de fuente como metadata. Con esta estructura, el agente de síntesis tiene todo lo necesario para producir un reporte con citas correctas.

### Ejemplo práctico: falla de atribución

Un sistema de investigación tiene tres agentes: búsqueda web, análisis de documentos, y síntesis. Búsqueda web devuelve resultados bien citados (URLs, títulos). Análisis de documentos devuelve análisis con referencias de página. El coordinador les pasa el contenido al agente de síntesis, pero **le quita la metadata** — envía las afirmaciones y el análisis en texto plano, sin URLs, nombres de documento ni números de página. El resultado: un resumen excelente, sin ninguna atribución.

**La solución no es tocar el prompt del agente de síntesis** (no puede citar fuentes que no tiene). La solución es que el coordinador pase la metadata estructurada junto al contenido, preservando URL, nombre de documento y número de página para cada hallazgo.

### 4. Invocación paralela vs. secuencial

- Cuando el coordinador necesita invocar varios subagentes para tareas **independientes**, debe **emitir múltiples llamadas al Task tool en una sola respuesta**, en vez de invocarlas una por una en turnos separados.
- La invocación secuencial de subagentes independientes agrega latencia sin ninguna ganancia: si el agente de búsqueda web y el de análisis de documentos trabajan de forma independiente, no hay razón para que uno espere al otro.

> [!note] Cómo detectar la respuesta correcta en el examen
> El examen evalúa conciencia de latencia. Ante tareas de subagentes independientes, la respuesta correcta involucra invocación paralela. Busca opciones que mencionen "en una sola respuesta" o "simultáneamente" — son la señal del patrón paralelo.

### 5. `fork_session`: ramas independientes desde una base compartida

- `fork_session` crea **ramas independientes a partir de una línea base de análisis compartida**. Después de que un coordinador completa un análisis inicial (leer un código, entender un problema), puede bifurcar la sesión para explorar enfoques divergentes.
- Ejemplo: tras analizar un código, el coordinador bifurca para comparar dos estrategias de testing distintas. Cada rama opera de forma independiente después del punto de bifurcación — no ven los resultados de la otra, y los cambios en una rama no afectan a la otra.

> [!warning] `fork_session` no es lo mismo que `--resume`
> **`--resume`** continúa una sesión nombrada específica — **agrega** a esa misma sesión. **`fork_session`** crea una rama nueva e independiente — **bifurca** en vez de continuar. Usa fork cuando necesites exploración divergente desde un punto de partida compartido; usa resume cuando quieras continuar la misma línea de investigación.

> [!note] Cómo se combinan en la práctica actual (septiembre 2026)
> En el Agent SDK, fork no es una alternativa a resume sino un modificador de resume: se pasan los dos juntos (`ClaudeAgentOptions(resume=session_id, fork_session=True)`). `resume` nombra la sesión de partida; `fork_session` indica bifurcar en vez de continuar sobre ella — si se omite, la misma llamada simplemente sigue agregando a la sesión original. El CLI funciona igual: `--fork-session` solo hace algo junto a `--resume` o `--continue`. La distinción real que evalúa el examen es **agregar vs. bifurcar**, no "un flag contra el otro". Para el examen, responde como lo enmarca la guía: resume para continuar una sesión, fork para bifurcarla.

## Trampas de examen

> [!warning] Trampa 1 — Asumir herencia automática del historial del coordinador o de resultados de otros subagentes
> Suponer que un subagente tiene acceso automático al historial de conversación del coordinador o a la salida de otros subagentes. **Por qué falla:** los subagentes tienen contexto aislado; cada pieza de información que necesitan debe incluirse explícitamente en su prompt por el coordinador. No hay herencia automática de contexto. **La forma correcta:** transmisión deliberada de cada dato necesario.

> [!warning] Trampa 2 — Culpar al agente de síntesis por citas faltantes cuando el problema real es la metadata
> Asumir que el agente de síntesis "no citó bien" por un fallo de su prompt. **Por qué falla:** el agente de síntesis solo puede citar fuentes que se le dieron; si el coordinador pasó contenido sin URLs ni nombres de documento, literalmente no puede producir citas. **La forma correcta:** identificar la falla de paso de contexto sin metadata estructurada — no tocar el prompt del agente de síntesis ni darle acceso directo a herramientas de búsqueda.

> [!warning] Trampa 3 — Proponer invocación secuencial para tareas que pueden correr de forma independiente
> Sugerir invocar subagentes uno por uno cuando sus tareas no dependen entre sí. **Por qué falla:** la invocación secuencial introduce latencia innecesaria. **La forma correcta:** invocar los subagentes independientes en paralelo, emitiendo múltiples llamadas al Task tool en una sola respuesta del coordinador.

> [!warning] Trampa 4 — Confundir `fork_session` con `--resume`
> Tratar ambos como si fueran intercambiables o como opciones mutuamente excluyentes. **Por qué falla:** `fork_session` bifurca — arranca una sesión nueva desde una copia del historial original; `--resume` continúa — agrega a la misma sesión. En el SDK, el flag de fork se coloca junto al de resume, así que la elección real es agregar o bifurcar, no "un comando u otro". **La forma correcta:** fork para comparar enfoques divergentes, resume para seguir la misma línea de trabajo.

## Fuentes citadas por la guía
- Claude Certification Guide — módulo "Subagent Invocation and Context Passing" (`/learn/1-agentic-architecture/1-3-subagent-invocation-context`)
- `examguide.pdf` — Task Statement 1.3: "Configure subagent invocation, context passing, and spawning"
- Claude Agent SDK Overview — Anthropic
- Agent SDK: Work with sessions — Anthropic

---
> [!tip] Sigue con este tema
> Repasa con [[3 cuestionario]], aplícalo en código en [[2 example]], y evalúate con [[4 test]].
