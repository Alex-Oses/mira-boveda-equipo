---
estado: en curso
responsable: Ruben (PC 3, OpenCode)
pc: PC 3
proyecto: Reserva con carta
depende_de: [M07, G04]
rama: feat/M15-promociones-panel
---
# M15 Editor de promociones en el panel

## Objetivo
Poder crear una campaña de promoción desde el panel y que la landing la muestre en «En promoción». Hoy solo existe `POST /v1/promotions`; no hay pantalla, y por eso producción devuelve `{"data":[]}`.

## Contexto (2026-10-23)
- Backend `POST /v1/promotions` (auth `empresa`/`admin`; el admin puede crear para cualquier restaurante):
  - Obligatorios: `restauranteId`, `fechaInicio`, `fechaFin`, `presupuestoTotal` (>0, ≤1.000.000).
  - Opcionales: `tipo` (def. `boost_visibilidad`), `cpc` (def. 0,20), `puntosExtraPorReserva` (def. 0).
  - Crea `estado: "activa"`, `prioridad: 100`, contadores a cero.
  - **409** si ya hay una promoción `activa` para ese restaurante, sin mirar fechas.
- `GET /v1/promotions?restauranteId=…` es público; sin sesión oculta `presupuestoTotal` y `gastoAcumulado`. El panel necesita pedirlo autenticado.
- `GET /v1/promotions/:id/stats` (empresa/admin) devuelve views, clicks, ctr, reservasGeneradas y gastoAcumulado.
- **No hay `PUT` ni `DELETE`**: no se puede editar ni desactivar desde la UI. Una campaña caducada sigue `activa` y bloquea la siguiente con 409.
- La landing ya pinta la sección si hay campañas vigentes (`landingApi.fetchPromocionesActivas`).

## Pasos
1. `promotionsApi.crear` y `list` con opción de auth en `src/services/api.js`.
2. `PromocionesPanel.jsx` reutilizable (patrón `OfertasPanel`): lista, crear, validación, 409 y stats.
3. Integrarlo en el panel que se decida (admin global y/o dashboard del restaurante).
4. i18n `promociones.*` en es/ca/en.
5. Tests de la lógica pura (validación/fechas) + `npm test` + `npm run build`.

## Terminada cuando
Se crea una campaña con fechas vigentes desde el panel, aparece en la lista y en «En promoción» de la landing, y el formulario muestra los errores 400/403/409.

## Decisiones pendientes
- [ ] ¿Panel admin global (`#/admin`), dashboard del restaurante (`#/dashboard`) o ambos?
- [ ] ¿Solo crear + listar (backend actual) o también editar/desactivar (requiere backend)?
- [ ] ¿Campos mínimos (fechas + presupuesto) o completos (tipo/CPC/puntos)?

## Limitación conocida
Sin `PUT /v1/promotions/:id`, una campaña mal creada o caducada no se puede cerrar desde la UI y bloquea nuevas (409). Si se quiere ciclo completo, PC 1 debe añadir el endpoint.

## Backend listo · Codex · 2026-10-23
@Rubén: `feat/M15-promociones-api`, commit `777b655`, subido para revisión. PUT /v1/promotions/:id permite fechaFin, presupuestoTotal y estado pausada/finalizada (parcial, al menos un campo). Contrato completo en [[Contratos API]], sección M15. POST ignora campañas caducadas y devuelve409 CONFLICT si queda alguna activa no caducada. 463 tests correctos/2 omitidos (emulador). Sin despliegue: todavía no disponible en la API real.
