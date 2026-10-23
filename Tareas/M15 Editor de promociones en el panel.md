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
- Alcance: **crear + listar + stats**. Sin editar/desactivar (el backend no tiene `PUT`/`DELETE`).
- Formulario **completo con defaults**: tipo `boost_visibilidad`, CPC 0,20, puntos extra 0, fechas hoy → +30 días y presupuesto 300 €. Tipo/CPC/puntos van en «Opciones avanzadas».

## Hecho
- `src/services/api.js`: `promotionsApi.crear` y `list` con `auth: true` (para ver presupuesto/gasto).
- `src/models/promocionesModel.js`: defaults, validación, fechas de Madrid, estados y errores de API. Test `test/promocionesModel.test.js` (9 casos).
- `src/components/dashboard/PromocionesPanel.jsx`: lista, alta, stats y aviso cuando ya hay una campaña activa.
- `src/components/ops/OpsPromociones.jsx`: buscador de restaurante (mismo patrón que `OpsOfertas`) y panel.
- `src/components/ops/OpsPanel.jsx`: sección `promociones` con icono `campaign`.
- i18n `promociones.*` y `ops.seccPromociones` en es/ca/en.
- `npm test`: **336/336**. `npm run build`: **OK**.
- Rama `feat/M15-promociones-panel`, commit `a6d61d2`, pusheada. PR: https://github.com/ruben-munoz-bl/restaurante-mira-frontend/pull/new/feat/M15-promociones-panel (no hay `gh` instalado para abrirlo desde aquí).

## Reparto con Codex (M15)
- **Codex (PC 1)**: rama `feat/M15-promociones-api` — `PUT /v1/promotions/:id` para editar `fechaFin`/`presupuestoTotal`/`estado` y 409 que ignore las caducadas o las finalice al crear. Avisará en esta nota (ver [[Codex - Cierre Reserva con carta]]).
- **Ruben (PC 3)**: editor del panel en esta rama, de momento solo crear/listar/stats. Cuando llegue el `PUT`, añadir editar y cerrar.

## Dependencias de backend (importante)
1. **Crear como admin**: en `origin/main`, `crearPromocion` comprueba `rest.ownerUid !== uid` y devuelve **403 incluso al admin**. La rama `fix/ia-respuesta-idioma` (cambios sin commitear) lo sustituye por `comprobarPropiedadRestaurante({ uid, role, … })`, que deja pasar al admin. **Hasta que eso se despliegue, el admin solo podrá crear en restaurantes sin `ownerUid`.**
2. **Ciclo completo**: `feat/M15-promociones-api` de Codex añadirá el `PUT`; sin él una campaña mal creada o caducada no se puede cerrar y bloquea las siguientes con 409. No se ha tocado el backend.

## Limitación conocida
La UI avisa y oculta «Nueva promoción» mientras haya una en `estado: activa` (aunque haya caducado), porque el backend la sigue contando.

## Pendiente
- Probar en local con una sesión admin real (no hay credenciales en el repo): crear una campaña, verla en la lista y en «En promoción».
- Abrir el PR (no hay `gh` instalado) y que lo revise PC 1, sobre todo por la dependencia de ownership.
- Cuando Codex entregue `PUT /v1/promotions/:id`, añadir editar/cerrar en el panel.
