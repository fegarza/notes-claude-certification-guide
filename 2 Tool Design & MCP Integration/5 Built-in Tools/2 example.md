---
tags:
  - claude-cert/dominio-2
  - task-statement/2.5
---

# 2 example — Built-in Tools

Aplicación práctica de [[1 resumen]]

Vamos a resolver, paso a paso, el escenario de deprecación completo: encontrar todos los llamadores de una función deprecada `processLegacyOrder()`, incluyendo los que la usan a través de un wrapper, encontrar sus tests, y reemplazar cada llamada — aplicando las cinco conclusiones del resumen: Grep vs. Glob, Edit vs. Read+Write, exploración incremental, rastreo a través de wrappers/barriles, y el patrón Grep → Glob → Grep.

## Paso 1 — Grep para encontrar los puntos de entrada

Aplicamos la [[1 resumen#3. Comprensión incremental del código base nunca leer todo de entrada|Conclusión 3]]: nunca empezamos leyendo todo el código base. Empezamos acotado, con Grep, porque `processLegacyOrder` es texto que buscamos **dentro** de archivos.

```
Grep: "processLegacyOrder"

-> src/OrderProcessor.ts:42:  await processLegacyOrder(orderId)
-> src/RefundHandler.ts:18:   const result = processLegacyOrder(orderId, { partial: true })
-> src/legacy/orders.ts:7:    export function processLegacyOrder(id: string, opts?: object) { ... }
```

> [!warning] Pregunta trampa en código — usar Glob para este paso
> La forma ingenua sería `Glob: "**/*processLegacyOrder*"`, razonando que "así encuentro los archivos relacionados con esa función". **Por qué es mala idea** (Trampa de examen 1 del resumen): Glob compara *rutas* de archivo por patrón de nombre, no contenido — como ningún archivo se llama literalmente `processLegacyOrder.ts`, esta búsqueda devuelve una lista vacía o irrelevante. La función vive *dentro* de `orders.ts`, `OrderProcessor.ts` y `RefundHandler.ts`, ninguno de los cuales tiene ese string en su nombre de archivo. Grep es la única herramienta que puede ver contenido.

## Paso 2 — Read para trazar el flujo, solo en los archivos relevantes

Aplicamos la [[1 resumen#3. Comprensión incremental del código base nunca leer todo de entrada|Conclusión 3]]: leemos únicamente los tres archivos que Grep identificó como relevantes, no el código base completo.

```
Read: src/legacy/orders.ts
```

```typescript
// src/legacy/orders.ts
export function processLegacyOrder(id: string, opts?: { partial?: boolean }) {
  // ... lógica legacy ...
}
```

```
Read: src/OrderProcessor.ts
```

```typescript
// src/OrderProcessor.ts
import { processLegacyOrder } from './legacy/orders';

export async function applyLegacyOrder(orderId: string) {
  return processLegacyOrder(orderId);
}
```

`OrderProcessor.ts` revela algo que el Grep del Paso 1 no podía ver por sí solo: expone `processLegacyOrder` bajo un nombre de wrapper, `applyLegacyOrder`. Esto activa la [[1 resumen#4. Trazar el uso de una función a través de módulos wrapper y archivos barril|Conclusión 4]].

> [!warning] Pregunta trampa en código — leer todo `src/` "ya que estamos"
> La forma ingenua sería `Read: src/**/*.ts` completo, razonando que "mejor tener todo el contexto de una vez". **Por qué es mala idea** (Trampa de examen 3 del resumen): en un código base real de cientos de archivos, esto consume el presupuesto de contexto en archivos que no tienen nada que ver con `processLegacyOrder` — React components, configuración, utilidades no relacionadas. Cada Read debe estar justificado por lo que el Grep anterior encontró, no por "por si acaso".

## Paso 3 — Grep otra vez, por el nombre del wrapper

Aplicamos la [[1 resumen#4. Trazar el uso de una función a través de módulos wrapper y archivos barril|Conclusión 4]]: un Grep de `processLegacyOrder` nunca encuentra a quienes llaman por el nombre del wrapper, porque ese string nunca aparece en sus archivos.

```
Grep: "applyLegacyOrder"

-> src/OrderProcessor.ts:5:    export async function applyLegacyOrder(orderId: string) {
-> src/checkout/CheckoutFlow.ts:31:  await applyLegacyOrder(order.id)
```

`CheckoutFlow.ts` es un consumidor indirecto que el Grep del Paso 1 nunca habría encontrado — exactamente el caso que la Conclusión 4 describe.

## Paso 4 — Glob para los archivos de test hermanos

Con los archivos llamadores ya identificados (`OrderProcessor.ts`, `RefundHandler.ts`, `CheckoutFlow.ts`), aplicamos la [[1 resumen#5. El escenario de deprecación el patrón Grep → Glob → Grep otra vez|Conclusión 5]]: Glob para encontrar sus tests hermanos por convención de nombre, no por contenido.

```
Glob: "**/OrderProcessor.test.*"
Glob: "**/RefundHandler.test.*"
Glob: "**/CheckoutFlow.test.*"

-> src/OrderProcessor.test.tsx
-> src/RefundHandler.test.tsx
-> src/checkout/CheckoutFlow.test.tsx
```

Esto encuentra `CheckoutFlow.test.tsx` aunque ese test nunca mencione `processLegacyOrder` ni `applyLegacyOrder` por nombre — solo ejercita el módulo que los envuelve. Ese es precisamente el punto de usar Glob aquí: coincidencia de ruta, no de contenido.

> [!warning] Pregunta trampa en código — usar Grep en vez de Glob para "encontrar los tests"
> La forma ingenua sería `Grep: "test"` sobre el directorio, razonando que "así encuentro cualquier archivo de test que mencione algo relacionado". **Por qué es mala idea** (Trampa de examen 2 del resumen): esto depende de que la palabra "test" aparezca como *contenido* en el archivo — funciona por accidente en algunos casos, pero es la herramienta equivocada para el trabajo, y se pierde archivos como `CheckoutFlow.test.tsx` si ese archivo no contiene el string `processLegacyOrder` en ningún lado (que es justo el caso aquí). Glob, por convención de nombre, los encuentra a los tres sin depender de qué haya escrito adentro.

## Paso 5 — Edit para reemplazar cada llamada, con ancla única

Aplicamos la [[1 resumen#2. Read, Write y Edit cada una para un caso de uso distinto|Conclusión 2]]: Edit es la herramienta por defecto para modificar, porque toca solo el texto exacto señalado.

```
Edit: src/OrderProcessor.ts
  old_string: "import { processLegacyOrder } from './legacy/orders';"
  new_string: "import { processOrder } from './orders';"

-> 1 reemplazo hecho (ancla única, funciona directo)
```

## Paso 6 — Cuando Edit falla por coincidencia no única

En `src/legacy/orders.ts`, `processLegacyOrder` aparece tres veces: la declaración de la función, un comentario que la menciona, y una llamada interna recursiva.

```
Edit: src/legacy/orders.ts
  old_string: "processLegacyOrder"
  new_string: "processOrder"

-> Error: old_string matches 3 locations
```

Aquí las dos respuestas del resumen divergen — la [[1 resumen#Qué hacer cuando Edit falla por coincidencia no única|nota sobre examen vs. Claude Code actual]] aplica directamente:

```
# Respuesta de trabajo real: ampliar el ancla hasta que sea única
Edit: src/legacy/orders.ts
  old_string: "export function processLegacyOrder(id: string"
  new_string: "export function processOrder(id: string"

-> 1 reemplazo hecho (ancla ampliada, sigue siendo Edit)
```

```
# Respuesta que espera el examen: Read + Write como fallback documentado
Read: src/legacy/orders.ts
# ... se construye el contenido completo del archivo con las tres ocurrencias renombradas ...
Write: src/legacy/orders.ts  (contenido completo, con processLegacyOrder -> processOrder en las 3 ocurrencias)
```

> [!warning] Pregunta trampa en código — saltar directo a Read + Write cuando Edit falla la primera vez
> La forma ingenua sería, ante `old_string matches 3 locations`, ir directo a `Read` + `Write` del archivo completo "porque es lo seguro". **Por qué es mala idea en trabajo real** (Trampa de examen 5 del resumen): eso gasta el contenido completo del archivo en tokens de contexto para lo que en este caso son tres reemplazos de una palabra — ampliar el ancla (como en el ejemplo de arriba) resuelve lo mismo sin salir de Edit y casi sin costo extra. **Pero en el examen**, si la pregunta pide la respuesta documentada del exam guide ante un fallo de Edit por no-unicidad, la respuesta correcta es Read + Write — no "ampliar el ancla", aunque esa sea la práctica real más barata.

## Paso 7 — Validar la cobertura completa con una función auxiliar

Como cierre, una pequeña utilidad en Python que resume el criterio de selección de herramienta aplicado en cada paso anterior — útil para verificar en retrospectiva que no se saltó ningún paso del patrón Grep → Glob → Grep:

```python
def select_tool(task: str, target: str) -> str:
    """
    Aplica la Conclusión 1 del resumen: decide entre Grep y Glob
    según si la tarea busca CONTENIDO o RUTAS por nombre.
    """
    content_tasks = {"find_callers", "find_error_message", "find_import", "trace_wrapper_usage"}
    path_tasks = {"find_test_files", "find_by_extension", "find_by_naming_convention"}

    if task in content_tasks:
        return f"Grep: \"{target}\""
    if task in path_tasks:
        return f"Glob: \"{target}\""
    raise ValueError(f"Tarea de búsqueda no reconocida: {task}")


# Los 3 Grep + 1 Glob del escenario de deprecación, en orden:
print(select_tool("find_callers", "processLegacyOrder"))        # -> Grep: "processLegacyOrder"
print(select_tool("trace_wrapper_usage", "applyLegacyOrder"))   # -> Grep: "applyLegacyOrder"
print(select_tool("find_test_files", "**/OrderProcessor.test.*"))  # -> Glob: "**/OrderProcessor.test.*"
```

> [!warning] Pregunta trampa en código — un `select_tool` que use Glob para `"find_callers"`
> La forma ingenua sería meter `"find_callers"` en `path_tasks` en vez de `content_tasks`, razonando que "buscar quién llama a una función es buscar dónde está esa función". **Por qué es mala idea:** confunde "dónde vive el archivo" con "qué contiene el archivo" — exactamente la Trampa de examen 1. Encontrar llamadores es siempre una búsqueda de contenido (Grep), nunca de ruta (Glob), sin importar qué tan relacionado suene el nombre de la tarea.

Con esto, el escenario completo respeta las cinco conclusiones del resumen: Grep para encontrar y trazar contenido (incluyendo el wrapper), Read solo sobre los archivos que el Grep justificó, Glob para los tests por convención de nombre, Edit como modificación por defecto con ancla ampliada en trabajo real (y Read + Write como la respuesta documentada del examen), y el patrón completo Grep → Glob → Grep del escenario de deprecación.
