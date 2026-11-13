# Dependencias — NEUR-32

Análisis del 2026-11-13. **Resultado: no se identifican dependencias reales pendientes de decisión.** Alcance analizado: demo de solo lectura (saldo adeudado, crédito disponible, moneda y fecha-hora de corte de una tarjeta Visa) sobre el simulador VPLUS ficticio; fuera de alcance: datos reales, cambios al contrato productivo de VPLUS y cualquier despliegue productivo (spec `credit-card-balance-demo`, `design.md`).

## Fuentes revisadas

- [VPLUS - Plataforma de Tarjetas de Credito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) (v2, espacio DDS): contrato `GET /credit-cards/{cardId}/balances` (`amountOwed`, `availableCredit`, `currency`, `asOf`), errores de ejemplo y sección «Resolución de dependencias de NEUR-2 (solo simulación)» (D1 contrato/modelo operativo y D2 acceso/conectividad documentados para simulación; D3 Redis fuera del dominio de VPLUS).
- Búsqueda CQL en el espacio DDS (`VPLUS`, `demo`, `NEUR-32`, `simulación`): solo devuelve la página anterior y [APSA - Visión General y Descripción Funcional](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1245185/APSA+-+Visi+n+General+y+Descripci+n+Funcional) (confirma que crédito/VPLUS está fuera de APSA; no aporta requisitos nuevos).
- Jira: NEUR-8, NEUR-9 y NEUR-10 (dependencias de NEUR-2, label `sdd_created`, estado `Finalizada`, categoría `Done`; resueltas el 2026-11-13 15:00). No existe ninguna issue de dependencia propia de NEUR-32 ni enlaces en NEUR-32. La spec previa está en `openspec/changes/neur-2-consultar-saldo-credito-visa/`.

## Evaluación de necesidades

| ID | Tipo | Necesidad evaluada | Clase | Evidencia | Conclusión |
|---|---|---|---|---|---|
| — | Modelo operativo / integración | Contrato y fuente de saldos (VPLUS simulado), semántica de saldo adeudado, crédito disponible y fecha de corte, soporte y SLO | Informativa | Página VPLUS (v2), secciones «Endpoints de ejemplo» y «D1 — Contrato y modelo operativo simulado»; NEUR-8 `Finalizada` (su alcance era la validación real para producción, «bloqueante para producción, no para la simulación») | **Cubierta para la demo**: la demo consume el contrato ficticio ya documentado, sin modificarlo. La validación real con el dueño de VPLUS queda fuera de alcance (datos reales y producción excluidos). No se requiere issue nueva. |
| — | Accesos / conectividad | Identidad, scope y ruta hacia el mock VPLUS (`mbbk-cards-svc-sim`, producto APIM `VPLUS-CreditCards-Read-SIM`, scope `vplus.credit-cards.read`, APIM interno no-productivo, sin firewall) | Informativa | Página VPLUS (v2), sección «D2 — Acceso y conectividad simulados»; NEUR-10 `Finalizada` (alcance: producción; «no para la simulación») | **Cubierta para la demo**: mismo consumidor y mismo contrato que NEUR-2; se reutiliza lo documentado. El acceso productivo no aplica (fuera de alcance). No se requiere issue nueva. |
| — | Aprovisionamiento | Redis P1→P2 de MBBK para caché de 30 s | No aplica | Página VPLUS (v2), «D3 — Redis MBBK»; NEUR-9 `Finalizada` (alcance: producción) | **No aplica**: ni la spec ni el diseño de NEUR-32 exigen caché ni capacidad adicional; la demo no se despliega en producción. |
| — | Otros (red, secretos, colas, BD, observabilidad, cumplimiento) | Recursos adicionales no contemplados por la spec ni por el diseño | — | `proposal.md`, `design.md`, `specs/credit-card-balance-demo/spec.md`, `tasks.md` | Sin necesidad identificada. La enmascaración de PAN/CVV/PIN/tokens y el 503 son requisitos de implementación (tareas 2.2 y 2.3), no dependencias. |

## Dependencias a decidir

Ninguna. No hay dependencias de aprovisionamiento, acceso o modelo operativo bloqueantes (para empezar o para desplegar) ni pendientes de decisión humana, por lo que no se genera tabla `D1..Dn` ni comentario de confirmación.

## Notas

- Reutilización: D1–D3 de NEUR-2 (NEUR-8/9/10) no se enlazan a NEUR-32 porque están acotadas a producción y a NEUR-2; la demo se apoya en la documentación de simulación de la página VPLUS, que es ficticia. Si más adelante la demo pasa a datos reales o producción, deberá rehacerse este análisis.
- Observación para el humano: el título de la issue aún dice «próximo pago»; la spec aceptada limita la demo a saldos y fecha de corte.
