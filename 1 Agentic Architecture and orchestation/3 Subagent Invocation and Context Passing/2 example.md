Aplicación práctica de [[1 resumen]] — configuración de un coordinador con Claude Agent SDK (Python) que invoca subagentes vía Task tool, pasa contexto con metadata estructurada, invoca en paralelo, y bifurca sesiones con `fork_session`.

## Escenario

El mismo sistema de investigación de energías renovables del tema anterior, pero ahora enfocado en la **mecánica de invocación**: configurar `allowedTools` correctamente, definir cada subagente con `AgentDefinition`, pasar hallazgos con metadata estructurada al agente de síntesis, invocar los subagentes de búsqueda en paralelo, y usar `fork_session` para comparar dos enfoques de síntesis sin que uno afecte al otro.

## Paso 1 — Habilitar el Task tool en el coordinador (el requisito binario)

Sin `"Task"` (o `"Agent"`) en `allowedTools`, el coordinador no puede invocar ningún subagente. Es lo primero que se configura, antes de definir un solo subagente.

```python
from claude_agent_sdk import ClaudeAgentOptions, AgentDefinition, query

opciones_coordinador = ClaudeAgentOptions(
    model="claude-sonnet-5",
    allowedTools=["Task"],  # requisito binario: sin esto, no hay subagentes
    agents={},  # se llena en el paso 2
)
```

> [!danger] Pregunta trampa — omitir Task/Agent de `allowedTools`
> ```python
> # ❌ NO HACER: coordinador sin "Task" en allowedTools
> opciones_coordinador_malo = ClaudeAgentOptions(
>     model="claude-sonnet-5",
>     allowedTools=["WebSearch", "Read"],  # falta "Task"/"Agent"
>     agents={"buscador": ...},  # no importa qué tan bien definido esté
> )
> ```
> **¿Por qué sería mala idea?** Aunque los subagentes estén perfectamente definidos en `agents`, el coordinador físicamente no puede invocarlos sin `Task` (o `Agent`) en `allowedTools` — cada intento de invocación se manda al callback de permisos, que en una ejecución desatendida la deniega. Es una compuerta binaria, no un detalle de configuración opcional.

## Paso 2 — Definir cada subagente con `AgentDefinition`

Cada subagente lleva descripción, system prompt y restricción de herramientas — nada más de lo que su rol necesita.

```python
agente_busqueda_web = AgentDefinition(
    description="Investiga un subtema específico en fuentes web y devuelve hallazgos con URL y título.",
    prompt=(
        "Eres un agente de búsqueda web. Investiga a fondo el subtema que se te dé "
        "y devuelve cada hallazgo con su claim, su source_url y un confidence level."
    ),
    tools=["WebSearch"],  # solo lo que necesita para su rol
)

agente_analisis_documentos = AgentDefinition(
    description="Analiza documentos internos y devuelve hallazgos con nombre de documento y página.",
    prompt=(
        "Eres un agente de análisis de documentos. Extrae hallazgos relevantes del "
        "documento que se te dé, incluyendo document_name y page_number para cada uno."
    ),
    tools=["Read"],  # solo lectura de documentos, no búsqueda web
)

agente_sintesis = AgentDefinition(
    description="Combina hallazgos de otros agentes en un reporte citado.",
    prompt=(
        "Eres un agente de síntesis. Combina los hallazgos recibidos en un reporte "
        "coherente. Cada afirmación debe incluir su fuente (URL o documento+página)."
    ),
    tools=[],  # no necesita herramientas, solo procesa lo que el coordinador le pasa
)

opciones_coordinador = ClaudeAgentOptions(
    model="claude-sonnet-5",
    allowedTools=["Task"],
    agents={
        "buscador_web": agente_busqueda_web,
        "analista_documentos": agente_analisis_documentos,
        "sintesis": agente_sintesis,
    },
)
```

## Paso 3 — Formato de metadata estructurada para los hallazgos

Antes de invocar nada, se define el contrato de datos: cada hallazgo separa `claim` (contenido) de su atribución (metadata).

```python
from typing import TypedDict, Literal

class Finding(TypedDict):
    claim: str                 # contenido
    source_url: str | None     # metadata
    document_name: str | None  # metadata
    page_number: int | None    # metadata
    confidence: Literal["low", "medium", "high"]
    retrieved_by: str
```

## Paso 4 — Invocar subagentes independientes en paralelo (Regla de latencia)

`buscador_web` y `analista_documentos` no dependen entre sí — se invocan en una sola llamada, no en turnos separados.

```python
async def investigar_subtema(subtema: str) -> list[Finding]:
    # una sola llamada a query() que dispara ambas invocaciones del Task tool
    # en la misma respuesta del coordinador
    resultado = await query(
        prompt=(
            f"Investiga el subtema '{subtema}' usando en paralelo el agente "
            f"buscador_web y el agente analista_documentos. Devuelve sus hallazgos "
            f"sin combinarlos todavía."
        ),
        options=opciones_coordinador,
    )
    return _parsear_findings(resultado)
```

> [!danger] Pregunta trampa — invocación secuencial de tareas independientes
> ```python
> # ❌ NO HACER: forzar al coordinador a esperar un subagente antes de invocar el otro
> async def investigar_subtema_malo(subtema: str) -> list[Finding]:
>     hallazgos_web = await query(
>         prompt=f"Invoca solo a buscador_web para '{subtema}' y espera su resultado.",
>         options=opciones_coordinador,
>     )
>     # el coordinador recién ahora invoca al segundo, en un turno separado
>     hallazgos_docs = await query(
>         prompt=f"Invoca solo a analista_documentos para '{subtema}'.",
>         options=opciones_coordinador,
>     )
>     return hallazgos_web + hallazgos_docs
> ```
> **¿Por qué sería mala idea?** `buscador_web` y `analista_documentos` no dependen entre sí — no hay ninguna razón para que uno espere al otro. Invocarlos en turnos separados agrega latencia por nada; la forma correcta es emitir ambas llamadas al Task tool en una sola respuesta del coordinador (Regla de invocación paralela del resumen).

## Paso 5 — Pasar hallazgos completos y con metadata al agente de síntesis

Esta es la Regla 1 y la Regla 2 del resumen aplicadas juntas: hallazgos completos, con contenido y metadata separados pero ambos presentes.

```python
import json

async def sintetizar(hallazgos: list[Finding]) -> str:
    # el coordinador pasa el array COMPLETO de findings, con toda su metadata intacta
    resultado = await query(
        prompt=(
            "Combina estos hallazgos en un reporte citado. Cada afirmación debe "
            "atribuirse a su fuente exacta.\n\n"
            f"Hallazgos:\n{json.dumps(hallazgos, indent=2)}"
        ),
        options=opciones_coordinador,
    )
    return _texto(resultado)
```

> [!danger] Pregunta trampa — pasar contenido sin metadata (la falla de atribución del resumen)
> ```python
> # ❌ NO HACER: extraer solo el texto del claim y perder la metadata en el camino
> async def sintetizar_malo(hallazgos: list[Finding]) -> str:
>     solo_texto = [h["claim"] for h in hallazgos]  # se descarta source_url, document_name, page_number
>     resultado = await query(
>         prompt=f"Combina estos hallazgos en un reporte:\n\n{chr(10).join(solo_texto)}",
>         options=opciones_coordinador,
>     )
>     return _texto(resultado)
> ```
> **¿Por qué sería mala idea?** El agente de síntesis va a producir un resumen bien escrito, pero **sin ninguna cita** — no porque su prompt esté mal diseñado, sino porque literalmente no tiene `source_url`, `document_name` ni `page_number` para incluir. Este es el patrón de examen más específico del tema: la causa raíz es la metadata perdida en el paso de contexto, no el prompt del agente de síntesis.

> [!danger] Pregunta trampa — "arreglar" el problema tocando el prompt de síntesis en vez del paso de contexto
> ```python
> # ❌ NO HACER: agregar instrucciones de "por favor cita tus fuentes" al agente que nunca las recibió
> agente_sintesis_parcheado = AgentDefinition(
>     description="Combina hallazgos de otros agentes en un reporte citado.",
>     prompt=(
>         "Eres un agente de síntesis. IMPORTANTE: siempre incluye citas y fuentes "
>         "para cada afirmación que hagas."  # el prompt no puede inventar datos que no existen
>     ),
>     tools=[],
> )
> ```
> **¿Por qué sería mala idea?** Ninguna instrucción en el prompt del agente de síntesis puede generar `source_url` o `page_number` que nunca llegaron en su contexto de entrada. El fix vive en `sintetizar()` (pasar el objeto `Finding` completo), no en el prompt de `agente_sintesis`.

## Paso 6 — Diseñar el prompt del coordinador orientado a objetivos, no a procedimientos

```python
prompt_coordinador_bueno = (
    "Objetivo: producir un reporte de investigación sobre energía solar que cumpla "
    "estos criterios de calidad: (1) toda afirmación tiene fuente atribuida, "
    "(2) cubre avances técnicos y adopción de mercado, (3) usa fuentes de los "
    "últimos 2 años cuando estén disponibles."
)
```

> [!danger] Pregunta trampa — prompt procedimental que encasilla al subagente
> ```python
> # ❌ NO HACER: dictarle el paso a paso exacto, sin margen de adaptación
> prompt_coordinador_malo = (
>     "Paso 1: busca 'solar panel efficiency 2024' en Google. "
>     "Paso 2: busca 'solar panel cost 2024' en Google. "
>     "Paso 3: copia los primeros 3 resultados de cada búsqueda. "
>     "Paso 4: pégalos en el reporte sin resumir."
> )
> ```
> **¿Por qué sería mala idea?** Si el subagente encuentra que esas dos búsquedas específicas no traen resultados útiles, un prompt procedimental no le deja margen para adaptarse — está encasillado a seguir pasos que ya no tienen sentido. Un prompt orientado a objetivos (qué lograr, qué criterios cumplir) le permite ajustar su enfoque ante lo inesperado.

## Paso 7 — `fork_session` para comparar dos enfoques de síntesis

Después del análisis inicial (los hallazgos ya recolectados), se bifurca la sesión para comparar dos estilos de síntesis sin que uno vea al otro.

```python
async def comparar_enfoques_de_sintesis(session_id: str):
    # rama A: síntesis con enfoque ejecutivo
    rama_ejecutiva = ClaudeAgentOptions(
        model="claude-sonnet-5",
        resume=session_id,
        fork_session=True,  # bifurca, no continúa la sesión original
    )
    reporte_ejecutivo = await query(
        prompt="Sintetiza los hallazgos en un resumen ejecutivo de una página.",
        options=rama_ejecutiva,
    )

    # rama B: síntesis con enfoque técnico detallado — independiente de la rama A
    rama_tecnica = ClaudeAgentOptions(
        model="claude-sonnet-5",
        resume=session_id,
        fork_session=True,
    )
    reporte_tecnico = await query(
        prompt="Sintetiza los hallazgos en un reporte técnico detallado con metodología.",
        options=rama_tecnica,
    )

    # ambas ramas parten de la misma base de análisis, pero no se ven entre sí
    return reporte_ejecutivo, reporte_tecnico
```

> [!danger] Pregunta trampa — confundir `fork_session` con simplemente usar `--resume`
> ```python
> # ❌ NO HACER: creer que resume, por sí solo, crea una rama independiente
> rama_mala = ClaudeAgentOptions(
>     model="claude-sonnet-5",
>     resume=session_id,  # sin fork_session=True
> )
> # esto AGREGA a la sesión original — no crea una rama nueva
> ```
> **¿Por qué sería mala idea?** Sin `fork_session=True`, `resume` simplemente continúa (agrega a) la sesión original — no bifurca nada. Si el objetivo es comparar dos enfoques sin que uno contamine al otro, hace falta `fork_session=True` explícitamente junto a `resume`; de lo contrario ambas "ramas" terminarían escribiendo sobre la misma sesión.

## Resultado

```python
import asyncio

async def main():
    hallazgos = await investigar_subtema("paneles solares de alta eficiencia")
    reporte = await sintetizar(hallazgos)
    print(reporte)
    # reporte con cada afirmación atribuida a su source_url o document_name+page_number,
    # producido por dos subagentes invocados en paralelo y un agente de síntesis
    # que recibió hallazgos completos con metadata intacta

asyncio.run(main())
```

El flujo completo aplicó: Task tool habilitado como requisito binario en `allowedTools`, subagentes definidos con `AgentDefinition` (description, prompt, tools), paso de contexto con hallazgos completos y metadata estructurada, invocación paralela de tareas independientes, prompts de coordinador orientados a objetivos, y `fork_session` para explorar enfoques divergentes desde una base de análisis compartida — evitando explícitamente las trampas de examen del resumen.
