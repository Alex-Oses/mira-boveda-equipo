---
estado: en revisión
responsable: Alex (PC 2)
depende_de: []
rama: feat/T06-pwa-descargas (frontend)
---
# T06 Convertir la web en PWA

## Objetivo
Que la web se pueda instalar en la pantalla de inicio (sobre todo en iOS).

## Pasos
- `vite-plugin-pwa`: manifest con `display: standalone`, nombre, colores e iconos (incluido `apple-touch-icon`).
- Service worker registrado; comprobar que el enrutado por hash sigue funcionando.
- Probar la instalación en iPhone (Safari) y en Android (Chrome).

## Terminada cuando
Al añadirla a inicio se abre a pantalla completa, sin la barra de Safari.

## Entrega 2026-10-15 (Alex, PC 2)
- Rama `feat/T06-pwa-descargas` (`0c4a52b`): manifest, iconos de MIRA, metas de iOS y `sw.js` (sin caché; `push` y `notificationclick` listos para T07).
- Página `#/app` (enlace en el pie): botón del APK para Android e instrucciones para añadirla a la pantalla de inicio en iPhone. Detecta el móvil y avisa si ya está instalada.
- **`VITE_APK_URL`** en Render = enlace del APK de EAS. Sin ella el botón sale como «Disponible muy pronto».
- Build OK; tests 190/194 (los 4 que fallan ya fallaban en main). Probada en Chrome: claro/oscuro, 375 px y escritorio; SW y manifest cargan. Sin conflictos con main, T03 ni G01.
