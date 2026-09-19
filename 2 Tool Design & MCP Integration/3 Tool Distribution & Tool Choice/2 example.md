---
tags:
  - claude-cert/dominio-2
  - task-statement/2.3
---

# 2 example — Tool Distribution & Tool Choice

Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, la distribución de tools de un sistema multi-agente de investigación (búsqueda web, análisis de documentos, síntesis, coordinador), usando la SDK de Anthropic en Python — aplicando las siete conclusiones del resumen: sobrecarga de tools, mal uso por falta de especialización, consolidación de tools casi-duplicadas, los tres modos de `tool_choice`, tools acotadas cruzando roles, reemplazo de tools genéricas por acotadas, y la distribución final por rol.

## Paso 1 — Consolidar tools casi-duplicadas en una sola tool parametrizada

Antes de repartir tools por rol, aplicamos la [[1 resumen#3. Consolidar tools casi-duplicadas en vez de solo dividir por rol|Conclusión 3]]: 19 operaciones de transformación de datos comparten el mismo patrón (entrada → operación → salida), así que las colapsamos en una sola tool con un parámetro `enum`.

```python
transform_data_tool = {
    "name": "transform_data",
    "description": (
        "Applies a single transformation operation to a tabular dataset. "
        "Use this instead of looking for a separate tool per operation."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "transform_type": {
                "type": "string",
                "enum": [
                    "pivot", "percentile", "normalise_currency",
                    "deduplicate", "aggregate_sum", "fill_missing",
                    # ... hasta cubrir las 19 operaciones originales
                ],
                "description": "Which transformation to apply.",
            },
            "dataset_ref": {"type": "string", "description": "Reference to the input dataset."},
            "params": {"type": "object", "description": "Operation-specific parameters."},
        },
        "required": ["transform_type", "dataset_ref"],
    },
}
```

> [!warning] Pregunta trampa en código — mantener 19 tools separadas "para que cada una tenga su propio schema claro"
> La forma ingenua sería definir `pivot_table_tool`, `calculate_percentile_tool`, `normalise_currency_tool`, etc. como 19 tools independientes, argumentando que así cada schema es más específico. **Por qué es mala idea:** el modelo tiene que elegir entre diecinueve descripciones casi idénticas cada vez que necesita transformar datos, exactamente el escenario de la Conclusión 3 que degrada la selección. Consolidarlas en un enum no pierde ninguna operación — solo mueve la elección a un parámetro dentro de una sola llamada.

## Paso 2 — Configurar `tool_choice` según el objetivo de cada llamada

Aplicamos la [[1 resumen#4. Configuración de tool_choice tres modos, tres trabajos distintos|Conclusión 4]]: no hay un solo `tool_choice` correcto, depende de si se necesita flexibilidad, salida estructurada garantizada, o un paso obligatorio.

```python
import anthropic

client = anthropic.Anthropic()

# Modo "auto" — el agente de síntesis puede decidir libremente si necesita
# llamar compile_report o simplemente responder con un resumen conversacional.
response_auto = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[compile_report_tool, verify_fact_tool, format_citation_tool, assess_coverage_tool],
    tool_choice={"type": "auto"},
    messages=[{"role": "user", "content": "Draft the synthesis section for the AI-in-art report."}],
)

# Modo "any" — un pipeline de extracción de documentos donde el tipo de
# documento no se conoce de antemano, pero SIEMPRE debe producir datos
# estructurados (nunca una respuesta conversacional).
response_any = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[extract_invoice_tool, extract_receipt_tool, extract_contract_tool],
    tool_choice={"type": "any"},
    messages=[{"role": "user", "content": "Extract structured data from this uploaded document."}],
)

# Selección forzada — el pipeline exige que extract_metadata corra ANTES
# que cualquier tool de enriquecimiento, sin excepción.
response_forced = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    tools=[extract_metadata_tool, enrich_with_tags_tool, enrich_with_summary_tool],
    tool_choice={"type": "tool", "name": "extract_metadata"},
    messages=[{"role": "user", "content": "Process this new document before enriching it."}],
)
# En el siguiente turno, tras procesar el resultado de extract_metadata,
# se puede volver a tool_choice={"type": "auto"} para los pasos de enriquecimiento.
```

> [!warning] Pregunta trampa en código — usar `tool_choice: auto` en el pipeline de extracción
> La forma ingenua sería usar `{"type": "auto"}` para `response_any`, asumiendo que "el modelo va a llamar una tool de todas formas porque el prompt lo pide". **Por qué es mala idea:** con `auto` nada garantiza que el modelo llame una tool — podría responder con texto conversacional describiendo el documento en vez de producir datos estructurados, rompiendo el pipeline. `any` es lo que garantiza la invocación (dejando la elección de cuál tool al modelo).

## Paso 3 — Reemplazar una tool genérica por una alternativa acotada

Aplicamos la [[1 resumen#6. Reemplazar tools genéricas por alternativas acotadas (principio de mínimo privilegio)|Conclusión 6]]: en vez de `fetch_url`, definimos `load_document`, que solo valida URLs de documentos.

```python
import re
from urllib.parse import urlparse

ALLOWED_DOCUMENT_EXTENSIONS = {".pdf", ".docx", ".txt", ".md"}


def load_document(url: str) -> str:
    """Loads a document from a URL, restricted to known document types."""
    parsed = urlparse(url)
    extension = re.search(r"\.\w+$", parsed.path)

    if not extension or extension.group().lower() not in ALLOWED_DOCUMENT_EXTENSIONS:
        raise ValueError(
            f"'{url}' does not look like a document URL. "
            f"load_document only accepts {sorted(ALLOWED_DOCUMENT_EXTENSIONS)}."
        )

    return f"<content of document at {url}>"  # en la práctica: descarga y extracción real


load_document_tool = {
    "name": "load_document",
    "description": (
        "Loads the contents of a document (PDF, DOCX, TXT, MD) from a URL. "
        "Rejects URLs that are not recognizable document files."
    ),
    "input_schema": {
        "type": "object",
        "properties": {"url": {"type": "string", "description": "URL of the document to load."}},
        "required": ["url"],
    },
}
```

> [!warning] Pregunta trampa en código — darle al agente de análisis de documentos una tool `fetch_url` genérica "por flexibilidad"
> La forma ingenua sería exponer una tool `fetch_url(url: str) -> str` sin restricciones, razonando que "así puede traer lo que necesite, documentos o cualquier otra cosa". **Por qué es mala idea** (Trampa de examen 4 del resumen): esto viola mínimo privilegio — el agente podría usarla para acceder a recursos completamente fuera de su rol (APIs internas, páginas arbitrarias), y la descripción genérica no comunica cuál es el uso previsto. `load_document` logra lo mismo para el caso real que el agente necesita, sin la superficie de mal uso.

## Paso 4 — Tool acotada cruzando roles: `verify_fact` en el agente de síntesis

Aplicamos la [[1 resumen#5. Tools acotadas cruzando roles (scoped cross-role tools)|Conclusión 5]]: en vez de que toda verificación pase por el coordinador, el agente de síntesis resuelve localmente el 85% de los casos simples.

```python
def verify_fact(claim: str, source_snippet: str) -> dict:
    """
    Scoped tool: resolves single-source, simple factual lookups
    (dates, names, statistics) without involving the coordinator.
    NOT meant for multi-source cross-referencing or substantial judgment calls —
    those still escalate through the coordinator to the full search agent.
    """
    is_supported = claim.lower() in source_snippet.lower()  # heurística simplificada
    return {"claim": claim, "supported_by_source": is_supported, "confidence": "high" if is_supported else "low"}


verify_fact_tool = {
    "name": "verify_fact",
    "description": (
        "Verifies a simple, single-source factual claim (a date, a name, a statistic) "
        "against a provided source snippet. For complex verifications requiring multiple "
        "sources or cross-referencing, escalate to the coordinator instead."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "claim": {"type": "string"},
            "source_snippet": {"type": "string"},
        },
        "required": ["claim", "source_snippet"],
    },
}
```

> [!warning] Pregunta trampa en código — rutear toda verificación al coordinador por "simplicidad de diseño"
> La forma ingenua sería no darle `verify_fact` al agente de síntesis y hacer que, para cualquier verificación (simple o compleja), regrese control al coordinador, que invoca al agente de búsqueda y vuelve a invocar síntesis con el resultado. **Por qué es mala idea** (Trampa de examen 1 del resumen, y Sample Question 9 de `examguide.pdf`): con 85% de verificaciones siendo simples, esto agrega 2-3 idas y vueltas innecesarias por tarea y hasta 40% más de latencia, cuando una tool acotada local resolvería la mayoría en milisegundos.

> [!warning] Pregunta trampa en código — darle al agente de síntesis todas las tools de búsqueda web en vez de solo `verify_fact`
> Otra forma ingenua sería, para evitar cualquier escalación, darle al agente de síntesis acceso completo a `search_web`, `fetch_page`, `extract_links` y `save_snippet` del agente de búsqueda. **Por qué es mala idea:** esto sobre-provisiona al agente de síntesis, violando separación de responsabilidades — y abre la puerta al problema de la Conclusión 2 (un agente con tools fuera de su rol tiende a mal-usarlas, por ejemplo repitiendo investigación completa en vez de solo verificar un dato puntual).

## Paso 5 — Distribución final de tools por rol en el sistema multi-agente

Aplicamos la [[1 resumen#7. Distribución de tools en la práctica un sistema multi-agente de investigación|Conclusión 7]]: cada rol recibe exactamente 4-5 tools acotadas a su función.

```python
AGENT_TOOLSETS = {
    "web_search_agent": [
        "search_web", "fetch_page", "extract_links", "save_snippet",
    ],
    "document_analysis_agent": [
        "extract_metadata", "extract_data_points", "summarize_content", "verify_claim",
    ],
    "synthesis_agent": [
        "compile_report", "verify_fact", "format_citation", "assess_coverage",  # verify_fact = tool acotada
    ],
    "coordinator_agent": [
        "Agent",  # invoca subagentes
        "review_output", "request_revision",
    ],  # sin tools de dominio: orquesta, no ejecuta investigación
}


def build_tools_for_role(role: str, tool_registry: dict) -> list[dict]:
    """Devuelve solo las tool definitions correspondientes al rol — nunca el registro completo."""
    return [tool_registry[name] for name in AGENT_TOOLSETS[role]]
```

> [!warning] Pregunta trampa en código — pasar el registro completo de tools a cada agente "por si acaso"
> La forma ingenua sería definir un único `ALL_TOOLS` con las 15+ tools del sistema y pasarlo tal cual a cada agente, dejando que "el prompt de sistema le diga cuáles usar". **Por qué es mala idea** (Trampa de examen 3 del resumen): aunque el prompt intente limitar el uso, el modelo sigue viendo todas las tools disponibles en cada llamada, lo cual sube la complejidad de decisión y el riesgo de mal uso cruzado de roles (Conclusión 2) — filtrar el toolset por rol antes de la llamada (`build_tools_for_role`) es lo que realmente lo previene, no una instrucción en texto.

Con esto, el sistema completo respeta las siete conclusiones del resumen: cada agente tiene 4-5 tools acotadas a su rol, las transformaciones casi-duplicadas viven en una sola tool parametrizada, `tool_choice` se elige según si se necesita flexibilidad, garantía de estructura, o un paso obligatorio, las tools genéricas se reemplazan por versiones acotadas, y la única capacidad cruzada de roles (`verify_fact`) está deliberadamente limitada a los casos simples y frecuentes.
