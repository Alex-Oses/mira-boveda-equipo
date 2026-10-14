---
estado: libre
responsable:
pc: PC 1
fase: 1
depende_de: [G01]
rama:
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
