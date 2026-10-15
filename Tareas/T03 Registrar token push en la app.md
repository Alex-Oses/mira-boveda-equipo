---
estado: en curso
responsable: Alex (PC 2)
depende_de: [T01, T04]
rama: feat/T03-push-android (frontend)
---
# T03 Registrar token push en la app

## Objetivo
Que la app pida permiso, saque el token y lo mande a mira-api.

## Pasos
- `mobile/src/services/notificaciones.js`: pedir permiso, crear el canal de Android, `getExpoPushTokenAsync({ projectId })`.
- Llamarlo después del login y enviar el token a `POST /v1/push/tokens` (tipo `expo`). Borrarlo al cerrar sesión.
- Al tocar una notificación, abrir la pantalla que indique `data.ruta` (expo-router).

## Terminada cuando
El token aparece en Firestore y al tocar una notificación se abre la pantalla correcta.

## Notas
Hasta que esté T04, mostrar el token por consola. Las notificaciones remotas no funcionan en Expo Go: hace falta un development build.

## Avance 2026-10-15 (Alex, PC 2)
- Rama `feat/T03-push-android` del frontend (commit `c0b1aa4`): servicio `mobile/src/services/notificaciones.js` según el contrato de T04, registro al entrar, DELETE antes de cerrar sesión y apertura de `data.ruta`.
- El mock (`npm run mock-api`) ya responde `POST/DELETE /v1/push/tokens` como el contrato, para probar sin T04.
- Falta: `eas init` (projectId), development build y probar en un Android real.
