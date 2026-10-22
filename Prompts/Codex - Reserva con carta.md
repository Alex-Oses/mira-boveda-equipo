# Prompt de Codex · Reserva con carta (PC 1)

Pégaselo a Codex tal cual al empezar. Proyecto: [[Reserva con carta]].

```
Trabajas en el proyecto MIRA (PC 1, con Izan y Claude). La memoria del equipo es
la bóveda de Obsidian: haz git pull y lee "00 Inicio.md", "Tablero.md",
"Proyecto/Reserva con carta.md", "Decisiones/Decisiones.md" y las 3 sesiones
más recientes de Sesiones/.

Repo: ruben-munoz-bl/restuarante-mira-backend, carpeta mira-api/ (Node 20,
Express, firebase-admin, zod). Documentación de endpoints: mira-api/docs/api.md.

Tus tareas, en este orden (cada una tiene su nota en Tareas/):
1. M02 Carta del restaurante en la API.
2. M03 Pedido ligado a la reserva.
3. M04 Check-in por QR y 50 puntos.

Reglas:
- NO empieces hasta que M01 (el contrato que escribe Claude) esté publicado en
  "Proyecto/Contratos API.md". Implementa exactamente ese contrato; si algo no
  encaja, propón el cambio en la nota de M01, no lo cambies por tu cuenta.
- Al coger una tarea: en su nota pon estado: en curso, muévela en el Tablero
  y haz push de la bóveda enseguida.
- Una rama por tarea en el backend: feat/M02-carta, feat/M03-pedido-reserva,
  feat/M04-checkin-puntos. Usa un worktree propio. Commits en español con
  Conventional Commits.
- Solo tocas mira-api/. No toques el frontend (web, panel ni mobile/): eso es de
  Claude (M10, M11) y del PC 2 (Alex).
- Reglas de negocio clave:
  - El pedido queda fijo desde que se reserva; las líneas copian nombre y precio
    del plato en ese momento. Platos de otro restaurante = error.
  - Sin pago online.
  - +50 puntos solo si la reserva llevaba pedido, al hacer el check-in por QR
    (no al reservar), una sola vez aunque se escanee dos veces.
  - Reaprovecha lo que ya existe: QR firmado y visitas (G08), puntos y la
    comprobación de dueño (comprobarRestaurante).
- Tests de cada endpoint (éxito, errores y permisos). npm test en verde antes
  de pasar a revisión. Nada de secretos en el código ni en la bóveda.
- No despliegues ni escribas en producción.

Al terminar cada tarea: estado en revisión en su nota con el enlace a la rama,
actualiza el Tablero y avisa: Claude revisa tu backend antes de unirlo.
Después revisa tú M10 y M11 de Claude: que las rutas, los cuerpos y los
errores que usa el frontend coincidan con tu backend.

Al terminar la sesión: Sesiones/AAAA-MM-DD Codex.md con Plantillas/Sesión.md
(máx. 20 líneas) y push de la bóveda.
```
