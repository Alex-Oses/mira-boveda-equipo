---
estado: en curso
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

## Reparto 2026-10-23
Web (panel): Claude. App y mock de `/checkin` y `/restaurante`: Codex ([[Codex - Sesión 2026-10-23]]). @Alex: visto, el pedido se mostrará al escanear en web y app.

## Mock listo para Claude · Codex · 2026-10-23
@Claude: tarea 2 terminada y subida en [feat/M11-mock-y-app](https://github.com/ruben-munoz-bl/restaurante-mira-frontend/tree/feat/M11-mock-y-app), commit `ae30364` (base main `0aedc23`, Alex ya unido). Puedes tomar ese commit; no incluye cambios del panel web ni de `src/models/escanerModel.js`.
- Arranque: `MOCK_RESERVAS_EJEMPLO=1 npm run mock-api` y `npm run dev:mock`; panel `demo-panel`. Para reservar desde catálogo: añade `MOCK_RESTAURANTE_ID=1` al arranque del mock (el panel pasa al restaurante 1).
- Pega `r1.demo-pedido.mock` (+50, 2 croquetas, total19), `r1.demo-sin-pedido.mock` (0), `r1.demo-usada.mock`, `r1.demo-cancelada.mock`, `r1.demo-fuera-fecha.mock`, `r1.demo-otro-restaurante.mock`, `r1.no-existe.mock` o una firma falsa.
- Check-in y listado con forma M01 exacta, fechas Madrid, rango inclusivo≤31 días y sin uid/email. Bearer requerido pero simulado: no valida Firebase ni roles. Firma `.mock`, nunca válida en producción.
- El check-in consume el QR; reinicia para repetir. Detalle en `docs/M11-pruebas-mock.md`. `npm test`331/331 y build correctos. App en curso (tarea3).
