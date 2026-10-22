# Tablero

Proyecto actual: [[Reserva con carta]] (desde 2026-10-22). Anterior: [[Ofertas por geolocalización]], cierre en [[Prueba conjunta]]. Formato: `[[tarea]] · responsable`. Al cambiar de estado, mueve la línea y cambia `estado` en la nota de la tarea.

## Libre

### Reserva con carta (2026-10-22)
M01 ya publicado. M08 no depende de nada. M07 espera respuestas del PC 1 ([[Contratos API]] §M07). Prompt de Codex: [[Codex - Reserva con carta]].
- [[M10 Pedidos y reservas en el panel]] · Izan (Claude) (PC 1)
- [[M11 Escáner QR para el check-in]] · Izan (Claude) (PC 1)
- [[M05 Pedir la comida al reservar]] · Alex (PC 2)
- [[M06 Pedido en Mis reservas]] · Alex (PC 2)
- [[M09 Editor de la carta del restaurante]] · Alex (PC 2)
- [[M08 Cabecera sin racha ni puntos]] · Marco (PC 3)

### Permisos y primer arranque (2026-10-16)
- [[R01 Revisión de G06 y G09 (unidas sin revisar)]] · Claude + Codex (PC 1)

### Próxima APK (android-v1.0.5)
- [[Próxima APK - perfil y animaciones]] · Alex (PC 2) · perfil: Puntos/Preferencias/Ajustes/Actividad no navegan en `v1.0.4`; sugerencias de animación

### Fase 1: Base

### Fase 2: Embudo de ofertas

### Fase 3: Llegada al restaurante
- [[G12 Enviar oferta a favoritos, cercanos y todos]] · aparcada (solo era para testeo, sin envíos masivos)

### Notificaciones push (apoyo a G06)
Ver [[Plan app móvil y notificaciones]]: T01 a T10.

## En curso
- [[M07 Landing por secciones]] · Ruben (PC 3) · empieza por estructura, buscador y Descubrir; espera al PC 1 para promociones, más reservados, nuevos y cartas
- [[T07 Web Push en la PWA]] · Alex (PC 2): clave VAPID hecha; falta ponerla en Vercel y probar en iPhone

## En revisión
- [[M04 Check-in por QR y 50 puntos]] · Izan (Codex) (PC 1) · menores corregidos, pendiente revisión del commit af4de43
- [[M14 Sembrar las cartas simuladas]] · Izan (Codex) (PC 1) · **aprobado por Claude**, falta unir y desplegar (decide Izan)
- [[M12 Endpoint de más reservados]] · Izan (Codex) (PC 1) · **aprobado por Claude**, falta unir y desplegar (decide Izan)
- [[M13 Fecha de alta de los restaurantes]] · Izan (Codex) (PC 1) · **aprobado por Claude**, falta unir y desplegar (decide Izan)
- [[M03 Pedido ligado a la reserva]] · Izan (Codex) (PC 1) · **aprobado por Claude**, falta unir y desplegar (decide Izan)
- [[M02 Carta del restaurante en la API]] · Izan (Codex) (PC 1) · **aprobado por Claude**, falta unir y desplegar (decide Izan)
- [[M01 Contrato API de carta y pedidos]] · Izan (Claude) (PC 1) · publicado en [[Contratos API]]; revisión de Codex resuelta; falta OK de Alex (PC 2)
- Rediseño web en la app (PR #16, `feat/app-rediseno-web`) · Alex (PC 2) · APK `android-v1.0.4` publicado; falta revisión y prueba en móvil
- Tanda 2 de bugs UI web BUG-146 a 152 · OpenCode · en `main` `8aac33e` y desplegado en Vercel; pendiente revisión visual del usuario (detalle en `docs/bugs/`)
- [[G06 Avisos por distancia en web y móvil]] · Alex (PC 2) · unida a main (PR #5) sin revisar: revisa PC 1 en main
- [[G09 Pantalla de llegada con QR]] · Alex (PC 2) · unida a main (PR #5) sin revisar: revisa PC 1 en main · estado canjeado en main `7dbba6c` (2026-10-20)
- [[T10 Preferencias de notificaciones]] · Codex (PC 1) · aprobado y desplegado (API); falta interruptor del PC 2
- [[T05 Notificar reservas y mensajes]] · Codex (PC 1) · desplegado (con P3), falta prueba real con PC 2
- [[T06 Convertir la web en PWA]] · Alex (PC 2): rama `feat/T06-pwa-descargas`, página `#/app`
- [[G01 Contrato API fase 1]] · Izan (Claude) · OK de PC 2 (Alex), falta PC 3

## Hecho
- Errores del APK BUG-138 a 145 · Codex, revisado por Claude · PR #10 en main y APK `android-v1.0.1` (descarga en `#/app`)
- [[C02 Primer arranque y permisos (web y app)]] · Izan (Claude) · en producción (web y APK `android-v1.0.1`)
- [[C01 Consentimientos en el perfil (API)]] · Codex (PC 1) · en producción (sin reglas: ver [[Permisos y primer arranque]])
- [[G11 Avisos solo de favoritos y destacados]] · Alex (PC 2) · en main y desplegado; Izan revisa el backend a posteriori
- [[T07 Web Push en la PWA]] · Alex (PC 2) · push recibido en iPhone (PWA)
- [[T04 Módulo push en mira-api]] · Codex (PC 1) · probado en Android e iPhone
- [[T01 Configurar EAS y app.json]] · Alex (PC 2) · APK en Release `android-v1.0.0`
- [[T03 Registrar token push en la app]] · Alex (PC 2) · push probado en Android real
- [[G08 QR firmado, visitas y comisión]] · Codex (PC 1) · desplegado con VISITAS_SECRET
- [[G07 Estadísticas de ofertas]] · Marco (PC 4) · en main
- [[G10 Escáner de QR y aviso de llegada]] · Marco (PC 4) · en main
- [[G04 Editor básico de ofertas por distancia]] · Ruben (PC 3) · en main
- [[G03 Consentimiento y permiso de ubicación]] · Izan (Claude), apoyo PC 1 · en main (web); app con el APK de Alex
- API de estadísticas para [[G07 Estadísticas de ofertas]] · Izan (Claude) · desplegada
- [[G05 Ofertas cerca de mí]] · Izan (Claude) · desplegado
- [[G02 Modelo de ofertas y endpoints base]] · Izan (Claude) · desplegado
