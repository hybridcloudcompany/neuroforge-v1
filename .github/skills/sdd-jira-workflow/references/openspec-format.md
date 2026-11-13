# Formato OpenSpec (cambios)

Estructura de un cambio:

```
openspec/
├─ specs/<capability>/spec.md            # verdad actual (si existe)
└─ changes/<issue-key>-<slug>/           # ej. proj-123-alta-pagos-qr
   ├─ proposal.md
   ├─ design.md                          # opcional pero recomendado si hay decisiones técnicas
   ├─ tasks.md
   ├─ open-questions.md                  # extensión del flujo SDD
   ├─ dependencies.md                    # extensión del flujo SDD (fase 2)
   └─ specs/<capability>/spec.md         # DELTA de requisitos
```

Antes de crear: revisar `openspec/specs/` para reutilizar capabilities existentes (usar `MODIFIED`/`REMOVED` en lugar de duplicar). Si existe el CLI, `openspec list` / `openspec validate <change-id> --strict`. Si `openspec/` no existe, crear la estructura mínima (no ejecutar `init` sin permiso).

## proposal.md
```markdown
# Change: <título breve>

Jira: <KEY> (<url>)

## Why
<problema/oportunidad en 1-3 frases, del requerimiento>

## What Changes
- <cambio 1>
- **BREAKING** <si aplica>

## Impact
- Capabilities afectadas: <lista>
- Componentes/sistemas (Confluence): <nombre — [título página](url)>
- Código/áreas esperadas: <si se conoce>

## Referencias
- [Título página Confluence](url) — qué aporta
```

## specs/<capability>/spec.md (delta)
```markdown
## ADDED Requirements
### Requirement: <nombre>
The system SHALL <comportamiento verificable>.

#### Scenario: <caso>
- **WHEN** <condición>
- **THEN** <resultado observable>

## MODIFIED Requirements
### Requirement: <nombre EXISTENTE>
<texto COMPLETO nuevo del requisito, no solo el diff>

#### Scenario: ...

## REMOVED Requirements
### Requirement: <nombre>
**Reason**: ...  **Migration**: ...

## RENAMED Requirements
- FROM: `### Requirement: A`  TO: `### Requirement: B`
```
Reglas: cada requisito con ≥1 `#### Scenario:` (exactamente 4 `#`); usar SHALL/MUST; el encabezado debe coincidir exactamente al modificar; un requisito = un comportamiento verificable; nada de detalles de implementación aquí (van en design.md).

## design.md
`## Context` · `## Goals / Non-Goals` · `## Decisions` (decisión + alternativas + porqué) · `## Risks / Trade-offs` · `## Migration Plan` · `## Open Questions` (solo IDs `Qn` de open-questions.md).

## tasks.md
```markdown
## 0. Prerequisitos (dependencias)      <- lo completa la fase de dependencias
- [ ] 0.1 <D1: descripción> — issue <KEY>
## 1. <Área>
- [ ] 1.1 <tarea pequeña y verificable>
```

## open-questions.md
```markdown
| ID | Pregunta | Por qué importa / impacto en spec | Fuente | Estado | Respuesta |
|----|----------|-----------------------------------|--------|--------|-----------|
| Q1 | ...      | ...                               | Jira / [Confluence](url) | abierta/resuelta | ... (autor, fecha) |
```
Una pregunta es válida solo si: no se puede responder con Jira/Confluence, y cambia requisitos, alcance, criterios o diseño. Agrupa por tema, máximo ~10 por ronda, prioriza bloqueantes. Cada pregunta propone una **suposición por defecto** para que el humano solo corrija.

## dependencies.md
```markdown
| ID | Tipo (aprovisionamiento/acceso/modelo operativo/otro) | Descripción | Evidencia Confluence | Equipo dueño | Proceso/plantilla de solicitud | Lead time | Issue existente | Decisión (CREAR/APROBADA/EXISTE/NO/pendiente) | Issue creada (`sdd_created`) o evidencia de aprobación |
```
En `Decisión = APROBADA` la última columna cita la evidencia humana (autor, fecha, referencia); no hay issue creada.
