## Context
NEUR-32 pide ayudar al titular autenticado de una tarjeta Visa a organizar sus pagos. La página VPLUS de simulación es documentación ficticia: su contrato devuelve tarjetas, `amountOwed`, `availableCredit`, `currency` y `asOf`; no define fecha de pago ni monto del próximo pago. [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2)

## Goals / Non-Goals
**Goals**
- Mostrar saldo adeudado, crédito disponible, moneda y fecha de corte de la tarjeta del titular.
- No presentar esos datos como próximo pago.

**Non-Goals**
- Fecha o monto de próximo pago, pago de cuotas, notificaciones push, tarjetas adicionales, datos reales y despliegue productivo.
- Analizar o resolver dependencias; corresponde a la fase SDD siguiente.

## Decisions
### D1. Alcance limitado a los datos documentados
La demo usa únicamente los campos del contrato de saldos de VPLUS (`GET /credit-cards/{cardId}/balances`) y no deriva un próximo pago de ellos.
- **Alternativa descartada:** mantener “próximo pago” con un fixture nuevo de fecha de vencimiento y monto; exigiría definir un contrato inexistente.
- **Por qué:** evita inventar campos y mantiene la demo verificable con el contrato documentado.

### D2. Reutilizar el contrato de saldos de NEUR-2
El comportamiento de lectura coincide con `neur-2-consultar-saldo-credito-visa`. Esta demo consume el mismo contrato simulado de solo lectura y no lo modifica.

## Risks / Trade-offs
- El título de la issue menciona “próximo pago”, pero el alcance acordado es saldos; la presentación de la demo debe rotular correctamente los datos.
- Los datos de VPLUS son ficticios y no acreditan contrato, acceso ni operación productiva.

## Migration Plan
No aplica: demo sin despliegue productivo.

## Open Questions
No hay preguntas abiertas.
