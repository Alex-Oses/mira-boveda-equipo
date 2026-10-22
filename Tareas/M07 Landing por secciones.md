---
estado: en revisión
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

## Estado (2026-10-22 · Ruben)

Implementado en `feat/M07-landing-secciones` (`570a717` + `08ca563`):
- Landing: Hero + `AvisoApp` + buscador plegado (Enter o «Buscar restaurantes» navega a `#/descubrir?…`) + `LandingSecciones` + bloque final. Fuera la lista infinita, el contador y los avisos de dieta.
- Secciones: promociones (`/v1/promotions?estado=activa`, fechas y prioridad filtradas en cliente), platos (`/v1/cartas/:id`, `foto: null` → `fotoDePlato`), más reservados (M12), Top 50 Barcelona y nuevos (`creado`, M13). Cada sección se oculta si falla o no hay datos.
- Descubrir: buscador plegado (abierto si llega con filtros), sin vuelo cuando hay filtros, lista infinita de 27 en 27 con `RestaurantList` y `id="dsc-todos"`; `?lista=1` baja directo a la lista.
- El buscador filtra solo por nombre en todo el sitio (`filterService`).
- Top 50 y nuevos piden `fetchRestaurants()` (catálogo completo, caché de 60 s); tarjetas compactas de ~240-260 px.

Pruebas: `npm test` 319/319 (el rojo previo era CRLF al quitar imports), `npm run build` en verde y comprobado en navegador con la API simulada (móvil 390 px y escritorio 1280 px).

Pendiente:
- Revisión de Izan/Claude antes de unir a `main`.
- M14: sembrar producción; hasta entonces «Platos» no aparece (en el emulador ya hay cartas).
- Promociones: sin campañas activas hoy, la sección se oculta.
