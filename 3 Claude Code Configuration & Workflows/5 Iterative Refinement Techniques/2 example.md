Aplicación práctica de [[1 resumen]] — una sesión de trabajo real con Claude Code migrando un sistema legacy de usuarios, donde se aplican (en orden) las cuatro decisiones del tema: ejemplos concretos ante interpretación inconsistente, test-driven iteration para la transformación compleja, interview pattern para el dominio desconocido, y batch vs sequential feedback.

## Escenario

Un equipo migra el servicio `accounts` de un formato legacy a uno nuevo. Hay tres tareas:

1. **Refactor de firmas**: envolver los tipos de retorno de las funciones async en `Result[T, ApiError]` (el developer sabe exactamente qué quiere).
2. **Script de migración**: convertir registros legacy (CSV exportado) a JSON — con muchos edge cases (nulos, vacíos, fechas raras) y un requisito de rendimiento.
3. **Capa de caché** para la API de lookup que usa la migración — un dominio que el developer no conoce bien.

Los prompts se envían con la CLI de Claude Code (`claude -p` para modo no interactivo, `claude -c` para continuar la conversación más reciente). El código de la aplicación es Python.

## Paso 1 — La versión ingenua: describir la transformación en prosa

El developer describe el refactor en prosa y lo corre tres veces sobre archivos distintos:

```bash
claude -p "Refactor the async functions in accounts/api.py so they return a Result \
type that wraps success values and API errors instead of raising exceptions."
```

Resultados observados en tres corridas:

```python
# Corrida 1 — envolvió el tipo, pero con otro nombre de error
async def get_user_data(user_id: str) -> Result[UserData, Exception]: ...

# Corrida 2 — devolvió una tupla en vez de Result
async def fetch_orders(customer_id: str) -> tuple[list[Order] | None, ApiError | None]: ...

# Corrida 3 — envolvió también funciones síncronas que no debía tocar
def format_name(first: str, last: str) -> Result[str, ApiError]: ...
```

> [!danger] Pregunta trampa — "arreglarlo" con prosa más precisa
> ```bash
> claude -p "Refactor ONLY the async functions in accounts/api.py. The return type \
> MUST be Result[T, ApiError] where T is the original success type, using the Result \
> generic from accounts.result. Do NOT use tuples. Do NOT touch synchronous functions. \
> Use precise algebraic-data-type semantics for the error channel."   # ❌
> ```
> **¿Por qué sería mala idea?** Es la Trampa 1 del resumen: una prosa más precisa **sigue dependiendo de interpretación** — solo cambias qué ambigüedad queda. El síntoma "interpreta distinto cada vez" pide cambiar de técnica (ejemplos), no pulir la redacción. Además, cuantas más reglas en prosa, más superficie para interpretarlas distinto.

## Paso 2 — Cambiar a 2-3 ejemplos concretos de input/output

Se aplica el proceso de 4 pasos del resumen: ya se **observó la inconsistencia**; ahora se **cambia a ejemplos**.

```bash
claude -p "Apply this exact transformation to every async function in accounts/api.py.

Input:
  async def get_user_data(user_id: str) -> UserData
Expected output:
  async def get_user_data(user_id: str) -> Result[UserData, ApiError]

Input:
  async def fetch_orders(customer_id: str) -> list[Order]
Expected output:
  async def fetch_orders(customer_id: str) -> Result[list[Order], ApiError]

Input (edge case — already Optional):
  async def find_invoice(invoice_id: str) -> Invoice | None
Expected output:
  async def find_invoice(invoice_id: str) -> Result[Invoice | None, ApiError]"
```

Dos pares cubren el **caso estándar** y el tercero cubre **un edge case clave** (tipo ya opcional) — no hace falta más.

**Verificar la generalización** (paso 3 del proceso): se corre sobre un archivo nuevo que el modelo no ha visto y se revisa que aplique el mismo patrón.

```bash
claude -c -p "Now apply the same transformation to accounts/billing.py and show me the diff."
```

```python
# Resultado esperado en billing.py — el patrón generalizó correctamente
async def get_balance(account_id: str) -> Result[Decimal, ApiError]: ...
async def list_payments(account_id: str) -> Result[list[Payment], ApiError]: ...
```

> [!danger] Pregunta trampa — apilar 20 ejemplos "para estar seguros"
> Pegar un ejemplo por cada función del módulo no mejora la generalización: 2-3 bien elegidos (estándar + edge case clave) fijan el patrón. Si en la verificación falla un edge case concreto (ej. funciones que devuelven `AsyncIterator`), se **agrega un ejemplo específico de ese edge case** (paso 4 del proceso), no una avalancha de ejemplos redundantes.

## Paso 3 — Test-driven iteration: los tests primero

El script de migración es una **transformación compleja con muchos edge cases** → test-driven iteration. Antes de pedir implementación, se escriben los tests que cubren happy path, edge cases y performance.

```python
# tests/test_migrate_users.py
import json
import time

from migrate_users import migrate_record, migrate_batch


# --- Happy path -----------------------------------------------------------
def test_migration_standard_record():
    legacy = {"id": "42", "full_name": "Ana López", "email": "ana@ex.com",
              "created": "2021-03-05", "phone": "555-0101"}
    out = json.loads(migrate_record(legacy))
    assert out == {
        "id": 42,
        "name": {"first": "Ana", "last": "López"},
        "email": "ana@ex.com",
        "created_at": "2021-03-05T00:00:00Z",
        "phone": "555-0101",
    }


# --- Edge cases -----------------------------------------------------------
def test_migration_handles_null_values():
    legacy = {"id": "7", "full_name": "Luis Pérez", "email": "l@ex.com",
              "created": "2020-01-01", "phone": None}
    out = json.loads(migrate_record(legacy))
    assert out["phone"] is None  # null debe preservarse, no convertirse en ""


def test_migration_handles_empty_name():
    legacy = {"id": "8", "full_name": "", "email": "x@ex.com",
              "created": "2020-01-01", "phone": None}
    out = json.loads(migrate_record(legacy))
    assert out["name"] == {"first": None, "last": None}


def test_migration_single_word_name():
    legacy = {"id": "9", "full_name": "Cher", "email": "c@ex.com",
              "created": "2020-01-01", "phone": None}
    out = json.loads(migrate_record(legacy))
    assert out["name"] == {"first": "Cher", "last": None}


# --- Performance requirement ---------------------------------------------
def test_migration_batch_performance():
    records = [{"id": str(i), "full_name": "A B", "email": f"{i}@ex.com",
                "created": "2020-01-01", "phone": None} for i in range(100_000)]
    start = time.perf_counter()
    migrate_batch(records)
    assert time.perf_counter() - start < 2.0  # 100k registros en < 2 s
```

Luego se pide la implementación contra esos tests:

```bash
claude -p "Implement migrate_record and migrate_batch in migrate_users.py so that \
all tests in tests/test_migrate_users.py pass. Do not modify the tests."
```

## Paso 4 — Iterar compartiendo los test failures

La primera implementación falla dos tests. En lugar de explicar en prosa qué está mal, se le pasa la salida de pytest tal cual:

```bash
pytest tests/test_migrate_users.py -q 2>&1 | claude -c -p "Fix migrate_users.py \
based on these test failures. Do not modify the tests."
```

Lo que Claude Code recibe (feedback inequívoco — "Expected X, got Y"):

```text
FAILED tests/test_migrate_users.py::test_migration_handles_null_values
  assert '' is None
   +  where '' = {...}['phone']
  Expected: null preserved in output JSON
  Actual: null replaced with empty string ""

FAILED tests/test_migrate_users.py::test_migration_single_word_name
  AssertionError: assert {'first': 'Cher', 'last': ''} == {'first': 'Cher', 'last': None}
2 failed, 3 passed in 1.41s
```

La corrección que produce es dirigida, sin interpretación:

```python
# migrate_users.py (fragmento corregido)
def _split_name(full_name: str) -> dict[str, str | None]:
    parts = full_name.split(maxsplit=1)
    return {
        "first": parts[0] if parts else None,
        "last": parts[1] if len(parts) > 1 else None,
    }


def migrate_record(legacy: dict) -> str:
    return json.dumps({
        "id": int(legacy["id"]),
        "name": _split_name(legacy["full_name"]),
        "email": legacy["email"],
        "created_at": f"{legacy['created']}T00:00:00Z",
        "phone": legacy["phone"],  # antes: legacy["phone"] or ""  ← el bug
    }, ensure_ascii=False)
```

> [!danger] Pregunta trampa — describir el bug en prosa en vez de compartir el fallo
> ```bash
> claude -c -p "The migration is mishandling missing phone numbers somehow, \
> and some names look off. Please make it more robust."   # ❌
> ```
> **¿Por qué sería mala idea?** "Más robusto" se puede interpretar como convertir nulos a `""` (¡justo el bug!), como lanzar excepciones, o como saltar registros. El test failure dice **exactamente** qué se esperaba y qué salió — no deja espacio a interpretación. Por eso el Task Statement pide "provide specific test cases with example input and expected output to fix edge case handling (e.g., null values in migration scripts)".

## Paso 5 — Interview pattern para la capa de caché (dominio desconocido)

La migración hace millones de lookups a la API de `accounts` y el developer necesita una caché, pero nunca ha diseñado una. Aquí **no** sirven los ejemplos (no sabe cuál es el output correcto) — el hueco de conocimiento está en el developer, así que se usa interview pattern.

> [!danger] Pregunta trampa — prescribir la solución en un dominio que no dominas
> ```bash
> claude -p "Build me a caching layer for the accounts API."   # ❌
> ```
> **¿Por qué sería mala idea?** Claude implementará *una* caché razonable, pero las decisiones críticas (qué pasa si un usuario cambia su email durante la migración, qué pasa si la caché se cae) quedan resueltas por defecto, sin que el developer sepa siquiera que existían.

```bash
claude -p "I need a caching layer for the accounts API used by the user migration. \
Before implementing, ask me questions about the requirements, edge cases, and \
constraints I should consider. Do not write any code until I've answered."
```

Preguntas típicas que surgen (consideraciones que un experto abordaría):

```text
1. Cache invalidation: if an account is updated in the source system during the
   migration, must the migration see the new value, or is a snapshot acceptable?
2. TTL policy: how stale can a cached account be? Minutes? The whole run?
3. Consistency: can two workers read different versions of the same account?
4. Failure modes: if the cache backend is down, should the migration fall back to
   the API, slow down, or stop?
5. Memory bounds: how many accounts fit in memory? Do we need LRU eviction?
```

Solo después de responderlas se pide la implementación:

```bash
claude -c -p "Answers: snapshot is fine; TTL = whole run; single worker; if the cache \
fails, fall back to the API and log a warning; cap at 500k entries with LRU. \
Now implement it."
```

> [!danger] Pregunta trampa — confundir interview con examples
> Si el developer hubiera intentado dar "ejemplos de input/output" de la caché, no habría sabido cuál es el output correcto para "cuenta modificada durante la migración" — ese es justamente el conocimiento que le falta. Al revés, en el Paso 1 usar interview pattern habría sido absurdo: el developer ya sabía **exactamente** qué transformación quería; el problema era que el modelo la interpretaba distinto. Diagnóstico: **¿el hueco está en ti (→ interview) o en la interpretación del modelo (→ examples)?**

## Paso 6 — Batch vs sequential feedback

Tras revisar el código, quedan cinco cosas por corregir. Se clasifican según si **arreglar una afecta a otra**:

```python
# Clasificación de los issues pendientes
issues = {
    # Grupo A — interactúan: cambiar el formato de error cambia el logging y los tipos
    "error_code": "Error responses must include an error_code field",
    "logging": "Structured logs must include the error_code",
    "sdk_types": "Client SDK type definitions must reflect the new error_code field",
    # Grupo B — independientes entre sí y del grupo A
    "naming": "Rename helper functions to snake_case consistently",
    "indent": "Reformat migrate_users.py to 4-space indentation",
}
```

**Grupo A → batch**: los tres en un solo mensaje, para que el modelo vea todas las restricciones a la vez y produzca un arreglo coherente.

```bash
claude -c -p "Three changes needed (they interact with each other):
1. Error responses must include an error_code field
2. Structured logging must include the error_code
3. The client SDK type definitions must reflect the new error_code field"
```

**Grupo B → secuencial**: uno por iteración, esperando el resultado entre cada uno.

```bash
claude -c -p "Rename the helper functions in migrate_users.py to snake_case consistently."
# [revisar el resultado]
claude -c -p "Now reformat migrate_users.py to 4-space indentation."
```

> [!danger] Pregunta trampa — secuenciar los issues que interactúan
> ```bash
> claude -c -p "Add an error_code field to error responses."          # ❌ iteración 1
> claude -c -p "Now add the error code to the logs."                   # ❌ iteración 2
> claude -c -p "Now update the SDK types for the new field."           # ❌ iteración 3
> ```
> **¿Por qué sería mala idea?** En la iteración 1 el modelo elige una forma para `error_code` (¿string? ¿int? ¿anidado en `error.code`?) sin saber que el logging y el SDK dependen de ella. Las iteraciones 2 y 3 tienen que adaptarse a esa decisión o romperla — cada arreglo puede chocar con el siguiente.

> [!danger] Pregunta trampa — meter todo en un solo mensaje "para ahorrar tiempo"
> Juntar los cinco issues (A + naming + indentación) en un mensaje mezcla problemas independientes con los interdependientes y puede confundir al modelo sobre qué feedback aplica a qué parte del código. La regla no es "siempre batch" ni "siempre secuencial": es **¿arreglar A afecta B?**

## Resumen de decisiones del ejemplo

| Paso | Situación | Técnica aplicada |
|---|---|---|
| 1-2 | Prosa interpretada distinto en cada corrida | Concrete input/output examples (2-3, con un edge case) |
| 3-4 | Migración compleja con nulos, vacíos y requisito de performance | Test-driven iteration + compartir test failures |
| 5 | Caché en un dominio desconocido | Interview pattern |
| 6 (A) | error_code + logging + SDK types se afectan | Batch feedback |
| 6 (B) | naming e indentación independientes | Sequential feedback |

---
> [!tip] Sigue con este tema
> Vuelve a la teoría en [[1 resumen]], repasa con [[3 cuestionario]] y evalúate con [[4 test]].
