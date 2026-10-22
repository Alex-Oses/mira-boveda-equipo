---
estado: en curso
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: []
rama: feat/M14-sembrar-cartas
---
# M14 Sembrar las cartas simuladas

## Objetivo
Que las cartas que hoy se generan en el frontend existan en Firestore, para que M07 muestre platos reales y se pueda pedir por adelantado.

## Pasos
- Script en `mira-api/scripts` que recorra `restaurants` y escriba `cartas/{id}` con las mismas categorías y platos que genera `cartaDelLocal()`/`cartaLibro()` en `src/models/restaurantModel.js` del frontend (deterministas por `id`).
- Formato del contrato de M01 (IDs válidos y únicos, precios a 2 decimales), `disponible: true`, `foto: null`.
- No sobrescribir cartas que el restaurante ya haya creado en el editor (M09).
- Modo de prueba (`--dry-run`) y contra el emulador primero. **No se ejecuta en producción sin el OK de Izan.**

## Terminada cuando
El script siembra las cartas en el emulador sin pisar cartas reales, y `GET /v1/cartas/:id` devuelve los mismos platos que veía el frontend.

## Notas
Lo pide Ruben (PC 3) para [[M07 Landing por secciones]]. Revisión cruzada: Claude revisa el código antes de unirlo.
