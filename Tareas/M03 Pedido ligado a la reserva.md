---
estado: en revisión
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
