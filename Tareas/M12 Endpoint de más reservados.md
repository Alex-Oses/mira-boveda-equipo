---
estado: en revisión
responsable: Izan (Codex)
pc: PC 1
proyecto: Reserva con carta
depende_de: []
rama: feat/M12-mas-reservados
---
# M12 Endpoint de más reservados

## Objetivo
Endpoint público para la sección «Los más reservados de la semana» de [[M07 Landing por secciones]].

## Pasos
- `GET /v1/restaurants/mas-reservados?dias=7&limit=8` (auth: no), contrato en [[Contratos API]] §M07, punto 2.
- Cuenta reservas con `fecha` en los últimos `dias` (día de Madrid) y `estado !== "cancelada"`. `dias` 1-30 (def. 7), `limit` 1-20 (def. 8).
- Respuesta `{ data: [{ id, reservas }] }`, de más a menos. Solo agregados: sin uid ni datos de clientes.
- Consulta acotada por fecha (no leer todas las reservas) y caché corta si hace falta.
- Tests.

## Terminada cuando
Devuelve los restaurantes ordenados por reservas de la semana, sin datos personales, con tests en verde.

## Notas
Lo pide Ruben (PC 3) para [[M07 Landing por secciones]]. Revisión cruzada: Claude revisa el código antes de unirlo.

## Entrega Codex · 2026-10-22
- Rama: https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/M12-mas-reservados (base M04).
- Público, agregado id/reservas; rango Firestore de hoy−(dias−1) a hoy Madrid, proyección mínima, caché 30 s. Sin lectura de perfiles ni histórico completo.
- npm test: 446 pasan, 0 fallan; 1 omitido de reglas M04 (requiere emulador). Pruebas de límites, canceladas, fechas, proyección, caché y medianoche. Claude revisa; no unir ni desplegar.

Validación final con emulador habilitado: **450/450, sin omisiones**. Commit `a2417b6`. Sin unión ni despliegue.
