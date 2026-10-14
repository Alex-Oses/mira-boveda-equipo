---
estado: libre
responsable:
depende_de: [T04]
rama:
---
# T10 Preferencias de notificaciones

## Objetivo
Que cada usuario elija qué notificaciones recibe.

## Pasos
- API: guardar las preferencias en `usuarios/{uid}` (reservas, mensajes, promociones...) y que `enviarPush` las respete.
- App (`cuenta.js`) y web (Cuenta): interruptores por tipo.

## Terminada cuando
Al desactivar "promociones" dejan de llegar y el resto sigue llegando.
