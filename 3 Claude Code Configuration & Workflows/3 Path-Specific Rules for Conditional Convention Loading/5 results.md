# Resultados — Tutor socrático: Path-Specific Rules for Conditional Convention Loading

> [!info] Sesión evaluada el 2026-09-27
> Repaso basado en [[1 resumen]].

## Resumen general del desempeño

**Nivel de entendimiento: 86/100**

De 9 preguntas, 7 fueron correctas y 2 parcialmente correctas. No hubo ninguna incorrecta. Dominas bien la mecánica de los rule files (`.claude/rules/`, frontmatter `paths`, carga automática), la tabla de decisión entre mecanismos y las 3 trampas de examen. Las respuestas parciales tuvieron el mismo patrón: la idea correcta, pero sin la consecuencia concreta que la guía espera, como el costo en tokens, la duplicación con drift o lo que se debería observar en `/context`. En el examen, esa precisión es justo lo que distingue la opción correcta de un distractor plausible.

## Conceptos con buen dominio

- **Ubicación y estructura de un rule file:** sabes que va en `.claude/rules/<nombre>.md`, con YAML frontmatter y un campo `paths` que contiene glob patterns.
- **Carga automática:** la regla se activa sola cuando Claude edita o lee un archivo que coincide con el patrón, sin que nadie la invoque.
- **Coincidencia con varios patrones:** con que el archivo coincida con **alguno** de los globs de `paths` basta. Lo aplicaste bien al caso de `auth.controller.ts` fuera de `src/api/`.
- **CLAUDE.md de directorio frente a rules:** elegiste bien el `CLAUDE.md` de directorio para convenciones acotadas a un solo paquete.
- **CLAUDE.md raíz para estándares universales:** descartaste con razón un rule con `**/*` para una convención que aplica siempre.
- **Trampa 3, Skills frente a Rules:** identificaste que los rules se disparan de forma determinística por el path, mientras que los skills se cargan on-demand y su carga no está garantizada.
- **Trampa 2, Terraform en el raíz:** reconociste el desperdicio de tokens en cada sesión y la carga condicional como solución.

## Áreas que necesitan revisión

### Consecuencias concretas de las alternativas (raíz frente a directorio)

- **Qué pasó:** en la primera pregunta identificaste bien que el raíz "aplica en archivos que no lo necesitan" y que los `CLAUDE.md` por directorio son "difíciles de mantener". Pero no nombraste el costo en **tokens en cada sesión** ni el mecanismo de fondo del problema de mantenimiento: la **duplicación en 50+ carpetas** y el **drift**.
- **Contenido del resumen:**
  > - El `CLAUDE.md` raíz se carga **en cada sesión, sin importar qué archivo edites** — si tienes convenciones de Terraform ahí, consumen tokens incluso cuando editas un componente de React.
  >
  > - El problema aparece cuando el tipo de archivo (ej. tests) está **disperso en 50+ directorios** junto al código que prueban (`Button.test.tsx` junto a `Button.tsx`): habría que poner una copia del mismo `CLAUDE.md` en cada una de esas 50+ carpetas.
  > - Consecuencias de esa duplicación: mantenimiento imposible, cada carpeta nueva con tests necesita su propia copia, cualquier cambio de convención obliga a actualizar 50+ archivos, y **drift inevitable** (copias que se desactualizan con el tiempo).
- **Sección:** [[1 resumen]] → "Ventaja de eficiencia de tokens" y "Por qué no alcanza con directory-level CLAUDE.md"
- **Analogía para reforzarlo:** es como avisar en un grupo de WhatsApp de 50 personas. Si mandas el aviso a todos cuando solo le importa a 3, les llenas el chat a los otros 47: ese es el costo del raíz. Si en cambio le copias el aviso a mano a cada uno por privado y luego cambias de opinión, tienes que corregir 50 chats, y seguro a alguno se te olvida: eso es el drift.

### Qué verificar con `/context`

- **Qué pasó:** acertaste en que `/context` es la herramienta para verificar, pero no dijiste qué resultado esperabas ver. La prueba de que la carga es condicional está en que las otras reglas **no aparezcan**.
- **Contenido del resumen:**
  > - Verificable con **`/context`**: al editar un `.test.ts`, solo aparece cargada la regla de testing (las de API/Terraform no aparecen); al editar un handler de API, aparece solo la regla de API.
- **Sección:** [[1 resumen]] → "Ventaja de eficiencia de tokens"
- **Analogía para reforzarlo:** es como revisar qué luces están prendidas en una casa con sensores de movimiento. No basta con ver que la luz de la cocina está prendida mientras estás ahí. La prueba de que los sensores funcionan es que las del garage y el cuarto estén **apagadas**.

## Siguiente paso recomendado

Ya estás listo para [[4 test]]. Al responder, entrénate para completar cada idea con su consecuencia concreta: tokens en cada sesión, duplicación con drift, y qué debería o no debería aparecer en `/context`. Los distractores del examen suelen fallar justo en esos detalles.
