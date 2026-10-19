# Resultados — Tutor socrático: MCP Server Integration

> [!info] Sesión evaluada el 2026-10-12
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 93/100**

Dominio sólido de prácticamente todo el tema: 18 de 19 preguntas correctas y 1 parcial. Las respuestas mostraron que entiendes el porqué y no solo el qué: scoping, resources vs. tools, build-vs-use y la competencia de las descripciones MCP contra las tools nativas. El único hueco real fue el comportamiento de Claude Code cuando falta una variable de entorno sin default: asumiste que el servidor simplemente no carga, y no que Claude Code solo advierte y continúa. Lo corregiste de inmediato en la pregunta de refuerzo.

## Conceptos con buen dominio

- **Scoping `.mcp.json` vs. `~/.claude.json`**: distinguiste con claridad el nivel proyecto (versionado y compartido) del nivel usuario (personal y experimental), incluido qué ve el resto del equipo.
- **Descubrimiento simultáneo**: sabes que todas las tools de los servidores de ambos niveles quedan disponibles al conectar, sin activación manual.
- **Formato de declaración**: identificaste que a un servidor remoto le falta `"type": "http"` y que sin él se interpreta como stdio y se salta.
- **Expansión `${VAR}` y `${VAR:-default}`**: escribiste correctamente la forma con default y entiendes que la rotación de tokens no toca `.mcp.json`.
- **Resources vs. tools**: aplicaste bien la regla de oro (mostrar vs. actuar), sabes que los resources son *application-controlled* (los decide el host) y que se traen con `@server:resource`.
- **Build-vs-use**: primero se evalúa el servidor de comunidad, y solo se construye uno propio para sistemas internos o necesidades que la comunidad no cubre.
- **Descripciones de tools MCP**: entiendes por qué el modelo prefiere Grep y qué agregar a la descripción (outputs y cuándo preferirla explícitamente).
- **Trampas de examen**: dos niveles para el examen (no los tres scopes de la documentación actual) y el secreto que sigue recuperable en el historial de git.

## Áreas que necesitan revisión

### Variable de entorno sin definir y sin default

- **Qué pasó:** cuando un desarrollador no define `GITHUB_TOKEN`, respondiste que Claude Code "no permitiría utilizar ese servidor y no cargaría las tools". Te faltó el punto clave del resumen: la carga **no se detiene**. Claude Code solo advierte y continúa, así que la falla puede pasar desapercibida. Lo corregiste bien en la pregunta de refuerzo.
- **Contenido del resumen:**
  > [!note] Forma con valor por defecto: `${VAR:-default}`
  > Existe una segunda forma: `${VAR:-default}` se expande a `VAR` cuando está definida, y usa `default` como respaldo cuando no lo está — útil para rutas específicas de cada máquina que tienen un valor de respaldo razonable (ej. `${WORKSPACE_ROOT:-.}`). Al 14 de agosto de 2026, una variable sin definir y sin default **no detiene** el resto de la carga de configuración: Claude Code advierte y continúa — no asumas que un token faltante va a fallar de forma ruidosa y visible.
- **Sección:** [[1 resumen]] → "3. Expansión de variables de entorno para credenciales"
- **Analogía para reforzarlo:** es como un coche que arranca aunque el sensor de presión de una llanta esté desconectado. No se niega a encender: solo prende un foquito en el tablero. Si no miras el tablero, sales a carretera creyendo que todo está bien.

## Siguiente paso recomendado

Repasa la nota sobre `${VAR:-default}` y el comportamiento "advierte y continúa" de las variables sin definir. Fuera de eso, estás listo para [[4 test]].
