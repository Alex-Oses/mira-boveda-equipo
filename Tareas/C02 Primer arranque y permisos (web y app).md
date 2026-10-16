---
estado: en revisión
responsable: Izan (Claude), PC 1
pc: PC 1
depende_de: [C01]
rama: feat/C02-primer-arranque
---
# C02 Primer arranque y permisos (web y app)

## Objetivo
Pantalla de bienvenida en pasos (cookies → ubicación → notificaciones) y permisos que se guardan y no se vuelven a pedir. Decisiones en [[Permisos y primer arranque]].

## Pasos
- Servicio común de consentimientos (dispositivo + perfil si hay sesión; gana el más reciente), en web y app.
- Primer arranque en 3 pasos en web y app (también invitados), reutilizando el banner de cookies como paso 1.
- «Activar ubicación» (Mi cuenta, tarjeta de la portada de la app) lanza el permiso del sistema directamente; si está bloqueado, guía o ajustes.
- App: consultar permisos al arrancar (`get…`) y pedirlos solo al aceptar; quitar la petición automática de notificaciones de `useAuth` (se pide en el paso 3 o en Mi cuenta). Reflejar en Mi cuenta si el sistema lo revocó.
- Web: notificaciones (Web Push, T07) en el paso 3; en iOS, solo si la PWA está instalada (si no, explicar cómo instalarla).
- Tests de la lógica pura y prueba en navegador; prueba en móvil con Alex.

## Terminada cuando
En una instalación limpia se ve la bienvenida una vez; aceptar lanza el permiso del sistema; al reabrir no vuelve a pedir nada; las decisiones aparecen en Mi cuenta y en el perfil.

## Hecho (2026-10-16, Izan + Claude)
Rama `feat/C02-primer-arranque` del frontend: web `bd1a942`, app `d1a526e`.
- **Lógica común** (`models/consentimientosModel.js`, igual en web y app): `{ estado, version, fecha }` por tipo; la ubicación usa la misma clave que G03 (quien ya aceptó no vuelve a ver la pregunta); `fusionar` (gana el más reciente entre dispositivo y perfil) y `pasoPendiente`. 5 tests.
- **Servicio** `consentimientosApi.js` (web y app): guarda en el dispositivo y, con sesión, `PUT /v1/users/me` con `consentimientos` (C01). **Mientras C01 no esté desplegado, el perfil falla en silencio y manda el dispositivo.** Al iniciar sesión se sincroniza.
- **Web:** `PrimerArranque` sustituye al banner de cookies: paso 1 cookies (el mismo banner), 2 ubicación, 3 notificaciones, con el mismo aspecto; «Aceptar» lanza el permiso del navegador; sin soporte de Web Push el paso 3 no aparece; en iPhone sin instalar enlaza a cómo instalar. «Activar ubicación» en Mi cuenta lanza el permiso directo (sin modal). Las notificaciones de Mi cuenta registran la decisión. Sin sesión solo se pide el permiso; el token se registra al iniciar sesión. Probado en el navegador con GPS y permiso simulados: los 3 pasos, se guardan las 3 decisiones (sin coordenadas) y **al recargar no pregunta nada**.
- **App:** el mismo `PrimerArranque` (Modal) en `AppShell`, en lugar de la tarjeta de la portada. **Bug de «vuelve a preguntar»:** el seguimiento ahora solo **consulta** el permiso al abrir (`getForegroundPermissionsAsync`) y solo lo **pide** tras un toque; el aviso «concedido» hace que las demás pantallas vuelvan a consultar. Errores nuevos: `permisoPendiente` («Dar permiso») y `permisoBloqueado` («Abrir ajustes», `Linking.openSettings`). Notificaciones: `useAuth` ya no pide el permiso al iniciar sesión; solo registra el dispositivo si estaban aceptadas (sin preguntar). 3 tests nuevos del arranque (8 en total). `npx expo lint` correcto.
- `npm test` 244/244 y build correcto.
- **Falta:** revisión de Codex; probar el APK en un móvil (Alex): instalación limpia → 3 pasos → cerrar y abrir → no pregunta nada; denegar y bloquear el permiso → «Dar permiso» / «Abrir ajustes». Y C01 desplegado para el sincronizado con el perfil.


## Revisión Codex · 2026-10-16 · cambios solicitados
Revisado `d1a526e` (incluye web `bd1a942`), en worktree propio `codex-C02-review`, contra C01 `90cd7e7`. No he modificado la implementación de C02.

1. **[P1] Sincronización no ligada a la sesión que inició el GET.** `src/services/consentimientosApi.js:43-50` y `mobile/src/services/consentimientosApi.js:41-47`: si GET de A está pendiente y se cambia a B, su respuesta se fusiona con el dispositivo ya usado por B y el siguiente PUT resuelve el Bearer de B. Los efectos de PrimerArranque/useAuth tampoco invalidan la operación ni su finally al cambiar UID. Puede aplicarse el consentimiento de la cuenta anterior y registrar push en el contexto nuevo. Reproducido ejecutando ambos servicios reales en VM con GET diferido y cambio A→B: se aplica aLocal del GET de A y el PUT posterior sale bajo B. Ligar toda sincronización/continuación a UID y generación de sesión; descartar resultados al salir/cambiar y probar ambos órdenes de respuesta. Coordinar la continuación de registro push con R01-A01, sin posponer la protección de consentimientos.
2. **[P2] Volver de Ajustes no reconsulta el permiso de ubicación.** `mobile/src/hooks/useUbicacion.js:48,69`: abrirAjustes solo llama Linking.openSettings, y el efecto solo depende de activa/intento. Con Mi cuenta montada, permisoBloqueado → conceder en Ajustes → volver mantiene el error y no arranca watch. Tampoco se refleja una revocación externa al volver. Reproducido con hook real y primitivas controladas: una sola consulta, cero watches tras conceder y volver. Suscribirse al regreso a primer plano/foco y consultar con get (nunca request); invalidar posición/seguimiento si se ha revocado. Añadir prueba de ida y vuelta a Ajustes y prueba física.
3. **[P2] Notificaciones rechazadas o pendientes de permiso en un móvil nuevo no pueden activarse desde Mi cuenta.** `mobile/src/components/PrimerArranque.jsx:75-78`: cualquier decisión hace desaparecer el paso. Tras Ahora no, el único llamador de pedirPermisoNotificaciones es ese paso oculto; Mi cuenta solo ofrece PreferenciasAvisos (categorías T10), que no pide permiso, guarda el consentimiento ni registra el token. Con un aceptado traído del perfil y permiso OS sin conceder pasa igual: useAuth consulta y no registra, sin botón para resolverlo. Añadir estado del consentimiento y permiso OS en Mi cuenta, acción explícita para aceptar/pedir o abrir ajustes si bloqueado, y para rechazar/desactivar. Probar rechazo→aceptación posterior y cuenta con aceptado en móvil nuevo.

### Comprobaciones correctas
- PUT `/v1/users/me` manda solo `{ consentimientos }`, con objetos estado/version/fecha compatibles con C01. Cookies sigue en su campo independiente. Fusión por campo de decisiones vigentes con fechas distintas sigue la regla del más reciente. C01 no requiere cambios de contrato.
- iniciarSeguimiento usa getForegroundPermissionsAsync por defecto y request únicamente con pedir=true. La notificación onConcedido está limitada al camino de petición concedida; las demás instancias reciben reintento sin pedir otra vez. permisoPendiente y permisoBloqueado distinguen canAskAgain, y las operaciones tardías de seguimiento comprueban sigueActiva.
- useAuth comprueba notificaciones aceptadas después de sincronizar y registrarPush usa pedir=false por defecto; no solicita permiso de notificaciones al login. Sigue pendiente la coordinación de carreras de R01-A01.
- `npm test` **244/244**, `npm run build` correcto (aviso de tamaño de chunk ya existente), `npm run lint` móvil correcto. Reproducciones adicionales: `/tmp/mira-C02-review.mjs` (servicios web/app y hook móvil reales con API/React/Location controlados). No son pruebas en dispositivo. Sin typecheck integral por falta de tsconfig.json en el proyecto móvil.
- Pendiente APK con Alex y nueva revisión tras corregir 1–3. C02 queda en revisión, **no aprobada para merge**. C01 ya ha sido publicado por Claude; sigue pendiente cerrar su P2 de compatibilidad del data test (complemento frontend fix/C01-data-test-perfil, aae000f, ver nota C01), integración y despliegue. El cliente podrá sincronizar con el contrato actual cuando esté disponible y ejecute una nueva sincronización.

## Reconfirmación de revisión · Codex · 2026-10-16
Tras `git fetch`, la rama remota feat/C02-primer-arranque continúa exactamente en **d1a526e** (web bd1a942 incluida). La revisión anterior sigue vigente: **cambios solicitados, no aprobar todavía para merge**.
- Reejecutada `/tmp/mira-C02-review.mjs`: reproducidos de nuevo el GET de A aplicado tras cambiar a B (web y app) y el estado permisoBloqueado que no se actualiza al volver de Ajustes. También sigue sin existir una acción de permiso/consentimiento de notificaciones en Mi cuenta de la app; las preferencias T10 no la sustituyen.
- Lo revisado sobre solo get al abrir, request tras un toque, onConcedido y canAskAgain es correcto en el camino normal. useAuth solo registra con consentimiento aceptado y registrarPush no solicita permiso por defecto. Coordinar el UID/generación de sincronización y su continuación con R01-A01, cuya rama debe partir de C02 ya corregido e integrado en main.
- Sin cambios nuevos de código en C02: se mantienen los resultados de la revisión del mismo commit (244/244 tests, build y lint móvil correctos). Sin prueba física del APK ni validación integral de tipos.
- C01 **ya tiene el P2 corregido y publicado en f9f2ba4**, en la misma rama feat/C01-consentimientos (402/402 tests). Ya no requiere integrar fix/C01-data-test-perfil; esa propuesta anterior queda sustituida por quitar strict de la raíz según indicación de Izan. El PUT de consentimientos de C02 encaja con C01 sin ajustes de contrato. Integración/despliegue los realiza Izan después de cerrar la revisión de C02.

## Corrección de la revisión de Codex (2026-10-16, Izan + Claude)
`20a1ecb` en `feat/C02-primer-arranque`:
1. **[P1] Sincronización ligada a la sesión.** La secuencia pasa a `sincronizar()` (pura, en `consentimientosModel.js`, igual en web y app), que comprueba `sigueVigente()` después del GET y antes de cada escritura. Los servicios comparan el UID con el de Firebase en ese momento; PrimerArranque (web) y useAuth (app) invalidan al salir o cambiar de cuenta y solo registran push si terminó vigente. 4 tests: sesión vigente; cambio de cuenta antes del GET (no aplica ni sube nada); cambio después del GET (no sube con la sesión nueva); sin sesión o error de red.
2. **[P2] Volver de Ajustes.** `services/primerPlano.js` (`alVolverAPrimerPlano`, con AppState inyectado): al volver a primer plano, `useUbicacion` vuelve a **consultar** (nunca pide); si se concedió arranca el seguimiento, si se revocó lo para y borra la posición. 2 tests de ida y vuelta.
3. **[P2] Notificaciones activables desde Mi cuenta (app).** `PreferenciasAvisos` muestra siempre el estado (`models/notificacionesModel.js`: desactivadas / activas / falta permiso / bloqueado, a partir del consentimiento y del permiso real) con su acción: Activar o Dar permiso (pide permiso, guarda la decisión y registra el dispositivo), Abrir ajustes o Desactivar (rechaza y borra el token). Reconsulta al volver de Ajustes. Los interruptores de T10 siguen, solo con sesión. 3 tests.
- `npm test` 253/253, build correcto, `npx expo lint` correcto. Primer arranque web probado otra vez en el navegador (aceptar y rechazar).
- Sigue faltando la prueba en un móvil (Alex). Repetir la revisión de Codex.

## Segunda revisión de Codex · 2026-10-16 · 20a1ecb
Revisado **20a1ecb** después de fetch, en worktree propio codex-C02-review. **Todavía no aprobado**: se cierran P1 (cambio de cuenta durante GET) y P2 de ubicación al volver de Ajustes; el P2 de notificaciones en Mi cuenta está parcialmente corregido y quedan dos casos reproducidos.

### Cerrados
- `/tmp/mira-C02-review.mjs` actualizado a la firma sincronizarConsentimientos(uid, sigueVigente), conservando el script anterior en `/tmp/mira-C02-review-d1a526e.mjs`: GET de A diferido y cambio a B no escribe en dispositivo ni hace PUT, tanto web como app. Comprobación del UID de Firebase simulada en su frontera; modelo y servicios reales ejecutados. Los efectos invalidan su continuación al cambiar UID/salir; no registran push cuando la sincronización devuelve false.
- Hook móvil real con AppState/Location/React controlados: abrir Ajustes → conceder → volver consulta por get y arranca watch; revocar externamente → volver limpia posición y deja permisoBloqueado. Ningún request al abrir ni al volver. El registro/notificación onConcedido del camino de toque se conserva.
- El formato de PUT y la fusión por fecha siguen siendo compatibles con C01 f9f2ba4. C01 aprobado por Claude en su tercera revisión; no necesita cambios por esta revisión.

### Cambios solicitados pendientes (Mi cuenta / notificaciones)
1. **[P2] Rechazado + permiso bloqueado no ofrece salida a Ajustes.** `mobile/src/models/notificacionesModel.js:10-14` prioriza el rechazo y devuelve desactivadas aunque permiso sea denied/canAskAgain:false. `PreferenciasAvisos.jsx:51-57,69-74` conecta ese estado a Activar: vuelve a pedir, obtiene denied, guarda rechazado y queda en el mismo estado sin ofrecer Abrir ajustes. Es el caso normal después de denegar el diálogo del sistema en el paso 3 (que guarda rechazado). Al pulsar Activar debe reconocer el bloqueo y permitir ir a Ajustes; no encerrar al usuario en solicitudes que el sistema ya no muestra. Añadir regresión con rechazado + denied/canAskAgain:false, no solo aceptado + bloqueado.
2. **[P2] Conceder notificaciones en Ajustes muestra activas sin registrar el dispositivo.** `mobile/src/components/PreferenciasAvisos.jsx:41-45`: reconsultar solo hace getPermissions y setPermiso. Si el consentimiento ya era aceptado pero el permiso bloqueado (p. ej. perfil traído a otro móvil), useAuth ya intentó registrar sin permiso y devolvió null. Abrir ajustes → conceder → volver cambia el texto a activas y el botón a Desactivar, pero no ejecuta registrarPush; useAuth no vuelve a ejecutarse al no cambiar UID. No habrá push remoto hasta otra entrada/arranque. Tras recuperar permiso, registrar sin pedirlo de nuevo cuando haya sesión y consentimiento aceptado; coordinar la vigencia de la sesión con R01-A01. Añadir prueba de que la transición dispara el registro efectivo y no solo cambia la etiqueta.

### Evidencia y validación
- `/tmp/mira-C02-notificaciones-review.mjs`: ejecuta **PreferenciasAvisos.jsx real**, transformado con Babel para JSX, con React/servicios nativos simulados. Reproducidos ambos casos: Activar permanece desactivadas sin abrir Ajustes; y bloqueado→Ajustes→granted muestra activas con **0 llamadas a registrarPush**. No se han modificado los archivos de implementación.
- **npm test 253/253**, **npm run build correcto**, **npm run lint móvil correcto**. Los 9 tests añadidos pasan, pero no cubren estos dos flujos completos del componente. Sin prueba física de APK; sigue pendiente con Alex. R01-A01 mantiene el alcance de carreras y limpieza de tokens ya anotado.
- Resultado: C02 queda **en revisión con cambios solicitados**; cerrar estos dos casos antes del merge conjunto. No se han unido ni desplegado ramas.

## Corrección de la segunda revisión de Codex (2026-10-16, Izan + Claude)
`73cdf9c` en `feat/C02-primer-arranque`:
1. **Rechazado + bloqueado → «Abrir ajustes».** `estadoNotificaciones` da prioridad al bloqueo del sistema (`canAskAgain: false` sin permiso) sobre la decisión guardada, así que tras denegar el diálogo del paso 3 el botón ya no es «Activar» sin efecto, sino «Abrir ajustes». Con permiso concedido pero rechazado sigue «desactivadas» (Activar no muestra diálogo y queda aceptado).
2. **Conceder en Ajustes registra el dispositivo.** `PreferenciasAvisos` recuerda el permiso anterior; al reconsultar, `debeRegistrarAlRecuperar(antes, ahora, consentimiento)` (aceptado y paso de no concedido a concedido) llama a `registrarPush` sin pedir permiso. La primera consulta no registra (de eso se ocupa useAuth al entrar). La vigencia de la sesión del registro sigue en el alcance de R01-A01.
- El botón: los textos en una tabla fija (`ACCIONES`) y la acción en `pulsar()` (el lint de React marcaba el objeto con callbacks).
- Tests en `test/notificacionesModel.test.js` (5): incluye rechazado + bloqueado y la transición que registra frente a las que no. **No hay test del componente montado** (el proyecto móvil no tiene entorno de render en node); lo cubre el modelo y la prueba en el APK.
- `npm test` 255/255, build y `npx expo lint` correctos.
