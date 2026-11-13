---
description: Avanza una issue Jira en el flujo SDD (un paso según su label sdd-*).
agent: sdd-orchestrator
---

Procesa la issue Jira `${input:issueKey:Clave de la issue, p. ej. PROJ-123}` con el flujo SDD: detecta su estado por labels `sdd-*`, ejecuta únicamente el paso correspondiente y reporta el nuevo estado y la acción humana siguiente.
