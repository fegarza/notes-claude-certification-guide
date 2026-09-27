Aplicación práctica de [[1 resumen]]

## Caso de uso

Un equipo tiene un monorepo con componentes React (`src/components/**`), handlers de API (`src/api/**`), configuración de infraestructura (`terraform/**`) y archivos de test dispersos junto a cada módulo (`Button.test.tsx` al lado de `Button.tsx`, `users.controller.test.ts` al lado de `users.controller.ts`, etc.). Queremos que Claude Code aplique automáticamente las convenciones correctas según el tipo de archivo que se edite, sin desperdiciar tokens cargando reglas irrelevantes.

Como este tema es específico de la configuración de Claude Code (no de la API de Anthropic), el "código" son los propios archivos de configuración: YAML frontmatter + Markdown dentro de `.claude/rules/`, y comandos de Claude Code para verificar el comportamiento.

### Paso 1 — Diagnosticar por qué el CLAUDE.md raíz no alcanza

Supongamos que hoy todas las convenciones viven en un único `CLAUDE.md` en la raíz:

```markdown
# CLAUDE.md (raíz) — enfoque ingenuo

## Convenciones de testing
- Use describe/it blocks...

## Convenciones de API
- All endpoints return { data, error, metadata }...

## Convenciones de Terraform
- State files must reference remote backends...
```

> [!warning] Pregunta trampa en código — ¿por qué NO consolidar todo aquí?
> Esta forma es "ingenua" porque el `CLAUDE.md` raíz se carga en **cada sesión, sin importar qué archivo se esté editando** (ver [[1 resumen#Trampas de examen]]). Si estás editando un componente React, Claude sigue cargando las convenciones de Terraform y de API aunque no apliquen — puro desperdicio de presupuesto de tokens. Además, obliga a Claude a "inferir" qué sección aplica según el archivo, en vez de una activación explícita y determinística.

### Paso 2 — Separar en rule files por tipo de archivo

En vez de un único archivo, creamos `.claude/rules/testing.md`, `.claude/rules/api-conventions.md` y `.claude/rules/terraform.md`. Cada uno usa `paths` en el frontmatter para declarar a qué archivos aplica:

```yaml
---
paths: ["**/*.test.ts", "**/*.test.tsx", "**/*.spec.ts", "**/*.spec.tsx"]
---
# Test Conventions

- Use describe/it blocks with descriptive names that read as sentences
- Each test file must have at least one happy path and one error case
- Use factory functions for test data, not inline object literals
- Mock external services at the module boundary, not individual functions
- Assert behaviour, not implementation details
```

```yaml
---
paths: ["src/api/**/*", "**/routes/**/*", "**/*.controller.ts"]
---
# API Conventions

- All endpoints return { data, error, metadata } response shape
- Use Zod schemas for request validation at the handler boundary
- Log request ID on every error response
- Rate limiting configuration must be explicit, not inherited from defaults
```

```yaml
---
paths: ["terraform/**/*", "**/*.tf", "infrastructure/**/*"]
---
# Infrastructure Conventions

- State files must reference remote backends, never local
- Use workspaces for environment separation
- Every module must be versioned with a CHANGELOG
```

Con esta estructura, cada archivo se activa **solo** cuando Claude edita un path que matchea su patrón `paths` ([[1 resumen#Cómo funcionan los rule files]]).

### Paso 3 — Resolver el caso de los tests dispersos en 50+ directorios

Este es el escenario exacto que el examen pone a prueba (ver [[1 resumen#Por qué no alcanza con directory-level CLAUDE.md]]): `Button.test.tsx` vive junto a `Button.tsx` en `src/components/Button/`, y lo mismo pasa en decenas de otras carpetas (`src/hooks/useAuth/useAuth.test.ts`, `src/api/users/users.controller.test.ts`, etc.).

```
src/components/Button/Button.test.tsx
src/hooks/useAuth/useAuth.test.ts
src/api/users/users.controller.test.ts
... (50+ carpetas más, cada una con su propio archivo de test)
```

> [!warning] Pregunta trampa en código — ¿por qué NO poner un CLAUDE.md en cada carpeta?
> La forma ingenua sería crear un `CLAUDE.md` con las reglas de testing dentro de `src/components/Button/`, otro idéntico en `src/hooks/useAuth/`, otro en `src/api/users/`... uno por cada carpeta que tenga tests. Esto es inviable: 50+ copias del mismo contenido, cada carpeta nueva con tests necesita su propia copia, y cualquier cambio de convención implica editar 50+ archivos a mano (con el riesgo de que algunas copias queden desactualizadas — *drift*).

La solución correcta ya la tenemos: el **único** `.claude/rules/testing.md` del Paso 2, con `paths: ["**/*.test.ts", "**/*.test.tsx", ...]`, cubre las 50+ carpetas sin duplicar nada — "un archivo, un patrón, cobertura universal".

### Paso 4 — Verificar la carga condicional con `/context`

Claude Code expone el comando `/context` para inspeccionar qué se cargó en la sesión actual. Lo usamos para confirmar que la carga es realmente condicional:

```bash
# Editando src/components/Button/Button.test.tsx
claude
> /context
# Salida esperada: .claude/rules/testing.md aparece cargado.
# .claude/rules/api-conventions.md y .claude/rules/terraform.md NO aparecen.
```

```bash
# Editando src/api/users/users.controller.ts
claude
> /context
# Salida esperada: .claude/rules/api-conventions.md aparece cargado.
# .claude/rules/testing.md y .claude/rules/terraform.md NO aparecen.
```

El conteo de tokens de configuración cargada es medible y menor comparado con tener las tres secciones consolidadas en el `CLAUDE.md` raíz del Paso 1 — es la evidencia concreta de la ventaja de eficiencia descrita en [[1 resumen#Ventaja de eficiencia de tokens]].

### Paso 5 — No confundir esto con un Skill

Para cerrar el ejemplo, comparamos con la alternativa incorrecta de la Trampa 3 ([[1 resumen#Trampas de examen]]): un skill invocado a demanda.

```yaml
---
name: apply-testing-conventions
description: Aplica las convenciones de testing del equipo a un archivo de test.
---
# Instrucciones para aplicar convenciones de testing
...
```

> [!warning] Pregunta trampa en código — ¿por qué NO usar un Skill aquí?
> Un skill como este solo se carga cuando el modelo decide invocarlo por intención, o cuando el usuario lo invoca explícitamente (ej. `/apply-testing-conventions`). No hay garantía de que se active automáticamente cada vez que se edita un `.test.tsx`. Para el requisito de este caso de uso — "aplicar automáticamente y siempre, según el path del archivo, sin invocación manual" — el mecanismo correcto es el rule file de `.claude/rules/` con `paths` en el frontmatter, no un skill.

## Resultado

Con los tres rule files del Paso 2, el proyecto logra: convenciones de testing aplicadas automáticamente a cualquier `.test.ts`/`.test.tsx` sin importar su carpeta, convenciones de API activas solo al tocar `src/api/**`, convenciones de Terraform activas solo al tocar `terraform/**`, y ningún desperdicio de tokens cargando reglas irrelevantes al contexto actual — verificable en cualquier momento con `/context`.

---

> [!tip] Repasa el concepto en [[1 resumen]], memorízalo en [[3 cuestionario]] y evalúate en [[4 test]]
