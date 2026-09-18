## Explícamelo como si tuviera 5 años

Imagina que le pides a un amigo que te vigile la casa mientras estás de viaje y le dices: "avísame solo si pasa algo importante, usa tu buen juicio". Ese amigo no sabe si una luz encendida es "importante", si un vecino tocando la puerta es "importante", o si el correo acumulado es "importante". Cada persona entiende "importante" de forma distinta, así que el resultado va a ser inconsistente: unas veces te llama por tonterías, otras veces ignora algo grave.

Ahora imagina que en vez de eso le das una lista: "avísame si ves una ventana rota, si suena la alarma, o si hay agua saliendo de la puerta. No me avises por correo acumulado, plantas necesitando agua, o luces encendidas". Ahora tu amigo sabe exactamente qué reportar y qué ignorar, sin importar su "buen juicio" personal.

Con Claude pasa lo mismo: decirle "sé conservador" o "reporta solo hallazgos de alta confianza" es como el primer amigo — suena razonable, pero no le da ninguna frontera de decisión real. Darle **criterios categóricos explícitos** (qué marcar, qué ignorar, con ejemplos concretos) es como la segunda lista.

## Argumento central

> [!note] Idea central
> Las instrucciones vagas ("sé conservador", "solo reporta hallazgos de alta confianza", "usa tu buen juicio") **no** establecen límites de decisión accionables — suenan razonables, por eso son distractores típicos del examen. Los **criterios categóricos explícitos**, con ejemplos de código concretos para cada nivel de severidad, siempre superan a las instrucciones vagas.

## Conclusiones clave

### 1. Vago vs. explícito: la diferencia real

- **Enfoque incorrecto** (sin criterios aplicables):
  ```
  Review this code. Be conservative. Only report high-confidence findings.
  ```
  Palabras como "conservador" o "alta confianza" no tienen un significado consistente entre contextos distintos.

- **Enfoque correcto** (categorías concretas y accionables):
  ```
  Flag comments only when claimed behaviour contradicts actual code behaviour.
  Report bugs and security vulnerabilities.
  Skip minor style preferences and local patterns.
  ```
  Aquí hay tres cosas concretas: qué reportar (bugs, seguridad), qué excluir (estilo, patrones locales) y un disparador explícito (contradicción entre comportamiento declarado y comportamiento real).

**Analogía:** es la diferencia entre un letrero que dice "maneja con cuidado" y uno que dice "límite 40 km/h en zona escolar de 7am a 4pm". El segundo es verificable; el primero es interpretación libre.

### 2. Los falsos positivos no se quedan en su categoría — contaminan la confianza total

> [!warning] El problema de la confianza no se compartimenta
> Si una categoría de revisión (ej. "documentation mismatch") tiene 40% de falsos positivos, los desarrolladores empiezan a ignorar **todas** las categorías del reporte — incluidas las de seguridad con 98% de precisión. La confianza se distribuye globalmente, no por categoría.

**Solución (contraintuitiva):** no es "seguir puliendo la categoría problemática mientras sigue activa" — eso sigue erosionando la confianza en todo el sistema mientras iteras. La jugada correcta es:

1. **Deshabilitar temporalmente** la categoría con alto falso positivo.
2. La confianza en las categorías que sí funcionan se recupera de inmediato.
3. Iterar esa categoría problemática usando ejemplos de código concretos.
4. Reactivarla solo cuando mejore la precisión.

Así preservas la categoría a largo plazo sin sacrificar la confianza del sistema completo mientras tanto.

### 3. Calibrar severidad con código, no con prosa

- **Descripción en prosa (insuficiente):**
  ```
  Critical: Issues that could cause system failures or data loss
  Minor: Issues that affect code readability but not functionality
  ```
  Esto obliga al modelo a *interpretar* qué significa "podría causar fallas del sistema" — ambiguo y variable entre invocaciones.

- **Ejemplos de código (correcto):**
  ```
  Critical — Unsanitised user input in SQL query:
    query = f"SELECT * FROM users WHERE id = {user_input}"

  Minor — Inconsistent variable naming:
    userName vs user_name in the same module
  ```
  Un patrón de código real clasificado en cada nivel de severidad elimina la ambigüedad y produce clasificaciones consistentes entre corridas.

### 4. Por qué el filtrado por confianza no resuelve el problema

- El modelo auto-reportando "alta confianza" **no** es un sustituto válido de criterios explícitos: **la confianza auto-reportada de un LLM está mal calibrada** — puede estar muy seguro de un hallazgo incorrecto y dudar de uno correcto.
- La confianza sí tiene un uso legítimo, pero como **mecanismo de enrutamiento** (ej. mandar hallazgos de baja confianza a revisión humana — tema de [[6 Multi-Instance and Multi-Pass Review/1 resumen|Multi-Instance and Multi-Pass Review]]), nunca como filtro primario de validez.
- **Secuencia correcta:** primero criterios explícitos, después (opcionalmente) enrutamiento por confianza. Nunca al revés, y nunca saltarse el primer paso.

## Trampas de examen

> [!warning] Trampa 1 — "Sé conservador" / "solo alta confianza" como mejora válida
> El examen presenta frecuentemente estas frases como opción tentadora porque *suenan* a buena ingeniería. Son incorrectas: no le dan al modelo ninguna interpretación accionable de qué es "conservador". La respuesta correcta siempre es definir criterios categóricos específicos de qué marcar y qué omitir.

> [!warning] Trampa 2 — Asumir que un umbral de confianza resuelve los falsos positivos
> La confianza auto-reportada por el LLM está mal calibrada, así que un umbral de confianza no arregla el problema de raíz. Los criterios explícitos con ejemplos de código superan al filtrado por confianza; el enrutamiento por confianza solo es útil *después* de tener criterios establecidos.

> [!warning] Trampa 3 — Mantener todas las categorías activas mientras se arregla la problemática
> Un alto índice de falsos positivos en una sola categoría destruye la confianza en **todas** las categorías, no solo en la afectada. La respuesta correcta es deshabilitar temporalmente esa categoría mientras se refina su prompt, para recuperar de inmediato la confianza en el resto del sistema. (Este es el escenario del pipeline de CI/CD con 40% de falsos positivos en "documentation mismatch" que aparece como pregunta de práctica en la guía.)

## En una frase

> Los criterios categóricos explícitos con ejemplos de código concretos —nunca las instrucciones vagas ni la confianza auto-reportada— son lo que hace que un system prompt de revisión sea preciso y consistente.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
