---
estado: en curso
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
