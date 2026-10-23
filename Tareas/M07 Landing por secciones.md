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

## Ajustes UI (2026-10-22 · tarde)

Rama `feat/M07-ajustes-ui` (worktree `.worktrees/m07`), desde `08ca563`:
- Tarjetas de las filas de la landing con el mismo ancho que la rejilla de Descubrir (1/2/3 columnas) y foto de 200 px (`05a6652`).
- Promocionados con carrusel de scroll-snap y flechas (ocultas en móvil); claves i18n `landing.anterior/siguiente`.
- «Platos» cae a la carta simulada local (`cartaLibro` + `fotoDePlato`) mientras M14 no siembre producción. Marcado como `shortcut:`.
- Descubrir: buscador movido justo encima de los destacados (debajo del vuelo).
- Badge «Promocionado» sobre la foto, abajo a la izquierda, sin tapar precio ni favorito (`9676623`).

### QA
- `f584136`: el mock sirve `/v1/restaurants/mas-reservados` y el test del buscador sigue el salto a Descubrir + Limpiar.
- `adab173`: la unitaria de CI ya bloquea (los 38 fallos quedaron resueltos en `08ca563`) y la regresión local encuentra el backend desde un worktree.
- `d7fb901`: 5 campañas de ejemplo en el mock y test del carrusel de promocionados.
- Verde: `npm test` 320/320 (Node 20 y 24), build, E2E 28/28, monkey semillas 20261020/1/2/3 y móvil, backend 87/87.
- Push hecho con `ruben-munoz-bl` (`9676623`). Falta abrir el PR y la revisión del equipo.
- Cuentas locales: GCM guarda `TontonR` (por defecto) y `ruben-munoz-bl` (frontend/backend del equipo).

## Todas las secciones en carrusel (2026-10-22 · noche)

- `09317a8`: platos, más reservados y Top Barcelona pasan al mismo `Carrusel` que promociones. La pista ocupa todo el ancho: la primera tarjeta alinea con el título y las siguientes llegan al borde de la ventana; anchos 82% móvil, 46% tableta y `clamp(320px, 27vw, 370px)` escritorio para que se vea la siguiente.
- Buscador: CTA «Buscar»/«Cercar»/«Search», sin salto de línea y con las acciones a la derecha (104×48 px a 1280).
- Test E2E generalizado a las cuatro secciones con datos. Verde: `npm test` 320/320, build, E2E 28/28 y monkey semilla 1.
- `b8d123a`: avance automático cada 5 s con vuelta al inicio, pausa al pasar el ratón o enfocar y respeto a `prefers-reduced-motion`; barra de scroll oculta, solo flechas y deslizamiento táctil. Nuevo test E2E: E2E 29/29.
- `9888be9`: el runner de Windows trae `prefers-reduced-motion: reduce` y el carrusel, con razón, no avanza. El E2E fija `no-preference` en `qa/navegador.py` salvo con `QA_REDUCED_MOTION`. `npm test` 320/320 y build en verde; CI del PR relanzado.
- PR [#20](https://github.com/ruben-munoz-bl/restaurante-mira-frontend/pull/20) abierto de `feat/M07-ajustes-ui` a `feat/M07-landing-secciones`; pendiente revisión y merge.

## Revisión de la preview (2026-10-23 · Ruben)

- CI: el run `39910922600` de `9888be9` terminó en verde; el fallo era el `prefers-reduced-motion` del runner de Windows.
- `67f2385`: en Descubrir, el contador se queda pegado arriba con un botón «Buscar y filtrar» que devuelve al buscador (probado a 1400 y 390 px). `npm test` 320/320 y build en verde.
- Promociones: la preview pide `GET /v1/promotions?estado=activa` y producción devuelve `{"data":[]}`. No es la sesión: el endpoint es público y sin filtros también devuelve vacío. La sección se oculta porque no hay campañas; para verla hay que crear una con fechas vigentes, sin tocar el frontend.
- Platos: la preview pide `/v1/cartas/:id` y recibe 404 (M14 sin sembrar); usa la carta simulada. Elige las 8 primeras fichas del catálogo y el primer plato de cada carta, con la foto resuelta por `fotoDePlato`. No hay curación del backend; M14 solo aportará las cartas reales.
- Endpoints: M12 y M13 desplegados (se ven «más reservados» y «Nuevos»); M14 pendiente; promociones sin datos.

## Cómo está hecho ahora (2026-10-23 · Ruben)

Rama de trabajo: `feat/M07-ajustes-ui` (`67f2385`), PR [#20](https://github.com/ruben-munoz-bl/restaurante-mira-frontend/pull/20).
Preview: https://restaurante-mira-frontend-git-feat-m07-ajustes-ui-rumu439-4703.vercel.app
CI: `9888be9` en verde (run `39910922600`) y `67f2385` en verde (run `39912468120`).

### Landing
- `src/components/LandingSecciones.jsx`: Hero intacto → buscador plegado → secciones → bloque final que baja a `#/descubrir?lista=1`. Cada sección se pinta solo si tiene datos.
- Orden de secciones: promociones → platos → más reservados → Top 50 Barcelona → nuevos → CTA final.
- Carrusel propio (`Carrusel`, dentro de `LandingSecciones.jsx`): scroll-snap nativo, flechas ‹ › (ocultas en móvil), avance automático cada 5 s con vuelta al inicio, pausa al pasar el ratón o al enfocar y respeto a `prefers-reduced-motion: reduce`. Sin barra de scroll.
- Estilos en `src/styles/landing.css`. Tarjeta: 82% móvil, 46% tableta y `clamp(320px, 27vw, 370px)` en escritorio, para que asome la siguiente.
- Textos `landing.*` en `src/i18n/{es,ca,en}.js`.

### Fuentes de datos (`src/services/landingApi.js`)
- **Promociones**: `GET /v1/promotions?estado=activa` (público). El estado no caduca solo, así que las fechas y la prioridad se filtran en cliente.
- **Más reservados**: `GET /v1/restaurants/mas-reservados?dias=7&limit=8` (M12).
- **Platos**: `GET /v1/cartas/:id` por restaurante; si no hay carta usa `primerPlatoLocal` (`shortcut:`) y la foto se resuelve con `fotoDePlato`. Ver «Platos» abajo.
- **Top 50 y nuevos**: catálogo completo `GET /v1/restaurants?all=1` (caché 60 s) + `topBarcelona()` y `nuevosRestaurantes()` de `src/models/landingModel.js` (campo `creado`, M13).
- Si un endpoint falla o viene vacío devuelve `[]` y su sección desaparece; nunca rompe el Hero.

### Descubrir (`src/components/Descubrir.jsx`)
- Vuelo de los 7 mejores; debajo, buscador y lista infinita de 27 en 27.
- Si llega con filtros en la URL (`#/descubrir?...`) no hay vuelo, el desplegable se abre y la lista ya viene filtrada.
- `?lista=1` (botón «ver más restaurantes») salta directo a `#dsc-todos`.
- `67f2385`: en el listado, el contador se queda pegado arriba con un botón «Buscar y filtrar» que devuelve al buscador. Estilo en `src/styles/descubrir.css`, texto `descubrir.filtros`.
- El buscador filtra solo por nombre (`filterService`).

### Platos: cómo se eligen
1. `LandingSecciones` pide `fetchPlatosDestacados` con el catálogo completo.
2. Se cogen las **8 primeras fichas** del catálogo (`LIMITE_PLATOS = 8`).
3. De cada una se pide `/v1/cartas/:id` y se toma **el primer plato con nombre** de la carta.
4. Si la carta no existe (404) se usa la carta simulada local (`primerPlatoLocal`).
5. La foto se resuelve con `fotoDePlato` a partir del nombre del plato.

No hay curación del backend ni flag de «destacado»: M14 solo aportará las cartas reales. En la preview todos los `/v1/cartas/:id` dan 404, así que ahora mismo se ven platos simulados.

### QA
- `qa/navegador.py` emula `prefers-reduced-motion: no-preference` salvo con `QA_REDUCED_MOTION` (el runner de Windows trae `reduce`).
- `qa/tests/test_flujo_home.py`: tests de carruseles y autoavance.

## Preguntas abiertas para Izan (2026-10-23)

1. **Promociones**: en producción no hay campañas (`GET /v1/promotions?estado=activa` → `{"data":[]}`), por eso la sección no se ve. ¿Creamos una campaña de ejemplo con fechas vigentes para la preview, o lo dejamos oculto hasta que exista una real? Solo existe el `POST /v1/promotions`; no hay pantalla ni script que las cree.
   - **No confundir** con las **ofertas por distancia** ([[G04 Editor básico de ofertas por distancia]], editor «Ofertas» en el dashboard) ni con el flag **partner** ([[G11 Avisos solo de favoritos y destacados]], «Gestión Restaurantes»). Ninguna de las dos crea campañas.
   - Hueco detectado (2026-10-23): no hay tarea asignada para un editor de promociones/campañas en el panel; `components/promotions/` del frontend son estilos de ofertas. Por eso «En promoción» no puede llenarse desde la app.
2. **Platos**: ahora se cogen las 8 primeras fichas y el primer plato de cada carta. ¿Qué criterio prefieres para «platos que apetecen»: aleatorio, un principal en vez del primero, o un flag de destacado desde el backend?
3. **Acceso a filtros**: la barra «Buscar y filtrar» es la solución mínima. ¿La mantenemos o prefieres otra (botón flotante, buscador fijo…)?

## Para retocar sin miedo
- Solo estética/UX: `src/styles/landing.css`, `src/styles/descubrir.css`, estructura del `Carrusel` en `src/components/LandingSecciones.jsx` y textos `landing.*` / `descubrir.*`.
- Fuera de alcance de M07: `Header`, modal de reserva, panel y backend.
