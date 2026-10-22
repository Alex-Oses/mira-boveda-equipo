---
estado: en revisión
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: [M01, M03]
rama: feat/M04-checkin-puntos
---
# M04 Check-in por QR y 50 puntos

## Objetivo
Al escanear el QR de la reserva, marcarla como presentada y dar +50 puntos si llevaba pedido.

## Pasos
- Reaprovechar el QR firmado de [[G08 QR firmado, visitas y comisión]] o uno equivalente para la reserva (decidir en M01).
- Estado de la reserva a «presentada», una sola vez.
- +50 puntos solo si había pedido, sin duplicarlos si se escanea dos veces.

## Terminada cuando
Escanear una reserva con pedido la marca presentada y suma 50 puntos una sola vez; sin pedido, no suma.

## Notas
Proyecto: [[Reserva con carta]]. Revisión cruzada: Claude revisa el código antes de unirlo.

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M04-checkin-puntos (`f98a795`), sobre M03/M02.
- QR `r1.` con fallback a VISITAS_SECRET; check-in del día de Madrid y del restaurante dueño; +50 con pedido, una sola transacción y movimiento por reserva. G08 sigue igual.
- Reglas preparadas: reservas solo lectura titular/admin y sin escrituras; cartas sin acceso directo; puntos_movimientos create/update/delete false.
- `FIRESTORE_EMULATOR_HOST=127.0.0.1:8080 npm test`: **446/446, sin omisiones**. Motor real Firestore 1.19.8, Java 21, proyecto local demo-mira-m04; Node 22.23.3. Incluye roles/permisos y escaneos concurrentes.
- Documentación y comandos reproducibles en `mira-api/docs/api.md`. Sin despliegues ni escrituras en producción.
- Claude: revisar antes de unir, orden M02 → M03 → M04. Ramas acumulativas; no unir M04 aisladamente.
- M10 y M11 siguen libres en la bóveda: revisión cruzada de llamadas pendiente hasta que Claude publique código. El OK de Alex a M01 sigue pendiente; cambios de contrato los publica Claude.
