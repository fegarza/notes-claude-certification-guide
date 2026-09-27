## Explícamelo como si tuviera 5 años

Imagina que escribiste una redacción y te la revisas tú mismo. Es difícil encontrarle errores: tú sabes por qué escribiste cada frase así, y tu cerebro tiende a decirte "sí, eso tiene sentido" — porque ya conoces la razón detrás de cada decisión. Por eso en la escuela es tan valioso que **otra persona** revise tu redacción: alguien que no sabe nada de por qué escribiste cada cosa la lee con ojos nuevos y sí nota lo que a ti se te escapó.

Ahora imagina que en vez de una redacción de una página, te piden revisar 14 capítulos de un libro, todos a la vez, en una sola sentada. Te vas a cansar: los primeros capítulos los revisas con cuidado, a la mitad empiezas a distraerte, y puede que dos capítulos distintos tengan el mismo error pero tú lo marques como grave en uno y ni lo notes en el otro.

Ese es el tema completo: revisar código (o cualquier output) con Claude tiene las mismas dos trampas — revisarte a ti mismo en la misma sesión, y revisar demasiado de golpe en un solo pase. La solución no es "pedirle que revise con más cuidado", es **rediseñar la arquitectura**: usar una instancia independiente sin el contexto de razonamiento original, y dividir revisiones grandes en pasadas más pequeñas y enfocadas.

## Argumento central

> [!note] Idea central
> El self-review dentro de la misma sesión y el single-pass review sobre múltiples archivos son **limitaciones estructurales**, no problemas de instrucción — no se arreglan pidiéndole al modelo que "revise con más cuidado" ni dándole un modelo más grande con más contexto. Se arreglan con **rediseño arquitectónico**: instancias independientes sin el razonamiento previo, revisión en múltiples pasadas (por archivo + integración), y ruteo de hallazgos por confianza **calibrada**, no por confianza cruda.

## Conclusiones clave

### 1. La limitación del self-review en la misma sesión

Cuando Claude revisa su propia salida dentro de la misma conversación en la que la generó, arrastra consigo el razonamiento que lo llevó a tomar cada decisión.

- Ya "sabe" por qué eligió cada clasificación, cada valor, cada línea de código.
- Al pedirle que revise, tiende a **confirmar** en vez de **cuestionar** críticamente sus propias elecciones.
- No es una falla técnica puntual — es una limitación inherente de revisar dentro del mismo contexto que generó la salida.

Una **instancia independiente** — una invocación completamente separada, sin acceso al razonamiento previo — encuentra la salida con ojos frescos: la evalúa solo por lo que observa, sin el sesgo de anclaje de "elegí este enfoque porque...". Esto la hace sustancialmente más efectiva para detectar defectos sutiles.

> [!note] Lo que evalúa el examen aquí
> Cuando se presentan opciones para mejorar la calidad de una revisión, la respuesta correcta casi siempre es desplegar una **instancia de modelo separada** — nunca "mejorar las instrucciones dentro de la misma sesión" ni "usar extended thinking durante la generación". Ninguna de esas dos alternativas elimina el sesgo de razonamiento retenido.

### 2. Arquitectura de multi-pass review

Para revisiones a gran escala — pull requests de muchos archivos, pipelines de extracción de datos complejos, auditorías de seguridad amplias — procesar todo en un solo pase produce **attention dilution** (dilución de atención). Se manifiesta en tres síntomas observables:

1. **Profundidad desigual entre archivos** — algunos reciben análisis detallado, otros trato superficial.
2. **Bugs dependientes de la posición** — defectos obvios se pasan por alto a la mitad de la revisión (fatiga de atención).
3. **Contradicciones internas** — el mismo patrón problemático se marca como crítico en un archivo y se pasa por alto o se aprueba en otro.

La solución divide la revisión en dos pasadas secuenciales:

- **Pass 1 — Análisis local por archivo**: cada archivo se revisa de forma independiente, en su propia invocación, con un prompt enfocado que examina *solo ese archivo* buscando bugs, problemas de seguridad y errores de lógica. Esto garantiza profundidad uniforme, porque cada archivo recibe la atención completa e indivisa de una invocación del modelo. Como son invocaciones independientes entre sí, se pueden paralelizar.
- **Pass 2 — Integración cross-file**: después de que terminan todos los análisis por archivo, una invocación separada recibe todos los hallazgos y revisa específicamente problemas *entre* archivos: datos que fluyen entre módulos en formatos incompatibles, patrones que se contradicen entre archivos, violaciones de contrato de API en los límites entre servicios, e inconsistencias en los propios hallazgos por archivo.

> [!note] Analogía
> Es como revisar los 14 capítulos del libro uno por uno, con la mente descansada en cada capítulo (Pass 1), y luego — solo al final — leer un resumen de todas las notas para ver si los personajes se contradicen entre capítulos (Pass 2). Ninguna de las dos pasadas sola resuelve ambos problemas: la primera no detecta inconsistencias globales, la segunda sola tendría el mismo problema de dilución si intentara ver los 14 capítulos completos de una vez.

Esta arquitectura resuelve cada síntoma de attention dilution directamente: las pasadas por archivo garantizan profundidad consistente, la pasada de integración detecta problemas sistémicos que ninguna revisión de un solo archivo encontraría, y la separación evita que aparezcan hallazgos contradictorios en la misma salida.

### 3. Por qué una ventana de contexto más grande NO resuelve esto

> [!warning] El distractor más común del examen
> "Cambiar a un modelo de nivel superior con ventana de contexto más grande" suena lógico: si el modelo tiene dificultades con 14 archivos a la vez, ¿por qué no darle más capacidad? Es un distractor plausible, no una solución real.

El problema no es capacidad de almacenamiento, es **calidad de atención**. Una ventana de contexto más grande permite que el modelo *contenga* más texto, pero no evita que la atención se distribuya de forma desigual sobre ese texto — el modelo puede seguir repartiendo su esfuerzo cognitivo de forma dispareja entre archivos, sin importar cuánto contexto tenga disponible. Solo pasadas estructuradas y enfocadas por archivo garantizan profundidad de atención consistente.

### 4. Ruteo por confianza calibrada

Para hallazgos donde existe incertidumbre, el sistema puede pedirle al modelo que reporte un **puntaje de confianza** junto a cada hallazgo. Esto habilita una estrategia de ruteo:

- **Hallazgos de alta confianza** → se envían directo a los desarrolladores, sin pasar por revisión humana.
- **Hallazgos de baja confianza** → se envían a una cola de revisión humana para validación.
- **Calibración del umbral** → se usa un dataset de validación etiquetado (donde ya se conoce la respuesta correcta) para medir qué tan bien la confianza reportada se correlaciona con la precisión real.

> [!warning] Confianza cruda ≠ confianza calibrada
> El puntaje de confianza que el propio modelo reporta es solo su sensación de certeza — **no está validado contra precisión real**. Usar ese número crudo directamente para decisiones automatizadas de ruteo es poco confiable. La calibración exige correr un dataset de ejemplos etiquetados a través del sistema y medir la correlación real entre confianza reportada y resultado verificado. Solo después de esa calibración es seguro usar umbrales para automatizar decisiones.

### 5. Arquitectura de producción completa

Sintetizando todo en un sistema de extremo a extremo:

1. **Generación** — una primera instancia crea el código, la extracción o el análisis.
2. **Revisión por archivo** — instancias independientes revisan cada unidad de salida individualmente, con prompts enfocados.
3. **Revisión de integración** — una instancia separada recibe todos los hallazgos por archivo y revisa consistencia cross-file, flujo de datos y contradicciones.
4. **Ruteo por confianza** — hallazgos bajo el umbral de confianza van a colas humanas; los de alta confianza van directo a desarrollo.
5. **Loop de calibración** — datasets de validación etiquetados miden continuamente qué tan bien la confianza reportada rastrea la precisión real, ajustando umbrales.

> [!note] El costo es real, y se acepta a propósito
> Esta arquitectura cuesta más que una revisión de un solo pase. El trade-off se justifica en sistemas de producción donde la calidad de la revisión impacta directamente la confiabilidad: pipelines de CI/CD, extracción de datos financieros, análisis de compliance, y cualquier sistema donde un problema no detectado se propaga a procesos posteriores.

## Trampas de examen

> [!warning] Trampa 1 — Confiar en self-review dentro de la misma sesión
> El modelo retiene el razonamiento de la generación y tiende a confirmar en vez de cuestionar sus propias decisiones. Pedirle "revisa con más cuidado" o "sé más crítico" en la misma sesión no elimina ese sesgo estructural — solo una instancia independiente, sin ese contexto de razonamiento previo, lo hace.

> [!warning] Trampa 2 — Un solo pase para revisiones grandes multi-archivo
> Un single-pass sobre muchos archivos produce tres fallas sistemáticas, no errores aleatorios: profundidad desigual, bugs perdidos (sobre todo a la mitad), y hallazgos contradictorios sobre el mismo patrón en archivos distintos. La solución es arquitectónica (dividir en pasadas), no "usar mejores prompts" ni "un modelo más grande".

> [!warning] Trampa 3 — Ventana de contexto más grande como solución a attention dilution
> Esta es la Pregunta 12 de muestra oficial de la guía (ver `4 test.md`). El problema no es cuánto texto cabe, es cómo se distribuye la atención sobre ese texto — un modelo de nivel superior con más contexto sigue repartiendo su atención de forma dispareja entre 14 archivos. La solución correcta es la separación estructural en pasadas por archivo, no una mejora de capacidad del modelo.

> [!warning] Trampa 4 — Usar confianza cruda (sin calibrar) para ruteo automatizado
> El examen distingue explícitamente entre confianza cruda (el número que el modelo reporta por sí solo, no confiable para automatizar) y umbrales calibrados (validados contra un dataset etiquetado, sí aptos para ruteo). Implementar scoring de confianza sin el paso de calibración es la trampa.

## En una frase

> Revisar la propia salida en la misma sesión y revisar demasiados archivos en un solo pase son limitaciones estructurales que no se arreglan con mejores instrucciones ni con más contexto — se arreglan con instancias independientes, revisión en dos pasadas (por archivo + integración), y ruteo de hallazgos por confianza calibrada contra datos etiquetados, nunca por confianza cruda.

---

> [!tip] Repasa esto en [[3 cuestionario]], aplícalo en [[2 example]] y evalúate en [[4 test]]
