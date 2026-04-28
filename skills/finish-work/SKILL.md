---
name: finish-work
description: Use when development work is complete and ready to merge to develop — runs all mandatory validations (lint:container, phpunit, Playwright, manual browser check) then creates a PR. Invoke when user says "terminé", "quiero crear el PR", "finish work", "listo para PR", "crear pull request", "voy a mergear", "todo listo".
---

# powersoft:finish-work — Validar y crear PR

## FASE 1 — Pre-check

Verificar que hay commits nuevos respecto a develop:

```bash
git log develop..HEAD --oneline
```

Si no hay commits:

> **BLOQUEADO:** No hay commits nuevos respecto a `develop`. No hay trabajo para integrar.

Verificar estado limpio:

```bash
git status --short
```

Si hay cambios sin commitear:

> **PAUSADO:** Hay cambios sin commitear. Hacer commit o stash antes de continuar.

## FASE 2 — Validaciones obligatorias

**Todas son obligatorias y en orden. No hay forma de saltarse ninguna.**

### 2.1 — lint:container

```bash
php bin/console lint:container
```

Si falla → mostrar error completo. **BLOQUEADO** hasta corregir y volver a ejecutar.

### 2.2 — phpunit

```bash
php bin/phpunit
```

Si falla → mostrar resumen de tests fallidos. **BLOQUEADO** hasta corregir y volver a ejecutar.

### 2.3 — Playwright

Detectar si Playwright está configurado:

```bash
ls playwright.config.* 2>/dev/null; ls tests/*.spec.* 2>/dev/null | head -3
```

**Si está configurado:**

```bash
npx playwright test
```

Si falla → mostrar reporte. **BLOQUEADO** hasta corregir.

**Si NO está configurado:**

> ⚠️ Playwright no detectado en este proyecto. Se añade una confirmación manual adicional.

### 2.4 — Verificación manual

Preguntar:

> **¿Verificaste el comportamiento esperado en el browser? (s/n)**

Si `n` → **BLOQUEADO.** Verificar antes de continuar.

Si Playwright no estaba configurado, preguntar también:

> **¿Probaste todos los flujos afectados manualmente? (s/n)**

## FASE 3 — Generar borrador del PR

Obtener datos del contexto:

```bash
# Rama actual
git branch --show-current

# Commits nuevos
git log develop..HEAD --oneline

# Fecha en nombre de rama (para inferir plan)
git branch --show-current | grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2}'
```

Buscar plan vinculado en `docs/superpowers/plans/` usando la fecha extraída del nombre de la rama:

```bash
ls docs/superpowers/plans/*<fecha>*.md 2>/dev/null
```

Si hay múltiples archivos con esa fecha → preguntar al usuario cuál es el plan vinculado.
Si no hay ninguno → dejar el campo vacío y notificar al usuario.

Construir borrador con esta estructura exacta:

```markdown
## Resumen
- <bullet por cada commit relevante>

## Plan vinculado
[<nombre-del-plan>](docs/superpowers/plans/<archivo>.md)

## Validaciones completadas
- [x] `php bin/console lint:container` — OK
- [x] `php bin/phpunit` — OK
- [x] Playwright — OK  (sustituir por "N/A — no configurado" si aplica)
- [x] Verificación manual en browser — confirmada

## Test plan
- Verificar en browser el flujo principal afectado por este cambio
- Verificar que no hay regresiones en features relacionadas
- Confirmar que los tests de regresión pasan: `php bin/phpunit`
```

Mostrar el borrador completo al usuario y permitir edición libre antes de confirmar.

## FASE 4 — Crear PR

Tras confirmación del usuario:

```bash
gh pr create \
  --base develop \
  --title "<tipo>: <descripción>" \
  --body "<body-final>"
```

Mostrar URL del PR creado.
