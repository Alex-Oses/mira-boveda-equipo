---
estado: hecho
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: [M01, M02]
rama: feat/M03-pedido-reserva
---
# M03 Pedido ligado a la reserva

## Objetivo
Que una reserva pueda llevar un pedido de la carta, fijo desde que se reserva.

## Pasos
- Aceptar el pedido opcional al crear la reserva y validar que los platos son de ese restaurante.
- El pedido no se edita después.
- El cliente lo ve en sus reservas; el restaurante en las suyas (nombre, ID y pedido).

## Terminada cuando
Una reserva con pedido se crea, el cliente y el restaurante la leen con el pedido, y un pedido con platos de otro restaurante se rechaza.

## Notas
Proyecto: [[Reserva con carta]]. Revisión cruzada: Claude revisa el código antes de unirlo.

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M03-pedido-reserva (sobre M02).
- Pedido copiado en transacción, rechazo de platos ajenos/agotados, listado privado del restaurante. `npm test`: 431/431.
- Claude: revisar antes de unir, primero M02. Reglas y QR en M04; sin despliegue.

## Revisión de Claude · 2026-10-22
**Aprobado**: revisado junto con M03 y M04, detalle en [[M04 Check-in por QR y 50 puntos]].

## Despliegue (Claude · 2026-10-22)
Unido a `main` del backend (`ebd274b`) en el orden M04 → M13 → M12 → M14, con 457 tests en verde, y desplegado en Render. Comprobado en producción: `GET /v1/cartas/:id` (404 sin carta), `POST /v1/reservations/checkin` y `GET /v1/reservations/restaurante` (401 sin sesión), `GET /v1/restaurants/mas-reservados` (200 con datos) y `creado` en los 705 restaurantes del catálogo (10 con fecha).
**Pendiente (lo hace Izan):** desplegar `firestore.rules` (`firebase deploy --only firestore:rules`) y configurar `RESERVAS_QR_SECRET` en Render; mientras no esté, el QR usa `VISITAS_SECRET`. Las cartas simuladas (M14) **no se han sembrado** en producción.
