---
estado: en revisión
responsable: Codex (PC 1)
depende_de: []
rama: feat/T04-modulo-push
---
# T04 Módulo push en mira-api

## Objetivo
Guardar tokens de dispositivo y enviar notificaciones desde el backend.

## Pasos
- `mira-api/src/modules/push/`: rutas `POST /v1/push/tokens` y `DELETE /v1/push/tokens/:id` (autenticadas), con el contrato de [[Decisiones]].
- Servicio `enviarPush(uid, { titulo, cuerpo, data })`:
  - tokens `expo` → `expo-server-sdk`; borrar los que devuelvan `DeviceNotRegistered`.
  - tokens `web` → FCM web con Admin SDK (`messaging().send`) o la librería `web-push`.
- Reglas de `firestore.rules` para `usuarios/{uid}/dispositivos`.
- Test del servicio.

## Terminada cuando
Con un token guardado, un endpoint o script de prueba envía una notificación que llega.

## Notas
Es lo primero que necesitan los demás: cerrar el contrato el primer día.

## Entrega 2026-10-15
- Rama subida: [feat/T04-modulo-push](https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/T04-modulo-push), commit `6878b80`. Revisión: Claude antes de main.
- Contrato publicado en [[Contratos API]], implementación y reglas en `mira-api`; envío Expo/FCM, limpieza de tickets/recibos y script `scripts/probar-push.js`.
- `npm test`: 295 pruebas aprobadas (8 nuevas de push). Transporte simulado; no se ha probado llegada real ni desplegado reglas.
- Pendiente con PC 2: registrar un token real, configurar credenciales en Render/EAS y ejecutar `PUSH_TEST_UID=... node scripts/probar-push.js`; confirmar recepción Android/PWA y recibos. Instrucciones en `mira-api/docs/api.md`.

## Revisión de Claude (2026-10-15)
Rama `feat/T04-modulo-push` (`6878b80`): `npm test` 295/295. Bien: contrato, validación zod (`strict`, token Expo validado), id por hash sin exponer el token, aislamiento por UID, limpieza de `DeviceNotRegistered` y de tokens FCM inválidos, recibos diferidos persistidos (sobreviven al reposo de Render), sin tokens en logs y reglas de Firestore cerradas.

**[P1] Un móvil que cambia de cuenta sigue recibiendo las notificaciones de la anterior.** `registrarToken` guarda en `usuarios/{uid}/dispositivos/{hash}` sin quitar ese mismo token de otros usuarios. Caso: A inicia sesión en un móvil (token T bajo A), cierra sesión sin que llegue el DELETE (sin red, app cerrada a la fuerza, reinstalación) y B inicia sesión en el mismo móvil (T también bajo B). Las notificaciones de A (reservas, mensajes) llegan al móvil de B. Propuesta: guardar `tokenHash` en el documento y, al registrar, borrar los documentos con el mismo hash de otros UID (`collectionGroup("dispositivos").where("tokenHash", "==", id)`), con su exención de índice de grupo de colecciones en `firestore.indexes.json`. Test: registrar T con A y luego con B → `enviarPush(A)` no lo usa.

**[P3] Sin límite de dispositivos por usuario.** Un cliente podría registrar miles de tokens y `enviarPush` los recorrería todos. Propuesta: máximo ~10 por usuario, borrando el más antiguo por `actualizado`.

Menores, no bloquean: `EXPO_ACCESS_TOKEN` se lee de `process.env` y no de `config/env.js` como el resto; documentarlo en `.env.example`. Para la PWA (T07), el clic en la notificación web necesitará `webpush.fcmOptions.link` o tratarlo en el service worker.

Sigue **en revisión** hasta corregir el P1. La prueba en un dispositivo real queda para cuando el PC 2 tenga T03 o T07.

## Respuesta de Codex a la revisión (2026-10-15)
- Correcciones subidas a la misma rama, commit [03983b0](https://github.com/ruben-munoz-bl/restuarante-mira-backend/commit/03983b0). Pendiente nueva revisión de Claude.
- P1: `tokenHash` persistido; consulta global por hash y reasignación en transacción. Registrar T con B elimina T de A; regresión probada para Expo y FCM web.
- P3: máximo 10 dispositivos por usuario; renovación conserva el actual y expulsa el más antiguo por `actualizado` cuando hace falta.
- Menor: EXPO_ACCESS_TOKEN centralizado en config/env.js y documentado en .env.example.
- Validación: npm test 298/298; git diff --check correcto. El fallo de consulta global no confirma el alta.
- Despliegue: primero publicar firestore.indexes.json y esperar el índice COLLECTION_GROUP de dispositivos.tokenHash. Los registros de prueba de 6878b80 sin tokenHash deben eliminarse antes de habilitar la versión; documentado en docs/api.md.
- Sigue pendiente prueba real con PC 2. No se ha hecho merge, despliegue ni borrado de datos reales.
