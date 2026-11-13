## ADDED Requirements

### Requirement: Consulta de saldo y crédito de una tarjeta Visa en la demo
The system SHALL permitir que el cliente autenticado titular de una tarjeta de crédito Visa consulte, en el entorno de demostración, el saldo adeudado, el crédito disponible, la moneda y la fecha-hora de corte de esa tarjeta, para orientar sus pagos sin llamar al Contact Center.

Cada importe SHALL mostrarse con su código de moneda ISO 4217 y sin conversión. El sistema SHALL NOT presentar el saldo adeudado ni la fecha de corte como fecha o monto del próximo pago, ni mostrar una fecha o un monto de próximo pago, porque el contrato de la demo no los documenta. Quedan fuera de alcance las tarjetas adicionales, el pago de cuotas, las notificaciones push, los datos reales, los cambios al contrato productivo de VPLUS y cualquier despliegue productivo.

_Trazabilidad: NEUR-32 (historia y fuera de alcance); [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) (`GET /credit-cards/{cardId}/balances`: `amountOwed`, `availableCredit`, `currency`, `asOf`). Datos ficticios, solo para demo._

#### Scenario: Consulta exitosa
- **WHEN** un cliente autenticado titular de una tarjeta Visa consulta su saldo en la demo
- **THEN** el sistema muestra el saldo adeudado y el crédito disponible de esa tarjeta, cada uno con su moneda
- **AND** muestra la fecha y hora de corte del dato

#### Scenario: No se muestra un próximo pago
- **WHEN** el cliente consulta su tarjeta en la demo
- **THEN** el sistema no muestra fecha ni monto de próximo pago
- **AND** el saldo adeudado y la fecha de corte se rotulan como tales

#### Scenario: Tarjeta con varias monedas
- **WHEN** la tarjeta tiene saldos en más de una moneda
- **THEN** el sistema muestra un par de saldo adeudado y crédito disponible por cada moneda, sin convertir importes

### Requirement: Acceso restringido al titular y protección de datos de tarjeta
The system SHALL entregar el saldo únicamente al titular autenticado, validando la titularidad en cada consulta, y SHALL NOT exponer ni registrar PAN completo, CVV, PIN ni tokens; la tarjeta SHALL identificarse solo por su identificador y sus últimos cuatro dígitos.

_Trazabilidad: [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) (reglas de seguridad y errores de ejemplo)._

#### Scenario: Tarjeta de otro cliente
- **WHEN** un cliente autenticado solicita el saldo de una tarjeta que no le pertenece
- **THEN** el sistema responde 404 con `application/problem+json` sin revelar si la tarjeta existe

#### Scenario: Sin sesión válida
- **WHEN** se solicita el saldo sin un token válido
- **THEN** el sistema responde 401 con `application/problem+json` y no devuelve importes

#### Scenario: Tarjeta enmascarada
- **WHEN** el sistema responde una consulta de saldo
- **THEN** la tarjeta se identifica solo por su identificador y los últimos cuatro dígitos, sin PAN completo

### Requirement: Degradación controlada de la fuente de la demo
The system SHALL responder 503 con `application/problem+json` cuando la fuente de saldos de la demo no esté disponible, sin stack trace ni mensajes internos, y SHALL mantener operativas las demás funciones del canal.

_Trazabilidad: [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) (error `VPLUS-SVC-503`; resto de MBBK disponible)._

#### Scenario: Fuente no disponible
- **WHEN** la fuente de saldos de la demo no responde o informa indisponibilidad
- **THEN** el sistema responde 503 con `application/problem+json`
- **AND** el cliente puede seguir usando el resto de las funciones del canal
