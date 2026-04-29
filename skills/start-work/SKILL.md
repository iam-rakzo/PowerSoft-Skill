---
name: start-work
description: Use before starting any new development work — captures work intent, syncs git with develop, creates a properly typed branch, then runs brainstorming to produce the spec and plan. Invoke when user says "voy a empezar", "nueva tarea", "nueva feature", "nuevo fix", "empecemos a trabajar", "start work", "nueva rama", "quiero implementar".
---

# powersoft:start-work — Preparar rama de trabajo

## FASE 1 — Capturar prompt del trabajo

Preguntar al usuario:

> **¿Qué vas a hacer?** Describe el trabajo con el detalle que tengas ahora — puede ser una frase corta o varios párrafos. Esta descripción se usará para nombrar la rama y como contexto inicial para el brainstorming.

Guardar la respuesta como `<prompt-trabajo>`. No continuar hasta tener al menos una descripción mínima.

## FASE 2 — Git sync

Ejecutar en orden:

```bash
git fetch origin --prune
git switch develop
git pull --rebase
```

Si `git pull --rebase` produce conflictos de rebase:

> **PAUSADO:** Conflictos de rebase detectados. Resolver los conflictos antes de crear la rama.

Confirmar estado:

```bash
git log --oneline -3
```

## FASE 3 — Tipo de trabajo

Inferir el tipo a partir de `<prompt-trabajo>` usando estas heurísticas:
- Palabras como fix, bug, error, falla, broken, roto → `fix`
- Palabras como agregar, nueva, integrar, implementar, añadir, crear → `feat`
- Palabras como refactor, reorganizar, limpiar, mover → `refactor`
- Palabras como test, prueba, cobertura → `test`
- Palabras como docs, documentación, README → `docs`

Mostrar tabla numerada y marcar el tipo inferido como sugerencia:

| # | Tipo | Uso |
|---|------|-----|
| 1 | `feat` | Nueva funcionalidad |
| 2 | `fix` | Corrección de bug |
| 3 | `docs` | Solo documentación |
| 4 | `style` | Formato, espacios (sin cambios de lógica) |
| 5 | `refactor` | Refactorización sin nueva funcionalidad ni fix |
| 6 | `test` | Añadir o modificar tests |
| 7 | `chore` | Build, dependencias, configuración |
| 8 | `build` | Cambios que afectan dependencias del sistema |
| 9 | `perf` | Mejoras de rendimiento |
| 10 | `revert` | Revertir un commit anterior |
| 11 | `ci` | Cambios en pipelines de CI/CD |

Preguntar: **Tipo sugerido: `<tipo-inferido>` (#N) — ¿Confirmás con Enter o escribís otro número?**

El usuario puede responder con el número de la tabla o con el nombre del tipo directamente. Mapear número → tipo antes de continuar.

## FASE 4 — Nombre de rama

Generar slug automáticamente desde `<prompt-trabajo>`:
- Tomar las primeras palabras significativas (ignorar artículos, preposiciones)
- Convertir a minúsculas
- Reemplazar espacios con guiones
- Eliminar caracteres especiales (acentos, ñ → n, puntuación)
- Truncar a máximo 40 caracteres

Obtener fecha:

```bash
date +%Y-%m-%d
```

Construir nombre: `<tipo>/<fecha>-<slug-generado>`

Ejemplo: si el prompt fue "agregar integración con Strava webhook":
→ `feat/2026-04-28-agregar-integracion-strava-webhook`

Verificar colisión:

```bash
git branch --list "<nombre-generado>"
```

Si ya existe: proponer `<nombre>-2` o pedir descripción diferente.

Mostrar preview y pedir confirmación antes de crear:

> **Rama propuesta:** `<nombre-rama>` — ¿Confirmás o querés cambiar algo?

## FASE 5 — Crear rama

```bash
git checkout -b <nombre-rama>
```

## FASE 6 — Release version

Usando `<tipo>` capturado en Fase 3, determinar el release type:

| Branch type | Release flag |
|-------------|-------------|
| `feat` | `--minor` |
| `perf` | `--minor` |
| `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `build`, `ci`, `revert` | `--patch` |

**Major override:** si `<prompt-trabajo>` contiene palabras como `"breaking change"`, `"versión mayor"`, `"major release"`, `"incompatible"` → proponer `--major` con advertencia explícita al usuario.

Mostrar propuesta y pedir confirmación:

> **Release propuesta:** `php bin/console app:release --<type>` — ¿Confirmás? (s/n o escribí `major`/`minor`/`patch` para override)

No continuar hasta confirmar.

Ejecutar:

```bash
php bin/console app:release --<type>
```

Si falla → mostrar error completo. **BLOQUEADO** hasta resolver.

Commitear el archivo de versión generado:

```bash
git add -A
git commit -m "chore: release version"
```

## FASE 7 — Brainstorming, spec y plan

Informar al usuario:

> Rama creada. Iniciando brainstorming para diseñar la solución y generar spec + plan.

Invocar `superpowers:brainstorming` pasando como contexto de entrada:

```
Contexto del trabajo:
<prompt-trabajo>

Rama de trabajo: <nombre-rama>
```

Brainstorming se encargará de:
1. Hacer preguntas de clarificación si es necesario
2. Diseñar la solución junto al usuario
3. Escribir el spec en `docs/superpowers/specs/`
4. Invocar `superpowers:writing-plans` para generar el plan en `docs/superpowers/plans/`

## Resumen final

Al completar brainstorming y writing-plans, mostrar:

```
✅ Rama activa:   <nombre-rama>
📋 Spec:          docs/superpowers/specs/<fecha>-<topic>-design.md
📄 Plan:          docs/superpowers/plans/<fecha>-<feature>.md

💡 Cuando estés listo para ejecutar: invocar superpowers:executing-plans
```
