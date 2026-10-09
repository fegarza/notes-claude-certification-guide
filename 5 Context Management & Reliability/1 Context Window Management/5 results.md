# Resultados — Tutor socrático: Context Window Management

> [!info] Sesión evaluada el 2026-10-01
> Repaso basado en [[1 resumen]]. Alcance limitado a lo evaluable del Task Statement 5.1 (se excluyó la mecánica de `prompt caching`, que la guía deja fuera del examen).

## Resumen general del desempeño

**Nivel de entendimiento: 85/100**

De 12 preguntas, 9 fueron correctas y 3 parcialmente correctas, sin respuestas incorrectas. Dominas los patrones centrales del tema (persistent case facts block, fix estructural del "lost in the middle", recorte antes de entrar al historial, API stateless y resumir vs. truncar). Lo que falta es precisión en los detalles concretos que la guía enumera: la unidad de separación en sesiones multi-issue, el porqué del costo acumulado de los tool results y los tipos exactos de metadata. Ahí es donde los distractores del examen suelen atrapar.

## Conceptos con buen dominio

- **Progressive summarisation trap**: identificaste que destruye números, fechas y porcentajes. Solo faltó mencionar las expectativas que el cliente expresó explícitamente.
- **Persistent case facts block**: explicaste bien que vive fuera del historial resumido, que va en cada prompt y que nunca se resume.
- **"Lost in the middle"**: lo definiste correctamente (el inicio y el final se procesan de forma confiable; lo de en medio se omite).
- **Fix estructural del "lost in the middle"**: descartaste el recordatorio en el prompt y propusiste un resumen de hallazgos clave al inicio más encabezados de sección explícitos.
- **Momento del tool result trimming**: el recorte va antes de que el resultado entre al historial, en un hook `PostToolUse`.
- **API stateless**: cada request debe llevar el historial completo para mantener coherencia.
- **Resumir vs. truncar**: elegiste resumir con el case facts block y entendiste que truncar rompe la coherencia.
- **Upstream agent optimisation**: el cambio se hace en los subagentes upstream, que deben devolver datos estructurados.
- **Beneficio de las salidas estructuradas más allá de los tokens**: el agente downstream no tiene que re-parsear prosa.

## Áreas que necesitan revisión

### Sesiones multi-issue en el case facts block

- **Qué pasó:** propusiste separar la capa "por cada hecho" en lugar de por cada issue, y no nombraste el riesgo concreto de que se mezclen los datos entre issues.
- **Contenido del resumen:**
  > Para sesiones con **múltiples issues** (un cliente que plantea varios problemas en una sola conversación), cada issue se extrae y persiste como una entrada separada en esta capa (order ID, monto, estado propios) — esto evita que la summarisation mezcle los datos de un issue con los de otro.
- **Sección:** [[1 resumen]] → "1. La trampa de la progressive summarisation"
- **Analogía para reforzarlo:** en un taller mecánico, cada coche tiene su propia orden de trabajo con su placa, su presupuesto y su estado. Nadie hace una orden por cada tornillo, ni una sola hoja para tres coches, porque terminarías cobrándole a un cliente la reparación del coche de otro.

### Por qué el costo de los tool results verbosos se multiplica

- **Qué pasó:** reconociste que los campos irrelevantes llenan el contexto, pero no explicaste que siguen consumiendo tokens **en cada turno subsecuente**, porque se quedan en el historial que se reenvía completo.
- **Contenido del resumen:**
  > - Una consulta de orden puede devolver 40+ campos (timestamps de auditoría interna, códigos de almacén, IDs de transportista) cuando el agente solo necesita 5.
  > - Esos 35 campos irrelevantes consumen tokens en **cada turno subsecuente**, porque quedan en el historial de la conversación conforme esta crece.
- **Sección:** [[1 resumen]] → "3. Tool result trimming"
- **Analogía para reforzarlo:** es como meter a la mochila una piedra "por si acaso" al inicio de una caminata. No pesa una sola vez: la cargas en cada paso que das después, y si cada parada agregas otra piedra, al final del camino ya no puedes moverte.

### Tipos concretos de metadata en las salidas de subagentes

- **Qué pasó:** identificaste que faltaba "metadata", pero no nombraste los tres tipos que exige la guía (fechas, ubicación de la fuente, contexto metodológico), que son justo los que resuelven el conflicto entre las cifras de 34% y 61%.
- **Contenido del resumen:**
  > - Se debe exigir a los subagentes que incluyan **metadata** (fechas, ubicación de la fuente, contexto metodológico) dentro de esas salidas estructuradas, para que la síntesis downstream sea precisa.
- **Sección:** [[1 resumen]] → "5. Upstream agent optimisation en sistemas multi-agente"
- **Analogía para reforzarlo:** dos termómetros marcan 15 °C y 30 °C. Sin saber **cuándo** se midió (¿de noche o al mediodía?), **dónde** (¿a la sombra o al sol?) ni **cómo** (¿en grados Celsius o en Fahrenheit?), no puedes saber si se contradicen o si los dos tienen razón.

## Siguiente paso recomendado

Antes de [[4 test]], vuelve a leer las Conclusiones 1, 3 y 5 de [[1 resumen]] fijándote en las listas concretas: las 4 categorías que destruye la summarisation, el "cada turno subsecuente" y los 3 tipos de metadata. Con eso, ya estás listo para el test de práctica.
