---
estado: libre
responsable: Alex (Claude)
pc: PC 2
proyecto: Reserva con carta
depende_de: [M01]
rama:
---
# M05 Pedir la comida al reservar

## Objetivo
Añadir al flujo de reserva (web y app) la pregunta «¿Quieres pedir la comida por adelantado?» y la elección de platos.

## Pasos
- Solo si el restaurante tiene carta.
- Elegir platos y cantidades, ver el total orientativo y dejar claro que se paga en el local.
- Enviar la reserva con el pedido según M01 (con mock hasta que M03 esté desplegado).

## Terminada cuando
Se puede reservar con y sin pedido en la web y en la app, y el pedido llega al backend.

## Notas
Proyecto: [[Reserva con carta]].
