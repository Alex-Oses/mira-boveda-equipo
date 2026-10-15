---
estado: en revisión
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
