---
estado: en revisión
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: [M01, M03]
rama: feat/M04-checkin-puntos
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

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M04-checkin-puntos (`f98a795`), sobre M03/M02.
- QR `r1.` con fallback a VISITAS_SECRET; check-in del día de Madrid y del restaurante dueño; +50 con pedido, una sola transacción y movimiento por reserva. G08 sigue igual.
- Reglas preparadas: reservas solo lectura titular/admin y sin escrituras; cartas sin acceso directo; puntos_movimientos create/update/delete false.
- `FIRESTORE_EMULATOR_HOST=127.0.0.1:8080 npm test`: **446/446, sin omisiones**. Motor real Firestore 1.19.8, Java 21, proyecto local demo-mira-m04; Node 22.23.3. Incluye roles/permisos y escaneos concurrentes.
- Documentación y comandos reproducibles en `mira-api/docs/api.md`. Sin despliegues ni escrituras en producción.
- Claude: revisar antes de unir, orden M02 → M03 → M04. Ramas acumulativas; no unir M04 aisladamente.
- M10 y M11 siguen libres en la bóveda: revisión cruzada de llamadas pendiente hasta que Claude publique código. El OK de Alex a M01 sigue pendiente; cambios de contrato los publica Claude.

## Revisión de Claude · 2026-10-22
**Aprobado** el conjunto M02 → M03 → M04 (`035446c`, `0a88897`, `f98a795`), revisado como una sola rama acumulada contra [[Contratos API]] (M01 y sus precisiones).
- `npm test` en `feat/M04-checkin-puntos`: **442 pasan, 0 fallan, 1 omitido** (el de reglas, que necesita el emulador; Codex lo pasó con emulador real: 446/446).
- Cumple el contrato: carta pública solo con disponibles y 404 sin carta; IDs únicos y precios a 2 decimales; pedido copiado **dentro de la transacción** de la reserva, con `PLATO_NO_DISPONIBLE` y `SIN_CARTA`; QR `r1.` con firma comparada en tiempo constante y 503 sin secreto (el listado sigue con `qr: null`); check-in que vuelve a comprobar el restaurante dentro de la transacción, marca `presentadaEn` sin tocar `estado` y da +50 con un marcador `pedido_anticipado_<id>` que impide duplicar; reglas cerradas para `reservas`, `cartas` y `puntos_movimientos`.
- Errores: el `errorHandler` devuelve `err.code`, así que `QR_USADO`, `OTRO_RESTAURANTE`, `FUERA_DE_FECHA`… salen como dice el contrato.
- **P2 · Listado del restaurante:** `delRestaurante` lee **todas** las reservas del restaurante (sin filtro de fecha en la consulta) y filtra en memoria, y luego hace una lectura de perfil por cliente. Con pocos datos va bien; con miles de reservas será lento. No bloquea; anotado para más adelante (filtrar `fecha` en la consulta con índice).
- **P3 · Cancelar tras llegar:** `cancelarReserva` no mira `presentadaEn`: una reserva ya presentada se puede cancelar. Propuesta: 409 `CONFLICT` si `presentadaEn`. No bloquea.
- Falta antes de producción: unir a `main` del backend, desplegar reglas y API y configurar `RESERVAS_QR_SECRET` en Render. **No se ha unido ni desplegado**: lo decide Izan.
