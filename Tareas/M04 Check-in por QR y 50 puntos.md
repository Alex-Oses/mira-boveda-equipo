---
estado: libre
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: [M01, M03]
rama:
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
