---
estado: hecho
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: [M01]
rama: feat/M02-carta
---
# M02 Carta del restaurante en la API

## Objetivo
Guardar y servir la carta de cada restaurante.

## Pasos
- Colección y reglas (solo el dueño o el admin editan; lectura pública).
- Endpoints según M01.
- Tests.

## Terminada cuando
Se crea, edita y lee una carta con los endpoints del contrato, con tests en verde.

## Notas
Proyecto: [[Reserva con carta]]. Revisión cruzada: Claude revisa el código antes de unirlo.

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M02-carta (`035446c`).
- Carta pública/editor/PUT, límites, precios e IDs únicos. `npm test`: 424/424.
- Claude: revisar antes de unir. Reglas se cierran y prueban con M04; sin despliegue.

## Revisión de Claude · 2026-10-22
**Aprobado**: revisado junto con M03 y M04, detalle en [[M04 Check-in por QR y 50 puntos]].

## Despliegue (Claude · 2026-10-22)
Unido a `main` del backend (`ebd274b`) en el orden M04 → M13 → M12 → M14, con 457 tests en verde, y desplegado en Render. Comprobado en producción: `GET /v1/cartas/:id` (404 sin carta), `POST /v1/reservations/checkin` y `GET /v1/reservations/restaurante` (401 sin sesión), `GET /v1/restaurants/mas-reservados` (200 con datos) y `creado` en los 705 restaurantes del catálogo (10 con fecha).
**Pendiente (lo hace Izan):** desplegar `firestore.rules` (`firebase deploy --only firestore:rules`) y configurar `RESERVAS_QR_SECRET` en Render; mientras no esté, el QR usa `VISITAS_SECRET`. Las cartas simuladas (M14) **no se han sembrado** en producción.
