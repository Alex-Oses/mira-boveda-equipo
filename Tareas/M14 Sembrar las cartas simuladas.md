---
estado: en revisión
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

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M14-sembrar-cartas (`ca72969`, base M04).
- scripts/sembrarCartas.js: copia determinista del generador del frontend (cf493de), IDs válidos, disponible true, foto null; páginas de 100 y comprobación transaccional para no sobrescribir ninguna carta existente.
- --dry-run sin escrituras; CLI restringido a emulador local y proyecto demo explícito. No se ejecutó contra producción.
- 72 muestras contrastadas con cartaLibro original. Suite con emulador: **452/452**, sin omisiones; CLI real dry-run, siembra, repetición y GET público coherente con frontend.
- Claude revisa antes de unir. Izan decide unión y despliegue de M12/M13/M14. Los tres parten de M04 de forma independiente.

## Revisión de Claude · 2026-10-22
**Aprobado** (`ca72969`). Reutiliza el generador del frontend, valida cada carta con el esquema de M02 y no sobrescribe cartas existentes (lo vuelve a comprobar dentro de la transacción). Comprobado: sin `FIRESTORE_EMULATOR_HOST` en localhost **se niega a arrancar**.
- **Para producción falta un paso:** el script solo admite el emulador y proyectos `demo-…`, como se pidió. Sembrar las cartas reales necesita una opción explícita para producción (por ejemplo `--produccion` con confirmación) y el OK de Izan. Hasta entonces, la sección de fotos de platos de M07 sigue oculta.
