---
tags:
  - claude-cert/dominio-2
  - task-statement/2.4
---

# 2 example — MCP Server Integration

Aplicación práctica de [[1 resumen]]

Vamos a construir, paso a paso, la configuración de servidores MCP de un equipo real: un servidor compartido de GitHub y otro de base de datos interna a nivel de proyecto, un servidor experimental a nivel de usuario, un resource de esquema de base de datos, y el refuerzo de una descripción de tool MCP — aplicando las seis conclusiones del resumen: scoping proyecto vs. usuario, descubrimiento simultáneo, expansión de variables de entorno, resources vs. tools, la decisión build-vs-use, y descripciones reforzadas.

## Paso 1 — Configurar servidores compartidos en `.mcp.json` (nivel de proyecto)

Aplicamos la [[1 resumen#1. La jerarquía de scoping .mcp.json (proyecto) vs. ~claude.json (usuario)|Conclusión 1]]: los servidores que **todo el equipo** necesita van en `.mcp.json`, en la raíz del repositorio, versionado en git.

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "internal_db": {
      "command": "npx",
      "args": ["-y", "@internal-tools/mcp-server-db", "${DATABASE_URL:-postgresql://localhost:5432/dev}"]
    }
  }
}
```

`github` es un servidor **remoto**: declara `"type": "http"` y una `url`, sin `command`. `internal_db` es **local**: declara `command` y `args`, y habla por stdio.

> [!warning] Pregunta trampa en código — declarar una URL sin `"type": "http"`
> La forma ingenua sería escribir `"github": {"url": "https://api.githubcopilot.com/mcp/"}`, asumiendo que Claude Code infiere el tipo de transporte a partir de la presencia de `url`. **Por qué es mala idea:** sin `"type": "http"` explícito, Claude Code interpreta la entrada como servidor stdio, la salta silenciosamente respecto a lo esperado, y solo avisa que falta el `type` — el servidor remoto nunca queda disponible aunque la entrada "se vea" bien formada.

## Paso 2 — Expansión de variables de entorno para credenciales

Aplicamos la [[1 resumen#3. Expansión de variables de entorno para credenciales|Conclusión 3]]: el `${GITHUB_TOKEN}` en el `.mcp.json` de arriba solo funciona si el header de autenticación referencia el nombre de la variable, nunca el valor.

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    }
  }
}
```

Cada desarrollador define `GITHUB_TOKEN` en su propio entorno (fuera del repositorio):

```bash
# En el perfil de shell de cada desarrollador, NO en el repositorio
export GITHUB_TOKEN="ghp_xxxxxxxxxxxxxxxxxxxx"
```

`git diff` sobre `.mcp.json` nunca debería mostrar un token real — solo la sintaxis `${GITHUB_TOKEN}`, que es segura de hacer commit.

> [!warning] Pregunta trampa en código — escribir el token directamente para "que funcione ya"
> La forma ingenua sería `"Authorization": "Bearer ghp_xxxxxxxxxxxxxxxxxxxx"` directamente en `.mcp.json`, razonando que "es más rápido para probar y ya lo cambio después". **Por qué es mala idea** (Trampa de examen 3 del resumen): en cuanto ese archivo se commitea, el token queda en el historial de git permanentemente — incluso si se borra en un commit posterior, sigue siendo recuperable. `${GITHUB_TOKEN}` evita ese riesgo desde el primer commit, sin fricción adicional para el equipo.

## Paso 3 — Servidor experimental a nivel de usuario, en `~/.claude.json`

Aplicamos la [[1 resumen#1. La jerarquía de scoping .mcp.json (proyecto) vs. ~claude.json (usuario)|Conclusión 1]]: un servidor que solo un desarrollador está probando —antes de proponerlo al equipo— va en `~/.claude.json`, fuera del repositorio y fuera de control de versiones.

```json
{
  "mcpServers": {
    "notion_experimental": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-notion"],
      "env": {
        "NOTION_TOKEN": "${NOTION_TOKEN}"
      }
    }
  }
}
```

Este archivo vive en el directorio home del desarrollador (`~/.claude.json`), no en la raíz del repositorio — nunca aparece en `git status` del proyecto.

Aplicamos también la [[1 resumen#2. Descubrimiento simultáneo no hay activación manual|Conclusión 2]]: al conectar, las tools de `github`, `internal_db` (nivel proyecto) **y** `notion_experimental` (nivel usuario) quedan disponibles simultáneamente para el agente de ese desarrollador, sin ningún paso extra de activación.

> [!warning] Pregunta trampa en código — poner `notion_experimental` en el `.mcp.json` del proyecto "para que quede guardado en algún lado"
> La forma ingenua sería agregar el servidor experimental directamente al `.mcp.json` compartido, pensando que así "no se pierde el trabajo de configurarlo". **Por qué es mala idea** (Trampa de examen 2 del resumen): eso lo convierte en configuración compartida con todo el equipo antes de que esté listo — cada teammate que haga pull heredaría un servidor experimental sin evaluar, potencialmente inestable. Un servidor personal o en prueba pertenece a `~/.claude.json`, no al `.mcp.json` versionado.

## Paso 4 — Exponer un resource de esquema de base de datos

Aplicamos la [[1 resumen#4. MCP Resources exponer catálogos de contenido para evitar tool calls exploratorias|Conclusión 4]]: en vez de forzar al agente a descubrir la estructura de la base de datos llamando `list_tables` y luego `describe_table` por cada tabla, exponemos el esquema completo como un **resource**.

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("internal_db")


@mcp.resource("db://schema/main")
def get_database_schema() -> dict:
    """
    Resource: exposes the full schema of the main database up front,
    so the agent never needs exploratory list_tables + describe_table
    calls just to understand what data is available.
    """
    return {
        "name": "main database schema",
        "description": "Tables, columns, and relationships for the main application database.",
        "mimeType": "application/json",
        "tables": {
            "customers": {"columns": {"id": "uuid", "email": "text", "created_at": "timestamptz"}},
            "orders": {"columns": {"id": "uuid", "customer_id": "uuid", "total_cents": "integer"}},
        },
    }


@mcp.tool()
def run_query(sql: str) -> list[dict]:
    """
    Tool: executes a read-only SQL query against the main database.
    Use get_database_schema (a resource, not a tool call) first to
    know which tables and columns exist before writing a query.
    """
    ...  # ejecución real de la consulta
```

`get_database_schema` es un **resource** (datos que el agente puede ver de antemano); `run_query` es una **tool** (una acción que el agente ejecuta). Esa es exactamente la distinción de la Conclusión 4: los resources muestran qué hay, las tools actúan sobre ello.

> [!warning] Pregunta trampa en código — exponer el esquema como una tool `describe_all_tables()` en vez de como resource
> La forma ingenua sería definir `describe_all_tables()` como una tool más, razonando que "total, el agente la puede llamar cuando la necesite". **Por qué es mala idea:** eso sigue requiriendo una llamada exploratoria antes de poder hacer cualquier consulta útil — exactamente el patrón `list_tables` + `describe_table` por tabla que la Conclusión 4 identifica como desperdicio de tool calls. Un resource pone esa información disponible de entrada, sin gastar un turno del agente solo para orientarse.

## Paso 5 — Reforzar la descripción de una tool MCP para competir con Grep

Aplicamos la [[1 resumen#6. Reforzar las descripciones de tools MCP para competir con las tools nativas|Conclusión 6]]: una tool MCP de búsqueda semántica de código, si su descripción es escueta, pierde frente a Grep aunque sea más capaz para el caso de uso correcto.

```python
# Descripción escueta — el modelo tiende a preferir Grep en su lugar
search_codebase_tool_weak = {
    "name": "search_codebase",
    "description": "Searches code",
    "input_schema": {
        "type": "object",
        "properties": {"query": {"type": "string"}},
        "required": ["query"],
    },
}

# Descripción reforzada — suficiente contexto para preferirla sobre Grep
# cuando el caso de uso es búsqueda por intención, no por string exacto
search_codebase_tool_strong = {
    "name": "search_codebase",
    "description": (
        "Performs semantic code search across the entire repository using "
        "AST-aware indexing. Returns matching functions, classes, and methods "
        "with full context including file path, line numbers, and surrounding "
        "code. More accurate than text-based grep for finding code by intent "
        "rather than exact string match. Use this instead of Grep when searching "
        "for code by what it does rather than what it contains."
    ),
    "input_schema": {
        "type": "object",
        "properties": {"query": {"type": "string"}},
        "required": ["query"],
    },
}
```

> [!warning] Pregunta trampa en código — dejar `search_codebase_tool_weak` tal cual y "compensar" con una instrucción en el system prompt
> La forma ingenua sería mantener la descripción escueta de la tool y en su lugar agregar al system prompt algo como "prefiere search_codebase sobre Grep para búsquedas semánticas". **Por qué es mala idea** (Trampa de examen 4 del resumen): el modelo elige entre tools principalmente por lo que dice la descripción de cada una en el momento de decidir, no por una instrucción general en el prompt de sistema compitiendo contra descripciones nativas ya detalladas. Reforzar la descripción de la tool misma es lo que realmente le da al modelo el contexto para preferirla cuando corresponde.

## Paso 6 — La decisión build-vs-use antes de escribir cualquier servidor propio

Aplicamos la [[1 resumen#5. Decisión build-vs-use comunidad primero, servidor propio solo si hace falta|Conclusión 5]]: antes de construir `internal_db` (Paso 1) como servidor propio, se justifica porque es un sistema interno propietario sin servidor de comunidad — pero para Jira, GitHub o Slack, la decisión es distinta.

```python
def should_build_custom_mcp_server(integration: str, team_specific_logic: bool) -> str:
    """
    Aplica el criterio de la Conclusión 5: comunidad primero, custom solo
    cuando hay lógica o requisitos específicos del equipo que la comunidad
    no puede cubrir.
    """
    community_servers = {"jira", "github", "slack", "linear", "notion"}

    if integration.lower() in community_servers and not team_specific_logic:
        return f"Usa el servidor de comunidad para {integration} — no construyas uno propio."

    return f"Construye un servidor MCP propio para {integration} — no hay cobertura de comunidad o hay lógica específica del equipo."


# El caso de Jira del equipo: sin lógica específica del equipo -> usar comunidad
print(should_build_custom_mcp_server("jira", team_specific_logic=False))
# -> "Usa el servidor de comunidad para jira — no construyas uno propio."

# El caso de internal_db: sistema propietario, no hay servidor de comunidad -> construir propio
print(should_build_custom_mcp_server("internal_db", team_specific_logic=True))
# -> "Construye un servidor MCP propio para internal_db — no hay cobertura de comunidad o hay lógica específica del equipo."
```

> [!warning] Pregunta trampa en código — construir un servidor MCP propio de Jira "porque así lo controlamos exactamente como queremos"
> La forma ingenua sería, para el caso de Jira, construir un servidor propio que exponga exactamente los endpoints que el equipo cree necesitar, en vez de evaluar primero el servidor de comunidad existente. **Por qué es mala idea** (Trampa de examen 1 del resumen): esto desperdicia tiempo de desarrollo y agrega mantenimiento continuo para un caso que un servidor de comunidad, ya probado y actualizado, resuelve de forma estándar. La respuesta correcta es evaluar el servidor de comunidad primero, y solo construir uno propio si el equipo describe explícitamente workflows que ese servidor no puede manejar.

Con esto, la configuración completa respeta las seis conclusiones del resumen: `github` e `internal_db` en `.mcp.json` (compartidos, versionados), `notion_experimental` en `~/.claude.json` (personal, no versionado), todas sus tools disponibles simultáneamente al conectar, credenciales referenciadas con `${VAR}` en vez de valores literales, el esquema de base de datos expuesto como resource en vez de forzar tool calls exploratorias, la decisión build-vs-use aplicada caso por caso, y la descripción de `search_codebase` reforzada para competir de igual a igual con Grep.
