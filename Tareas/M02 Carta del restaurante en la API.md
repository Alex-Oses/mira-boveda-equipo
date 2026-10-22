---
estado: en revisión
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
