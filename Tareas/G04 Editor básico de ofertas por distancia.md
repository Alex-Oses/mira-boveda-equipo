---
estado: hecho
responsable: Ruben (PC 3, Opencode)
pc: PC 3
fase: 1
depende_de: [G01]
rama: feat/G04-editor-ofertas
---
# G04 Editor básico de ofertas por distancia

## Objetivo
Que el restaurante cree y edite sus ofertas por niveles desde su dashboard.

## Pasos
- Sección nueva en `components/dashboard/`: lista de ofertas, crear y editar.
- Un formulario por nivel (1 km, 500 m, 100 m y llegada) con texto y descuento. Avisar si un nivel no mejora al anterior.
- Primero contra el mock y después contra la API real (G02).

## Terminada cuando
Se crea una oferta completa y vuelve a aparecer al recargar la página.

## Hecho (2026-10-15, rama `feat/G04-editor-ofertas`, commit 0204bca)
- `src/models/ofertasModel.js` (reglas puras espejo del backend) + `test/ofertasModel.test.js` (8 tests).
- `src/services/ofertasApi.js` (CRUD `/v1/ofertas`) y `src/components/dashboard/OfertasPanel.jsx`: lista, crear/editar con un formulario por nivel, aviso «mejora al anterior», vigencia, borrar con confirmación.
- Integrado en `Dashboard.jsx`: botón «Ofertas» en la topbar que abre el panel (mismo patrón que la Ficha). Claves `ofertas.*` en es/ca/en.
- Verificado contra el mock de G01. `npm test` 199/203 (los 4 fallos son preexistentes en `esperaQuitar` y `miraNuevoChat` por Node 24) y `npm run build` en verde.
- Falta: revisión cruzada y probar contra la API real (G02) cuando esté mergeada; OK de PC 1 a G01.

## Revisión de Claude (PC 1, 2026-10-15)
Rama `feat/G04-editor-ofertas` (`6a80260`): `npm test` 204/204 en mi equipo (los 4 fallos que viste son de Node 24, no de G04) y `npm run build` correcto. Bien: reglas puras en `ofertasModel.js` iguales a las del backend, con tests; `ofertasApi.js` encaja con el contrato y con `httpClient` (`err.status`, `err.data.details`, `api.del`); el formulario por nivel avisa en directo si un nivel empeora; los detalles de un 400 se pintan en su campo; los textos están en es/ca/en.

**[P2] En el formulario no se ve el error general al guardar.** `errorGeneral` solo se pinta con `modo === "lista"` (`OfertasPanel.jsx`, bloque de `ofertas-error` de arriba). Si al guardar la API responde **409** (ya hay otra activa: el caso más habitual al crear una segunda oferta activa), 403, 404 o falla la red, `erroresDeApi` devuelve null y el usuario sigue en el formulario **sin ningún mensaje**. Propuesta: pintar también `errorGeneral` dentro de `ofertas-form` (por ejemplo, encima de los botones Guardar/Cancelar).

**[P3] La vigencia se compara con la fecha UTC.** `estadoOferta` usa `new Date().toISOString().slice(0, 10)`; el backend usa Europe/Madrid. Entre las 00:00 y las 02:00 (hora de Madrid) el distintivo puede decir «fuera de vigencia» un día antes o después. Usar la fecha en Madrid, como `hoyMadrid` del backend (`Intl.DateTimeFormat("en-CA", { timeZone: "Europe/Madrid" })`).

Menor: `docs/coordenacion/` (hilo PC 3 ↔ PC 4) en el repo de código; la coordinación encaja mejor en la bóveda, para no mezclarla con el código. No bloquea.

Cuando esté el P2: probar contra la **API real** (`main` del backend ya tiene `/v1/ofertas` en producción) con una cuenta de empresa dueña de un restaurante: crear, recargar y ver que sigue ahí, que es lo que pide «Terminada cuando». Después, a «hecho».

## Cierre (2026-10-15, Ruben, commit 5d629e8)
- **P2 resuelto**: `errorGeneral` ahora se pinta también dentro de `ofertas-form`, encima de Guardar/Cancelar (`data-test="ofertas-error-form"`), así un 409/403/404 o caída de red se ve desde el formulario.
- **P3 resuelto**: `estadoOferta` usa `hoyMadrid()` (`Intl.DateTimeFormat("en-CA", { timeZone: "Europe/Madrid" })`), la misma fecha que el backend; test nuevo `hoyMadrid usa la fecha de Madrid…` que cubre el salto de día entre UTC y Madrid.
- `npm test` 200/204 (los 4 fallos, los mismos preexistentes de `esperaQuitar`/`miraNuevoChat` con Node 24) y `npm run build` en verde.
- **Prueba contra la API real (producción)**, cuenta `restaurante@gmail.com`, restaurante «Super Restaurante Mira» (`IrSLEIhEfkt0qvJEbSRI`):
  - `POST /v1/ofertas` → **201** con id `XkVzkxR2d8eJjiZo0zYJ`.
  - `GET /v1/ofertas/mias` → la oferta **persiste** tras la recarga.
  - Segunda oferta activa → **409 CONFLICT** («Ya hay otra oferta activa»).
  - `PUT /v1/ofertas/:id` → 200 y el cambio **persiste** al recargar.
  - Oferta final dejada **inactiva** con título «SMOKE TEST G04 (borrar)» para poder verla/borrarla desde el panel.
- Observación menor (no de G04): `GET /v1/dashboard/my-restaurants` devuelve el id `V2Zjzmay4kZRzuuSG56t` («Restaurante test») que ya no existe en `restaurants` (404 en ownership); viene probablemente de un `restaurantId` huérfano en el perfil. Revisar cuando haga falta.

Estado: **hecho** (pendiente de merge en la revisión de fase por PC 4 / Marco).

## Segunda revisión de Claude y unión a main (2026-10-15)
`5d629e8`: P2 y P3 corregidos (error general en el formulario; `hoyMadrid` con test del cambio de día). Prueba contra producción de Ruben: 201, persiste al recargar, 409 con la segunda activa, PUT 200. **Aprobado.** Unido a `main` del frontend (`npm test` 214/214, build correcto) y desplegado en Vercel: botón «Ofertas» en el panel del restaurante.

