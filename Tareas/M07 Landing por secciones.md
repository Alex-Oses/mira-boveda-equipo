---
estado: libre
responsable: Ruben (OpenCode)
pc: PC 3
proyecto: Reserva con carta
depende_de: []
rama:
---
# M07 Landing por secciones

## Objetivo
Rehacer la landing en secciones compactas siguiendo el esquema de [[Reserva con carta]].

## Pasos
- Orden: cabecera, foto principal, filtros, promociones, más reservados, para ti (fotos de platos), Top 50 Barna, nuevos y bloque final «Sube tu restaurante / ventajas de ser cliente».
- Usar datos y endpoints que **ya existen**. Si falta alguno, pedirlo a PC 1, no tocar el backend.
- Solo los componentes de la landing: no tocar `Header`, el modal de reserva ni el panel.
- Responsive (móvil, tableta y escritorio) y modo claro y oscuro.

## Terminada cuando
La landing muestra las 9 secciones en ese orden, compactas, en móvil y escritorio, con `npm test` y `npm run build` en verde.

## Notas
Proyecto: [[Reserva con carta]].
