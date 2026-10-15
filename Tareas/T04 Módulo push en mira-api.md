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
