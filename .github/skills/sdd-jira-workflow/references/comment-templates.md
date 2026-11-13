# Plantillas de comentarios Jira (markdown)

## [SDD-AGENT:questions] — preguntas abiertas
```markdown
**[SDD-AGENT:questions]** Ronda <n> — <KEY>

Revisé el requerimiento y la documentación de Confluence (<lista breve de páginas con links>). Borrador de spec: `<ruta del cambio>`.

**Resumen de entendimiento:** <3-5 líneas>

**Preguntas abiertas** (responde citando el ID; si estás de acuerdo con la suposición, escribe "Qn: OK"):
- **Q1** <pregunta> — _Suposición por defecto:_ <...> — _Impacto:_ <...>
- **Q2** ...

El campo Jira `Especificación` contiene el estado vigente y solo las preguntas pendientes; las respuestas aceptadas ya se integran en sus requisitos y decisiones, no se mantienen como Q&A resueltas.
Los criterios de aceptación derivados de los escenarios OpenSpec se mantienen en el campo Jira `Criterios de Aceptación`, cada uno con título y pasos Gherkin `DADO`, `CUANDO`, `ENTONCES`; lo que dependa de preguntas abiertas se marca provisional.

**Siguiente paso:** responde en un comentario citando los IDs; después pon el label `sdd-answers-provided`.
```

## [SDD-AGENT:review] — spec lista para validar
```markdown
**[SDD-AGENT:review]** Spec sin preguntas abiertas — <KEY>

- Cambio: `<ruta>` (proposal, design, tasks, specs/<capability>)
- Requisitos: <n> ADDED / <n> MODIFIED / <n> REMOVED
- Decisiones tomadas a partir de tus respuestas: <lista Qn → decisión>
- Validación `openspec validate --strict`: <OK/no ejecutada>

Se añadió el label auxiliar `BDD`, que indica que la spec está lista para revisión, no que esté aprobada. `DOR` se añadirá únicamente cuando el análisis confirme que no hay dependencias o todas estén cerradas/resueltas.

**Siguiente paso:** revisa y pon `sdd-spec-approved`. Si tienes observaciones, coméntalas y pon `sdd-answers-provided`.
```

## [SDD-AGENT:dependencies] — dependencias detectadas
```markdown
**[SDD-AGENT:dependencies]** Dependencias para <KEY>

Según Confluence ([página](url), ...) la implementación requiere:

El inventario y su estado también se mantienen en el campo Jira `Dependencias`.

| ID | Tipo | Dependencia | Fuente | Equipo dueño | Lead time | ¿Ya existe issue? |
|----|------|-------------|--------|--------------|-----------|-------------------|
| D1 | aprovisionamiento | ... | [pág](url) | ... | ... | no / KEY |

Responde por ID con una de estas opciones:
- `D1: CREAR` — el agente crea una issue relacionada que bloquea a <KEY> (descripción Como/Quiero/Para, label `sdd_created`). Opcional: `proyecto=<KEY> asignado=<persona> prioridad=<...>`.
- `D1: APROBADA <evidencia>` — ya la gestionaste; indica qué se hizo, referencia/enlace/ticket, quién y cuándo. No se crea issue.
- `D1: EXISTE <KEY>` — usar una issue existente como bloqueante.
- `D1: NO` — no aplica / ya resuelto sin acción adicional.
Si falta alguna dependencia, descríbela como `D+: <descripción>`.

**Siguiente paso:** pon `sdd-deps-confirmed`.
```

## [SDD-AGENT:dependency-issues] — resultado
```markdown
**[SDD-AGENT:dependency-issues]** Gestión de dependencias — <KEY>

| ID | Decisión | Issue / evidencia | Resumen | Estado |
|----|----------|-------------------|---------|--------|
| D1 | CREAR | <KEY-n> (bloquea a <KEY>, `sdd_created`) | ... | creada / abierta / cerrada |
| D2 | APROBADA | evidencia de <autor>, <fecha>: <resumen> | ... | resuelta por aprobación humana |
| D3 | EXISTE | <KEY-m> | ... | ya existía / abierta / cerrada |
| D4 | NO | — | ... | omitida (no aplica) |

`tasks.md` actualizado con claves de subtareas. Estado final: `sdd-awaiting-deps-closure` si queda alguna issue bloqueante abierta o una decisión pendiente; `sdd-completed` con `DOR` solo cuando todas estén cerradas, aprobadas con evidencia o no apliquen y las subtareas estén sincronizadas.

**Subtareas**: `<ID de tasks.md> → <KEY> — <resumen>`.

**Archivos adjuntos** (solo al completar): `proposal.md`, `design.md` — <adjuntados / ya existían sin cambios / versión nueva / fallaron: motivo>. No adjuntar otros archivos SDD.
```

## [SDD-AGENT:blocked]
```markdown
**[SDD-AGENT:blocked]** No puedo continuar: <motivo concreto>. Necesito: <acción humana>. Al resolverlo, devuelve el label `<label previo>`.
```

## Plantilla de descripción para issues de dependencia
Las issues de dependencia llevan el label `sdd_created` y se enlazan como bloqueantes de la issue origen. La descripción empieza siempre con `Como / Quiero / Para`.
```markdown
**Como** <rol o equipo que necesita la dependencia; p. ej. equipo implementador del componente>,
**Quiero** <resultado concreto y verificable: recurso, acceso o acuerdo operativo, sin prescribir la solución>,
**Para** <valor o habilitador, ligado a <KEY origen> — <resumen de la historia>>.

## Contexto
Bloquea a <KEY origen> — <resumen>. Spec: `<ruta>`.

## Qué se solicita
<detalle del recurso/acceso/acuerdo, tomado del proceso de Confluence: entornos, tamaños, roles, ventanas>

## Información requerida por el proceso
<campos/datos que el proceso de Confluence exige para la solicitud; marca ❓ los que falten>

## Criterios de aceptación
**CA1 — <título>**
- **DADO** <contexto>
- **CUANDO** <acción o hito>
- **ENTONCES** <resultado verificable>

## Proceso y responsables (Confluence)
- Página: [título](url)
- Equipo dueño / canal de solicitud: ...
- Lead time / SLA: ...
- Aprobaciones necesarias: ...

## Notas
Creada por el agente SDD tras la decisión `CREAR` en <KEY origen>.
```

Ejemplo de cabecera bien definida:
```markdown
**Como** equipo de desarrollo de MBBK,
**Quiero** disponer de la capacidad de caché Redis ampliada (P2) en el entorno de producción,
**Para** servir la consulta de saldo y crédito de tarjetas (NEUR-2) con el pico de 10 TPS sin superar el 70 % de memoria.
```
Si no puedes definir con rigor Como/Quiero/Para a partir de la spec y Confluence, marca ❓ lo que falta y pregúntalo antes de crear la issue.
