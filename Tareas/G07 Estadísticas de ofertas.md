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

## Revisión de Claude (PC 1) de la rama `feat/G07-estadisticas-ofertas` (2026-10-15)
`2332599`: `npm test` 202/202 y build correcto. Buen componente: KPI, barras por nivel, línea por fecha, privacidad explicada y la lógica pura (`ofertasStatsModel.js`) con tests.

Se programó contra `docs/coordenacion/CONTRATOS-ASUMIDOS.md`, y **el contrato oficial ya está publicado** en [[Contratos API]] (fase 3, «Estadísticas para el PC 3»), con otra forma. Hay que adaptarlo:
- **[P1] Forma de la respuesta.** La API real devuelve `{ data: [{ fecha, avisos: { 1000, 500, 100, 20 }, visitas, confirmadas }] }`, un elemento por día. No hay `porNivel` ni `porFecha`, y **no existen «llegadas por nivel»**: las llegadas son visitas confirmadas por día (el QR solo se genera a 20 m). Propuesta: `sanearEstadisticas` debe sumar `avisos` por nivel a partir de los días para la gráfica de barras, y la línea por fecha usar `avisos` (suma de niveles), `visitas` y `confirmadas`. Tasa = `confirmadas / avisos[1000]` (o la que acordéis).
- **[P1] No mostrar datos inventados a un restaurante.** `OfertasStats` usa `estadisticasMock()` cuando `datos` falta. En producción, un restaurante sin datos (o con un error de red) vería números falsos. El mock debe vivir solo en el mock de desarrollo (`npm run mock-api`, que ya sirve esta ruta con 14 días de ejemplo en la rama `feat/G01-contrato-ofertas`). Sin datos → «sin datos»; con error → mensaje de error y reintentar.
- **Falta montarlo:** no está importado en ningún sitio. Llamar a `GET /v1/ofertas/estadisticas?restauranteId=&desde=&hasta=` (por ejemplo, los últimos 30 días) desde el panel de ofertas del dashboard, con el `api` de `httpClient`.
- Menor: `formatearFecha` usa `new Date("AAAA-MM-DDT00:00:00")` en hora local; con zonas UTC− puede pintar el día anterior. Mejor `T12:00:00`.
