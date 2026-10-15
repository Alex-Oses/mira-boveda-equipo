---
estado: libre
responsable:
pc: PC 3
fase: 2
depende_de: [G04, G05]
rama:
---
# G07 Estadísticas de ofertas

## Objetivo
Que el restaurante vea cómo funcionan sus ofertas.

## Pasos
- Gráficos con Recharts: avisos mostrados por nivel y (en la fase 3) llegadas confirmadas.
- Acordar con el PC 1 qué contadores agregados guarda la API, siempre sin datos de ubicación.

## Terminada cuando
El dashboard muestra los datos de prueba por nivel y por fecha.

## Parte API lista (2026-10-15, Izan + Claude, PC 1)
Contrato en [[Contratos API]] (fase 3, «Estadísticas para el PC 3»).
- Backend, rama `feat/G07-estadisticas-api` (`c5a299e`), pendiente de revisión de Codex y despliegue: `POST /v1/ofertas/:id/avisos` (la app suma un aviso por nivel) y `GET /v1/ofertas/estadisticas?restauranteId=&desde=&hasta=` (por día, máximo 92 días). Contadores en `ofertas_stats`, sin datos de usuario ni ubicación. 354/354 tests.
- **Mock para el PC 3** en el frontend (`feat/G01-contrato-ofertas`): las dos rutas, con 14 días de datos de ejemplo en el restaurante `1`. Ya se pueden montar los gráficos con `npm run mock-api`.
- PC 2: llamad a `POST /v1/ofertas/:id/avisos` cada vez que mostréis un aviso de nivel (G06).
