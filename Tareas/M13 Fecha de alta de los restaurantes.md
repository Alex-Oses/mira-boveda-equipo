---
estado: en curso
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
