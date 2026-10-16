---
estado: en curso
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

## Revisión Codex terminada · 2026-10-16
Base revisada: frontend `origin/main` **cc31bab** (incluye merge PR #5 `0900b00`), worktree propio `codex-R01-review`; contrato contrastado con backend `origin/main` **808ff00**. Sin reescribir G06/G09 ni tocar el trabajo de C02. Parte de Claude pendiente de registrar por su responsable.

### Hallazgos y fixes pequeños en revisión local
1. **[P2] Sondeos solapados y respuesta de visita antigua sobre un QR nuevo.** `src/hooks/useLlegada.js:57-62` y copia móvil: intervalos cada 4 s sin exclusión de petición pendiente ni guardia de limpieza; GET antiguo puede sobrescribir la nueva visita al renovar QR. Además se ignoran 401/403/404 y sigue mostrándose el QR. Fix **fix/R01-sondeo-visitas**, **4633d80**, worktree `codex-R01-qr`: una petición en vuelo por efecto, se ignoran respuestas tras cleanup y se retira QR/visita ante errores terminales. Fallos temporales se reintentan. 18 tests de hooks con red diferida, errores de POST y reloj controlado; contra main fallan 8, con fix pasan 18. Suite **254/254**, build correcto, lint móvil correcto. No incluye rediseño de ciclo de sesión/pantalla (ver A02).
2. **[P2] La web cuenta y deduplica avisos que nunca muestra.** `src/hooks/useAvisosOfertas.js:31-38`: cinco candidatos nuevos producen cinco POST /avisos, aunque slice limita la lista a tres; los otros dos quedan vistos. Fix **fix/R01-avisos-visibles**, **43b24f0**, worktree `codex-R01-avisos`: seleccionar máximo tres nuevos antes de guardar vistos y contar; los restantes pueden aparecer en la siguiente consulta. Suite **237/237**, build correcto. La app requiere revisar el caso sin permiso de notificaciones (A03).

### Hallazgos de mayor alcance → tareas para Alex
- **[P1] Registro/borrado push y cambio de cuenta:** [[R01-A01 Ciclo push ligado a la sesión]]. Un registro en vuelo puede finalizar después de cerrar sesión; el borrado elimina el ID local antes de confirmar DELETE y oculta el fallo. El token remoto puede quedar asociado a una cuenta tras salir; requiere ciclo de sesión, reintentos y prueba concurrente, no solo reordenar dos líneas.
- **[P2] Llegada ligada a sesión, restaurante y pantalla:** [[R01-A02 Ciclo de llegada y navegación]]. El hook no recibe UID/foco; pedidaAuto no se reinicia al cambiar restaurante, y no invalida POST en vuelo ni QR al cambiar de cuenta. En app el intervalo solo termina al limpiar efecto/desmontar, sin comprobar si la ruta sigue visible. Necesita integrar navegación y auth en ambas plataformas.
- **[P2] Deduplicación móvil entre primer/segundo plano:** [[R01-A03 Deduplicación y avisos móviles]]. Primer plano lee el shim de localStorage y segundo plano escribe AsyncStorage directamente, sin refrescar la copia de memoria; puede repetir notificación/POST al volver a primer plano. Además se marca/contabiliza antes de saber si se mostró notificación local; más de tres candidatos con permiso de notificación denegado cuentan avisos no visibles.

### Contratos y errores comprobados
- `ofertasCercaApi`: GET público con lat/lng codificados y `auth: false`; POST /ofertas/:id/avisos público, solo `{ nivel }`, 204 soportado por httpClient. Throttle 30 s **o** 50 m, exclusión en vuelo en cada hook; dedup por restaurante/nivel/día y no repite un nivel peor. No es exclusión global entre instancias ni entre foreground/background (A03).
- `visitasApi`: POST autenticado `{ restauranteId, lat, lng }`, GET autenticado con ID codificado; ambos httpClient conservan `status` y `data`. POST 422 se refleja como `lejos` con `data.distanciaM`, 401 `sinSesion`, 404 `sinOferta`, 503 `noDisponible`. Tests ejecutados en los hooks reales mediante primitivas controladas, sin dispositivo.
- **409 QR_USADO y 410 QR_CADUCADO pertenecen a POST /visitas/validar**, no a crear/consultar. El panel `escanerModel.js:39-56` los transforma en `ya_usado`/`caducado`, con tests existentes en verde. GET usa estado `caducada`, no HTTP 410. No se ha cambiado el contrato.
- Push: POST tiene los cuatro campos del contrato; idioma/plataforma admitidos y Bearer del usuario actual. Logout invoca DELETE antes de signOut en el camino nominal; la carrera y fallo de red siguen en A01. Permisos solicitados al login: ya asignados a C02.
- Para la revisión de privacidad de Claude: `mobile/src/services/ubicacionFondo.js:49` **persiste coordenadas** bajo `mira:avisosFondoUltima`. Se registra aquí para su parte; no se modifica ese módulo.

### Límites y estado de entrega
- Revisiones de código + tests automatizados; sin prueba física en móvil ni validación E2E contra producción. Proyecto móvil no contiene tsconfig.json: `tsc --noEmit` global devuelve ayuda/error; comprobación dirigida de sintaxis JS del hook con tsc correcta (comando en docs/revisiones/R01-sondeo-visitas.md). No se presenta como typecheck integral.
- Las dos ramas están comprometidas y limpias, pendientes revisión cruzada; sin merge ni despliegue. Publicación pendiente: auto-review rechazó push de C01 por no considerar explícitamente autorizada la exportación al remoto; se solicita autorización antes de publicar las tres ramas. No se ha creado PR remoto.
