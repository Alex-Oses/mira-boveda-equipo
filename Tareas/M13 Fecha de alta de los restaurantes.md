---
estado: en revisión
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: []
rama: feat/M13-creado
---
# M13 Fecha de alta de los restaurantes

## Objetivo
Exponer la fecha de alta para la sección «Nuevos» de [[M07 Landing por secciones]].

## Pasos
- Exponer `creado` como texto ISO en `GET /v1/restaurants?all=1` (y en `/mapa` si sale barato). Los aprobados desde Negocio ya lo guardan como fecha de Firestore.
- Los importados de Yelp: `creado: null` (sin backfill inventado).
- Comprobar que las altas hechas por el admin también lo guardan.
- Tests.

## Terminada cuando
Los restaurantes nuevos llevan `creado` en ISO en el catálogo y los de Yelp `null`, con tests en verde.

## Notas
Lo pide Ruben (PC 3) para [[M07 Landing por secciones]]. Revisión cruzada: Claude revisa el código antes de unirlo.

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M13-creado (base M04).
- creado ISO/null en catálogo y mapa, caché con versión actualizada; sin backfill. Alta/aprobación admin probada (ya guarda fecha real).
- npm test: 445 pasan, 0 fallan; 1 omitido de reglas (M04, requiere emulador). Claude revisa; no unir ni desplegar.

Validación final con emulador habilitado: **449/449, sin omisiones**. Commit `533c7b4`. Sin unión ni despliegue.

## Revisión de Claude · 2026-10-22
**Aprobado** (`533c7b4`). `creado` sale en ISO en el catálogo y en `/mapa`, y `null` si no hay fecha real (Yelp), sin inventar ninguna. Acepta Timestamp de Firestore, `Date` y texto ISO. El documento `_catalogo` se regenera solo, porque ahora exige que su versión incluya `creado`.
