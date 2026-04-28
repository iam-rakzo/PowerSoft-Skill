---
name: start-work
description: Use before starting any new development work — verifies an approved plan exists, syncs git with develop, and creates a properly typed branch. Invoke when user says "voy a empezar", "nueva tarea", "nueva feature", "nuevo fix", "empecemos a trabajar", "start work", "nueva rama", "quiero implementar".
---

# powersoft:start-work — Preparar rama de trabajo

## FASE 1 — Verificar plan aprobado (GATE)

Buscar planes aprobados en el proyecto actual:

```bash
ls -lt docs/superpowers/plans/*.md 2>/dev/null | head -10
```

**Si el directorio no existe o no hay archivos `.md`:**

> **BLOQUEADO:** No existe ningún plan aprobado en `docs/superpowers/plans/`.
> Invocar `superpowers:brainstorming` → `superpowers:writing-plans` antes de continuar.
> No se puede crear una rama sin un plan que respalde el trabajo.

**Si existen planes:** mostrar la lista (nombre de archivo + fecha) y preguntar al usuario cuál aplica a este trabajo. Anotar el path del plan seleccionado — se usa en FASE 5 y en `finish-work`.

## FASE 2 — Git sync

Ejecutar en orden:

```bash
git fetch origin --prune
git switch develop
git pull --rebase
```

Si `git pull --rebase` produce conflictos de rebase:

> **PAUSADO:** Conflictos de rebase detectados. Resolver los conflictos antes de crear la rama.
> No crear rama sobre estado roto.

Confirmar que develop está actualizado:

```bash
git log --oneline -3
```

## FASE 3 — Tipo de trabajo

Mostrar tabla de tipos:

| Tipo | Uso |
|------|-----|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Solo documentación |
| `style` | Formato, espacios (sin cambios de lógica) |
| `refactor` | Refactorización sin nueva funcionalidad ni fix |
| `test` | Añadir o modificar tests |
| `chore` | Build, dependencias, configuración |
| `build` | Cambios que afectan dependencias del sistema |
| `perf` | Mejoras de rendimiento |
| `revert` | Revertir un commit anterior |
| `ci` | Cambios en pipelines de CI/CD |

Preguntar: **¿Qué tipo de trabajo es este?**

## FASE 4 — Nombre de rama

Preguntar: **Escribe una descripción corta del trabajo** (ej: "agregar strava webhook", "fix login redirect")

Normalizar automáticamente:
- Convertir a minúsculas
- Reemplazar espacios con guiones
- Eliminar caracteres especiales (acentos, ñ, puntuación)
- Truncar a máximo 40 caracteres

Obtener fecha:

```bash
date +%Y-%m-%d
```

Construir nombre: `<tipo>/<fecha>-<descripcion-normalizada>`

Ejemplo: `feat/2026-04-24-agregar-strava-webhook`

Verificar colisión:

```bash
git branch --list "<nombre-generado>"
```

Si ya existe: proponer `<nombre>-2` o pedir descripción diferente al usuario.

Mostrar preview y pedir confirmación antes de crear.

## FASE 5 — Crear rama

```bash
git checkout -b <nombre-rama>
```

## Resumen

Mostrar al usuario:

```
✅ Rama activa:   <nombre-rama>
📄 Plan vinculado: <path-del-plan>

💡 Si el plan tiene fases detalladas, considera invocar superpowers:executing-plans
```