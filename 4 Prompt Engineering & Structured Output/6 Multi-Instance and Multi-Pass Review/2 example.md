Aplicación práctica de [[1 resumen]]

## Caso de uso

Un pipeline de CI/CD necesita revisar un pull request que modifica 14 archivos del módulo de stock tracking (el mismo escenario de la Pregunta 12 oficial del exam guide). Vamos a construir el pipeline aplicando, en orden, los conceptos del resumen: por qué no basta con pedirle a la misma instancia que generó el diff que se autorrevise, por qué no basta con un solo pase sobre los 14 archivos, la arquitectura de dos pasadas (por archivo + integración), y el ruteo de hallazgos por confianza calibrada.

### Paso 0 — La versión ingenua (para ver por qué falla)

Antes de construir el pipeline correcto, así se ve la forma más intuitiva pero incorrecta: la misma instancia que generó una corrección de código se revisa a sí misma en la misma conversación.

```python
import anthropic

client = anthropic.Anthropic()

def naive_self_review(pr_diff: str) -> str:
    messages = [
        {"role": "user", "content": f"Fix the bugs in this diff:\n{pr_diff}"},
    ]
    generation = client.messages.create(
        model="claude-sonnet-5", max_tokens=4096, messages=messages,
    )
    messages.append({"role": "assistant", "content": generation.content})
    messages.append({
        "role": "user",
        "content": "Now review your fix carefully and point out any remaining issues.",
    })
    review = client.messages.create(
        model="claude-sonnet-5", max_tokens=2048, messages=messages,
    )
    return review.content[0].text
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> Es [[1 resumen#1. La limitación del self-review en la misma sesión]]: `review` se ejecuta dentro de la *misma* lista `messages` que contiene el razonamiento original de `generation`. El modelo ya "sabe" por qué escribió cada línea de la corrección, así que tiende a confirmar su propio trabajo en vez de cuestionarlo críticamente — sin importar qué tan insistente sea la instrucción "revisa con cuidado".

### Paso 1 — Revisión con una instancia independiente

La corrección: una invocación *separada*, sin el historial de mensajes de la generación — solo recibe el resultado final, como si lo viera por primera vez.

```python
def independent_review(pr_diff: str, generated_fix: str) -> str:
    review = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=2048,
        messages=[{
            "role": "user",
            "content": (
                "You are reviewing a code fix with no knowledge of why it was written. "
                f"Original diff:\n{pr_diff}\n\nProposed fix:\n{generated_fix}\n\n"
                "Identify any bugs, security issues, or logic errors."
            ),
        }],
    )
    return review.content[0].text
```

> [!note] Conexión con el resumen
> Esta invocación no lleva ningún mensaje previo con el razonamiento de `generation` — es [[1 resumen#1. La limitación del self-review en la misma sesión]] aplicado: una instancia que "encuentra la salida con ojos frescos", sin el sesgo de anclaje de "elegí este enfoque porque...".

### Paso 2 — La versión ingenua de revisar 14 archivos de golpe

Con la independencia resuelta, el siguiente error natural es mandar los 14 archivos del PR en un solo prompt.

```python
def naive_single_pass_review(files: dict[str, str]) -> str:
    all_files_text = "\n\n".join(f"### {path}\n{content}" for path, content in files.items())
    review = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=4096,
        messages=[{
            "role": "user",
            "content": f"Review all 14 files in this PR for bugs:\n{all_files_text}",
        }],
    )
    return review.content[0].text
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> Es [[1 resumen#2. Arquitectura de multi-pass review]]: meter los 14 archivos en un solo prompt produce attention dilution — profundidad desigual entre archivos, bugs perdidos a la mitad de la revisión, y el mismo patrón marcado como crítico en un archivo pero aprobado en otro. Cambiar a un modelo con ventana de contexto más grande *no* resuelve esto — ver [[1 resumen#3. Por qué una ventana de contexto más grande NO resuelve esto]] y la Trampa 3 del resumen.

### Paso 3 — Pass 1: análisis local por archivo, en paralelo

Cada archivo se revisa en su propia invocación independiente, con un prompt enfocado solo en ese archivo. Al ser invocaciones independientes entre sí, se paralelizan.

```python
import asyncio

async_client = anthropic.AsyncAnthropic()

async def review_single_file(path: str, content: str) -> dict:
    response = await async_client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": (
                f"Review ONLY this file for bugs, security issues, and logic errors. "
                f"Do not assume anything about other files in the PR.\n\n"
                f"### {path}\n{content}"
            ),
        }],
    )
    return {"path": path, "findings": response.content[0].text}


async def per_file_pass(files: dict[str, str]) -> list[dict]:
    tasks = [review_single_file(path, content) for path, content in files.items()]
    return await asyncio.gather(*tasks)
```

> [!note] Conexión con el resumen
> `asyncio.gather` paraleliza las 14 invocaciones porque son independientes entre sí — cada una recibe la atención completa e indivisa del modelo sobre un solo archivo, garantizando la profundidad uniforme que describe [[1 resumen#2. Arquitectura de multi-pass review]].

### Paso 4 — Pass 2: integración cross-file

Una vez que las 14 pasadas locales terminan, una invocación separada recibe *todos* los hallazgos y busca problemas que solo aparecen al comparar archivos entre sí.

```python
async def integration_pass(per_file_findings: list[dict]) -> str:
    findings_summary = "\n\n".join(
        f"### {f['path']}\n{f['findings']}" for f in per_file_findings
    )
    response = await async_client.messages.create(
        model="claude-sonnet-5",
        max_tokens=2048,
        messages=[{
            "role": "user",
            "content": (
                "Here are per-file review findings from a 14-file PR. Check for: "
                "data flowing between files in incompatible formats, the same pattern "
                "flagged differently across files, API contract violations at module "
                "boundaries, and contradictions between the per-file findings themselves.\n\n"
                f"{findings_summary}"
            ),
        }],
    )
    return response.content[0].text
```

> [!warning] Trampa de examen aplicada aquí
> Saltarse este paso y quedarse solo con el Paso 3 sería incompleto: las pasadas por archivo, por diseño, nunca ven más de un archivo a la vez, así que jamás detectarían por sí solas un contrato de API roto entre dos módulos. Es exactamente lo que pide la Pregunta 12 oficial (ver `4 test.md`): "análisis local por archivo" **más** "pasada separada de integración cross-file" — ninguna de las dos sola es la respuesta completa.

### Paso 5 — Confianza cruda vs. confianza calibrada

Cada hallazgo se reporta con un puntaje de confianza, pero ese número crudo no se usa todavía para ruteo automático — primero se calibra contra un dataset etiquetado.

```python
import json

def parse_finding_with_confidence(raw_finding: str) -> dict:
    # En producción esto vendría de tool use con un schema; simplificado aquí.
    return json.loads(raw_finding)


def is_calibration_reliable(labeled_validation_set: list[dict]) -> float:
    """Corre el sistema sobre ejemplos ya etiquetados y mide la correlación
    entre confianza reportada y precisión real verificada."""
    correct_at_high_confidence = sum(
        1 for item in labeled_validation_set
        if item["reported_confidence"] >= 0.8 and item["was_actually_correct"]
    )
    total_high_confidence = sum(
        1 for item in labeled_validation_set if item["reported_confidence"] >= 0.8
    )
    if total_high_confidence == 0:
        return 0.0
    return correct_at_high_confidence / total_high_confidence


def route_finding(finding: dict, calibrated_threshold: float) -> str:
    if finding["confidence"] >= calibrated_threshold:
        return "direct_report"
    return "human_review"
```

```python
example_finding = {
    "severity": "medium",
    "confidence": 0.65,
    "reasoning": "Pattern resembles an off-by-one error, but the surrounding "
                 "loop bounds are defined in a file not included in this review batch.",
    "custom_id": "inventory_sync.py-finding-3",
}
```

> [!warning] Por qué NO usar `finding["confidence"] >= 0.8` directamente sin calibrar
> Es [[1 resumen#4. Ruteo por confianza calibrada]]: el 0.8 en `example_finding` es la propia sensación de certeza del modelo, no una cifra validada contra precisión real. Rutear directo a `direct_report` con ese número crudo — sin antes correr `is_calibration_reliable` sobre un dataset etiquetado — es la Trampa 4 del resumen: se automatizan decisiones sobre una señal que nunca se verificó que fuera confiable.

### Paso 6 — Ensamblar el pipeline completo

```python
async def review_pull_request(pr_files: dict[str, str], calibrated_threshold: float) -> dict:
    per_file_findings = await per_file_pass(pr_files)          # Pass 1
    integration_findings = await integration_pass(per_file_findings)  # Pass 2

    routed = {"direct_report": [], "human_review": []}
    for f in per_file_findings:
        finding = parse_finding_with_confidence(f["findings"])
        routed[route_finding(finding, calibrated_threshold)].append(finding)

    return {
        "per_file": per_file_findings,
        "integration": integration_findings,
        "routing": routed,
    }
```

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
