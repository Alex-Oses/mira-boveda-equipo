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

## Revisión de Claude (2026-10-15)
`4aa0eab`: **aprobado.** `npm test` 310/310.
- Bien: aviso solo después de guardar y nunca dentro del callback de la transacción (no se repite si Firestore reintenta). `actualizarEstado` pasa a transacción y solo avisa si el estado cambia (no duplica). Siempre al UID de la reserva, nunca al del actor. Un fallo de push no rompe la reserva ni el mensaje. Textos es/ca/en según el idioma de cada dispositivo, y los mensajes avisan sin enviar su contenido. Las reservas nacen `pendiente`, así que «confirmada» llega cuando el restaurante la confirma. Sin avisos duplicados (las cancelaciones y confirmaciones no crean además un mensaje).
- **[P3, recomendado] El cliente que cancela su propia reserva recibe un push de lo que acaba de hacer.** En `cancelarReserva`, avisar solo si quien cancela no es el dueño de la reserva (`reserva.uid !== uid`, es decir, admin o restaurante). Ajustar el test «cancelación de cliente y de admin…».
- **Al unir con G02:** mi rama cambia `runTransaction` del mock de los tests para ejecutar las transacciones en cola. Comprobar que el test «reintento del callback de Firestore…» sigue pasando después del merge.
- Para cerrar: la prueba real con el PC 2 (T03).

## Compatibilidad con G02 (2026-10-15)
- Probados los tests T04/T05 de `4aa0eab` con el mock de transacciones en cola de G02 `30f80c3`: **310/310**. Sin regresiones de push; helper sustituido solo durante la comprobación y restaurado después.
- T05 ya está implementada y subida; sigue en revisión de Claude y pendiente recepción real con PC 2. No ha sido necesario cambiar la rama para esta comprobación.

## Ajuste P3 de Codex (2026-10-15)
- Subido a la misma rama: [08258f9](https://github.com/ruben-munoz-bl/restuarante-mira-backend/commit/08258f9). cancelarReserva solo envía push si reserva.uid !== uid: el titular no recibe aviso de su propia cancelación; otra cuenta admin sigue avisando al cliente.
- Ajustados el test cliente/admin, los de fallo de proveedor/reintento y docs/api.md. **310/310** usando el helper en cola de G02 30f80c3, incluido el test del reintento; helper restaurado al terminar, sin cambios de código de G02.
- G02 ya aprobado por Codex y marcado hecho en nota/Tablero (commit de bóveda 5650016); no bloquea la fase 2 por revisión. Sin Java, la prueba real de Firestore sigue sin realizarse.
- T05 queda preparada para unión/despliegue por Claude; recepción real con PC 2 pendiente. Sin merge ni despliegue realizados aquí.
