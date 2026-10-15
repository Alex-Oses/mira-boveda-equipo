---
estado: en revisión
responsable: Izan (Claude)
pc: PC 1
fase: 1
depende_de: [G01]
rama: feat/G02-modulo-ofertas
---
# G02 Modelo de ofertas y endpoints base

## Objetivo
Implementar en mira-api lo que dice el contrato de G01.

## Pasos
- Módulo `src/modules/ofertas/` con `routes.js` (zod), `service.js` y `logica.js` (puro).
- Validar los niveles: distancias decrecientes y ofertas cada vez mejores.
- Reglas de Firestore e índices. Tests con `node --test` y supertest.
- Documentarlo en `mira-api/docs/api.md`.

## Terminada cuando
`npm test` pasa y los endpoints funcionan en local contra Firestore.

## Hecho (2026-10-15, Izan + Claude)
- Rama `feat/G02-modulo-ofertas` del backend (`d232874`): módulo `src/modules/ofertas/` (`logica.js` puro, `service.js`, `routes.js`), montado en `/v1/ofertas`. Cumple el contrato de [[Contratos API]] fase 1.
- Reglas de Firestore (`ofertas` solo vía API). Documentado en `docs/api.md` §22 bis.
- `npm test` 299/299 (12 tests nuevos: niveles, vigencia, 409 de única activa, pública vs mias, permisos de dueño/otra empresa/cliente/admin, editar y borrar).
- **Sin probar contra Firestore real** (solo con el mock de los tests). Se puede probar al desplegar.
- Falta: revisión de Codex.

