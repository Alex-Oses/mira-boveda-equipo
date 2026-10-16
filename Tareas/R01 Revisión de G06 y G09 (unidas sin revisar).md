---
estado: libre
responsable: Claude + Codex (PC 1)
pc: PC 1
depende_de: []
rama: main (frontend), PR #5
---
# R01 Revisión de G06 y G09 (unidas sin revisar)

`feat/mira-app` se unió a `main` (PR #5) sin la revisión cruzada. Revisar ahora en `main`, en worktrees propios:
- **Codex:** llamadas a la API (`/v1/ofertas/cerca` con throttle, deduplicación de avisos y `POST /v1/ofertas/:id/avisos`, `POST /v1/visitas` y sondeo de `GET /v1/visitas/:id`, registro push), autenticación y manejo de errores (422 LEJOS, 410, 409).
- **Claude:** pantallas (avisos, `Llegada`, QR), segundo plano en Android (`ubicacionFondo.js`, permisos y textos de `app.json`), privacidad (que la posición no salga más que en `cerca` y `visitas`) y que respete el consentimiento.
Hallazgos como fixes pequeños en ramas nuevas desde `main`, o como tareas para Alex si son grandes. No reescribir su trabajo.
