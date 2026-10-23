---
estado: en revisión
responsable: Ruben (PC 3, OpenCode)
pc: PC 3
proyecto: Reserva con carta
depende_de: [M07, G04]
rama: feat/M15-promociones-panel
---
# M15 Editor de promociones en el panel

## Objetivo
Poder crear una campaña de promoción desde el panel y que la landing la muestre en «En promoción». Hoy solo existe `POST /v1/promotions`; no hay pantalla, y por eso producción devuelve `{"data":[]}`.

## Decisiones (2026-10-23)
- Panel **admin global** (`#/admin`, sección «Promociones»), no el dashboard del restaurante.
- Alcance: **crear + listar + stats + editar + finalizar**. Sin borrar (el backend no tiene `DELETE`); editar cambia fecha de fin y presupuesto, finalizar cierra la campaña.
- Formulario **completo con defaults**: tipo `boost_visibilidad`, CPC 0,20, puntos extra 0, fechas hoy → +30 días y presupuesto 300 €. Tipo/CPC/puntos van en «Opciones avanzadas».

## Hecho
- `src/services/api.js`: `promotionsApi.crear`, `actualizar` y `list` con `auth: true` (para ver presupuesto/gasto).
- `src/models/promocionesModel.js`: defaults, validación, edición, fechas de Madrid, estados y errores de API. Test `test/promocionesModel.test.js` (12 casos).
- `src/components/dashboard/PromocionesPanel.jsx`: lista, alta, stats, **editar** (fecha de fin y presupuesto) y **finalizar** con confirmación.
- `src/components/ops/OpsPromociones.jsx`: buscador de restaurante (mismo patrón que `OpsOfertas`) y panel.
- `src/components/ops/OpsPanel.jsx`: sección `promociones` con icono `campaign`.
- i18n `promociones.*` y `ops.seccPromociones` en es/ca/en.
- `npm test`: **339/339**. `npm run build`: **OK**.
- Rama `feat/M15-promociones-panel`, commits `a6d61d2`, `c14e4c0` y el de editar/finalizar, pusheada. PR: https://github.com/ruben-munoz-bl/restaurante-mira-frontend/pull/new/feat/M15-promociones-panel (no hay `gh` instalado para abrirlo desde aquí).

## Reparto con Codex (M15)
- **Codex (PC 1)**: rama `feat/M15-promociones-api` — `PUT /v1/promotions/:id` para editar `fechaFin`/`presupuestoTotal`/`estado` y 409 que ignore las caducadas o las finalice al crear. Ya está desplegado (ver abajo).
- **Ruben (PC 3)**: editor del panel en esta rama, ahora con crear/listar/stats/editar/finalizar.

## Backend resuelto y verificado (2026-10-23)
Izan desplegó los dos bloqueos y los he probado contra la API real (`https://mira-api-xveu.onrender.com`) con la cuenta admin de Ruben:
1. **Crear como admin**: ya no da 403 en restaurantes con dueño. El panel usa `comprobarPropiedadRestaurante` del backend.
2. **PUT `/v1/promotions/:id`**: responde y deja editar `fechaFin`/`presupuestoTotal` y `estado: "finalizada"`.

Prueba de extremo a extremo en el panel (`#/admin` → Promociones → Ari's Restaurant): crear 150 €, editar a 200 € y 1 dic, finalizar; el estado pasa a «Finalizada», desaparece el bloqueo y vuelve «Nueva promoción». Los datos de prueba (2 campañas) se borraron de Firestore al terminar; producción queda como estaba (`{"data":[]}`).

## Limitación conocida
La UI oculta «Nueva promoción» mientras haya una en `estado: activa` (aunque haya caducado), porque el backend la sigue contando. Ahora se puede resolver con «Finalizar», que sí cambia el estado.

## Pendiente
- Abrir el PR (no hay `gh` instalado) y que lo revise PC 1.
- Confirmar en la landing que una campaña activa aparece en «En promoción»; la prueba se hizo sobre la lista del panel y la API.
