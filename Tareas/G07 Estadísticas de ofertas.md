---
estado: libre
estado_api: hecho
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
- Backend, rama `feat/G07-estadisticas-api` (`2b6f5b6`), **API hecha y aprobada por Codex**, pendiente de unión y despliegue: `POST /v1/ofertas/:id/avisos` (la app suma un aviso por nivel) y `GET /v1/ofertas/estadisticas?restauranteId=&desde=&hasta=` (por día, máximo 92 días). Contadores en `ofertas_stats`, sin datos de usuario ni ubicación. 355/355 tests.
- **Mock para el PC 3** en el frontend (`feat/G01-contrato-ofertas`): las dos rutas, con 14 días de datos de ejemplo en el restaurante `1`. Ya se pueden montar los gráficos con `npm run mock-api`.
- PC 2: llamad a `POST /v1/ofertas/:id/avisos` cada vez que mostréis un aviso de nivel (G06).

## Revisión de Claude (PC 1) de la rama `feat/G07-estadisticas-ofertas` (2026-10-15)
`2332599`: `npm test` 202/202 y build correcto. Buen componente: KPI, barras por nivel, línea por fecha, privacidad explicada y la lógica pura (`ofertasStatsModel.js`) con tests.

Se programó contra `docs/coordenacion/CONTRATOS-ASUMIDOS.md`, y **el contrato oficial ya está publicado** en [[Contratos API]] (fase 3, «Estadísticas para el PC 3»), con otra forma. Hay que adaptarlo:
- **[P1] Forma de la respuesta.** La API real devuelve `{ data: [{ fecha, avisos: { 1000, 500, 100, 20 }, visitas, confirmadas }] }`, un elemento por día. No hay `porNivel` ni `porFecha`, y **no existen «llegadas por nivel»**: las llegadas son visitas confirmadas por día (el QR solo se genera a 20 m). Propuesta: `sanearEstadisticas` debe sumar `avisos` por nivel a partir de los días para la gráfica de barras, y la línea por fecha usar `avisos` (suma de niveles), `visitas` y `confirmadas`. Tasa = `confirmadas / avisos[1000]` (o la que acordéis).
- **[P1] No mostrar datos inventados a un restaurante.** `OfertasStats` usa `estadisticasMock()` cuando `datos` falta. En producción, un restaurante sin datos (o con un error de red) vería números falsos. El mock debe vivir solo en el mock de desarrollo (`npm run mock-api`, que ya sirve esta ruta con 14 días de ejemplo en la rama `feat/G01-contrato-ofertas`). Sin datos → «sin datos»; con error → mensaje de error y reintentar.
- **Falta montarlo:** no está importado en ningún sitio. Llamar a `GET /v1/ofertas/estadisticas?restauranteId=&desde=&hasta=` (por ejemplo, los últimos 30 días) desde el panel de ofertas del dashboard, con el `api` de `httpClient`.
- Menor: `formatearFecha` usa `new Date("AAAA-MM-DDT00:00:00")` en hora local; con zonas UTC− puede pintar el día anterior. Mejor `T12:00:00`.

## Revisión de Codex de la API (2026-10-15)
- Revisado `c5a299e` en `codex-review-G07`, sin modificar código. `npm test`: **354/354**. Permisos de dueño/admin, validación de fechas (92 días inclusivos), forma de respuesta, ceros por nivel y contadores agregados sin ubicación ni UID: correctos en esta revisión. Reglas de `ofertas_stats` cerradas; consulta simple sin índice compuesto.
- **[P2] El límite de avisos se reinicia al cambiar de oferta.** `mira-api/src/modules/ofertas/routes.js:50` usa la clave predeterminada de `rateLimit`, que incluye `req.path` (con el ID concreto). El contrato y docs indican límite por IP, pero actualmente son 30/min por IP **y oferta**. En esta ruta pública con escrituras, recorrer IDs permite superar el límite previsto.
- Reproducción con Supertest y Firestore simulado, misma IP y dos ofertas activas: 30 POST a `/v1/ofertas/oferta-a/avisos` con `{ nivel: 20 }` → 204; el siguiente a A → 429; otro a `/v1/ofertas/oferta-b/avisos` → **204**, cuando debería ser 429. No hace falta autenticación. Reproducción independiente de la suite, con aserciones.
- Propuesta: pasar una clave específica estable para avisos, por ejemplo `req => "ofertas-avisos:" + req.ip`, sin el ID; añadir regresión que alterne ofertas y compruebe 429 al superar 30 en total. Mantener separados los límites de las otras rutas.
- **Resultado: API en revisión, pendiente del P2.** El estado general de G07 sigue correspondiendo a la UI del PC 4; sus observaciones de Claude son independientes. La integración G08/`contarVisita` queda a cargo de Claude según lo acordado.

## Corrección del P2 de Codex en la API (2026-10-15, Izan + Claude)
`2b6f5b6` en `feat/G07-estadisticas-api`: `POST /v1/ofertas/:id/avisos` usa la clave `ofertas-avisos:<ip>`, sin el id, así que el límite es de 30/min por IP para todas las ofertas juntas. Regresión: 30 avisos alternando dos ofertas y, después, 429 en las dos. `npm test` 355/355.

## Segunda revisión de Codex de la API (2026-10-15)
- **Estado API: hecho. Aprobado `2b6f5b6`** en `feat/G07-estadisticas-api`; P2 cerrado, sin nuevos hallazgos en la corrección.
- La clave `ofertas-avisos:<ip>` agrupa todas las ofertas, separada de los límites de otras rutas. La regresión alterna dos ofertas: los primeros 30 avisos devuelven 204 y después ambas devuelven 429.
- Verificado en `codex-review-G07`: `npm test` **355/355**, worktree limpio y `git diff --check` correcto; sin cambios de código de Codex.
- Claude puede unir estadísticas con G08 y desplegar tras configurar `VISITAS_SECRET` y publicar el índice de visitas **sin `--force`**. La integración debe conservar su mock y adaptar G08 a `ofertas.contarVisita`, según lo acordado.
- Esta aprobación cierra la **parte API**. La UI del PC 4 y sus observaciones de Claude conservan su estado independiente; por eso el estado general de G07 no se marca hecho.

## API desplegada (2026-10-15, Izan + Claude)
En `main` del backend `808ff00` (Render): `GET /v1/ofertas/estadisticas` (401 sin sesión) y `POST /v1/ofertas/:id/avisos` funcionan en producción. Falta la UI del PC 4 adaptada al contrato oficial.
