---
estado: libre
responsable: Alex (PC 2)
pc: PC 2
depende_de: [C02]
rama:
---
# R01-A01 Ciclo push ligado a la sesión

Origen: [[R01 Revisión de G06 y G09 (unidas sin revisar)]], hallazgo Codex [P1].

Evidencia en origin/main cc31bab: mobile/src/controllers/useAuth.js:88-92 inicia registrarPush sin esperar/cancelar al salir; notificaciones.js:55-59 permite POST en vuelo tras logout. borrarPush:70-75 borra el ID antes de DELETE y oculta el fallo. pushWeb.js:65-75 hace lo mismo y el logout continúa aunque falle el servidor.

Reproducir con red lenta: entrar A y salir antes de terminar el POST; o registrar A, forzar DELETE 500/sin red, salir y volver a abrir. Puede quedar dispositivo remoto vinculado a A sin ID local recuperable; al cambiar a B, también hay carreras con Bearer resuelto en cada llamada.

Resolver ciclo completo: registro ligado al UID/generación de sesión, coordinar con logout/cambio de cuenta y limpieza de registros tardíos; conservar identidad suficiente y política explícita de reintento ante DELETE fallido. No almacenar Bearers. Quitar el log del token Expo. Coordinar con C02 para registrar solo con consentimiento y permiso ya concedidos.

Terminada cuando: tests de POST en vuelo + logout, A→B, DELETE 500/offline y recuperación; no queda A recibiendo push tras la baja efectiva; pruebas en web y development build. Mantener contrato T04.

## Coordinación con C02 · 2026-10-16
Por indicación de Izan: **Alex debe crear su rama desde main después de integrar C02**, o coordinarlo explícitamente antes. C02 (`feat/C02-primer-arranque`, web bd1a942 + app d1a526e) modifica mobile/src/controllers/useAuth.js y services/notificaciones.js: conservar la separación entre consultar y pedir permiso, y registrar solo con consentimiento aceptado. No restaurar request automático al login.

La revisión de Codex de C02 solicita cambios (ver su nota): sincronización ligada a la sesión, reconsulta al volver de Ajustes y control de notificaciones en Mi cuenta. En particular, coordinar el UID/generación del finally de sincronización con el registro push; evitar arreglar la misma carrera en dos ramas. C02 todavía no se ha aprobado ni unido a main.
