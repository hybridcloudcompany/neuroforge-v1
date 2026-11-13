## 0. Prerequisitos (dependencias)
- [x] 0.1 D1 — Validar contrato y operación real de VPLUS — NEUR-8 (Finalizada)
- [x] 0.2 D2 — Habilitar identidad, APIM y conectividad hacia VPLUS — NEUR-10 (Finalizada)
- [x] 0.3 D3 — Ampliar Redis de MBBK (P1 a P2) para la caché de saldos — NEUR-9 (Finalizada)

## 1. Definición y contrato
- [ ] 1.1 Antes de producción, validar con el responsable de VPLUS si expone el saldo y sustituir el contrato OpenAPI, SLA, volumen y canal ficticios por datos aprobados — NEUR-13
- [ ] 1.2 Definir el contrato aditivo de MBBK `v8` (listado de tarjetas de crédito y saldo por tarjeta) con `value`+`currency`, `problem+json` y códigos `MBBK-CARD-*` — NEUR-12
- [ ] 1.3 Actualizar el catálogo de APIs y las secciones "Alcance"/"Fuera de alcance" en Confluence tras la aprobación (solo el equipo dueño; este flujo no edita Confluence) — NEUR-11

## 2. Backend de canal (`mbbk-cards-svc`)
- [ ] 2.1 Implementar el adaptador de lectura hacia la fuente de crédito con timeout, 1 reintento en lecturas y circuit breaker — NEUR-14
- [ ] 2.2 Validar la titularidad de la tarjeta en cada consulta y devolver 404 sin revelar existencia — NEUR-15
- [ ] 2.3 Responder error de negocio para tarjetas que no son de crédito y aplicar las reglas definidas para cada estado — NEUR-16
- [ ] 2.4 Cachear con TTL ≤ 30 s en Redis y verificar que todas las claves tienen TTL — NEUR-17
- [ ] 2.5 Enmascarar PAN (últimos 4) y verificar que no hay PAN/CVV/PIN/tokens en logs, trazas ni eventos — NEUR-22

## 3. BFF y configuración remota
- [ ] 3.1 Agregar la ruta en `mbbk-bff-gateway` que orquesta la pantalla de saldo — NEUR-18
- [ ] 3.2 Añadir el flag de habilitación en `mbbk-config-svc` (desactivado por defecto) — NEUR-23

## 4. App móvil (iOS y Android)
- [ ] 4.1 Nueva pantalla de selección de tarjeta de crédito y de saldo (saldo adeudado, crédito disponible, moneda, fecha de corte) — NEUR-21
- [ ] 4.2 Manejo de estados de error y de funcionalidad deshabilitada (mensaje de reintento ante 503) — NEUR-19
- [ ] 4.3 Accesibilidad WCAG 2.1 AA, español es-PE, compatibilidad iOS 15+/Android 8+ — NEUR-20

## 5. Observabilidad y operación
- [ ] 5.1 Métricas RED, logs JSON con `correlationId` y trazas con `traceparent` — NEUR-24
- [ ] 5.2 Alertas, sonda sintética y runbook del endpoint (N1/N2/N3) — NEUR-26

## 6. Pruebas y calidad
- [ ] 6.1 Pruebas unitarias ≥ 80% y escenarios del spec (titularidad, 401, 404, 503, circuit breaker, flag) — NEUR-25
- [ ] 6.2 Pruebas de contrato Pact con 100% de los contratos existentes sin cambios — NEUR-28
- [ ] 6.3 Prueba de carga al 150% del pico durante 30 min y resistencia de 4 h al 80% — NEUR-31
- [ ] 6.4 SAST/SCA/DAST sin hallazgos críticos/altos y pentest previo a PRD — NEUR-29

## 7. Liberación
- [ ] 7.1 Promoción DEV → QA → PRD con CAB, en ventana martes/jueves 22:00-02:00 (Lima) y fuera de congelamiento — NEUR-27
- [ ] 7.2 Activación gradual del flag y verificación de SLO (p95 ≤ 900 ms, 5xx < 0.3%) — NEUR-30
