Aplicación práctica de [[1 resumen]] — implementación de un sistema de investigación multi-agente hub-and-spoke con la Messages API de Claude en Python.

## Escenario

Un sistema de investigación sobre energías renovables: un **coordinador** descompone el tema, invoca **subagentes de búsqueda** especializados por subtema, y un **subagente de síntesis** arma el reporte final. Vamos a construirlo aplicando cada idea clave del resumen: hub-and-spoke estricto, aislamiento de contexto, descomposición amplia (no estrecha), refinamiento iterativo, y evitando explícitamente las 4 trampas de examen.

## Paso 1 — Definir el subagente de búsqueda (aislado, sin memoria propia)

Cada subagente es una función independiente: recibe *solo* lo que el coordinador le pasa como `prompt`, y no tiene acceso a nada más — ni al historial del coordinador, ni a resultados de otros subagentes, ni a invocaciones anteriores de sí mismo.

```python
import anthropic

client = anthropic.Anthropic()

def subagente_busqueda(subtema: str) -> str:
    """Aislado: solo conoce el subtema que el coordinador le pasó explícitamente."""
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        system="Eres un agente de búsqueda. Investiga a fondo el subtema dado y devuelve hallazgos con fuentes.",
        messages=[{"role": "user", "content": f"Investiga: {subtema}"}],
    )
    return "".join(b.text for b in response.content if b.type == "text")
```

> [!danger] Pregunta trampa — Trampa 2 en código (asumir herencia de contexto)
> ```python
> # ❌ NO HACER: asumir que el subagente "ya sabe" el tema general de la investigación
> def subagente_busqueda_malo(subtema: str) -> str:
>     response = client.messages.create(
>         model="claude-sonnet-5",
>         max_tokens=1024,
>         messages=[{"role": "user", "content": f"Sigue investigando el subtema: {subtema}"}],
>         # "Sigue" implica contexto previo que este subagente nunca recibió
>     )
>     ...
> ```
> **¿Por qué sería mala idea?** El subagente no tiene memoria de invocaciones anteriores ni del tema general que el coordinador está investigando — cada invocación es independiente. Escribir el prompt como si "ya supiera" algo del contexto general asume una herencia de contexto que no existe (Trampa 2 del resumen).

## Paso 2 — Definir el subagente de síntesis (mismo aislamiento)

```python
def subagente_sintesis(hallazgos: list[str]) -> str:
    """Solo recibe los hallazgos que el coordinador decide pasarle — nada más."""
    contexto = "\n\n---\n\n".join(hallazgos)
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=2048,
        system="Eres un agente de síntesis. Combina los hallazgos recibidos en un reporte coherente.",
        messages=[{"role": "user", "content": f"Hallazgos a sintetizar:\n\n{contexto}"}],
    )
    return "".join(b.text for b in response.content if b.type == "text")
```

## Paso 3 — El coordinador: descomposición amplia del tema

Esta es la parte crítica del resumen: la descomposición debe cubrir el **alcance completo** del tema, no solo los subtemas más obvios. Aplicando el ejemplo del resumen (energías renovables), el coordinador reparte el tema en categorías representativas de *todo* el dominio, no solo dos.

```python
def descomponer_tema(tema: str) -> list[str]:
    """Devuelve la lista de subtemas que se asignarán a subagentes de búsqueda."""
    # energías renovables: cubre TODO el dominio, no solo los subtemas más populares
    return [
        "avances en paneles solares",
        "ingeniería de turbinas eólicas",
        "energía geotérmica",
        "energía mareomotriz",
        "biomasa",
        "fusión nuclear",
    ]
```

> [!danger] Pregunta trampa — Trampa 4 y descomposición estrecha en código
> ```python
> # ❌ NO HACER: descomposición estrecha (el bug del ejemplo del resumen)
> def descomponer_tema_malo(tema: str) -> list[str]:
>     return ["avances en paneles solares", "ingeniería de turbinas eólicas"]
> ```
> **¿Por qué sería mala idea?** Aunque cada subagente de búsqueda investigue su subtema a la perfección, el reporte final nunca mencionará geotérmica, mareomotriz, biomasa ni fusión — no porque los subagentes fallaron, sino porque el coordinador nunca los asignó. **Y agregar más subagentes de búsqueda para "solar" o "eólica" no arregla nada** (Trampa 4): el problema está en la lista de `descomponer_tema`, no en cuántos subagentes ejecutan esa lista incompleta.

## Paso 4 — El coordinador: enrutar toda la comunicación (hub-and-spoke)

El coordinador invoca cada subagente de búsqueda, recolecta sus resultados, y es el **único** que le pasa esos resultados al agente de síntesis — los subagentes de búsqueda nunca hablan entre sí ni con el de síntesis directamente.

```python
def coordinador(tema: str) -> str:
    subtemas = descomponer_tema(tema)

    # el coordinador invoca cada subagente y centraliza los resultados
    hallazgos = [subagente_busqueda(st) for st in subtemas]

    # el coordinador decide qué le pasa al agente de síntesis
    reporte = subagente_sintesis(hallazgos)
    return reporte
```

> [!danger] Pregunta trampa — Trampa 3 en código (comunicación directa entre subagentes)
> ```python
> # ❌ NO HACER: dejar que un subagente invoque a otro directamente "para ahorrar una vuelta"
> def subagente_busqueda_que_llama_a_sintesis(subtema: str) -> str:
>     hallazgo = "..."  # investigación de este subagente
>     return subagente_sintesis([hallazgo])  # ¡llamada directa, sin pasar por el coordinador!
> ```
> **¿Por qué sería mala idea?** Aunque parezca más "eficiente", esto rompe los tres beneficios del hub-and-spoke: el coordinador pierde observabilidad sobre esa comunicación, no puede aplicar manejo de errores consistente, y pierde control sobre qué contexto recibe cada subagente (Trampa 3 del resumen).

## Paso 5 — Refinamiento iterativo: evaluar huecos de cobertura

El coordinador no se detiene en una sola pasada — evalúa la síntesis, detecta huecos, y re-delega con consultas más específicas antes de aceptar el resultado.

```python
def hay_huecos_de_cobertura(reporte: str, subtemas_esperados: list[str]) -> list[str]:
    """Evaluación simplificada: subtemas que el reporte no menciona."""
    return [st for st in subtemas_esperados if st.split()[-1].lower() not in reporte.lower()]

def coordinador_con_refinamiento(tema: str, max_rondas: int = 2) -> str:
    subtemas = descomponer_tema(tema)
    hallazgos = [subagente_busqueda(st) for st in subtemas]
    reporte = subagente_sintesis(hallazgos)

    for _ in range(max_rondas):
        faltantes = hay_huecos_de_cobertura(reporte, subtemas)
        if not faltantes:
            break
        # re-delegar solo los subtemas con hueco, con consultas más específicas
        hallazgos += [subagente_busqueda(f"detalle adicional sobre {st}") for st in faltantes]
        reporte = subagente_sintesis(hallazgos)

    return reporte
```

## Paso 6 — Manejo de errores centralizado en el coordinador

Siguiendo el beneficio de "manejo de errores consistente" del hub-and-spoke: si un subagente falla, el error se maneja en el coordinador, no se esconde silenciosamente dentro del subagente.

```python
def subagente_busqueda_seguro(subtema: str) -> dict:
    try:
        return {"subtema": subtema, "ok": True, "resultado": subagente_busqueda(subtema)}
    except Exception as e:
        # el coordinador decide qué hacer con esto: reintentar, omitir, o alertar
        return {"subtema": subtema, "ok": False, "error": str(e)}
```

> [!danger] Pregunta trampa — Trampa 1 en código (culpar al subagente equivocado)
> ```python
> # ❌ NO HACER: si el reporte final tiene huecos, "arreglar" el subagente de síntesis
> def subagente_sintesis_parcheado(hallazgos: list[str]) -> str:
>     # agregar lógica extra de "detección de huecos" AQUÍ, dentro de síntesis,
>     # como si el problema fuera que síntesis no se dio cuenta de que faltaba algo
>     ...
> ```
> **¿Por qué sería mala idea?** Si el reporte final solo cubre solar y eólica porque `descomponer_tema` nunca asignó geotérmica/mareomotriz/biomasa/fusión, el agente de síntesis está combinando *correctamente* los hallazgos que recibió — no tiene forma de sintetizar información que nunca le llegó. Parchar síntesis (o búsqueda) en vez de corregir la descomposición del coordinador es la Trampa 1: culpar a un subagente que funcionó bien dentro del alcance que se le asignó.

## Resultado

```python
reporte_final = coordinador_con_refinamiento("energías renovables")
print(reporte_final)
# Reporte que cubre solar, eólica, geotérmica, mareomotriz, biomasa y fusión —
# porque descomponer_tema() cubrió el dominio completo desde el inicio.
```

El flujo completo aplicó: hub-and-spoke estricto (toda comunicación pasa por `coordinador`), aislamiento de contexto (cada subagente solo recibe lo que se le pasa explícitamente), descomposición amplia del tema, un loop de refinamiento iterativo, manejo de errores centralizado, y evitó explícitamente las 4 trampas de examen del resumen.
