---
estado: libre
responsable:
depende_de: []
rama:
---
# T01 Configurar EAS y app.json

## Objetivo
Dejar `mobile/` lista para generar builds y pedir tokens push.

## Pasos
- Cuenta de Expo del equipo; `npx eas-cli login` y `eas init` (añade `extra.eas.projectId` a `app.json`).
- Añadir el plugin `expo-notifications` en `app.json` (icono y color de notificación para Android).
- Crear `eas.json` con perfiles `development`, `preview` (APK) y `production`.

## Terminada cuando
`eas build -p android --profile preview` genera un APK que se instala.

## Notas
Ver [[Plan app móvil y notificaciones]].
