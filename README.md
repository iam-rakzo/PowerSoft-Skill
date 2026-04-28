# powersoft

Claude Code plugin — development workflow skills for Symfony/PHP projects.

## Skills

| Skill | Trigger |
|-------|---------|
| `powersoft:start-work` | "voy a empezar", "nueva tarea", "nueva feature", "start work" |
| `powersoft:finish-work` | "terminé", "quiero crear el PR", "finish work", "listo para PR" |

## What each skill does

**`powersoft:start-work`**
1. Verifies an approved plan exists in `docs/superpowers/plans/`
2. Syncs `develop` branch (`fetch → switch → pull --rebase`)
3. Asks for work type (feat/fix/refactor/etc.)
4. Generates a normalized branch name with date prefix
5. Creates the branch

**`powersoft:finish-work`**
1. Verifies commits exist ahead of `develop`
2. Runs mandatory validations: `lint:container` → `phpunit` → Playwright → manual browser check
3. Generates PR draft body from commits + linked plan
4. Creates the PR via `gh pr create`

## Install

```bash
claude /install-plugin github:oscrodriguez/powersoft
```

## Adding new skills

Create `skills/<skill-name>/SKILL.md` with frontmatter:

```markdown
---
name: skill-name
description: When to invoke this skill...
---

# powersoft:skill-name — Title
...
```
