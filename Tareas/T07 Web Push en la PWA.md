---
estado: hecho
responsable: Alex (PC 2)
depende_de: [T06, T04]
rama: feat/mira-app (frontend)
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

## Avance 2026-10-15 (Alex, PC 2)
- Código en `feat/mira-app` (`4e2b36b`): `src/services/pushWeb.js` + sección «Notificaciones» en Mi cuenta (botón de activar e interruptores de T10, ocultos si la API aún no los tiene). Borra el token al cerrar sesión.
- `sw.js` acepta `data.ruta` como `"reservas"` o `"/reservas"` (el backend la manda sin barra) y solo rutas internas.
- Falta: clave VAPID de Firebase → `VITE_FIREBASE_VAPID_KEY` en Render, desplegar y probar en un iPhone con la PWA instalada.

## Avance 2026-10-16 (Alex, PC 2)
- Clave VAPID generada en Firebase (Cloud Messaging → Certificados push web). Es pública: está en `.env.example` del frontend, rama `chore/T07-vapid`.
- Ojo: la web está en **Vercel**, no en Render. Falta añadir `VITE_FIREBASE_VAPID_KEY` en Vercel (Production), redeploy y probar en iPhone con la PWA instalada.

## Cerrada (2026-10-16)
VAPID en Vercel (Ruben) y redeploy. PWA instalada en iPhone (iOS), avisos activados, token web `ios` guardado en Firestore. Mensaje de prueba desde Firebase Messaging (solo a ese token) **recibido en el iPhone**.
