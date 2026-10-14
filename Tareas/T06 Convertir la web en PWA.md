---
estado: libre
responsable:
depende_de: []
rama:
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
