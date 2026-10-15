# Tablero

Proyecto actual: [[Ofertas por geolocalización]]. Formato: `[[tarea]] · responsable`. Al cambiar de estado, mueve la línea y cambia `estado` en la nota de la tarea.

## Libre

### Fase 1: Base

### Fase 2: Embudo de ofertas
- [[G06 Avisos por distancia en web y móvil]] · PC 2
- [[G07 Estadísticas de ofertas]] · PC 4 (Marco)

### Fase 3: Llegada al restaurante
- [[G09 Pantalla de llegada con QR]] · PC 2
- [[G10 Escáner de QR y aviso de llegada]] · PC 4 (Marco)

### Notificaciones push (apoyo a G06)
Ver [[Plan app móvil y notificaciones]]: T01 a T10.

## En curso
- [[T01 Configurar EAS y app.json]] · Alex (PC 2): falta `eas login` + `eas init` y el primer APK
- [[T03 Registrar token push en la app]] · Alex (PC 2): código hecho, espera a T04 y al dev build

- [[T07 Web Push en la PWA]] · Alex (PC 2): código en `feat/mira-app`, falta clave VAPID y prueba en iPhone

## En revisión
- [[G08 QR firmado, visitas y comisión]] · Codex (PC 1) · desplegado; falta VISITAS_SECRET en Render
- [[T10 Preferencias de notificaciones]] · Codex (PC 1) · aprobado y desplegado (API); falta interruptor del PC 2
- [[G04 Editor básico de ofertas por distancia]] · Ruben (PC 3) · revisado por Claude: P2 (error del formulario), falta prueba con API real
- [[T05 Notificar reservas y mensajes]] · Codex (PC 1) · desplegado (con P3), falta prueba real con PC 2
- [[T06 Convertir la web en PWA]] · Alex (PC 2): rama `feat/T06-pwa-descargas`, página `#/app`
- [[T04 Módulo push en mira-api]] · Codex (PC 1) · desplegado, falta prueba en dispositivo real (T03)
- [[G01 Contrato API fase 1]] · Izan (Claude) · falta OK de PC 2 y PC 3

## Hecho
- [[G03 Consentimiento y permiso de ubicación]] · Izan (Claude), apoyo PC 1 · en main (web); app con el APK de Alex
- API de estadísticas para [[G07 Estadísticas de ofertas]] · Izan (Claude) · desplegada
- [[G05 Ofertas cerca de mí]] · Izan (Claude) · desplegado
- [[G02 Modelo de ofertas y endpoints base]] · Izan (Claude) · desplegado
