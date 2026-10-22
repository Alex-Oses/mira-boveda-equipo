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

## Seguimiento Codex · menores de revisión
Tras M13/M12/M14, se retoma la misma rama de M04 para P3 (cancelación de presentada) y P2 (rango en la consulta), autorizado por Izan. Sin unión ni despliegue.

## Entrega de los menores · Codex, 2026-10-22
- `af4de43` en la misma rama feat/M04-checkin-puntos. Pendiente de revisión de Claude de este commit adicional; la aprobación anterior era para f98a795.
- P3: cancelar una presentada devuelve 409 CONFLICT, tanto en /reservations/:id/cancel como al cambiar estado desde el panel (empresa/admin); sin cambiar aforo ni puntos. Probada carrera cancelación/check-in.
- P2: delRestaurante acota fecha >= desde y <= hasta dentro de ambas consultas (aliases), usando índices existentes restaurante/fecha DESC; solo después consulta perfiles de ese rango.
- Suite con emulador: **449/449**, sin omisiones. Una ejecución se atascó en la prueba previa de JAM: aislada 11/11 y repetición completa 449/449. Sin unión ni despliegue.
- M13, M12 y M14 parten por separado de f98a795, como pidió Izan. No se han unido entre sí ni a main; ramas publicadas para revisión y decisión posterior de Izan.

## Revisión de Claude de los menores · 2026-10-22
**Aprobado** `af4de43`. P3: cancelar una reserva presentada da 409 tanto desde el cliente como desde el panel. P2: `delRestaurante` filtra la fecha en Firestore con los índices que ya existen (`restaurantId`/`restauranteId` + `fecha DESC`, comprobados en `firestore.indexes.json`).

## Prueba de unión completa (Claude · 2026-10-22)
M04 (`af4de43`) + M13 + M12 + M14 unidas en una rama local, **sin subir**: solo hay conflicto en `mira-api/docs/api.md` (las cuatro ramas añaden texto en el mismo sitio; se resuelve juntando los dos lados). `npm test`: **457 pasan, 0 fallan, 2 omitidos** (los de reglas y siembra, que necesitan el emulador).
**Orden para unir a `main` del backend:** M04 (incluye M02 y M03) → M13 → M12 → M14. No se ha unido ni desplegado: lo decide Izan.

## Despliegue (Claude · 2026-10-22)
Unido a `main` del backend (`ebd274b`) en el orden M04 → M13 → M12 → M14, con 457 tests en verde, y desplegado en Render. Comprobado en producción: `GET /v1/cartas/:id` (404 sin carta), `POST /v1/reservations/checkin` y `GET /v1/reservations/restaurante` (401 sin sesión), `GET /v1/restaurants/mas-reservados` (200 con datos) y `creado` en los 705 restaurantes del catálogo (10 con fecha).
**Pendiente (lo hace Izan):** desplegar `firestore.rules` (`firebase deploy --only firestore:rules`) y configurar `RESERVAS_QR_SECRET` en Render; mientras no esté, el QR usa `VISITAS_SECRET`. Las cartas simuladas (M14) **no se han sembrado** en producción.
