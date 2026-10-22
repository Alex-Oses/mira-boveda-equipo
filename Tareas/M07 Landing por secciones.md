---
estado: en curso
responsable: Ruben (OpenCode)
pc: PC 3
proyecto: Reserva con carta
depende_de: [M02]
rama: feat/M07-landing-secciones
---
# M07 Landing por secciones

## Objetivo
Rehacer la landing en secciones compactas y llevar el listado infinito a Descubrir.

## Pasos
- Landing: Hero (no tocar) → buscador con filtros en desplegable (cerrado por defecto) → promociones → fotos de platos → más reservados de la semana → Top 50 Barcelona → nuevos → bloque final con botón a `#/descubrir`.
- Fuera de la landing: la lista infinita, el contador y los avisos de dieta (se van con la lista, a Descubrir).
- Descubrir: mantiene el vuelo de los 7 mejores; debajo, lista infinita de 27 en 27. Si llega con filtros desde la landing, se abre con ellos, el desplegable abierto y sin vuelo. El botón «ver más restaurantes» baja directo a la lista.
- Filas horizontales deslizables con tarjeta compacta; responsive (móvil, tableta y escritorio) y modo claro y oscuro.
- Solo componentes de la landing y Descubrir: no tocar `Header`, el modal de reserva ni el panel.

## Terminada cuando
Las 5 secciones y el bloque final se ven en móvil y escritorio, Descubrir pagina de 27 en 27 y `npm test` + `npm run build` están en verde.

## Notas
Proyecto: [[Reserva con carta]]. Peticiones a PC 1 en [[Contratos API]] §M07: promociones (confirmar), más reservados (nuevo), campo `creado` y siembra de cartas simuladas. Mientras falten, esas secciones se ocultan.
