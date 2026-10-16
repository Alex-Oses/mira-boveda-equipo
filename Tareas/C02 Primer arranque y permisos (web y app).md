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

