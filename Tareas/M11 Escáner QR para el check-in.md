---
estado: libre
responsable: Izan (Claude)
pc: PC 1
proyecto: Reserva con carta
depende_de: [M04]
rama:
---
# M11 Escáner QR para el check-in

## Objetivo
Conectar el escáner que ya existe ([[G10 Escáner de QR y aviso de llegada]]) al check-in de la reserva.

## Pasos
- Al escanear: confirmar la llegada, mostrar nombre y pedido para que la cocina empiece.
- Errores: QR caducado, ya usado o de otro restaurante.
- En la web del panel y en la app del restaurante (`mobile/`).

## Terminada cuando
Escaneando el QR de una reserva con pedido se marca presentada, se ve el pedido y el cliente recibe sus 50 puntos.

## Notas
Proyecto: [[Reserva con carta]]. Revisión cruzada: Codex revisa que las llamadas coincidan con su backend.

## Aviso de Alex (PC 2) · 2026-10-23
@Izan: el lado del cliente ya está (rama `feat/M05-M06-M09-reserva-carta`): Mis reservas enseña el QR `r1.…` y el pedido. Falta M11 para que el restaurante, al escanear, **vea en pantalla el nombre del cliente y los platos** (`reserva.pedido` de la respuesta de `POST /v1/reservations/checkin`), en la web del panel y en la app. Para probar sin backend, el mock (`scripts/mock-api.mjs`) devuelve `qr: r1.<id>.mock` pero aún no tiene `/checkin`.
