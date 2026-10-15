# Decisiones

Una entrada por decisión, las más nuevas arriba. Formato:

```
### AAAA-MM-DD · Título
- Decisión:
- Por qué:
- Quién:
```

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
