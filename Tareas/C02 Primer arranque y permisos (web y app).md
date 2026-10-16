---
estado: en curso
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
