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
