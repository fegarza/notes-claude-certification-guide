---
tags:
  - claude-cert/dominio-2
  - task-statement/2.4
---

# 3 cuestionario — MCP Server Integration

Repaso de [[1 resumen]]. Preguntas cortas, una idea por pregunta. Respóndelas mentalmente antes de abrir cada respuesta.

## La jerarquía de scoping

> [!question]- ¿Cuál es la diferencia fundamental entre `.mcp.json` y `~/.claude.json`?
> `.mcp.json` vive en la raíz del proyecto, está versionado en git y se comparte automáticamente con todo el equipo al clonar o hacer pull. `~/.claude.json` vive en el home del usuario, es personal, y no está versionado ni se comparte.

> [!question]- ¿Qué tipo de servidores deberían ir en `.mcp.json` y cuáles en `~/.claude.json`?
> Servidores que todo el equipo necesita (Jira, GitHub, conectores internos) van en `.mcp.json`. Servidores experimentales, personales, o en prueba antes de proponerse al equipo van en `~/.claude.json`.

> [!question]- ¿Cómo se distingue en `.mcp.json` una entrada de servidor remoto de una local?
> Un servidor remoto declara `"type": "http"` y una `url`. Un servidor local declara `command` y `args`, y habla por stdio.

> [!question]- ¿Qué pasa si una entrada de servidor tiene `url` pero no declara `"type": "http"`?
> Claude Code la interpreta como servidor stdio, la salta, y avisa que falta el `type` — el servidor remoto no queda disponible aunque la entrada parezca correcta.

> [!question]- El exam guide describe la configuración de MCP en dos niveles, pero la documentación actual describe tres scopes (`local`, `project`, `user`). ¿Con cuál se debe responder en el examen?
> Con el split de dos niveles proyecto-vs-usuario — esa es la respuesta esperada por el exam guide v1.0, aunque la documentación más reciente describa un tercer scope (`local`, el default).

## Descubrimiento simultáneo de tools

> [!question]- Si un desarrollador tiene servidores configurados tanto en `.mcp.json` como en `~/.claude.json`, ¿necesita activar manualmente las tools de cada uno?
> No — todas las tools de todos los servidores configurados (proyecto y usuario) se descubren al conectar y quedan disponibles simultáneamente. No hay paso de activación manual.

## Expansión de variables de entorno

> [!question]- ¿Por qué usar `${GITHUB_TOKEN}` en vez de escribir el token directamente en `.mcp.json`?
> Porque `${GITHUB_TOKEN}` referencia solo el *nombre* de la variable, nunca el valor — así el archivo es seguro de commitear, cada desarrollador define su propio token localmente, y ningún secreto queda expuesto en el historial del repositorio.

> [!question]- ¿Qué ventaja adicional da la expansión de variables de entorno respecto a rotar tokens?
> La rotación de tokens no requiere cambios en el archivo de configuración — solo se actualiza la variable de entorno localmente en cada máquina.

> [!question]- ¿Qué hace la sintaxis `${VAR:-default}` y cuándo conviene usarla?
> Se expande a `VAR` cuando está definida, y usa `default` como respaldo cuando no lo está — útil para rutas u otros valores específicos de cada máquina que tienen un valor razonable por defecto.

> [!question]- ¿Qué pasa si una variable de entorno referenciada en `.mcp.json` no está definida y no tiene default?
> Claude Code advierte pero continúa cargando el resto de la configuración — no asumas que una variable faltante va a fallar de forma ruidosa y detener todo.

## MCP Resources

> [!question]- ¿Cuál es la diferencia conceptual entre un resource y una tool en MCP?
> Los resources muestran qué datos están disponibles (un catálogo que el agente puede ver de antemano); las tools permiten actuar sobre esos datos (ejecutar una acción).

> [!question]- ¿Por qué exponer un esquema de base de datos como resource en vez de como una tool de "listar tablas"?
> Porque sin un resource, el agente tendría que llamar `list_tables` y luego `describe_table` para cada tabla —gastando múltiples tool calls solo para orientarse—, mientras que con el resource esa estructura está disponible de inmediato, sin exploración previa.

> [!question]- ¿Qué significa que los resources de MCP sean "application-controlled"?
> Que aunque el servidor los lista, es el cliente (el host, ej. Claude Code) quien decide cuándo adjuntar un resource al contexto del modelo — el agente solo lo "ve" cuando el host lo expone explícitamente (ej. mediante menciones `@server:resource`).

## La decisión build-vs-use

> [!question]- ¿Cuándo es correcto usar un servidor MCP de la comunidad en vez de construir uno propio?
> Para integraciones estándar (Jira, GitHub, Slack, Linear, Notion) — están mantenidos, probados por la comunidad, cubren casos de uso estándar, y ahorran tiempo de desarrollo y mantenimiento.

> [!question]- ¿Bajo qué condiciones se justifica construir un servidor MCP propio?
> Solo cuando el equipo tiene workflows específicos que los servidores de comunidad no cubren, necesita lógica de negocio personalizada embebida en la capa de tools, o requiere integración con sistemas internos propietarios sin servidor de comunidad disponible.

> [!question]- ¿Qué pasaría si un equipo construye un servidor MCP propio para Jira sin verificar primero si el servidor de comunidad cubre sus necesidades?
> Desperdiciaría tiempo de desarrollo y agregaría carga de mantenimiento continuo para un caso que probablemente ya está resuelto de forma estándar y probada por el servidor de comunidad — la respuesta del examen favorece siempre evaluar comunidad primero.

## Reforzar descripciones de tools MCP

> [!question]- ¿Por qué un agente puede preferir una tool nativa como Grep sobre una tool MCP más capaz?
> Porque cuando la descripción de la tool MCP es escueta, el modelo tiene mejor contexto sobre las tools nativas —sus descripciones son más ricas y detalladas— y por eso tiende a preferirlas, aunque la tool MCP sea objetivamente mejor para esa tarea.

> [!question]- ¿Cuál es la forma correcta de resolver que un agente prefiera Grep sobre una tool MCP de búsqueda semántica más capaz?
> Reforzar la descripción de la tool MCP para explicar sus capacidades y outputs en detalle, dejando explícito cuándo y por qué usarla en vez de la alternativa nativa — no compensarlo con instrucciones genéricas en el system prompt.

> [!question]- ¿Qué elementos debería incluir una descripción de tool MCP reforzada, según el ejemplo de `search_codebase`?
> Qué hace la tool (búsqueda semántica AST-aware), qué devuelve (funciones/clases/métodos con ruta y número de línea), cuándo usarla (buscar por intención, no por string exacto), y una comparación explícita con la alternativa nativa (más precisa que grep para ese caso, úsala en vez de Grep cuando aplica).

## Síntesis

> [!question]- Un equipo pone un servidor MCP de Jira personal de un desarrollador en `.mcp.json`, con el token de autenticación escrito directamente en el archivo, y la tool tiene la descripción "Interacts with Jira". ¿Qué tres errores de diseño hay aquí?
> Primero, scoping incorrecto: un servidor personal/experimental no debería estar en `.mcp.json` compartido. Segundo, credencial expuesta en texto plano en vez de usar `${VARIABLE_NAME}`. Tercero, descripción de tool demasiado escueta, lo cual hace que el agente probablemente prefiera tools nativas en vez de la tool MCP de Jira aunque sea más capaz para esa tarea.
