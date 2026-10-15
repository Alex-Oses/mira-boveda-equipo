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


## Revisión de Codex (2026-10-15)
Revisado `d232874` desde un worktree propio (`codex-review-G02`), sin modificar el módulo de Claude. `npm test`: **299/299**. Niveles fijos, descuentos no decrecientes con mejora final, fechas inclusivas Europe/Madrid, esquemas strict, permisos con comprobarRestaurante y filtro activa/vigente: sin otros hallazgos bloqueantes.

**[P1] La única activa por restaurante no está garantizada ante concurrencia.** En `mira-api/src/modules/ofertas/service.js:43-45` (crear) y `:57-58` (editar), comprobarUnicaActiva y la escritura son operaciones separadas. Dos peticiones pueden leer que no hay activa y escribir ambas. Documentarlo en docs/api.md §22 bis no sustituye la restricción acordada en [[Decisiones]] y [[Contratos API]]; basta un doble envío, dos pestañas o empresa y admin a la vez. La consulta pública devuelve ambas, dejando ambiguo qué oferta deben usar PC 2 y las fases siguientes.

Reproducción comprobada con el servicio y el mock de Firebase (sin Firestore real):
- Tras seedBase, `Promise.allSettled([svc.crear(datosActivos), svc.crear(datosActivos)])`: **2 altas aceptadas y 2 ofertas públicas activas** para r1.
- Crear dos inactivas y ejecutar `Promise.allSettled([svc.editar(a.id, datosActivos), svc.editar(b.id, datosActivos)])`: **2 activaciones aceptadas y 2 ofertas públicas activas**.
- La suite actual solo comprueba el 409 de forma secuencial; no detecta estos casos.

Corrección solicitada: comprobación y escritura atómicas en Firestore con serialización por restaurante, tanto en crear como en editar. Añadir regresiones de altas/activaciones paralelas: una operación debe ganar y la otra devolver 409, quedando exactamente una activa pública; verificar también en emulador o Firestore porque el mock no reproduce el aislamiento de transacciones real. No basta un bloqueo en memoria si hay varias instancias.

Resultado: **sigue en revisión**, pendiente corregir P1 y repetir revisión. No se ha hecho merge ni cambiado código de ofertas.
