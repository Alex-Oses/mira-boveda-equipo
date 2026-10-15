---
estado: en curso
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
