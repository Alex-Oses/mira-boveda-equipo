# Decisiones

Una entrada por decisión, las más nuevas arriba. Formato:

```
### 2026-10-22 · Reglas de Firestore: todo cerrado
- **Decisión:** las reglas son las publicadas en la consola: ningún cliente lee ni escribe Firestore directamente (`allow read, write: if false` para todo). El archivo `mira-api/firestore.rules` del repo pasa a ser exactamente eso (backend `788a302`).
- **Por qué:** la web y la app solo usan Firebase Auth; todo lo demás va por mira-api (Admin SDK, que no pasa por las reglas). El archivo antiguo del repo era mucho más permisivo y desplegarlo habría abierto restaurantes, aforo, finanzas, invitaciones…
- **Para todos:** no añadáis lecturas o escrituras directas a Firestore desde la web o la app; si hace falta un dato, pedid un endpoint al PC 1.
- **Quién:** Izan (Claude).

### 2026-10-22 · Contrato de reserva con carta (M01)
- **Decisión:** carta en `cartas/{restauranteId}` (un documento, se sustituye entera). El pedido va dentro de la reserva con nombre y precio copiados y no se edita. La reserva tiene un QR propio firmado `r1.<reservaId>.<firma>` (HMAC, `RESERVAS_QR_SECRET`), válido el día de la reserva y una vez. `POST /v1/reservations/checkin` marca `presentadaEn` sin cambiar `estado` y da +50 puntos (`pedido_anticipado`) si hay pedido, en la misma transacción.
- **Por qué:** el `codigo` `MIRA-…` se puede adivinar; no tocar `estado` evita romper ticket, comisión y puntos de reserva que ya funcionan.
- **Quién:** Izan (Claude). Detalle en [[Contratos API]]. Falta OK de Alex y Codex.

### 2026-10-22 · Reserva con carta (aprobado)
- **Decisión:** el cliente puede añadir a la reserva un pedido de la carta. El pedido es un compromiso, queda fijo en la web y se paga en el local después de comer (sin pago online). Los cambios y platos agotados se hablan al llegar. Escanear el QR marca la reserva como presentada y, si llevaba pedido, da +50 puntos (al escanear, no al reservar). La carta la crea el restaurante al darse de alta.
- **Reparto:** PC 1 backend + panel de pedidos y escáner; PC 2 flujo del cliente + editor de carta; PC 3 landing por secciones y cabecera sin racha ni puntos, sin depender de la API nueva (OpenCode va más lento).
- **Fuera:** empresas de reparto (JustEat, UberEats), miniproyecto posterior.
- **Quién:** Izan (con Claude). Detalle en [[Reserva con carta]].

### AAAA-MM-DD · Título
- Decisión:
- Por qué:
- Quién:
```

### 2026-10-22 · Todas las secciones de la landing en carrusel
- **Decisión:** las cuatro secciones con datos usan el mismo carrusel de scroll-snap y flechas. La pista ocupa todo el ancho de la ventana: la primera tarjeta alinea con el título y las siguientes llegan al borde, con anchos 82% móvil, 46% tableta y `clamp(320px, 27vw, 370px)` escritorio. El CTA del buscador es corto («Buscar»/«Cercar»/«Search») y no se parte. El carrusel avanza solo cada 5 s y vuelve al inicio; se pausa al pasar el ratón o al enfocar con el teclado, y se queda quieto con `prefers-reduced-motion`. Sin barra de scroll: solo flechas y deslizamiento táctil.
- **Por qué:** las filas de 3 tarjetas parecían una rejilla estática y dejaban franjas vacías; el carrusel muestra que hay más contenido y llena los lados. El botón anterior se partía en dos líneas y parecía desproporcionado.
- **Quién:** Ruben (PC 3), commit `09317a8`.

### 2026-10-22 · Landing por secciones y Descubrir infinito (M07)
- **Decisión:** la landing deja de listar restaurantes. Queda: buscador con filtros plegados (solo el nombre a la vista), 5 secciones (promociones, fotos de platos, más reservados de la semana, Top 50 Barcelona, nuevos) y bloque final con botón a Descubrir. El listado infinito de 27 en 27 pasa a Descubrir, que mantiene el vuelo de los 7 mejores; si llega con filtros desde la landing se abre filtrado, sin vuelo y con el desplegable abierto. «Promoción» = campañas (`promociones`), no las ofertas por distancia. Los platos deben ser reales: se siembran las cartas simuladas actuales en Firestore (M02).
- **Por qué:** la landing se vuelve curada y compacta; el catálogo completo vive en Descubrir, con un solo motor de paginación.
- **Quién:** Ruben (PC 3). Peticiones a PC 1 en [[Contratos API]] §M07.

### 2026-10-16 · Avisos por distancia solo de partners o favoritos (G11)
- **Decisión:** los avisos por distancia (G06) solo saltan si el restaurante es **partner** (lo marca el admin con 1 clic) o está en los **favoritos** del usuario. No hay envíos masivos de ofertas (G12 aparcada).
- **Por qué:** no saturar al usuario con avisos de cualquier restaurante.
- **Quién:** Alex (PC 2); el backend lo revisa PC 1.

### 2026-10-15 · G06 y G09 se quedan con Alex (PC 2); PC 3, de momento, fuera de la app
- **Decisión:** al asignar la parte de app a Ruben (PC 3) apareció a la vez Alex (PC 2) con G06 y G09 **en curso** en `feat/mira-app`. El usuario decide: **se los deja Alex** y PC 3 no toca la app de momento. A futuro se valorá **llevar el panel del restaurante a la app** (ahí entraría PC 3).
- **Por qué:** no duplicar trabajo en la misma rama; Alex ya estaba en ello cuando se acordó el reparto web/app.
- **Quién:** decisión del usuario con Ruben (PC 3); sustituye en lo que se refiere a G06/G09 a la entrada siguiente (el reparto **web = Marco** se mantiene).

### 2026-10-15 · Reparto web/app del proyecto de ofertas (cambio)
- **Decisión:** Marco (PC 4) se queda con **toda la parte web** del proyecto de ofertas: revisión y merge de G04, G07 (estadísticas), G10 (escáner) y los avisos web de G06. Ruben (PC 3) pasa a la **app móvil** (`mobile/`, Expo, rama de base `feat/mira-app`): G06 móvil (avisos por distancia con notificaciones locales) y G09 móvil (llegada a 20 m con QR), además de la prueba en móvil real de G03. La web del cliente (consentimiento y avisos en la landing) sigue en PC 2.
- **Por qué:** así cada agente web lleva un solo lado sin pisarse (G04 tocó `Dashboard.jsx`, ahora lo revisa/mergea Marco) y la app Expo —que sigue desarrollándose con Alex— avanza en paralelo; en localhost no se veía la parte de la app.
- **Quién:** decisión del usuario con Ruben (PC 3). Comunicado en el hilo #3 de `docs/coordenacion/HILO-PC3-PC4.md` (ramas `docs/pc4-marco` y `feat/G04-editor-ofertas`); instrucciones de Marco actualizadas en `PC4-INSTRUCCIONES.md`.

### 2026-10-15 · Reemitir QR mantiene la caducidad original (G08)
- **Decisión:** al devolver una visita pendiente no se renuevan sus 15 minutos. Se firma de nuevo ID/fecha originales: el texto HMAC es determinista y coincide. La siguiente visita se crea solo tras caducar o confirmar la anterior.
- **Por qué:** no prolongar indefinidamente una pendiente al repetir POST. Idempotencia por cliente/restaurante en una transacción con bloqueo privado visitas_pendientes.
- **Quién:** Izan confirma mantener la caducidad original; Codex implementa G08 (ed2b8b8), pendiente revisión de Claude.

### 2026-10-15 · Promociones push solo tras activación explícita (corrección T10)
- **Decisión:** prevalece sobre el valor por defecto inicial de T10: promociones false salvo `preferenciasPush.promociones === true`; reservas/mensajes mantienen true por defecto. El interruptor de PC 2 debe activarse solo por acción del usuario.
- **Por qué:** corrección P2 solicitada en la revisión de Claude: no interpretar ausencia o valores no booleanos como aceptación de promociones.
- **Quién:** Izan + Claude; Codex aplica el cambio en la misma rama y publica este aviso para PC 2 antes de implementar.

### 2026-10-15 · Reparto del panel: PC 3 (Ruben) y PC 4 (Marco)
- **Decisión:** el trabajo del panel del restaurante del proyecto de ofertas se reparte entre dos agentes Opencode: **Ruben (PC 3)** con **G04** (editor de ofertas) y **Marco (PC 4)** con **G07** (estadísticas) y **G10** (escáner QR, jsQR + fallback manual). La comunicación entre ambos es por git con `.md`: instrucciones en `docs/coordenacion/PC4-INSTRUCCIONES.md` y hilo `docs/coordenacion/HILO-PC3-PC4.md` del repo del frontend (rama `docs/pc4-marco`).
- **Por qué:** tener varios agentes trabajando a la vez sin instalarse nada extra; git ya está en uso y el hilo `.md` se lee desde el navegador si hace falta.
- **Quién:** Ruben (PC 3), con OK del usuario. En fase 1, PC 4 no toca `Dashboard.jsx` (lo monta G04).

### 2026-10-15 · Preferencias push por categoría (T10)
- **Decisión:** usuarios/{uid}.preferenciasPush contiene reservas/mensajes/promociones. GET/PUT autenticados en /v1/push/preferencias, PUT parcial transaccional; claves ausentes true para mantener el comportamiento previo. Cada enviarPush exige tipo y comprueba preferencias sin caché, sin enviar si no puede leerlas.
- **Por qué:** interruptores comunes a todos los dispositivos, sin perder ajustes concurrentes ni confundir tipos de negocio con canales Expo/web; el buzón y las acciones siguen guardándose.
- **Quién:** Codex (PC 1), según T10 y contrato publicado antes de implementar. 9f0a452 pendiente revisión de Claude; UI de PC 2. Nuevas categorías requieren ampliar contrato.

### 2026-10-15 · Avisos T05 por idioma y después del commit
- **Decisión:** avisos de confirmación/cancelación al UID de la reserva y de mensaje al UID destinatario, después de persistir. Textos por idioma del token; el aviso de buzón es genérico para no mostrar contenido privado en la pantalla de bloqueo.
- **Por qué:** mantener el idioma de cada dispositivo y evitar avisar de escrituras fallidas o revertir una acción por un fallo de push. Sin cola persistida de eventos en T05; pruebas reales con PC 2 pendientes.
- **Quién:** Codex (PC 1), según T05; implementado en 4aa0eab, pendiente revisión de Claude.

### 2026-10-15 · Despliegue de índices sin force
- **Decisión:** nunca usar `--force` al desplegar índices de Firestore. Partir de main alineado con los índices publicados (86add00); Firebase devuelve los de restaurants sin el __name__ implícito.
- **Por qué:** el despliegue forzado habría borrado índices existentes. Si pide force, revisar y alinear el fichero primero.
- **Quién:** Izan + Claude; aviso de producción a Codex.

### 2026-10-15 · Corrección T04 tras revisión de Claude
- **Decisión:** un token push solo pertenece a una cuenta; alta/reasignación por tokenHash en transacción y máximo 10 dispositivos por UID (expulsión del más antiguo). Índice de grupo en dispositivos.tokenHash previo al despliegue.
- **Por qué:** evitar que un móvil con otra sesión reciba avisos de la cuenta anterior si falla DELETE; acotar el envío por usuario.
- **Quién:** revisión de Claude y solicitud de Izan; implementado por Codex, pendiente nueva revisión (03983b0). Sin cambios de canales ni formato público.

### 2026-10-15 · Detalles de implementación T04 (push)
- **Decisión:** registro idempotente con SHA-256 de `tipo:token`, `web` usa token FCM; solo la API escribe dispositivos. Recibos diferidos de Expo en `push_recibos`, privados para la API, revisados por cron para limpiar DeviceNotRegistered.
- **Por qué:** evita duplicados y permite limpiar tokens después de la aceptación inicial del proveedor, incluso tras reiniciar la API. Contrato publicado en [[Contratos API]].
- **Quién:** Codex (PC 1), T04; pendiente revisión de Claude. Se mantienen pendientes de confirmación del equipo las decisiones de canales del 2026-10-14.

### 2026-10-15 · Visitas con QR y estadísticas (contrato fase 3)
- **Decisión:** QR = token `v1.<visitaId>.<caducaEnMs>.<firma>` firmado con HMAC-SHA256 (`VISITAS_SECRET`), caducidad de 15 min, una sola validación (transacción). La visita se crea a ≤ 50 m (20 m + margen de GPS) y no guarda la posición. Al confirmar se copia `comisionPct`; el importe queda para el ticket. Estadísticas como contadores agregados por restaurante y día, sin datos de usuario.
- **Por qué:** el GPS falla a 20 m; el QR firmado es la prueba de llegada y no se puede falsificar ni reutilizar.
- **Pendiente para el equipo:** ¿tarifa fija por visita confirmada además del porcentaje?
- **Quién:** PC 1 (Izan + Claude). Detalle en [[Contratos API]].

### 2026-10-15 · Modelo de ofertas (contrato fase 1)
- **Decisión:** colección `ofertas`, con 4 niveles fijos (1000, 500, 100 y 20 m) que no pueden empeorar al acercarse, y **una sola oferta activa por restaurante**. Lectura pública solo de la activa y vigente; CRUD para la empresa dueña o el admin, con la comprobación de dueño que ya existe (`comprobarRestaurante`).
- **Por qué:** simplifica «ofertas cerca de mí» (fase 2) y el QR (fase 3): cada restaurante tiene como mucho una oferta que mostrar.
- **Quién:** PC 1 (Izan + Claude). Detalle en [[Contratos API]].

### 2026-10-14 · Ofertas por geolocalización (aprobado)
- **Decisión:** ofertas que mejoran a 1 km, 500 m y 100 m, y QR firmado por visita a 20 m, que valida el restaurante. Sin login por SMS. RGPD: consentimiento, se puede desactivar y no se guarda el recorrido.
- **Por qué:** el GPS no es fiable a 20 m; el QR prueba la llegada y sirve para la comisión.
- **Reparto:** PC 1 backend (Izan + Claude + Codex), PC 2 app del cliente (2 personas), PC 3 panel del restaurante. 3 fases con unión de ramas y revisión cruzada al final de cada una. El PC 1 publica el contrato en [[Contratos API]].

### 2026-10-14 · Android con Expo, iOS como PWA
- **Decisión:** Android usa la app Expo de `mobile/` (Expo Push, que por debajo usa FCM). iOS, de momento, usa la web como PWA con Web Push (iOS 16.4+, instalada en la pantalla de inicio) hasta que haya cuenta de Apple Developer.
- **Por qué:** la app Expo ya existe y la cuenta de Apple cuesta 99 €/año.
- **Quién:** Alex (propuesta; falta que el equipo la confirme).

### 2026-10-14 · Un solo endpoint de tokens para los dos tipos
- **Decisión:** `POST /v1/push/tokens` con `{ token, tipo: "expo" | "web", plataforma, idioma }`, guardado en `usuarios/{uid}/dispositivos/{id}`. `DELETE /v1/push/tokens/:id` al cerrar sesión. `enviarPush(uid, { titulo, cuerpo, data })` envía por el canal de cada token.
- **Por qué:** la app Android y la PWA comparten el mismo backend.
- **Quién:** Alex (propuesta).
