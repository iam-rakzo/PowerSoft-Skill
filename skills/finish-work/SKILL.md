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
```

Buscar plan vinculado en `docs/superpowers/plans/` mostrando los más recientes primero:

```bash
ls -lt docs/superpowers/plans/*.md 2>/dev/null | head -5
```

Mostrar la lista al usuario y preguntar cuál corresponde a este trabajo.
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

Guardar URL del PR creado como `<pr-url>`. Mostrar al usuario.

## FASE 5 — Code Review + Security Review

Informar al usuario:

> Iniciando análisis de código y seguridad sobre la PR recién creada. Ejecutando en paralelo…

Invocar ambos en paralelo usando `superpowers:dispatching-parallel-agents`:

- **Agente 1:** `code-review:code-review` — pasando `<pr-url>` como input
- **Agente 2:** `security-review` — ejecutado sobre el branch actual

Esperar resultados de ambos antes de continuar.

### Evaluación de findings

**Security review:**
- Hallazgo crítico (vulnerabilidad, exposición de datos, inyección) → **BLOQUEADO.** Mostrar hallazgos completos. No continuar hasta que el usuario corrija y vuelva a ejecutar `finish-work`.
- Hallazgo menor → mostrar como advertencia, continuar.

**Code review:**
- Hallazgos presentes → mostrar lista completa, luego preguntar:
  > **Code review encontró findings. ¿Confirmás que procedés con estos pendientes para resolver en PR de seguimiento? (s/n)**
  - `n` → **PAUSADO.** El usuario debe resolver antes de continuar.
  - `s` → continuar.
- Sin hallazgos → continuar.

### Resumen final

Mostrar estado consolidado:

```
✅ PR creado:        <pr-url>
✅ Security review:  OK  (o lista de advertencias menores)
✅ Code review:      OK  (o "X findings — aceptados por usuario")
```
