Aplicación práctica de [[1 resumen]] — un agente de revisión de código para un PR de 14 archivos del módulo de inventario, construido primero de forma ingenua (una sola pasada) y luego rediseñado con arquitectura multi-pass, más una función de decisión que elige entre pipeline fijo y decomposición dinámica.

## Escenario

Un PR modifica 14 archivos del módulo `stock_tracking`. El equipo quiere un agente que revise el PR completo y, por separado, otro agente que se encargue de "agregar tests exhaustivos a un codebase legacy" — una tarea de alcance abierto. Vamos a construir ambos, aplicando en cada uno el patrón correcto del framework de decisión del resumen.

## Paso 1 — La función de decisión: pipeline fijo vs. decomposición dinámica

Antes de escribir cualquier agente, se decide el patrón según una sola pregunta: ¿los pasos se conocen de antemano?

```python
from dataclasses import dataclass
from enum import Enum


class Patron(Enum):
    PIPELINE_FIJO = "fixed_sequential_pipeline"
    DECOMPOSICION_DINAMICA = "dynamic_adaptive_decomposition"


@dataclass
class Tarea:
    nombre: str
    pasos_conocidos_de_antemano: bool


def elegir_patron(tarea: Tarea) -> Patron:
    if tarea.pasos_conocidos_de_antemano:
        return Patron.PIPELINE_FIJO
    return Patron.DECOMPOSICION_DINAMICA


revision_pr = Tarea("revisar PR de 14 archivos", pasos_conocidos_de_antemano=True)
tests_legacy = Tarea("agregar tests a codebase legacy", pasos_conocidos_de_antemano=False)

assert elegir_patron(revision_pr) == Patron.PIPELINE_FIJO
assert elegir_patron(tests_legacy) == Patron.DECOMPOSICION_DINAMICA
```

> [!danger] Pregunta trampa — usar decomposición dinámica "por si acaso" en la revisión del PR
> Los 14 archivos y su estructura ya se conocen antes de empezar (es un `git diff`, no una investigación). Usar decomposición dinámica aquí no aporta nada y agrega complejidad innecesaria: no hay nada que "descubrir" sobre qué archivos revisar. **¿Por qué sería mala idea?** Porque el framework de decisión no es "el patrón más flexible siempre gana" — es adaptabilidad *solo* cuando el problema no está completamente definido. Aquí sí lo está.

## Paso 2 — La versión ingenua: revisión en una sola pasada (dilución de atención)

Este es el punto de partida que produce el fallo descrito en el resumen: los 14 archivos se meten en un solo contexto y se le pide al modelo que los revise todos juntos.

```python
def revisar_pr_una_sola_pasada(archivos: dict[str, str], revisar_con_llm) -> str:
    contenido_completo = "\n\n".join(
        f"### Archivo: {ruta}\n{codigo}" for ruta, codigo in archivos.items()
    )
    prompt = (
        "Review all 14 files below for bugs, inefficiencies, and code quality "
        "issues. Be thorough and consistent across all files.\n\n" + contenido_completo
    )
    return revisar_con_llm(prompt)
```

> [!danger] Pregunta trampa — "arreglarlo" con un prompt más insistente
> ```python
> prompt = (
>     "Review all 14 files below. IMPORTANT: give EQUAL thoroughness to every "
>     "single file, including the last ones. Do not skim any file.\n\n" + contenido_completo
> )  # ❌ sigue siendo una sola pasada
> ```
> **¿Por qué sería mala idea?** Reforzar el prompt es exactamente la Trampa 2 del resumen: mejora el promedio, pero no cambia que el modelo sigue repartiendo el mismo presupuesto de atención entre 14 archivos en una sola pasada. El síntoma real (feedback detallado en los primeros archivos, superficial en los últimos, veredictos contradictorios entre archivos) es estructural — ninguna instrucción en el prompt cambia cuántos "tokens de atención" le tocan a cada archivo.

> [!danger] Pregunta trampa — "arreglarlo" subiendo de modelo
> Cambiar a un modelo con ventana de contexto más grande deja que quepan los 14 archivos sin truncarse, pero no resuelve la dilución: el problema nunca fue que los archivos no cupieran, fue que la profundidad de análisis se degrada al procesar muchos ítems en una sola pasada. **¿Por qué sería mala idea?** Porque confunde "capacidad de contexto" con "calidad de atención por ítem" — son dos cosas distintas, y solo la segunda causa el síntoma observado.

## Paso 3 — La solución: pasadas de análisis local por archivo

Se reemplaza la pasada única por 14 pasadas independientes, una por archivo, cada una con todo el presupuesto de atención enfocado en un solo ítem.

```python
def analisis_local_por_archivo(archivos: dict[str, str], revisar_con_llm) -> dict[str, str]:
    hallazgos_por_archivo = {}
    for ruta, codigo in archivos.items():
        prompt = (
            f"Review this single file for bugs, inefficiencies, and code quality "
            f"issues. Focus only on this file — do not reference other files.\n\n"
            f"### Archivo: {ruta}\n{codigo}"
        )
        hallazgos_por_archivo[ruta] = revisar_con_llm(prompt)
    return hallazgos_por_archivo
```

Cada llamada ve un solo archivo, así que el archivo 14 recibe exactamente el mismo presupuesto de atención que el archivo 1 — no hay degradación progresiva.

## Paso 4 — La pieza que falta si solo se hace batching: la pasada de integración cruzada

Agrupar los 14 archivos en lotes de 5 (batching) reduce la dilución dentro de cada lote, pero por sí solo deja huecos entre lotes. La arquitectura completa agrega una segunda capa: una pasada separada, después de que todas las locales terminaron, que compara hallazgos entre archivos.

```python
def pasada_integracion_cruzada(hallazgos_por_archivo: dict[str, str], revisar_con_llm) -> str:
    resumen_hallazgos = "\n\n".join(
        f"### {ruta}\n{hallazgos}" for ruta, hallazgos in hallazgos_por_archivo.items()
    )
    prompt = (
        "Below are per-file review findings for all 14 files in this PR. "
        "Look across all of them for cross-cutting concerns: identical patterns "
        "flagged in one file but not another, data-flow issues between files, "
        "and inconsistent verdicts. Report only cross-file issues.\n\n" + resumen_hallazgos
    )
    return revisar_con_llm(prompt)


def revisar_pr_multi_pass(archivos: dict[str, str], revisar_con_llm) -> dict[str, str]:
    hallazgos_locales = analisis_local_por_archivo(archivos, revisar_con_llm)
    hallazgos_cruzados = pasada_integracion_cruzada(hallazgos_locales, revisar_con_llm)
    return {"por_archivo": hallazgos_locales, "integracion_cruzada": hallazgos_cruzados}
```

> [!danger] Pregunta trampa — batching sin pasada de integración cruzada
> ```python
> def revisar_por_lotes_incompleto(archivos: dict[str, str], revisar_con_llm) -> list[str]:
>     rutas = list(archivos.items())
>     lotes = [dict(rutas[i:i + 5]) for i in range(0, len(rutas), 5)]  # 3 lotes de ~5
>     return [revisar_con_llm(construir_prompt(lote)) for lote in lotes]  # ❌ sin paso final
> ```
> **¿Por qué sería mala idea?** Reduce la dilución *dentro* de cada lote de 5, pero un patrón marcado como problemático en el lote 1 puede seguir aprobándose sin objeción en el lote 3 — nadie compara entre lotes. Es la Trampa 4 del resumen: batching sin una pasada de integración cruzada separada es una solución parcial, no la arquitectura multi-pass completa.

## Paso 5 — Decomposición dinámica: agregar tests a un codebase legacy

Para la segunda tarea (`tests_legacy`), el patrón es distinto porque el alcance no se conoce de antemano. Se decompone en fases que se adaptan a lo que se va descubriendo, no en una lista fija de pasos.

```python
def decomponer_tarea_legacy(mapear_estructura, identificar_areas_alto_impacto, investigar) -> list[str]:
    estructura = mapear_estructura()  # fase 1: descubrir qué módulos existen
    areas_prioritarias = identificar_areas_alto_impacto(estructura)  # fase 2: priorizar

    plan = list(areas_prioritarias)  # plan inicial, basado en lo descubierto hasta ahora
    completadas = []
    while plan:
        area = plan.pop(0)
        resultado = investigar(area)  # fase 3: ejecutar, y el resultado puede alterar el plan
        completadas.append(area)
        for dependencia_nueva in resultado.get("dependencias_descubiertas", []):
            if dependencia_nueva not in plan and dependencia_nueva not in completadas:
                plan.append(dependencia_nueva)  # el plan crece con lo que se descubre
    return completadas
```

> [!danger] Pregunta trampa — escribir de antemano la lista completa de módulos a testear
> ```python
> plan_fijo = ["auth", "billing", "inventory", "shipping"]  # ❌ decidido antes de investigar
> for modulo in plan_fijo:
>     investigar(modulo)
> ```
> **¿Por qué sería mala idea?** Esto es exactamente la Trampa 3 del resumen: aplicar un pipeline fijo a una tarea de alcance abierto. En un codebase legacy real, investigar `auth` puede revelar que depende de un módulo `legacy_session` que no estaba en la lista original — un plan fijo nunca lo descubre porque nunca vuelve a mirar lo que encontró en el camino.

---
> [!tip] Sigue con este tema
> Repasa la teoría en [[1 resumen]], refuerza con [[3 cuestionario]], y practica con preguntas estilo examen en [[4 test]].
