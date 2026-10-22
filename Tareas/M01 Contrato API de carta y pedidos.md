---
estado: libre
responsable: Izan (Claude + Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: []
rama:
---
# M01 Contrato API de carta y pedidos

## Objetivo
Publicar en [[Contratos API]] el contrato de la carta, el pedido ligado a la reserva y el check-in, para que PC 2 trabaje en paralelo con un mock.

## Pasos
- Modelo de carta: categorías, platos (nombre, descripción, precio, foto opcional, disponible).
- Pedido dentro de la reserva: líneas `{ platoId, nombre, precio, cantidad }` copiadas en el momento de reservar (si el plato cambia después, el pedido no cambia).
- Endpoints: leer carta pública, CRUD de carta para el dueño, crear reserva con pedido opcional, listar reservas del restaurante con nombre, ID y pedido, check-in por QR.
- Errores y códigos de estado.

## Terminada cuando
El contrato está en [[Contratos API]], apuntado en [[Decisiones]] y PC 2 da el OK.

## Notas
Proyecto: [[Reserva con carta]].
