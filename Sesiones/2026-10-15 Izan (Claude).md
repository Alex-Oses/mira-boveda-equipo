---
fecha: 2026-10-15
persona: Izan
ia: Claude Code
tareas: [G01, G02]
---
# Sesión 2026-10-15 · Izan (Claude)

## Hecho
- [[G01 Contrato API fase 1]]: contrato de ofertas publicado en [[Contratos API]] y mock en el frontend (`feat/G01-contrato-ofertas`). En revisión: falta OK de PC 2 y PC 3.
- [[G02 Modelo de ofertas y endpoints base]]: módulo `ofertas` en el backend (`feat/G02-modulo-ofertas`), 299/299 tests. En revisión por Codex.
- Revisión de [[T04 Módulo push en mira-api]] (Codex): P1, un móvil que cambia de cuenta recibiría las notificaciones de la anterior. Detalle en la nota de T04.

## Pendiente / siguiente paso
- Contrato de la fase 2 y [[G05 Ofertas cerca de mí]] cuando G02 esté revisado.

## Problemas o dudas para el equipo
- Hoy se reescribió el historial de la bóveda (push forzado) y se perdió un cambio. **No hagáis push --force.**

## Ramas y archivos tocados
- Frontend: `feat/G01-contrato-ofertas` (`scripts/mock-api.mjs`).
- Backend: `feat/G02-modulo-ofertas` (`src/modules/ofertas/`, `app.js`, `firestore.rules`, `docs/api.md`, `test/ofertas.test.js`).
