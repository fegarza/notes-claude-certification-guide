Aplicación práctica de [[1 resumen]] — un flujo de trabajo de dos días sobre un codebase de 50 archivos: día 1 se analiza y se encuentran tres problemas de autenticación, de noche se corrigen, y día 2 hay que decidir cómo continuar sin caer en el problema de contexto obsoleto. Construimos la función de decisión, la reanudación ingenua que falla, el fork para exploración divergente, y el inicio limpio con resumen inyectado que sí funciona.

## Escenario

Un desarrollador usa Claude Code para auditar un codebase de 50 archivos. Día 1: sesión nombrada `auth-audit` encuentra tres problemas en `auth.ts`, `session.ts` y `middleware.ts`. Esa noche corrige los tres. Día 2: necesita retomar el trabajo, pero además quiere comparar dos estrategias de tests para cubrir esos módulos. Vamos a modelar las tres opciones del resumen y una función que elige entre ellas.

## Paso 1 — Modelar el estado de la sesión y la función de decisión

Antes de tocar la CLI o el SDK, se decide la estrategia con una sola pregunta por rama: ¿el contexto sigue siendo válido, hay que explorar en paralelo, o los resultados están obsoletos?

```python
from dataclasses import dataclass, field
from enum import Enum


class EstrategiaSesion(Enum):
    RESUME = "resume"
    FORK_SESSION = "fork_session"
    FRESH_START_CON_RESUMEN = "fresh_start_con_resumen"


@dataclass
class EstadoSesion:
    nombre_sesion: str
    archivos_modificados_desde_ultima_sesion: list[str] = field(default_factory=list)
    quiere_explorar_enfoques_divergentes: bool = False


def elegir_estrategia(estado: EstadoSesion) -> EstrategiaSesion:
    if estado.quiere_explorar_enfoques_divergentes:
        return EstrategiaSesion.FORK_SESSION
    if estado.archivos_modificados_desde_ultima_sesion:
        return EstrategiaSesion.FRESH_START_CON_RESUMEN
    return EstrategiaSesion.RESUME


continuar_sin_cambios = EstadoSesion("auth-audit", archivos_modificados_desde_ultima_sesion=[])
retomar_tras_fix = EstadoSesion(
    "auth-audit",
    archivos_modificados_desde_ultima_sesion=["auth.ts", "session.ts", "middleware.ts"],
)
comparar_estrategias_test = EstadoSesion(
    "auth-audit", quiere_explorar_enfoques_divergentes=True
)

assert elegir_estrategia(continuar_sin_cambios) == EstrategiaSesion.RESUME
assert elegir_estrategia(retomar_tras_fix) == EstrategiaSesion.FRESH_START_CON_RESUMEN
assert elegir_estrategia(comparar_estrategias_test) == EstrategiaSesion.FORK_SESSION
```

> [!danger] Pregunta trampa — priorizar `fork_session` sobre el contexto obsoleto cuando ambas condiciones aplican
> Si el equipo quisiera bifurcar *y* los archivos cambiaron, bifurcar primero heredaría contexto obsoleto en ambas ramas (Trampa 4 del resumen). Por eso la función revisa divergencia primero solo cuando NO hay archivos modificados en juego — en un caso real con ambas condiciones, el orden correcto es: primero un inicio limpio con resumen para eliminar lo obsoleto, y recién ahí bifurcar desde esa base limpia si hace falta explorar caminos distintos.

## Paso 2 — Día 1: sesión nombrada con hallazgos

En la CLI, la sesión se nombra desde el arranque para poder identificarla el día 2.

```bash
claude --name auth-audit \
  "Analiza el codebase en src/ buscando problemas de autenticación. \
   Sé exhaustivo con auth.ts, session.ts y middleware.ts."
```

El agente termina el día 1 con tres hallazgos en el historial de la sesión `auth-audit`: un token de sesión que no expira, una comparación de contraseñas no constante en el tiempo, y una validación de rol ausente en un endpoint de middleware.

## Paso 3 — La versión ingenua: `--resume` después de corregir los archivos (contexto obsoleto)

Esto es lo que produce el fallo descrito en el resumen: los tres archivos ya fueron corregidos de noche, pero la sesión reanudada sigue teniendo en su historial las lecturas *viejas* de esos archivos.

```bash
# ❌ Los tres problemas ya fueron corregidos de noche, pero el historial
# de auth-audit todavía contiene el contenido viejo de estos archivos.
claude --resume auth-audit \
  "Vuelve a leer auth.ts, session.ts y middleware.ts y confirma si los \
   problemas siguen presentes."
```

> [!danger] Pregunta trampa — "arreglarlo" solo con "vuelve a leer los archivos"
> Pedirle al agente que relea los archivos modificados dentro de la misma sesión reanudada es la Trampa 2 del resumen. **¿Por qué sería mala idea?** Aunque el agente relea el contenido actual, los resultados de herramientas viejos (mostrando el código sin corregir) siguen en el historial de conversación. El modelo puede seguir citando el hallazgo original de más atrás en el contexto, produciendo una respuesta que mezcla "el problema sigue ahí" (dato viejo) con "el problema ya no está" (lectura nueva) — exactamente el patrón de consejos contradictorios del caso guía.

## Paso 4 — La solución: inicio limpio con resumen estructurado inyectado

Se construye un resumen curado (sin resultados de herramientas crudos) y se inyecta como contexto inicial de una sesión completamente nueva.

```python
@dataclass
class ResumenSesionPrevia:
    hallazgos: list[str]
    archivos_afectados: list[str]
    estado: str  # "corregido" o "pendiente"


def construir_prompt_inicial(resumen: ResumenSesionPrevia, archivos_cambiados: list[str]) -> str:
    hallazgos_texto = "\n".join(f"- {h}" for h in resumen.hallazgos)
    return (
        "El análisis previo (sesión anterior) identificó estos problemas de "
        f"autenticación:\n{hallazgos_texto}\n\n"
        f"Estado reportado: {resumen.estado}.\n"
        f"Los siguientes archivos fueron modificados desde entonces: "
        f"{', '.join(archivos_cambiados)}.\n\n"
        "Re-analiza únicamente estos archivos para verificar las correcciones "
        "y detectar si se introdujeron problemas nuevos. No asumas que el "
        "resto del codebase cambió."
    )


resumen_previo = ResumenSesionPrevia(
    hallazgos=[
        "Token de sesión sin expiración en session.ts",
        "Comparación de contraseñas no constante en el tiempo en auth.ts",
        "Validación de rol ausente en un endpoint de middleware.ts",
    ],
    archivos_afectados=["auth.ts", "session.ts", "middleware.ts"],
    estado="corregido",
)

prompt_dia_2 = construir_prompt_inicial(
    resumen_previo, archivos_cambiados=["auth.ts", "session.ts", "middleware.ts"]
)
```

```bash
# ✅ Sesión nueva, sin --resume: el historial empieza vacío y el único
# contexto previo es el resumen curado inyectado en el primer mensaje.
claude --name auth-audit-verificacion "$prompt_dia_2"
```

Esta sesión no tiene ningún resultado de herramienta obsoleto: el agente lee `auth.ts`, `session.ts` y `middleware.ts` en su estado actual, verifica las correcciones contra el resumen inyectado, y reporta de forma consistente — sin mezclar datos viejos y nuevos.

## Paso 5 — `fork_session`: comparar dos estrategias de tests desde la misma base

Una vez verificadas las correcciones, el equipo quiere comparar dos estrategias de tests para esos módulos, partiendo del mismo análisis. Esto es exploración divergente, no continuación — el caso de uso de `fork_session`.

Con el SDK (Python), `resume` selecciona la sesión base y `fork_session=True` copia su historial en vez de seguir escribiendo sobre él:

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

id_sesion_verificada = "auth-audit-verificacion"

opciones_rama_unit_tests = ClaudeAgentOptions(
    resume=id_sesion_verificada,
    fork_session=True,
    system_prompt=(
        "A partir del análisis ya hecho, propone una estrategia de unit "
        "tests para auth.ts, session.ts y middleware.ts."
    ),
)

opciones_rama_integration_tests = ClaudeAgentOptions(
    resume=id_sesion_verificada,
    fork_session=True,
    system_prompt=(
        "A partir del análisis ya hecho, propone una estrategia de "
        "integration tests para el flujo completo de autenticación."
    ),
)

async def explorar_estrategias_en_paralelo():
    async with ClaudeSDKClient(options=opciones_rama_unit_tests) as rama_unit:
        await rama_unit.query("Genera el plan de unit tests.")
    async with ClaudeSDKClient(options=opciones_rama_integration_tests) as rama_integration:
        await rama_integration.query("Genera el plan de integration tests.")
```

Ambas ramas parten del mismo análisis verificado, pero ninguna ve los resultados de la otra — cambios en la rama de unit tests no contaminan la rama de integration tests.

En la CLI, el mismo par de flags logra lo equivalente:

```bash
claude --resume auth-audit-verificacion --fork-session \
  "Propón una estrategia de unit tests para los tres módulos."

claude --resume auth-audit-verificacion --fork-session \
  "Propón una estrategia de integration tests para el flujo de autenticación."
```

> [!danger] Pregunta trampa — usar `fork_session` para "arreglar" el contexto obsoleto del Paso 3
> ```python
> # ❌ Bifurcar desde la sesión auth-audit original, contaminada
> opciones_incorrectas = ClaudeAgentOptions(
>     resume="auth-audit",  # todavía tiene los resultados viejos de los 3 archivos
>     fork_session=True,
> )
> ```
> **¿Por qué sería mala idea?** `fork_session` copia el historial existente tal cual está — si ese historial ya contiene resultados de herramientas obsoletos, la rama nueva los hereda íntegros (Trampa 4 del resumen). Bifurcar no limpia nada; solo distribuye el mismo contexto (bueno o malo) en dos caminos. La limpieza solo ocurre con un inicio limpio y resumen inyectado, como en el Paso 4 — por eso las ramas del Paso 5 bifurcan desde `auth-audit-verificacion` (ya limpia), no desde `auth-audit` (todavía contaminada).

## Paso 6 — Función de re-análisis dirigido reutilizable

Generalizando el Paso 4: dado un resumen previo y una lista de archivos cambiados, se construye siempre el mismo tipo de prompt de re-análisis dirigido, evitando tanto el `--resume` ingenuo como la re-exploración completa del codebase.

```python
def necesita_reexploracion_completa(archivos_cambiados: list[str], total_archivos_codebase: int) -> bool:
    # Umbral simple: si cambió una fracción muy grande del codebase,
    # el resumen dirigido pierde valor frente a una exploración nueva.
    return len(archivos_cambiados) / total_archivos_codebase > 0.5


def preparar_sesion_dia_siguiente(
    resumen: ResumenSesionPrevia, archivos_cambiados: list[str], total_archivos_codebase: int
) -> str:
    if not archivos_cambiados:
        raise ValueError("Sin archivos modificados: usa --resume, no esta función.")
    if necesita_reexploracion_completa(archivos_cambiados, total_archivos_codebase):
        return (
            "Más de la mitad del codebase cambió desde el último análisis. "
            "Realiza una exploración completa nueva, no un re-análisis dirigido."
        )
    return construir_prompt_inicial(resumen, archivos_cambiados)


# 3 de 50 archivos cambiados: re-análisis dirigido, como en el caso guía.
assert "Re-analiza únicamente" in preparar_sesion_dia_siguiente(resumen_previo, ["auth.ts", "session.ts", "middleware.ts"], 50)
```

> [!danger] Pregunta trampa — re-explorar todo el codebase aunque solo cambiaron 3 de 50 archivos
> ```python
> # ❌ Ignora el resumen previo y vuelve a analizar los 50 archivos desde cero
> claude --name auth-audit-v2 "Analiza todo src/ desde cero, busca problemas de autenticación."
> ```
> **¿Por qué sería mala idea?** Es la Trampa 1 del resumen: desperdicia tiempo y presupuesto de contexto re-analizando 47 archivos que no cambiaron, cuando el resumen inyectado ya preserva ese conocimiento sin resultados obsoletos. El re-análisis dirigido del Paso 4 es más rápido y, a diferencia del `--resume` ingenuo del Paso 3, no arrastra contexto contaminado.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y practica con preguntas estilo examen en [[4 test]].
