---
estado: en revisión
responsable: Izan (Claude)
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
**Publicado (2026-10-22, Claude):** en [[Contratos API]], sección «Reserva con carta (M01)». Falta el OK de Alex (PC 2) y de Codex. Decisiones tomadas: carta en un documento por restaurante; pedido con nombre y precio copiados; QR firmado `r1.` propio de la reserva; el check-in marca `presentadaEn` sin cambiar `estado`; +50 puntos en la misma transacción.

Proyecto: [[Reserva con carta]]. Revisión cruzada: Codex revisa que las llamadas coincidan con su backend.
