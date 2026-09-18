Aplicación práctica de [[1 resumen]]

## Caso de uso

Vamos a construir, paso a paso, el system prompt de un **pipeline de revisión de código en CI/CD** que aplica los cuatro conceptos del resumen: criterios explícitos, protección de la confianza ante falsos positivos, calibración de severidad con ejemplos de código, y la jerarquía "criterios primero, confianza después".

### Paso 1 — La versión ingenua (para ver por qué falla)

Antes de construir la versión correcta, así es como se ve el error típico que describe el resumen: instrucciones vagas.

```python
NAIVE_SYSTEM_PROMPT = """
You are a code reviewer. Review this pull request.
Be conservative. Only report high-confidence findings.
"""
```

> [!warning] Por qué esta forma "ingenua" es mala idea
> `"Be conservative"` y `"high-confidence"` no le dan al modelo ninguna frontera de decisión verificable — ver [[1 resumen#1. Vago vs. explícito: la diferencia real]]. En la práctica, correr este prompt varias veces contra el mismo PR produce clasificaciones distintas cada vez, porque el modelo está "interpretando" en lugar de "aplicando reglas". Nunca la uses como prompt de producción.

### Paso 2 — Definir categorías explícitas de qué reportar y qué omitir

Reemplazamos las instrucciones vagas por categorías concretas y un disparador explícito.

```python
CATEGORY_CRITERIA = """
Report findings only in these categories:

1. BUGS — logic errors that produce incorrect behavior (off-by-one errors,
   incorrect conditionals, unhandled edge cases with observable impact).
2. SECURITY — vulnerabilities such as unsanitised user input reaching a
   query, command, or file path.
3. DOCUMENTATION MISMATCH — flag a comment/docstring ONLY when the claimed
   behaviour contradicts the actual code behaviour. Do not flag comments
   that are merely incomplete or terse.

Skip entirely:
- Minor style preferences (naming conventions, formatting, line length).
- Local patterns already established elsewhere in the same module.
"""
```

Cada categoría tiene un límite verificable: "contradice el comportamiento real" es un hecho comprobable en el código, no una opinión.

### Paso 3 — Calibrar severidad con ejemplos de código, no con prosa

Ahora agregamos ejemplos de código concretos por nivel de severidad, en vez de descripciones abstractas.

```python
SEVERITY_CALIBRATION = """
Classify each finding using these severity examples as reference patterns:

CRITICAL — e.g. unsanitised user input in a query:
    query = f"SELECT * FROM users WHERE id = {user_input}"

MAJOR — e.g. resource not released on the error path:
    conn = get_connection()
    process(conn)  # missing conn.close() if process() raises

MINOR — e.g. inconsistent naming within the same module:
    userName = fetch_user()   # elsewhere in this file: user_name
"""
```

> [!warning] Trampa de examen aplicada aquí
> Si en vez de esto hubiéramos escrito `"Critical: issues that could cause system failures or data loss"`, el modelo tendría que *interpretar* qué cuenta como "falla del sistema" — la misma ambigüedad que con "conservador". Ver [[1 resumen#3. Calibrar severidad con código, no con prosa]]. Los ejemplos de código concretos eliminan esa interpretación.

### Paso 4 — Ensamblar el system prompt final

```python
SYSTEM_PROMPT = f"""
You are a code reviewer for pull requests.

{CATEGORY_CRITERIA}

{SEVERITY_CALIBRATION}

Output one finding per issue as JSON: {{"category": ..., "severity": ...,
"file": ..., "line": ..., "explanation": ...}}. Do not filter findings by
your own confidence — report every finding that matches a category above,
regardless of how certain you feel about it.
"""
```

La última línea aplica directamente la conclusión del resumen: **nunca uses la confianza auto-reportada como filtro primario** — ver [[1 resumen#4. Por qué el filtrado por confianza no resuelve el problema]].

### Paso 5 — Medir falsos positivos por categoría

Simulamos la medición que dispararía la estrategia de recuperación de confianza.

```python
def false_positive_rate(findings: list[dict], verified: list[dict]) -> dict[str, float]:
    """Calcula el % de falsos positivos por categoría, comparando contra
    una verificación humana (`verified`) de los mismos findings."""
    by_category: dict[str, list[bool]] = {}
    for f in findings:
        is_false_positive = f["id"] not in {v["id"] for v in verified if v["confirmed"]}
        by_category.setdefault(f["category"], []).append(is_false_positive)

    return {
        category: sum(flags) / len(flags)
        for category, flags in by_category.items()
    }

# Ejemplo de resultado observado en producción:
# {"BUGS": 0.05, "SECURITY": 0.02, "DOCUMENTATION MISMATCH": 0.40}
```

### Paso 6 — Aplicar la estrategia de recuperación de confianza

Con un 40% de falsos positivos en `DOCUMENTATION MISMATCH`, aplicamos la solución contraintuitiva del resumen: **deshabilitar la categoría problemática, no seguir puliéndola mientras sigue activa**.

```python
FALSE_POSITIVE_THRESHOLD = 0.25

def active_categories(rates: dict[str, float]) -> list[str]:
    """Deshabilita temporalmente cualquier categoría por encima del umbral,
    para proteger la confianza en las categorías que sí funcionan."""
    return [category for category, rate in rates.items() if rate <= FALSE_POSITIVE_THRESHOLD]

rates = {"BUGS": 0.05, "SECURITY": 0.02, "DOCUMENTATION MISMATCH": 0.40}
print(active_categories(rates))
# ['BUGS', 'SECURITY'] — DOCUMENTATION MISMATCH queda fuera del reporte
# mientras se refinan sus criterios con más ejemplos de código.
```

```python
CATEGORY_CRITERIA_REFINED = CATEGORY_CRITERIA.replace(
    "3. DOCUMENTATION MISMATCH — flag a comment/docstring ONLY when the claimed\n"
    "   behaviour contradicts the actual code behaviour. Do not flag comments\n"
    "   that are merely incomplete or terse.",
    """3. DOCUMENTATION MISMATCH — flag ONLY when the docstring/comment makes a
   verifiable factual claim contradicted by the code. Examples:
     FLAG:  "# returns sorted list" but the function never sorts
     SKIP:  "# helper function" (vague, not a verifiable factual claim)
     SKIP:  a docstring missing a @param description (incomplete, not wrong)""",
)
```

> [!warning] Otra trampa aplicada aquí
> No caigas en la tentación de "arreglar" `DOCUMENTATION MISMATCH` agregando `"only flag with high confidence"` al prompt de esa categoría — sería reintroducir exactamente el problema del Paso 1. La forma correcta de refinarla es añadir más pares FLAG/SKIP concretos, como arriba, no lenguaje de confianza.

### Paso 7 — Reactivar solo cuando mejora la precisión

```python
def maybe_reenable(category: str, new_rate: float) -> bool:
    return new_rate <= FALSE_POSITIVE_THRESHOLD

# Tras refinar con ejemplos FLAG/SKIP, medimos de nuevo:
new_rates = {"DOCUMENTATION MISMATCH": 0.12}
if maybe_reenable("DOCUMENTATION MISMATCH", new_rates["DOCUMENTATION MISMATCH"]):
    print("Reactivar DOCUMENTATION MISMATCH en el pipeline")
```

Con esto el pipeline recupera las tres categorías activas sin haber sacrificado la confianza de los desarrolladores en `BUGS` y `SECURITY` mientras se refinaba `DOCUMENTATION MISMATCH`.

---

> [!tip] Repasa los conceptos en [[1 resumen]], memorízalos en [[3 cuestionario]] y evalúate en [[4 test]]
