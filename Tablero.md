# Tablero

Proyecto actual: [[Ofertas por geolocalización]]. Cierre: [[Prueba conjunta]] (cuando G06 y G09 estén en main). Formato: `[[tarea]] · responsable`. Al cambiar de estado, mueve la línea y cambia `estado` en la nota de la tarea.

## Libre

### Permisos y primer arranque (2026-10-16)
- [[C01 Consentimientos en el perfil (API)]] · Codex (PC 1)
- [[R01 Revisión de G06 y G09 (unidas sin revisar)]] · Claude + Codex (PC 1)

### Fase 1: Base

### Fase 2: Embudo de ofertas

### Fase 3: Llegada al restaurante

### Notificaciones push (apoyo a G06)
Ver [[Plan app móvil y notificaciones]]: T01 a T10.

## En curso
- [[T07 Web Push en la PWA]] · Alex (PC 2): clave VAPID hecha; falta ponerla en Vercel y probar en iPhone

## En revisión
- [[C02 Primer arranque y permisos (web y app)]] · Izan (Claude) · revisa Codex; falta APK en móvil (Alex) y C01
- [[G06 Avisos por distancia en web y móvil]] · Alex (PC 2) · unida a main (PR #5) sin revisar: revisa PC 1 en main
- [[G09 Pantalla de llegada con QR]] · Alex (PC 2) · unida a main (PR #5) sin revisar: revisa PC 1 en main
- [[T10 Preferencias de notificaciones]] · Codex (PC 1) · aprobado y desplegado (API); falta interruptor del PC 2
- [[T05 Notificar reservas y mensajes]] · Codex (PC 1) · desplegado (con P3), falta prueba real con PC 2
- [[T06 Convertir la web en PWA]] · Alex (PC 2): rama `feat/T06-pwa-descargas`, página `#/app`
- [[G01 Contrato API fase 1]] · Izan (Claude) · OK de PC 2 (Alex), falta PC 3

## Hecho
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
