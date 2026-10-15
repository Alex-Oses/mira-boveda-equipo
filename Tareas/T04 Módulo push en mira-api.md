---
estado: en curso
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
