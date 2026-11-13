## Context
NEUR-2 pide que un titular de tarjeta de crédito Visa consulte saldo adeudado y crédito disponible. La documentación de Confluence (espacio DDS, sitio bcp-neuroforge) describe dos aplicativos relacionados, ambos en producción sobre AKS (East US 2):

- **APSA** (APP-0457, Tier 1): APIs de **tarjetas de débito**. Su "Fuera de alcance" incluye explícitamente las tarjetas de crédito, cuya plataforma el usuario identificó como **VPLUS**. Su catálogo (15 endpoints) no tiene saldo ni crédito disponible, y el procesador `CardSwitch` se usa para estado, límites, controles y movimientos de débito. Fuente: [APSA - Visión General](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1245185/APSA+-+Visi+n+General+y+Descripci+n+Funcional), [APSA - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1146881/APSA+-+Cat+logo+de+APIs).
- **MBBK** (APP-0312, Tier 1): banca móvil; su módulo Tarjetas es un proxy de APSA (débito) y "créditos y seguros" son plataformas propias fuera de alcance. Cualquier endpoint nuevo de tarjetas exige cambios en `mbbk-cards-svc` y en el BFF. Fuente: [MBBK - Visión General](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474579/MBBK+-+Visi+n+General+y+Descripci+n+Funcional), [MBBK - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1638401/MBBK+-+Cat+logo+de+APIs).

**Alcance acordado:** la historia habla de Visa, pero la funcionalidad aplica a tarjetas de crédito de todas las marcas del titular. APSA y MBBK no cubren crédito en su documentación operativa actual. VPLUS es la fuente de crédito para esta simulación; la página Confluence contiene un contrato y métricas ficticios, no una especificación operativa. El diseño de simulación está definido; la integración productiva requiere validar los datos reales con los responsables de VPLUS.

**Contradicciones/inconsistencias detectadas entre páginas:**
- `MBBK - Visión General` describe "Cuentas y saldos" (Core) y "Tarjetas" (APSA) sin cubrir tarjetas de crédito; "Fuera de alcance" excluye créditos.
- `APSA - Decisiones` (roadmap) solo planifica tarjetas virtuales *de débito*, tokenización y controles por MCC; no hay hito de crédito.
- La página [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) define un contrato y SLOs sintéticos para esta demostración; deben reemplazarse con datos aprobados antes de producción. Crédito permanece fuera del alcance de APSA.

## Goals / Non-Goals
**Goals**
- Describir el comportamiento observable de la consulta de saldo adeudado y crédito disponible para el titular.
- Cumplir los LET y la seguridad bancaria aplicables (ver tabla de LET abajo).
- No romper contratos existentes (APSA v1, MBBK v8).

**Non-Goals**
- Modificar el alcance de APSA (tarjetas de crédito), que sigue limitado a débito.
- Pago de tarjeta, estados de cuenta, movimientos de crédito, límites/controles de crédito, PIN.
- Analizar dependencias de infraestructura/acceso (fase siguiente, `sdd-dependency-analyst`).

## Decisions
### D1. No extender APSA con crédito
- **Decisión para la simulación:** VPLUS es la fuente de tarjetas de crédito. Se usa el contrato ficticio de lectura REST/APIM de la página VPLUS para el prototipo; antes de producción debe sustituirse por el contrato aprobado por sus responsables. APSA no cambia.
- **Alternativas:** (a) ampliar APSA a crédito → requiere cambiar su alcance/ADR, el procesador `CardSwitch` y sus 8 servicios Spring Boot, y contradice "Fuera de alcance"; (b) `mbbk-cards-svc` consume el contrato simulado de VPLUS para el prototipo, sujeto a sustitución por el contrato aprobado; (c) servicio nuevo intermediario entre canales y VPLUS.
- **Por qué:** respeta los límites documentados de cada aplicativo y evita mezclar PCI/débito con crédito. Si se adopta un servicio Java nuevo, **debe crearse con CNF** (obligatorio para servicios nuevos desde 2026-01-01; [CNF - Migración desde Spring Boot y Adopción](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/983063/CNF+-+Migraci+n+desde+Spring+Boot+y+Adopci+n) §1), con los módulos `cnf-rest`, `cnf-http-client`, `cnf-security`, `cnf-audit` si aplica ([CNF - Framework](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/950297/CNF+-+Framework+Cloud+Native+Quarkus) §7).

### D2. Adaptador en `mbbk-cards-svc` + ruta en el BFF
- `mbbk-cards-svc` (Spring Boot 3.3, soportado hasta 2027-12-31) se propone como adaptador al contrato simulado de VPLUS para el prototipo; el BFF (`mbbk-bff-gateway`, Node.js) agrega la pantalla móvil. Las rutas son **aditivas** en MBBK `v8` (no se toca `GET /cards`). Convenciones MBBK: `x-channel-id: MOBILE`, `x-correlation-id`, errores `application/problem+json` con prefijo `MBBK-CARD-*`. El contrato debe reemplazarse por el aprobado antes de producción.
- **Alternativa:** reutilizar `GET /cards` mezclando débito y crédito → cambia el contrato de una ruta existente y exige coordinación con APSA; descartada.

### D3. Caché de lectura con TTL corto
- Caché en Redis con TTL ≤ 30 s (patrón ya usado para saldos). `LET_DAT_040` exige TTL en todas las claves y memoria ≤ 70% en pico; MBBK está hoy en 78% (no conformidad abierta, ampliación a P2 pendiente), por lo que el volumen de claves nuevas debe medirse.

### D4. Resiliencia y degradación
- Para la simulación, timeout 3 s / 1 reintento solo GET de lectura y circuit breaker en `cards-svc` siguen los valores **ficticios** de la página VPLUS y el precedente MBBK→APSA; no son política aprobada para VPLUS. Con la fuente caída se responde 503 y el resto de la app sigue operativa. La funcionalidad se puede apagar con remote config (`mbbk-config-svc`, ADR-006 de MBBK; `LET_CIC_080`).

### D5. Seguridad
- JWT/sesión existente del cliente + validación de titularidad en cada consulta; scopes y `client_id` de MBBK ante la fuente de crédito (equivalente a `cards.read` en APSA). Sin token digital adicional para la consulta; no se trata como operación sensible de auditoría inmutable. Sin PAN/CVV/PIN en datos/logs (`LET_SEC_080`); secretos en Key Vault con Workload Identity (`LET_SEC_020`, `LET_SEC_030`). Pentest y SAST/SCA/DAST de la ruta nueva (`LET_SEC_060`, `LET_SEC_100`). PCI DSS: MBBK está fuera de alcance directo porque no maneja PAN/PIN; mantener ese invariante.

### D6. Calidad y liberación
- Pruebas de contrato Pact 100% (`LET_CIC_030`), carga al 150% del pico (`LET_WLD_150`), promoción DEV→QA→PRD con CAB y ventana de cambio de canales/tarjetas martes y jueves 22:00-02:00 hora Lima (`LET_CIC_040`, `LET_CIC_050`); congelamiento del 15-dic al 5-ene y últimos 2 días hábiles de cada mes (`LET_CIC_060`) — relevante para la fecha actual. Ciclo de tiendas de la app: 2-5 días, compatibilidad con las 3 últimas versiones.

## LET aplicables (referencia)
Fuente: [LET - Lineamientos Técnicos de Plataforma](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1179679/LET+-+Lineamientos+T+cnicos+de+Plataforma) y sus páginas hijas.

| Código | Aplicación a este cambio |
|---|---|
| `LET_WLD_010`, `LET_WLD_030`, `LET_WLD_040`, `LET_WLD_060` | Solo si se crea un servicio nuevo (D1.c): 3 réplicas en PRD, PDB, topología por zona, HPA 60-70% ([LET_WLD](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/458788/LET_WLD+-+Cargas+de+Trabajo+Pods+y+Escalabilidad)) |
| `LET_WLD_140`, `LET_WLD_150` | Capacidad de diseño ≥ 140% del pico y prueba de carga al 150% |
| `LET_SEC_020`, `LET_SEC_030`, `LET_SEC_060`, `LET_SEC_080`, `LET_SEC_100` | Workload Identity, Key Vault, SAST/SCA/DAST, sin PAN/PIN/CVV en logs, pentest ([LET_SEC](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/688162/LET_SEC+-+Seguridad)) |
| `LET_SEC_090` | Auditoría de operaciones sensibles: esta consulta de solo lectura no requiere auditoría inmutable |
| `LET_OBS_020`, `LET_OBS_030`, `LET_OBS_040`, `LET_OBS_050`, `LET_OBS_060`, `LET_OBS_090`, `LET_OBS_100` | Logs JSON, métricas RED, trazas, alertas, SLOs, sonda sintética y runbook ([LET_OBS](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1277975/LET_OBS+-+Observabilidad+y+Logging)) |
| `LET_NET_070`, `LET_NET_110`, `LET_NET_140` | Salida por Azure Firewall con FQDN aprobado, TLS 1.2+/mTLS on-premise, solicitud de reglas `REQ-CAT-NET-020` si VPLUS no es accesible vía APIM; host y conectividad de la página VPLUS son ficticios ([LET_NET](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/786434/LET_NET+-+Red+y+Network+Security+Groups)) |
| `LET_DAT_040` | Redis: TTL en todas las claves; MBBK en no conformidad (78%) ([LET_DAT](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1114157/LET_DAT+-+Datos+y+Mensajer+a)) |
| `LET_REG_030`, `LET_REG_040`, `LET_REG_060` | DR Tier 1 (RPO 15 min / RTO 4 h): MBBK opera con la excepción EXC-2026-015 (DR parcial hasta 2027-06-30); residencia de datos en `eus2`/`cus` ([LET_REG](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/491552/LET_REG+-+Regiones+Alta+Disponibilidad+y+DR)) |
| `LET_CIC_030`, `LET_CIC_040`, `LET_CIC_050`, `LET_CIC_060`, `LET_CIC_070`, `LET_CIC_080`, `LET_CIC_090` | Calidad, promoción, ventanas, congelamiento, rollback, feature flags y compatibilidad ([LET_CIC](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1048625/LET_CIC+-+CI+CD+y+Gesti+n+de+Cambios)) |

## Risks / Trade-offs
- **Integración productiva no verificada:** el usuario identificó VPLUS como plataforma de crédito, pero el contrato, SLA, acceso y continuidad descritos en la página de ejemplo son ficticios y deben validarse antes de producción.
- **Continuidad:** el módulo de tarjetas de MBBK ya depende del DR parcial de APSA; añadir una segunda dependencia (VPLUS) sin DR documentado amplía el riesgo regional.
- **Capacidad de Redis en MBBK** (78% en pico) y cuota de vCPU (320 vs ~380 requeridas) pueden limitar el crecimiento hasta resolver sus no conformidades.
- **Ventana de cambios:** próximas ventanas y congelamiento de fin de año pueden desplazar el despliegue.
- **Datos de ejemplo:** la documentación indica "datos ficticios con fines de ejemplo"; los valores se toman como referencia de diseño.

## Migration Plan
Sin migración de datos. Despliegue aditivo detrás de feature flag desactivada por defecto; activación progresiva por remote config tras pruebas de contrato, carga y pentest. Rollback: desactivar el flag y/o rollback automático por 5xx > 2% o p95 > 1.5 s en 5 min (`LET_CIC_070`).

## Open Questions
No hay preguntas abiertas para el alcance de simulación. La validación del contrato, acceso, SLA y continuidad reales de VPLUS es un prerrequisito de producción.
