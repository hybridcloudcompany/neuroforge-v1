# Change: Consultar saldo adeudado y crédito disponible de una tarjeta de crédito Visa

Jira: NEUR-2 (https://bcp-neuroforge.atlassian.net/browse/NEUR-2)

## Why
Un cliente titular de una tarjeta de crédito Visa necesita conocer su deuda (saldo adeudado) y cuánto puede utilizar en nuevas compras (crédito disponible) sin llamar al Contact Center ni acudir a una agencia (historia NEUR-2: "Como cliente titular de una tarjeta de crédito Visa, quiero consultar el saldo adeudado y el crédito disponible, para conocer mi deuda y cuánto puedo utilizar en nuevas compras").

**Decisiones incorporadas:** APSA cubre solo tarjetas de débito; VPLUS es la plataforma de tarjetas de crédito para el escenario. Se creó una página Confluence con contrato y valores ficticios para NEUR-2, claramente marcados como no verificados ni válidos para producción. El canal confirmado es MBBK móvil; antes de producción se deben validar los detalles reales de integración VPLUS.

## What Changes
- Nueva capacidad `credit-card-balance`: el titular autenticado consulta, para sus tarjetas de todas las marcas (sin adicionales), saldo adeudado, crédito disponible, moneda e instante de corte.
- Reglas de acceso (solo titular, solo sus tarjetas), protección de datos de tarjeta, manejo de errores/degradación y observabilidad de la consulta.
- Habilitación/deshabilitación remota de la funcionalidad sin liberar la app (feature flag).
- Sin cambios incompatibles en contratos existentes (APSA v1 y MBBK v8 permanecen intactos).

## Impact
- Capabilities afectadas: `credit-card-balance` (nueva). No existen specs previas en `openspec/specs/`.
- Componentes/sistemas:
  - Canal confirmado: MBBK móvil — [MBBK - Mobile Banking](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1441793/MBBK+-+Mobile+Banking) (`mbbk-cards-svc` + `mbbk-bff-gateway` + app).
  - Fuente de datos de la simulación: VPLUS (tarjetas de crédito). La página [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) contiene un contrato de ejemplo ficticio, no el contrato real.
  - APSA — [APSA - APIs de Tarjetas de Débito](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/851969/APSA+-+APIs+de+Tarjetas+de+D+bito): fuera de alcance (solo débito); permanece sin cambios.
- Código/áreas esperadas: adaptador de tarjetas y BFF de MBBK, pantalla en la app iOS/Android, configuración remota (`mbbk-config-svc`).

## Referencias
- [APSA - Visión General y Descripción Funcional](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1245185/APSA+-+Visi+n+General+y+Descripci+n+Funcional) — §1 y §4: APSA = débito; crédito (VPLUS) fuera de alcance.
- [APSA - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1146881/APSA+-+Cat+logo+de+APIs) — ningún endpoint de saldo/crédito disponible; convenciones (OAuth2, `problem+json`, `x-correlation-id`).
- [APSA - Decisiones, Restricciones y Roadmap](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1343489/APSA+-+Decisiones+Restricciones+y+Roadmap) — ADR-004 (no almacenar PAN), roadmap (sin crédito).
- [MBBK - Visión General y Descripción Funcional](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474579/MBBK+-+Visi+n+General+y+Descripci+n+Funcional) — §3 módulo Tarjetas (delegado a APSA, débito) y §4 fuera de alcance (créditos).
- [MBBK - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1638401/MBBK+-+Cat+logo+de+APIs) — rutas 19-23 son proxies de APSA; versión `v8`; convenciones y errores `MBBK-CARD-*`.
- [MBBK - Catálogo de Microservicios](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1507329/MBBK+-+Cat+logo+de+Microservicios) — `mbbk-cards-svc` (adaptador), comportamiento ante fallas.
- [MBBK - Integraciones y Dependencias con Otros Sistemas](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1572865/MBBK+-+Integraciones+y+Dependencias+con+Otros+Sistemas) — VPLUS no figura entre las integraciones.
- [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) — contrato y métricas sintéticos, solo para el escenario de demostración.
- [MBBK - Seguridad y Cumplimiento](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1114135/MBBK+-+Seguridad+y+Cumplimiento) y [MBBK - Requisitos No Funcionales, SLOs y Capacidad](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474561/MBBK+-+Requisitos+No+Funcionales+SLOs+y+Capacidad) — controles y SLOs.
- [MBBK - Decisiones, Restricciones y Roadmap](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/589826/MBBK+-+Decisiones+Restricciones+y+Roadmap) — ADR-004/006, DR parcial, roadmap sin crédito.
- [LET - Lineamientos Técnicos de Plataforma](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1179679/LET+-+Lineamientos+T+cnicos+de+Plataforma) y páginas `LET_WLD`, `LET_SEC`, `LET_OBS`, `LET_NET`, `LET_REG`, `LET_DAT`, `LET_CIC` (ver `design.md`).
- [CNF - Framework Cloud Native (Quarkus)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/950297/CNF+-+Framework+Cloud+Native+Quarkus), [CNF - APIs REST](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1638426/CNF+-+APIs+REST), [CNF - Seguridad y Configuración](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474608/CNF+-+Seguridad+y+Configuraci+n), [CNF - Migración desde Spring Boot y Adopción](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/983063/CNF+-+Migraci+n+desde+Spring+Boot+y+Adopci+n) — obligatorio CNF para servicios Java nuevos.
- [Plataforma AKS - Infraestructura Mínima](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/819201/Plataforma+AKS+-+Infraestructura+M+nima) — estándares de plataforma y aprobaciones.
