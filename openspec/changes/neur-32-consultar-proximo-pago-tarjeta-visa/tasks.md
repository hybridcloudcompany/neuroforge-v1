## 0. Prerequisitos (dependencias)
- Ninguno: el análisis de dependencias (2026-11-13) no identificó dependencias pendientes para la demo; ver `dependencies.md`.

## 1. Consulta de saldo
- [ ] 1.1 Exponer en la demo el saldo adeudado, el crédito disponible, la moneda y la fecha-hora de corte de la tarjeta Visa del titular. — NEUR-34
- [ ] 1.2 Rotular los datos como saldo y fecha de corte, sin mostrar fecha ni monto de próximo pago. — NEUR-33
- [ ] 1.3 Mostrar un par de importes por moneda sin conversión. — NEUR-38

## 2. Seguridad y errores
- [ ] 2.1 Validar la titularidad en cada consulta y responder 404 sin revelar existencia; 401 sin sesión válida. — NEUR-36
- [ ] 2.2 Identificar la tarjeta solo por identificador y últimos 4 dígitos, sin PAN/CVV/PIN/tokens en respuestas ni logs. — NEUR-37
- [ ] 2.3 Responder 503 `problem+json` si la fuente no está disponible, manteniendo el resto del canal operativo. — NEUR-35

## 3. Verificación de la demo
- [ ] 3.1 Probar los escenarios de la spec con datos ficticios de VPLUS. — NEUR-39
