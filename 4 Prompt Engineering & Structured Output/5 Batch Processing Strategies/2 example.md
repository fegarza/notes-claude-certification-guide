Aplicación práctica de [[1 resumen]]

## Caso de uso

Un equipo tiene dos workflows que hoy usan llamadas síncronas a Claude: (1) un **pre-merge check** que bloquea el merge de un PR hasta tener resultado, y (2) un **reporte de deuda técnica** que se genera de noche sobre 200 archivos, para revisión la mañana siguiente. Vamos a aplicar la regla de matching del resumen, construir el envío del batch con `custom_id`, manejar fallos reenviando solo lo que falló, calcular el scheduling contra un SLA, y aplicar el paso de refinamiento sobre una muestra antes de escalar.

### Paso 0 — La versión ingenua (para ver por qué falla)

Antes de construir el pipeline correcto, así se ve la propuesta que el resumen descarta: mover *todo* a batch porque es más barato.

```python
import anthropic

client = anthropic.Anthropic()

def naive_migrate_everything_to_batch(pr_diff: str, debt_report_docs: list[str]):
    all_requests = [
        {
            "custom_id": "pre-merge-check",
            "params": {
                "model": "claude-sonnet-5",
                "max_tokens": 4096,
                "messages": [{"role": "user", "content": pr_diff}],
            },
        }
    ] + [
        {
            "custom_id": f"debt-report-{i}",
            "params": {
                "model": "claude-sonnet-5",
                "max_tokens": 4096,
                "messages": [{"role": "user", "content": doc}],
            },
        }
        for i, doc in enumerate(debt_report_docs)
    ]
    return client.messages.batches.create(requests=all_requests)
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> Meter `pre-merge-check` en el mismo batch que los reportes de deuda técnica ignora [[1 resumen#2. La regla de matching: síncrono vs. batch]]: el desarrollador que abrió el PR está bloqueado esperando ese resultado *ahora*, y la API de batch no da ningún SLA de latencia — puede tardar minutos o hasta 24 horas. El ahorro del 50% no compensa dejar a un desarrollador bloqueado un día entero por un merge.

### Paso 1 — Clasificar workflows antes de tocar código

Aplicamos la regla de matching primero, en texto, no en código: ¿alguien está esperando el resultado en tiempo real?

```python
WORKFLOWS = {
    "pre_merge_check": {"blocking": True, "api": "sync"},
    "technical_debt_report": {"blocking": False, "api": "batch"},
}
```

El pre-merge check se queda en la API síncrona. Solo el reporte de deuda técnica es candidato a batch.

### Paso 2 — Refinar el prompt sobre una muestra antes de escalar

Antes de enviar los 200 documentos, se prueban 8 documentos representativos (distintos lenguajes, tamaños, estilos) con la API síncrona, iterando el prompt hasta lograr buena precisión.

```python
def refine_on_sample(sample_docs: list[str], prompt_template: str) -> str:
    for doc in sample_docs:
        response = client.messages.create(
            model="claude-sonnet-5",
            max_tokens=2048,
            messages=[{"role": "user", "content": prompt_template.format(document=doc)}],
        )
        print(response.content[0].text[:200])
    # Se ajusta prompt_template manualmente entre iteraciones hasta que
    # la muestra completa produce salidas consistentes y bien formadas.
    return prompt_template
```

> [!note] Conexión con el resumen
> Este paso es [[1 resumen#5. Refinar antes de escalar: el paso que más ahorra]]: saltárselo y enviar el prompt sin probar directamente a los 200 documentos es la forma más cara de descubrir sus fallas — cada fallo detectado tarde cuesta un reenvío completo dentro de otro batch.

### Paso 3 — Construir y enviar el batch con `custom_id`

Cada documento del reporte de deuda técnica se envía como un item del batch, con un `custom_id` que permite emparejar la respuesta con el documento original más adelante.

```python
def submit_debt_report_batch(documents: dict[str, str], refined_prompt: str):
    requests = [
        {
            "custom_id": doc_id,
            "params": {
                "model": "claude-sonnet-5",
                "max_tokens": 4096,
                "messages": [{"role": "user", "content": refined_prompt.format(document=text)}],
            },
        }
        for doc_id, text in documents.items()
    ]
    return client.messages.batches.create(requests=requests)
```

### Paso 4 — Calcular la frecuencia de envío contra el SLA

El negocio exige que el reporte esté listo en un máximo de 30 horas desde que se generan los documentos. Aplicamos el cálculo del resumen.

```python
SLA_HOURS = 30
MAX_BATCH_WINDOW_HOURS = 24

def submission_interval_hours(sla_hours: int = SLA_HOURS, batch_window_hours: int = MAX_BATCH_WINDOW_HOURS) -> int:
    buffer_hours = sla_hours - batch_window_hours
    if buffer_hours <= 0:
        raise ValueError("El SLA no deja margen: la ventana máxima de batch ya lo excede.")
    # Se somete un batch nuevo con suficiente frecuencia para que, aun si uno
    # tarda el máximo de 24h, siempre haya otro en vuelo dentro del margen.
    #
    # El "-2" es un valor arbitrario de este ejemplo, no una cifra que exija
    # la guía de certificación: reserva parte del buffer como margen operativo
    # (validar inputs, absorber demoras propias) en vez de gastar el buffer
    # completo (6h) como intervalo de envío, lo cual dejaría cero margen extra.
    OPERATIONAL_SAFETY_MARGIN_HOURS = 2
    return buffer_hours - OPERATIONAL_SAFETY_MARGIN_HOURS

interval = submission_interval_hours()  # 30 - 24 = 6h de buffer -> cada 4h
```

> [!warning] Trampa de examen aplicada aquí
> Es [[1 resumen#3. Cálculo de scheduling contra un SLA]]: las 24 horas son el **peor caso**, no el tiempo típico. Calcular `submission_interval_hours` contra el mejor caso (ej. "normalmente tarda 2 horas, así que hay margen de sobra") es exactamente la Trampa 2 del resumen — funcionaría en el caso feliz y fallaría el día que un batch sí tarde el máximo.

### Paso 5 — Poll hasta que termine y separar fallos de éxitos

`batches.results()` es un iterable, no una lista lista para usar de inmediato — hay que esperar a que el batch termine antes de leerlo, y tratar `errored` y `expired` como el mismo tipo de fallo.

```python
import time

def poll_until_done(batch_id: str, poll_seconds: int = 60):
    batch = client.messages.batches.retrieve(batch_id)
    while batch.processing_status != "ended":
        time.sleep(poll_seconds)
        batch = client.messages.batches.retrieve(batch_id)
    return batch


def split_results(batch_id: str) -> tuple[dict[str, str], list[str]]:
    succeeded: dict[str, str] = {}
    failed_ids: list[str] = []

    for entry in client.messages.batches.results(batch_id):
        if entry.result.type == "succeeded":
            succeeded[entry.custom_id] = entry.result.message.content[0].text
        elif entry.result.type in ("errored", "expired"):
            failed_ids.append(entry.custom_id)

    return succeeded, failed_ids
```

> [!note] Conexión con el resumen
> Tratar `"expired"` igual que `"errored"` aplica [[1 resumen#4. Manejo de fallos de batch: reenviar solo lo que falló]]: un item que no alcanzó a procesarse dentro de las 24 horas no tuvo éxito silenciosamente, necesita el mismo tratamiento de reenvío que un error explícito.

### Paso 6 — Reenviar solo los fallidos, con modificaciones

Se reenvían únicamente los documentos identificados por `custom_id` en `failed_ids`, nunca el batch completo — y con una modificación dirigida al motivo probable del fallo (documentos que excedieron el contexto se dividen en chunks).

```python
def chunk_document(text: str, max_chars: int = 20_000) -> str:
    return text[:max_chars]  # simplificado: en producción se dividiría en varias requests


def resubmit_failures(documents: dict[str, str], failed_ids: list[str], refined_prompt: str):
    retry_requests = [
        {
            "custom_id": f"{doc_id}-retry-1",
            "params": {
                "model": "claude-sonnet-5",
                "max_tokens": 4096,
                "messages": [{
                    "role": "user",
                    "content": refined_prompt.format(document=chunk_document(documents[doc_id])),
                }],
            },
        }
        for doc_id in failed_ids
    ]
    return client.messages.batches.create(requests=retry_requests)
```

> [!warning] Trampa de examen aplicada aquí
> Reenviar `documents` completo (los 200) en vez de solo `documents[doc_id] for doc_id in failed_ids` sería la Trampa 1 en miniatura: pagar de nuevo por documentos que ya tuvieron éxito. El punto de `custom_id` es precisamente permitir este filtro quirúrgico — ver [[1 resumen#4. Manejo de fallos de batch: reenviar solo lo que falló]].

### Paso 7 — El paso que debe quedarse fuera del batch

Si una parte del análisis de deuda técnica necesitara ejecutar una tool (ej. consultar un linter externo) y usar su resultado antes de seguir razonando en la misma conversación, ese paso no puede vivir dentro de una request de batch con una client tool.

```python
def run_linter_step(file_content: str) -> dict:
    # Esta llamada SÍ requiere la API síncrona: el modelo necesita ejecutar
    # la tool `run_linter` (client tool, código propio) y usar su resultado
    # para continuar razonando en el mismo turno — el batch no lo soporta.
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=2048,
        tools=[{
            "name": "run_linter",
            "description": "Runs a static linter over the given file content.",
            "input_schema": {
                "type": "object",
                "properties": {"file_content": {"type": "string"}},
                "required": ["file_content"],
            },
        }],
        messages=[{"role": "user", "content": f"Lint this file:\n{file_content}"}],
    )
    return response.content
```

> [!warning] Por qué no meter esto en el batch de deuda técnica
> Es [[1 resumen#6. La limitación de multi-turn tool calling]]: la API de batch no puede ejecutar `run_linter` a mitad de una request y seguir razonando con su resultado dentro del mismo item — `run_linter` es una client tool que ejecuta el propio código del pipeline, no una server tool. Intentar forzar este paso dentro del batch de deuda técnica requeriría partirlo en dos requests de batch separadas (una que termina en `tool_use`, otra de seguimiento con el resultado ya inyectado), lo cual es más complejo que simplemente correr este paso específico en la API síncrona.

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
