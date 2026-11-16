# story-workflow

Plugin de Claude Code con los agentes y la skill que uso para implementar historias de Azure DevOps en tres repos (backend, frontend y Azure Functions).

## Contenido

| Tipo | Nombre | Qué hace |
|---|---|---|
| Skill | `implement-story` | Orquesta el flujo completo: plan → aprobación → ramas → código → pruebas → revisión → PRs |
| Agente | `story-planner` | Lee la historia y escribe el plan de implementación para revisión humana |
| Agente | `git-manager` | Sincroniza la rama base, crea ramas (y `-db`), hace push y abre PRs |
| Agente | `backend-developer` | Implementa pasos del plan en backend y Azure Functions |
| Agente | `frontend-developer` | Implementa pasos del plan en el frontend |
| Agente | `test-engineer` | Crea y ajusta pruebas unitarias y verifica que corran |
| Agente | `code-reviewer` | Revisa todo contra el plan y los estándares (solo lectura) |

No incluye hooks por ahora. Si se agregan, van en `hooks/hooks.json`.

## Instalación

Dentro de Claude Code:

```
/plugin marketplace add <url-del-repo>
/plugin install story-workflow@story-workflow-marketplace
```

Para probarlo sin instalar, desde una copia local:

```
claude --plugin-dir <ruta-a-este-repo>
```

## Uso

```
/story-workflow:implement-story <numeroDeHistoria>
```

## Datos locales (no están en el repo)

Los agentes leen y escriben en `~/.claude/story-plans/`, que vive solo en tu máquina:

- `azure-devops.md` — organización y proyecto de Azure DevOps (se pregunta la primera vez)
- `branch-conventions.md` — estándar de nombres de ramas y commits (se confirma la primera vez)
- `<id>-plan.md` — el plan de cada historia

## Requisitos

- MCP de Azure DevOps, o `az` con la extensión `azure-devops`, para leer historias y abrir PRs
- `dotnet` y `node` según el repo que se toque
