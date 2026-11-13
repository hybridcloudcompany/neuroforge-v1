# NeuroForge — Agentes SDD (Jira + Confluence → OpenSpec)

Agentes de GitHub Copilot (VS Code / Copilot CLI) que convierten una issue Jira en una especificación **OpenSpec**, usando Confluence como contexto, y gestionan las dependencias (aprovisionamiento, accesos, modelo operativo) como issues documentadas. El avance se controla con **labels de Jira**.

## Contenido
| Archivo | Rol |
|---|---|
| [.vscode/mcp.json](.vscode/mcp.json) | Servidor MCP Atlassian (Jira + Confluence) |
| [sdd.config.yaml](sdd.config.yaml) | Sitio, espacios Confluence, proyecto/tipo de issues de dependencia |
| [.github/skills/sdd-jira-workflow](.github/skills/sdd-jira-workflow/SKILL.md) | Máquina de estados, gates, seguridad, formato OpenSpec y plantillas |
| [.github/agents/sdd-orchestrator.agent.md](.github/agents/sdd-orchestrator.agent.md) | Punto de entrada: enruta según el label |
| [.github/agents/sdd-spec-author.agent.md](.github/agents/sdd-spec-author.agent.md) | Lee Jira + Confluence, redacta spec, publica preguntas |
| [.github/agents/sdd-dependency-analyst.agent.md](.github/agents/sdd-dependency-analyst.agent.md) | Detecta dependencias en Confluence y pide confirmación |
| [.github/agents/sdd-dependency-issues.agent.md](.github/agents/sdd-dependency-issues.agent.md) | Crea las issues de dependencia confirmadas |
| [.github/prompts/sdd-run.prompt.md](.github/prompts/sdd-run.prompt.md) | `/sdd-run PROJ-123` |

## Flujo

```
sdd-ready ─▶ [spec-author] ─▶ sdd-awaiting-answers ─(humano: sdd-answers-provided)─┐
                 ▲                                                                  │
                 └──────────────────────────────────────────────────────────────────┘
             (sin preguntas) ─▶ sdd-spec-review ─(humano: sdd-spec-approved)─▶ [dependency-analyst]
 sin dependencias ─▶ sdd-completed + DOR
 con dependencias ─▶ sdd-awaiting-deps-confirmation ─(humano: D1: CREAR... + sdd-deps-confirmed)─▶ [dependency-issues] ─▶ sdd-awaiting-deps-closure ─(todas cerradas)─▶ sdd-completed + DOR
```

**Gates humanos** (el agente nunca los pone): `sdd-ready`, `sdd-answers-provided`, `sdd-spec-approved`, `sdd-deps-confirmed`.

El agente mantiene en el campo Jira `Especificación` el estado vigente de la spec: integra las respuestas aceptadas en los requisitos y decisiones, y deja allí únicamente preguntas todavía pendientes; nunca conserva Q&A resueltas. Sincroniza los escenarios OpenSpec con el campo `Criterios de Aceptación`; cada criterio tiene título y formato Gherkin en español (`DADO`, `CUANDO`, `ENTONCES`), y los afectados por preguntas abiertas se marcan provisionales. El analista y el agente de dependencias sincronizan el campo `Dependencias` con el inventario, las decisiones humanas y las issues creadas/encontradas. El campo `Descripción` es de solo lectura para los agentes. Cuando no quedan preguntas ni observaciones y la spec pasa a `sdd-spec-review`, agrega `BDD`; `DOR` se agrega únicamente cuando no hay dependencias o todas están cerradas/resueltas. Ninguno sustituye la aprobación humana `sdd-spec-approved`.

## Puesta en marcha
1. Instala/arranca el MCP `atlassian` desde VS Code (autenticación OAuth) y ajusta [sdd.config.yaml](sdd.config.yaml).
2. (Recomendado) `npm i -g @fission-ai/openspec` y `openspec init` para validar con `openspec validate --strict`.
3. En Jira, el humano pone `sdd-ready` y en Copilot Chat ejecuta `/sdd-run PROJ-123` (o selecciona el agente `sdd-orchestrator`).
4. Responde en la issue citando IDs (`Q1: ...`), pon el label de gate y vuelve a ejecutar `/sdd-run PROJ-123`.

## Sobre "esperar"
Un agente de chat no puede quedarse bloqueado esperando a una persona. La espera se modela con labels y cada ejecución avanza un paso. Para automatizarlo (sin intervención manual) se puede disparar `/sdd-run` desde una **Jira Automation** (cuando se añade un label de gate) hacia un runner (Copilot CLI/GitHub Actions) — no incluido aquí.

## Notas
- Confluence es solo lectura. Jira: los agentes editan labels, comentarios y únicamente los campos dedicados `Especificación`, `Criterios de Aceptación` y `Dependencias`; nunca modifican `Descripción`. El agente de dependencias crea solo las issues confirmadas.
- Los labels y IDs de enlace/tipo de issue pueden requerir ajuste según el esquema de tu Jira (`sdd.config.yaml`).
- Los enlaces entre issues usan `createIssueLink` (tipos del sitio: Blocks, Cloners, Duplicate, Relates).
