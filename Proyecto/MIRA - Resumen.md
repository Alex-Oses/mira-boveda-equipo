# MIRA: resumen

Actualizado: 2026-10-14.

Plataforma de restaurantes de Cataluña (Barcelona, Girona, Lleida y Tarragona) con datos de Yelp y Google Places. Incluye:
- **Buscador** con filtros y mapa (Leaflet).
- **Ficha del restaurante**: reserva, carta en libro, parkings y meteo.
- **MIRA JAM**: reserva en grupo con votación en directo (SSE).
- **MIRA Points**: racha diaria, ruleta e invitaciones por correo (Brevo).
- Lista de espera, favoritos, mensajes, tickets y reseñas.
- **Agente IA «MIRA»** con Gemini.
- **Paneles**: dashboard del restaurante (con parte fiscal) y panel Ops/Admin.
- Idiomas es/ca/en y modo claro y oscuro.

## Repos y despliegue
| Parte | Repo | Despliegue |
|---|---|---|
| Web | ruben-munoz-bl/restaurante-mira-frontend | Vercel: restaurante-mira-frontend.vercel.app (`main`) |
| API | ruben-munoz-bl/restuarante-mira-backend (sic) | Render gratis: mira-api-xveu.onrender.com (`main`) |
| Móvil | `mobile/` dentro del frontend | Expo 57 / RN 0.86 |

- **Firebase:** proyecto `restaurante-mira-18e0c` (Auth + Firestore). Sin Storage, porque exigiría el plan Blaze.
- **Orden de despliegue:** primero el backend y después el frontend.
- **Limitaciones de Render gratis:** bloquea el SMTP (por eso se usa Brevo por HTTPS) y el servicio se duerme cuando no se usa.

## Stack
- **Frontend:** Vite 5 + React 18, Zustand, Firebase SDK 12, Leaflet, Recharts.
  - Enrutado por hash (`#/ruta`), sin react-router.
  - La lógica pura va en `models/` y `services/`, las vistas en `components/`, el estado en `stores/`, las traducciones en `i18n/` (`useT`) y los temas en `styles/tokens.css`.
- **Backend (`mira-api/`):** Node 20, Express 4, firebase-admin, zod, helmet, pino, node-cron y Gemini.
  - Los routers cuelgan de `/v1`. Cada módulo tiene `src/modules/<nombre>/routes.js` (con zod) y `service.js`.
  - Middlewares: `verifyFirebaseAuth`, `authorize("admin")`, `validate` y `rateLimit`.
  - Documentación de los endpoints en `mira-api/docs/api.md`.
- **Scripts de datos:** `yelp-connection/`.

## Comandos
- Frontend: `npm run dev`; `npm run dev:mock` + `npm run mock-api` (puerto 3001); `npm test` y `npm run build`, que tienen que pasar antes de cerrar algo.
- Backend (`mira-api/`): `npm run dev` y `npm test`.

## Normas de trabajo en el código
Están en el `AGENTS.md` de cada repo:
- Bugs en `docs/bugs/`. Estados: pendiente → en curso (agente) → en revisión → hecho.
- Una rama por bug o por tanda. Cada agente trabaja en su propio git worktree.
- Revisión cruzada antes de hacer merge.
- Commits en español con Conventional Commits.
- Cambios mínimos, imitando el estilo del código que rodea el cambio.

## Estado (2026-10-14)
- **En producción:** frontend `95f913f`, backend `fe3e95c`. Hechos los bugs hasta el BUG-122.
- **Pendiente:**
  - BUG-123: un soft bounce con "MX inexistente" debería marcar la invitación como inexistente.
  - La rama `feat/fotos-places` está sin revisar.
  - Hay un test intermitente: `POST /v1/reviews` sin token → 401.

Siguiente proyecto: [[Ofertas por geolocalización]].
