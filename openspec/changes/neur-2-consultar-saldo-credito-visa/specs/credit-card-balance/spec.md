## ADDED Requirements

### Requirement: Consulta de saldo adeudado y crédito disponible
The system SHALL permitir al cliente titular autenticado consultar, para una de sus tarjetas de crédito, el saldo adeudado y el crédito disponible, ambos con su importe decimal y su moneda.

La funcionalidad se entrega solo en MBBK móvil (`MOBILE`), detrás de un feature flag. Aplica a tarjetas de todas las marcas del titular; no incluye tarjetas adicionales. La respuesta contiene deuda total a fecha de corte, crédito disponible y fecha-hora de corte, sin línea total, pago mínimo ni fechas de pago/cierre. Se devuelve un par de importes por cada moneda habilitada, sin conversión, con importe decimal y código ISO 4217. La consulta se hace en línea y cualquier caché tiene TTL máximo de 30 segundos.

_Trazabilidad: NEUR-2; decisiones aceptadas sobre canal, titularidad, datos, monedas y frescura._

#### Scenario: Consulta exitosa
- **WHEN** un cliente titular autenticado solicita el saldo de una de sus tarjetas de crédito
- **THEN** el sistema responde con el saldo adeudado y el crédito disponible de esa tarjeta, cada uno con valor decimal y código de moneda
- **AND** la respuesta incluye la fecha y hora de corte del dato

#### Scenario: Tarjeta sin deuda
- **WHEN** el titular consulta una tarjeta de crédito cuyo saldo adeudado es cero
- **THEN** el sistema responde saldo adeudado igual a 0 con su moneda y el crédito disponible vigente

#### Scenario: Tarjeta no activada
- **WHEN** el titular consulta una tarjeta de crédito que todavía no está activada
- **THEN** el sistema responde 409 con `application/problem+json` y no devuelve saldos

#### Scenario: Tarjeta cancelada, vencida o bloqueada definitivamente con deuda
- **WHEN** el titular consulta una tarjeta cancelada, vencida o bloqueada definitivamente que mantiene deuda
- **THEN** el sistema responde la deuda adeudada y crédito disponible igual a 0

#### Scenario: Tarjeta bloqueada temporalmente
- **WHEN** el titular consulta una tarjeta bloqueada temporalmente
- **THEN** el sistema responde la deuda y el crédito disponible vigentes

### Requirement: Acceso restringido al titular y a sus tarjetas
The system SHALL entregar el saldo y el crédito disponible únicamente al titular autenticado de la tarjeta y SHALL validar la titularidad de la tarjeta solicitada en cada consulta, no solo la validez de la sesión.

_Trazabilidad: NEUR-2 ("cliente titular"); [CNF - Seguridad y Configuración](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474608/CNF+-+Seguridad+y+Configuraci+n) §6 ("valida titularidad/propiedad del recurso"); [APSA - Seguridad y Cumplimiento](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1277953/APSA+-+Seguridad+y+Cumplimiento) §2 (validación de titularidad en cada operación)._

#### Scenario: Tarjeta de otro cliente
- **WHEN** un cliente autenticado solicita el saldo de una tarjeta que no le pertenece
- **THEN** el sistema responde 404 con `application/problem+json` indicando "tarjeta no encontrada o no pertenece al cliente"
- **AND** no revela si la tarjeta existe ni ningún dato de ella

#### Scenario: Solicitud sin sesión válida
- **WHEN** se solicita el saldo sin un token de acceso válido o con uno expirado
- **THEN** el sistema responde 401 con `application/problem+json` y no devuelve ningún importe

#### Scenario: Tarjeta que no es de crédito
- **WHEN** el titular solicita el saldo de una tarjeta de débito mediante esta consulta
- **THEN** el sistema responde un error de negocio `application/problem+json` (no un importe) indicando que la operación no aplica al tipo de tarjeta

### Requirement: Protección de datos de tarjeta
The system SHALL NOT exponer, persistir ni registrar en logs, trazas o eventos el número completo de la tarjeta (PAN), el CVV, el PIN ni claves; la tarjeta SHALL identificarse en respuestas solo por un identificador interno y los últimos 4 dígitos.

_Trazabilidad: `LET_SEC_080`; [APSA - Visión General](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1245185/APSA+-+Visi+n+General+y+Descripci+n+Funcional) regla 3; [MBBK - Decisiones](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/589826/MBBK+-+Decisiones+Restricciones+y+Roadmap) ADR-004 (MBBK no almacena datos de tarjeta)._

#### Scenario: Respuesta con tarjeta enmascarada
- **WHEN** el sistema responde una consulta de saldo
- **THEN** la respuesta identifica la tarjeta solo con su identificador interno y los últimos 4 dígitos, sin PAN completo

#### Scenario: Registro de la consulta
- **WHEN** se procesa una consulta de saldo
- **THEN** los logs y trazas de la consulta no contienen PAN, CVV, PIN ni tokens de acceso, y los datos personales del cliente aparecen enmascarados

### Requirement: Degradación controlada ante falla de la fuente de datos
The system SHALL responder con un error 503 `application/problem+json` con código propio del canal cuando la fuente de saldo de tarjetas de crédito no esté disponible o exceda el tiempo de espera, y SHALL mantener operativas el resto de las funcionalidades del canal. En el escenario VPLUS de simulación, el timeout es de 3 segundos y se permite un único reintento solo para solicitudes GET.

_Trazabilidad: [MBBK - Catálogo de Microservicios](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1507329/MBBK+-+Cat+logo+de+Microservicios) §5 (APSA no disponible: circuit breaker, el resto de la app opera); [MBBK - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1638401/MBBK+-+Cat+logo+de+APIs) §5 (`MBBK-CARD-050`). Los parámetros específicos de VPLUS son ficticios y aplican solo a la simulación._

#### Scenario: Fuente no disponible
- **WHEN** la fuente de datos de tarjetas de crédito no responde dentro del tiempo de espera o devuelve un error de disponibilidad
- **THEN** el sistema responde 503 con `application/problem+json` y un código de error con el prefijo del canal, sin stack trace ni mensajes internos
- **AND** el cliente puede seguir usando el resto de las funcionalidades del canal

#### Scenario: Circuit breaker abierto
- **WHEN** el circuit breaker hacia la fuente de datos está abierto
- **THEN** el sistema responde 503 de inmediato, sin invocar a la fuente, y el mensaje al cliente invita a reintentar más tarde

### Requirement: Habilitación remota de la funcionalidad
The system SHALL permitir habilitar o deshabilitar la consulta de saldo de tarjeta de crédito mediante configuración remota, sin requerir un nuevo release de la aplicación móvil ni un nuevo despliegue del backend.

_Trazabilidad: [MBBK - Decisiones](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/589826/MBBK+-+Decisiones+Restricciones+y+Roadmap) ADR-006 (remote config para degradar módulos); `LET_CIC_080` (feature flags para funcionalidades de riesgo)._

#### Scenario: Funcionalidad deshabilitada
- **WHEN** la configuración remota marca la funcionalidad como deshabilitada
- **THEN** la aplicación no muestra la opción de consulta de saldo de tarjeta de crédito
- **AND** el backend rechaza las consultas con un error de funcionalidad no disponible `application/problem+json`

#### Scenario: Funcionalidad habilitada
- **WHEN** la configuración remota marca la funcionalidad como habilitada
- **THEN** el cliente titular ve la opción y puede consultar el saldo de sus tarjetas de crédito

### Requirement: Observabilidad de la consulta
The system SHALL emitir, para cada consulta de saldo, logs JSON estructurados con `correlationId`, métricas de tasa, errores y duración por endpoint, y trazas distribuidas que propaguen `x-correlation-id` y `traceparent`.

_Trazabilidad: `LET_OBS_020`, `LET_OBS_030`, `LET_OBS_040`; [MBBK - Requisitos No Funcionales](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474561/MBBK+-+Requisitos+No+Funcionales+SLOs+y+Capacidad) §6._

#### Scenario: Correlación extremo a extremo
- **WHEN** la app envía una consulta con `x-correlation-id`
- **THEN** el mismo identificador aparece en los logs de cada componente que atiende la consulta y en la respuesta de error, si la hubiera

#### Scenario: Métricas disponibles
- **WHEN** se atienden consultas de saldo en PRD
- **THEN** existen métricas de tasa, tasa de error y duración (p95) del endpoint, visibles en el panel de salud del módulo de tarjetas

### Requirement: Latencia de la pantalla de saldo
The system SHALL cumplir una latencia p95 menor o igual a 900 ms para la pantalla de consulta de saldo de tarjeta de crédito en PRD.

_Trazabilidad: [MBBK - Requisitos No Funcionales](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1474561/MBBK+-+Requisitos+No+Funcionales+SLOs+y+Capacidad) §1 (p95 pantallas ≤ 900 ms); `LET_WLD_150` (prueba de carga al 150% del pico). El p95 de VPLUS de 600 ms es ficticio, no un objetivo aprobado para producción._

#### Scenario: Medición de latencia
- **WHEN** se mide la latencia de la consulta de saldo durante la prueba de carga previa a producción al 150% del pico esperado durante 30 minutos
- **THEN** el p95 es menor o igual a 900 ms y no hay errores 5xx

### Requirement: Integración VPLUS para la simulación
Para la simulación NEUR-2, el sistema SHALL usar VPLUS como fuente de tarjetas de crédito, con integración REST de solo lectura por APIM, OAuth 2.0 client credentials y scope `vplus.credit-cards.read`. El escenario usa `GET /customers/{customerId}/credit-cards` para listar tarjetas y `GET /credit-cards/{cardId}/balances` para consultar saldos.

Los valores de capacidad simulados son 2.4 millones de consultas al mes, 10 TPS de pico, 14 TPS de capacidad diseñada y prueba a 15 TPS durante 30 minutos; la disponibilidad VPLUS es 99.95% y su p95 es 600 ms. Estos datos y el contrato de la página [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) son ficticios, no medidos ni aprobados y no constituyen evidencia productiva. Antes de producción deben validarse con los responsables reales de VPLUS.

La autenticación usa la sesión/JWT existente; no se requiere token digital adicional para esta consulta. Se emiten logs y métricas correlacionados, sin registrar secretos ni tratar la consulta como una operación sensible con auditoría inmutable.

_Trazabilidad: decisiones aceptadas de NEUR-2 y página VPLUS de simulación; las afirmaciones específicas de VPLUS son solo para el escenario ficticio._

### Requirement: Compatibilidad con contratos existentes
The system SHALL introducir la consulta de saldo de tarjeta de crédito sin modificar de forma incompatible los contratos publicados de APSA `v1` ni de MBBK `v8`, y las apps ya instaladas SHALL seguir funcionando sin cambios.

_Trazabilidad: `LET_CIC_090`; [APSA - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1146881/APSA+-+Cat+logo+de+APIs) §6; [MBBK - Catálogo de APIs](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1638401/MBBK+-+Cat+logo+de+APIs) §6._

#### Scenario: App instalada sin la nueva funcionalidad
- **WHEN** una versión previa de la app consume los endpoints de tarjetas existentes después del despliegue
- **THEN** las respuestas de los endpoints existentes no cambian en estructura ni en semántica

#### Scenario: Pruebas de contrato
- **WHEN** se ejecutan las pruebas de contrato (Pact) con los consumidores en QA
- **THEN** el 100% de los contratos existentes pasa sin cambios
