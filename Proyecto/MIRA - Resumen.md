# MIRA: resumen

Web de restaurantes de Cataluña: buscador, mapa, reservas, reseñas, puntos, dashboard de empresa y panel de admin.

## Repos
- **Frontend** (github.com/ruben-munoz-bl/restaurante-mira-frontend): Vite 5 + React 18, Zustand, Leaflet, Recharts, i18n es/ca/en. Rutas por hash (`#/cuenta`, `#/admin`...).
  - `mobile/`: app Expo 57 / React Native con unas 20 pantallas en `src/app/` (expo-router). Tiene `expo-notifications` instalado, pero no se usa todavía. No hay `eas.json`.
- **Backend** (`restaurante-mira-backend`):
  - `mira-api/`: Express + Firestore (Admin SDK), endpoints `/v1/...`, desplegado en Render. No tiene nada de push todavía.
  - `yelp-connection/`: scripts de datos (Yelp → Faker → Firestore) y recordatorios.

## Firebase
Proyecto `restaurante-mira-18e0c` (plan Spark). El frontend solo usa Auth; el ID token va como Bearer a mira-api.

## Arrancar el frontend
- `npm install` y `npm run dev` (puerto 5173). En `.env`, `VITE_API_URL`.
- `npm test` para los tests y `npm run build` antes de dar algo por terminado.
- Las normas para agentes del repo están en su `AGENTS.md` (bugs en `docs/bugs/`).

Ver también: [[Plan app móvil y notificaciones]]
