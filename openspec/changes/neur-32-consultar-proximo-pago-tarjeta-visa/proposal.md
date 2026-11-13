# Change: Consultar saldo y crédito de una tarjeta Visa en la demo (NEUR-32)

Jira: [NEUR-32](https://bcp-neuroforge.atlassian.net/browse/NEUR-32)

## Why
El titular autenticado de una tarjeta Visa quiere organizar sus pagos sin llamar al Contact Center. El contrato de la demo VPLUS no documenta fecha ni monto del próximo pago, solo saldos y fecha de corte; por decisión acordada, la demo se ajusta a esos datos documentados.

## What Changes
- Nueva capability `credit-card-balance-demo`: consulta de saldo adeudado, crédito disponible, moneda y fecha-hora de corte de una tarjeta Visa del titular.
- La demo no muestra fecha ni monto de próximo pago, ni presenta el saldo o la fecha de corte como tales.
- Acceso solo para el titular, sin PAN/CVV/PIN, y degradación controlada con 503.
- Fuera de alcance: tarjetas adicionales, pago de cuotas, notificaciones push, datos reales, cambios al contrato productivo de VPLUS y despliegue productivo.

## Impact
- Capabilities afectadas: `credit-card-balance-demo` (nueva).
- Componentes/sistemas: MBBK móvil y la simulación VPLUS de solo lectura — [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2).
- Solapamiento: el comportamiento de lectura de saldos coincide con el cambio `neur-2-consultar-saldo-credito-visa`; esta demo reutiliza su contrato de saldos y no lo modifica.
- Código/áreas esperadas: por definir en el diseño de implementación.

## Referencias
- [VPLUS - Plataforma de Tarjetas de Crédito (SIMULACION NEUR-2)](https://bcp-neuroforge.atlassian.net/wiki/spaces/NEUR/pages/1540128/VPLUS+-+Plataforma+de+Tarjetas+de+Credito+SIMULACION+NEUR-2) — endpoints, campos de saldo, errores y reglas de seguridad; todo ficticio y solo para demo.
