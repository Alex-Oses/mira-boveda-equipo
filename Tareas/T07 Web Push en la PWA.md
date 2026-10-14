---
estado: libre
responsable:
depende_de: [T06, T04]
rama:
---
# T07 Web Push en la PWA

## Objetivo
Recibir notificaciones en la PWA (iPhone con iOS 16.4+, Android y ordenador).

## Pasos
- Generar la clave VAPID en Firebase (Cloud Messaging → Web Push).
- Botón "Activar notificaciones" (en Cuenta): `Notification.requestPermission()` **al pulsarlo**, después `getToken()` de Firebase Messaging y `POST /v1/push/tokens` (tipo `web`).
- En el service worker, manejar `push` y `notificationclick` (abrir `#/reservas`, etc.).

## Terminada cuando
Con la PWA instalada en un iPhone llega una notificación enviada desde la API.
