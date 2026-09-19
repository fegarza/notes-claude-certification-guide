---
tags:
  - claude-cert/dominio-2
  - task-statement/2.4
---

# 2.4 — MCP Server Integration

## Explícamelo como si tuviera 5 años

Imagina que tu equipo de trabajo comparte una caja de herramientas común guardada en la oficina — todos la usan, todos ven lo que hay dentro, y si alguien agrega un martillo nuevo, el resto del equipo lo ve al día siguiente. Pero tú también tienes tu propia caja de herramientas personal en tu casa, con cosas que estás probando antes de proponerlas para la caja de la oficina — nadie más la ve, y no aparece en el inventario compartido.

Eso es exactamente la diferencia entre un servidor MCP configurado a **nivel de proyecto** (la caja compartida, versionada, que todo el equipo hereda al clonar el repositorio) y uno a **nivel de usuario** (tu caja personal, que vive solo en tu máquina). Mezclar las dos —poner algo que todo el equipo necesita en tu caja personal, o poner un experimento a medias en la caja compartida— es la fuente de la mayoría de los problemas de configuración de MCP.

## Argumento central

> [!note] Idea central
> Un servidor MCP (Model Context Protocol) extiende las capacidades de Claude conectándolo a sistemas externos — bases de datos, APIs, rastreadores de issues, herramientas de desarrollo. Configurarlo correctamente no es solo cuestión de que "funcione": determina si el equipo comparte un toolset consistente y seguro, o si cae en caos de configuración. Esto exige acertar en cuatro decisiones: **dónde** vive la configuración (scoping proyecto vs. usuario), **cómo** se manejan las credenciales (variables de entorno, nunca secretos en texto plano), **qué** se expone como resource vs. como tool, y **cuándo** construir un servidor propio en vez de usar uno de la comunidad.

## Conclusiones

### 1. La jerarquía de scoping: `.mcp.json` (proyecto) vs. `~/.claude.json` (usuario)

- **Nivel de proyecto — `.mcp.json`**: vive en la raíz del repositorio, está versionado (va en git) y se comparte automáticamente con cualquiera que clone o haga pull del repo. Es para servidores que **todo el equipo** necesita — la integración de Jira, las tools de GitHub, conectores internos de API.
- **Nivel de usuario — `~/.claude.json`**: vive en el directorio home del usuario, es personal, **no** está versionado ni se comparte con el equipo. Es para servidores experimentales, integraciones personales, o servidores que se están probando antes de proponerlos al equipo.

> [!note] Dos formas de declarar un servidor en `.mcp.json`
> Un servidor **remoto** declara `"type": "http"` y una `url` (ej. los endpoints oficiales de GitHub y Atlassian). Un servidor **local** declara `command` y `args`, y habla por stdio (ej. `npx -y @modelcontextprotocol/server-filesystem ...`). Una entrada con `url` pero sin `type` es un error de configuración: Claude Code la interpreta como servidor stdio, la salta, y avisa que falta el `type`.

> [!warning] Estado actual vs. respuesta esperada en el examen
> El exam guide v1.0 describe la configuración de MCP en **dos** niveles (proyecto y usuario), y esa es la respuesta esperada en el examen. La documentación actual de MCP (al 14 de agosto de 2026) documenta **tres** scopes — `local` (default), `project` y `user` — seleccionables con `-s`/`--scope`, con `~/.claude.json` guardando tanto las entradas `local` como `user`. **En el examen, responde con el split de dos niveles proyecto-vs-usuario**, no con los tres scopes de la documentación más reciente.

> [!tip] Analogía mental
> `.mcp.json` es el pizarrón de la oficina que todos ven al llegar. `~/.claude.json` es tu libreta personal en el bolsillo — útil para anotar ideas propias, pero nadie más la lee.

### 2. Descubrimiento simultáneo: no hay activación manual

Todas las tools de todos los servidores MCP configurados —tanto a nivel proyecto como a nivel usuario— se descubren al momento de la conexión y quedan disponibles **simultáneamente** para el agente. No existe un paso de activación manual: si un servidor está configurado y es alcanzable, sus tools aparecen en el toolkit del agente automáticamente.

> [!tip] Analogía mental
> Es como conectar un dispositivo USB nuevo a la computadora: no hay que "activarlo" aparte — en cuanto está conectado y la computadora lo reconoce, ya está disponible para usarse.

### 3. Expansión de variables de entorno para credenciales

`.mcp.json` soporta la sintaxis `${VARIABLE_NAME}` para expansión de variables de entorno. Así se mantienen las credenciales fuera del control de versiones mientras se sigue compartiendo la configuración del servidor con el equipo.

- Cada desarrollador define su propio token localmente (perfil de shell, archivo `.env`, gestor de secretos).
- El archivo `.mcp.json` referencia **nombres** de variable, nunca valores reales — por eso es seguro hacer commit de ese archivo.
- La rotación de tokens no requiere cambios en el archivo de configuración.
- Ningún secreto queda expuesto en el historial del repositorio.

> [!note] Forma con valor por defecto: `${VAR:-default}`
> Existe una segunda forma: `${VAR:-default}` se expande a `VAR` cuando está definida, y usa `default` como respaldo cuando no lo está — útil para rutas específicas de cada máquina que tienen un valor de respaldo razonable (ej. `${WORKSPACE_ROOT:-.}`). Al 14 de agosto de 2026, una variable sin definir y sin default **no detiene** el resto de la carga de configuración: Claude Code advierte y continúa — no asumas que un token faltante va a fallar de forma ruidosa y visible.

### 4. MCP Resources: exponer catálogos de contenido para evitar tool calls exploratorias

Los **resources** de MCP exponen catálogos de contenido a los agentes sin requerir llamadas exploratorias a tools. En vez de que el agente llame una tool para descubrir qué datos existen, esa información ya está disponible de antemano.

- **Resúmenes de issues** — una lista de issues actuales de Jira con títulos y estados.
- **Jerarquías de documentación** — una tabla de contenidos de la documentación interna.
- **Esquemas de base de datos** — nombres de tablas, tipos de columna, relaciones.

> [!note] Caso: esquema de base de datos como resource
> Sin un resource, un agente podría llamar `list_tables` y luego `describe_table` para *cada* tabla, gastando múltiples tool calls solo para orientarse. Con un resource de esquema de base de datos, el agente conoce esa estructura de inmediato — sin exploración previa.

> [!tip] Regla de oro
> **Los resources muestran qué datos están disponibles. Las tools permiten actuar sobre ellos.** Es la diferencia entre un catálogo que ya tienes en la mano y una herramienta para hacer algo con lo que hay en ese catálogo.

> [!note] Precisión: los resources son application-controlled
> El exam guide enmarca los resources como "reducen tool calls exploratorias" — esa es la respuesta que evalúa el examen. Una precisión de la especificación de MCP (septiembre 2026): los resources son *application-controlled* — el servidor los lista, pero es el cliente (el host) quien decide cuándo adjuntar uno al contexto del modelo, así que el agente solo "ve" un resource cuando el host lo expone. Claude Code hace esto mediante menciones `@server:resource` y una tool de listado de resources.

### 5. Decisión build-vs-use: comunidad primero, servidor propio solo si hace falta

**Usar servidores de la comunidad para integraciones estándar:**

- Jira, GitHub, Slack, Linear, Notion — todos tienen servidores MCP de comunidad mantenidos.
- Cubren casos de uso estándar, están probados por la comunidad, y reciben actualizaciones.
- Usarlos ahorra tiempo de desarrollo y carga de mantenimiento.

**Construir servidores propios solo cuando:**

- El equipo tiene workflows específicos que los servidores de comunidad no pueden manejar.
- Se necesita lógica de negocio personalizada embebida en la capa de tools.
- Se requiere integración con sistemas internos propietarios que no tienen servidor de comunidad.

> [!tip] Analogía mental
> Construir un servidor MCP propio para una integración estándar (como Jira) es como fabricar tu propio martillo cuando la ferretería de la esquina ya vende uno probado por miles de personas — tiene sentido solo si tu trabajo necesita un martillo con una forma que nadie más fabrica.

### 6. Reforzar las descripciones de tools MCP para competir con las tools nativas

Cuando una tool MCP tiene una descripción escueta, el agente puede preferir tools nativas (como Grep) aunque la tool MCP sea más capaz para esa tarea. La razón no es que el modelo sea torpe: simplemente tiene mejor contexto sobre las tools nativas, porque sus descripciones son más ricas y detalladas.

> [!note] Antes / después
> Escueta: `search_codebase: "Searches code"`.
> Reforzada: una descripción que explica que hace búsqueda semántica *AST-aware* en todo el repositorio, devuelve funciones/clases/métodos con ruta de archivo y número de línea, es más precisa que una búsqueda de texto tipo grep para buscar código por intención en vez de por coincidencia exacta de string, y **especifica explícitamente** que debe usarse en vez de Grep cuando se busca código por lo que hace, no por lo que contiene.

La descripción reforzada le da al modelo suficiente contexto para preferir la tool MCP cuando genuinamente es más capaz que la alternativa nativa.

## En una frase

> Integrar servidores MCP bien no es solo "conectarlos": es poner cada uno en el scope correcto (`.mcp.json` compartido vs. `~/.claude.json` personal), mantener credenciales fuera del código con `${VAR}`, usar resources para exponer catálogos de datos en vez de forzar tool calls exploratorias, preferir servidores de comunidad sobre construir uno propio, y escribir descripciones de tools MCP lo bastante detalladas como para competir con las tools nativas por la atención del modelo.

## Trampas de examen

> [!warning] Trampa 1 — Construir un servidor MCP propio para una integración estándar (ej. Jira)
> Existen servidores MCP de comunidad para integraciones estándar y deben evaluarse primero. **Por qué es un error:** construir uno propio para un caso ya cubierto desperdicia tiempo de desarrollo y agrega mantenimiento innecesario. La forma correcta es evaluar servidores de comunidad primero, y solo construir uno propio cuando el escenario describe explícitamente workflows específicos del equipo que la comunidad no puede cubrir.

> [!warning] Trampa 2 — Poner configuración de servidor MCP compartida por todo el equipo en `~/.claude.json`
> `~/.claude.json` es de nivel usuario y personal — no está versionado ni se comparte. **Por qué es un error:** si el servidor debe estar disponible para todo el equipo, ponerlo en el archivo personal de un solo desarrollador significa que el resto del equipo nunca lo tiene. Los servidores de todo el equipo van en `.mcp.json` en la raíz del proyecto.

> [!warning] Trampa 3 — Hacer commit de credenciales directamente en `.mcp.json` en vez de usar expansión de variables de entorno
> Las credenciales en control de versiones son un riesgo de seguridad. **Por qué es un error:** cualquiera con acceso al historial del repositorio (incluso después de "borrar" el secreto en un commit posterior) puede recuperar el token. La forma correcta es usar sintaxis `${GITHUB_TOKEN}`, para que cada desarrollador defina sus tokens localmente y ningún secreto entre nunca al historial del repositorio.

> [!warning] Trampa 4 — Dejar las descripciones de tools MCP escuetas, provocando que el agente prefiera tools nativas
> El modelo por defecto favorece las tools que mejor entiende. **Por qué es un error:** descripciones MCP escuetas pierden frente a descripciones de tools nativas más detalladas, aunque la tool MCP sea objetivamente más capaz para la tarea — el agente termina usando Grep cuando una tool MCP semántica habría dado mejores resultados. La forma correcta es reforzar las descripciones MCP para explicar capacidades y outputs a fondo, dejando explícito cuándo preferirla sobre la alternativa nativa.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
