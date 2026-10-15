---
estado: en revisión
responsable: Codex (PC 1)
depende_de: [T04]
rama: feat/T05-notificar-reservas-mensajes
---
# T05 Notificar reservas y mensajes

## Objetivo
Enviar notificaciones reales en los primeros eventos.

## Pasos
- Reserva confirmada o cancelada → `enviarPush` al cliente (`data.ruta: "reservas"`).
- Mensaje nuevo → `enviarPush` al destinatario (`data.ruta: "mensajes"`).
- Textos en es/ca/en según el `idioma` del token.
- Más adelante: recordatorio de reserva (ver `yelp-connection/recordatorios.js`), incidencias, puntos y promociones.

## Terminada cuando
Al hacer una reserva desde otra cuenta, al cliente le llega la notificación.

## Entrega de Codex (2026-10-15)
- Rama [feat/T05-notificar-reservas-mensajes](https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/T05-notificar-reservas-mensajes), commit `4aa0eab`, desde main `86add00`, en worktree propio codex-T05-backend. Pendiente revisión de Claude antes de main.
- Confirmación/cancelación de reservas avisa al UID de la reserva; mensajes internos y del admin al dueño avisan al destinatario. data.ruta reservas/mensajes, textos es/ca/en por token. Push del buzón genérico, sin contenido privado.
- Avisos posteriores al commit y fuera del callback de transacción; repetir estado no repite aviso. Fallo de push no cambia el resultado de la reserva/mensaje. Reservas nuevas pendientes no se anuncian como confirmadas.
- npm test: **310/310**, 12 nuevas; git diff --check correcto. Tests con proveedores/Firestore simulados, sin prueba en dispositivo real.
- Documentado en mira-api/docs/api.md (al final, sección T05). Sin cambios de índices, reglas, ofertas ni frontend. No se ha desplegado ni unido a main.
- Pendiente con PC 2: dispositivo de T03/T07, reserva confirmada/cancelada desde empresa/admin y mensaje de buzón; comprobar idioma, recepción y navegación por data.ruta. No hay cola persistida de eventos: caída tras commit puede perder el aviso.
