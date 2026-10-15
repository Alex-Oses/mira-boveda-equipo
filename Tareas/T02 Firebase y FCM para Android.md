---
estado: libre
responsable:
depende_de: [T01]
rama:
---
# T02 Firebase y FCM para Android

## Objetivo
Que Expo pueda enviar a Android a través de FCM.

## Pasos
- En Firebase `restaurante-mira-18e0c`, añadir una app Android con el paquete `app.mira.restaurante`.
- Descargar `google-services.json` y enlazarlo en `app.json` (`android.googleServicesFile`). Decidir si se sube al repo o va como secreto de EAS.
- Generar la clave de cuenta de servicio (FCM v1) y subirla a EAS (`eas credentials`). **No subirla al repo.**

## Terminada cuando
Una notificación de prueba desde expo.dev/notifications llega a un Android.
