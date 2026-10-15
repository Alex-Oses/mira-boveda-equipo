# Contratos API (ofertas por geolocalización)

Lo escribe el **PC 1** al empezar cada fase. Los PC 2 y 3 programan contra esto con datos simulados. Cualquier cambio se avisa aquí y en [[Decisiones]].

Formato por endpoint:
```
### MÉTODO /v1/ruta  (auth: sí/no/admin)
Petición: { ... }
Respuesta 200: { ... }
Errores: 400 ..., 401 ..., 404 ...
```

## Fase 1: Base
Publicado el 2026-10-15 por el PC 1 (Izan + Claude). Mock disponible: `npm run mock-api` en el repo del frontend (rama `feat/G01-contrato-ofertas` hasta que se una a `main`).

### Modelo `Oferta` (colección `ofertas`)
```
{
  id: "abc123",
  restauranteId: "1",
  titulo: "Menú de mediodía",          // 1-80 caracteres
  activa: true,
  desde: "2026-10-20" | null,          // vigencia opcional, AAAA-MM-DD (incluido)
  hasta: "2026-12-31" | null,
  niveles: [                           // siempre 4, en este orden
    { distancia: 1000, texto: "Café gratis con el menú", descuentoPct: 5 },
    { distancia: 500,  texto: "10 % en la cuenta",       descuentoPct: 10 },
    { distancia: 100,  texto: "15 % en la cuenta",       descuentoPct: 15 },
    { distancia: 20,   texto: "20 % + postre gratis",    descuentoPct: 20 }
  ],
  creado: "2026-10-15T09:00:00.000Z",
  actualizado: "2026-10-15T09:00:00.000Z"
}
```
Reglas:
- `niveles` tiene exactamente las distancias **1000, 500, 100 y 20** (metros), en ese orden. El de 20 m es el de la llegada: el que va en el QR de la fase 3.
- `texto`: 1-140 caracteres. `descuentoPct`: entero de 0 a 100.
- **Cada nivel tiene que ser igual o mejor que el anterior** (`descuentoPct` no baja al acercarse), y el de 20 m tiene que ser mejor que el de 1000 m. Si no, 400.
- `desde` ≤ `hasta` si vienen los dos.
- **Una sola oferta activa por restaurante.** Activar una segunda da 409; hay que desactivar antes la otra.
- La oferta no guarda nada del usuario ni de su ubicación.

Errores comunes: `400 { error: "VALIDATION_ERROR", details: { campo: ["mensaje"] } }`, `401 { error: "UNAUTHORIZED" }`, `403 { error: "FORBIDDEN" }` (no eres dueño del restaurante), `404 { error: "NOT_FOUND" }`, `409 { error: "CONFLICT", message }`.

### GET /v1/ofertas?restauranteId=1  (auth: no)
Para la app del cliente (PC 2). Solo devuelve la oferta **activa y vigente hoy** del restaurante.
Respuesta 200: `{ data: [Oferta] }` (lista vacía si no tiene)
Errores: 400 si falta `restauranteId`.

### GET /v1/ofertas/mias?restauranteId=1  (auth: empresa dueña o admin)
Para el editor del restaurante (PC 3). Todas las ofertas del restaurante, activas o no, de la más nueva a la más antigua.
Respuesta 200: `{ data: [Oferta] }`
Errores: 400, 401, 403, 404 (restaurante no existe).

### POST /v1/ofertas  (auth: empresa dueña o admin)
Petición: `{ restauranteId, titulo, activa, desde?, hasta?, niveles: [4 niveles] }`
Respuesta 201: `Oferta`
Errores: 400, 401, 403, 404, 409 (ya hay otra activa).

### PUT /v1/ofertas/:id  (auth: empresa dueña o admin)
Sustituye los campos editables. Petición: igual que el POST, pero sin `restauranteId` (no se puede mover de restaurante).
Respuesta 200: `Oferta`
Errores: 400, 401, 403, 404, 409.

### DELETE /v1/ofertas/:id  (auth: empresa dueña o admin)
Respuesta 204, sin cuerpo.
Errores: 401, 403, 404.

### Avance de la fase 2 (orientativo, se cerrará en G05)
`GET /v1/ofertas/cerca?lat=&lng=` devolverá las ofertas de los restaurantes en 1 km, con `distanciaM` y el `nivel` alcanzado. La posición no se guardará.

### Revisión del mock de fase 1 (Alex, PC 2 · 2026-10-15)
Probado `feat/G01-contrato-ofertas` (`d60f007`): GET público, 400, 409, 201, PUT/DELETE y validación de niveles cumplen el contrato. Dos diferencias, que afectan al PC 3:
- `GET /v1/ofertas/mias` sin Bearer da **200** (el contrato dice 401): el mock no comprueba la auth.
- `GET /v1/ofertas/mias?restauranteId=999` da **200 `[]`** (el contrato dice 404).
El mock de `main` todavía no lo tiene: sigue en la rama.

## Fase 2: Embudo de ofertas
Publicado el 2026-10-15 por el PC 1 (Izan + Claude). La API de la fase 1 ya está en producción.

### GET /v1/ofertas/cerca?lat=41.3874&lng=2.1686  (auth: no)
Para los avisos por distancia (PC 2). Devuelve las ofertas activas y vigentes de los restaurantes a **1 km o menos**, de la más cercana a la más lejana, con el nivel alcanzado.
- `lat`: −90 a 90; `lng`: −180 a 180 (números decimales).
- **La posición no se guarda ni se registra** (ni en logs). Solo se usa para calcular la distancia.
- Límite: 60 peticiones por minuto. Llamadla con throttle (por ejemplo, cada 30 s o cada 50 m de movimiento).

Respuesta 200:
```
{
  data: [
    {
      restaurante: { id: "1", nombre: "Can Solé", coordenadas: { lat: 41.3881, lng: 2.1690 } },
      oferta: Oferta,                     // el modelo de la fase 1, completo
      distanciaM: 84,                     // metros, entero
      nivel: { distancia: 100, texto: "15 % en la cuenta", descuentoPct: 15 }
    }
  ]
}
```
- `nivel` es el más cercano alcanzado: el de menor `distancia` que cumpla `distanciaM <= distancia`. A 900 m → nivel 1000; a 400 m → 500; a 80 m → 100; a 15 m → 20 (llegada).
- Los restaurantes sin coordenadas no aparecen.
- La lista de candidatos se cachea 60 s en el servidor: una oferta recién creada o desactivada puede tardar hasta 1 minuto en reflejarse (en otra instancia).

Errores: `400 VALIDATION_ERROR` (lat/lng ausentes o fuera de rango), `429` (límite).

Pendiente de acordar con el PC 3 para [[G07 Estadísticas de ofertas]]: qué contadores agregados guarda la API (por ejemplo, avisos mostrados por nivel y día), siempre sin ubicación.

## Fase 3: Llegada al restaurante
_(pendiente)_

## Push · T04 (PC 1, Codex · 2026-10-15)
Contrato cerrado para PC 2; se mantienen las decisiones de canales pendientes de confirmación del equipo.

### POST /v1/push/tokens (auth: sí, Firebase Bearer)
Petición: `{ token, tipo: "expo" | "web", plataforma: "android" | "ios" | "web", idioma: "es" | "ca" | "en" }`. Todos obligatorios; no admite campos extra.
- `token`: cadena sin espacios, 1–4096 caracteres; para `expo`, formato ExpoPushToken[...] o ExponentPushToken[...]. Para `web`, token de registro FCM (no una suscripción PushSubscription).
- Respuesta 200: `{ id, tipo, plataforma, idioma }`. `id` es SHA-256 de `tipo:token` (64 caracteres hexadecimales). Guardarlo para el logout. Repetir actualiza plataforma/idioma sin duplicar.
- Persiste en `usuarios/{uid}/dispositivos/{id}`; el UID sale del Bearer, nunca del body. Campos: token, tokenHash (igual al ID), tipo, plataforma, idioma, actualizado (fecha ISO).
- Un token pertenece a una sola cuenta: registrar en B lo elimina de otras cuentas en una transacción. Máximo 10 dispositivos por UID; al superarlo se elimina el más antiguo por actualizado, conservando el que se está registrando. Desplegar antes el índice de grupo dispositivos.tokenHash (firestore.indexes.json).
- Errores: 400 `{ error: "VALIDATION_ERROR", details }`; 401 `{ error: "MISSING_TOKEN" | "INVALID_TOKEN", message }`.

### DELETE /v1/push/tokens/:id (auth: sí, Firebase Bearer)
Respuesta 204 sin cuerpo, también si no existe. Solo borra bajo el UID autenticado.
Errores: 400 si el ID no son 64 caracteres hexadecimales minúsculos; 401 como POST.
Al cerrar sesión, borrar **antes** de invalidar el Bearer. Al cambiar de cuenta, borrar el registro anterior y registrar en la nueva.

### Servicio interno y configuración
`enviarPush(uid, { titulo, cuerpo, data? })`: Expo Server SDK para expo y Firebase Admin Messaging para web; `data` es un mapa de cadenas compatible con FCM. No hay endpoint público de envío.
Los tickets y recibos Expo con DeviceNotRegistered y los tokens FCM no registrados se limpian; errores temporales conservan el dispositivo.
Credenciales de Firebase en Secret File de Render mediante GOOGLE_APPLICATION_CREDENTIALS; EXPO_ACCESS_TOKEN solo si se habilita seguridad push en Expo. FCM/VAPID/EAS se configuran fuera del repo y de la bóveda.

### T05 · Avisos de reservas y mensajes (Codex, 2026-10-15)
- Sin endpoints nuevos. Al pasar a confirmada/cancelada, UID cliente de la reserva recibe `{ ruta: "reservas", reservaId, estado }` en data. Crear una pendiente todavía no envía confirmación.
- Mensaje nuevo desde la API: UID destinatario recibe `{ ruta: "mensajes", mensajeId }`. Incluye enviarMensaje (espera/JAM) y mensaje de admin al dueño; sin UID se conserva el mensaje por email sin push.
- Textos es/ca/en por idioma del token. Aviso de buzón genérico, sin mostrar contenido privado en pantalla de bloqueo.
- Extensión interna compatible de enviarPush: `textos?: { es?: { titulo, cuerpo }, ca?: { titulo, cuerpo }, en?: { titulo, cuerpo } }`; si falta traducción, usa título/cuerpo base. El formato público de tokens no cambia.
- Avisos tras commit, fuera de callbacks repetibles; repetir el mismo estado no vuelve a avisar. Best-effort sin cola persistida: fallo de push no revierte el negocio y una caída tras commit puede perder aviso.
- Rama feat/T05-notificar-reservas-mensajes (4aa0eab), en revisión; aún no desplegada. Instrucciones de prueba conjunta en mira-api/docs/api.md §T05.

- Ajuste T05 (08258f9): en PUT /v1/reservations/:id/cancel no se envía push si el actor es el titular; si cancela otra cuenta admin autorizada, sí se avisa al cliente. Estado, aforo y 409 no cambian.

## T10 · Preferencias push (PC 1, Codex · 2026-10-15)
Contrato publicado antes de implementar para los interruptores de PC 2. Las preferencias afectan a todos los dispositivos del UID y solo a push, no al buzón ni a la operación de negocio.

### GET /v1/push/preferencias (auth: sí, Firebase Bearer)
Respuesta 200: `{ reservas: true, mensajes: true, promociones: true }`.
Lee usuarios/{uid}.preferenciasPush; UID exclusivamente del Bearer. Usuario/campo/clave ausente implica true (compatibilidad con comportamiento anterior); GET no crea documentos. Solo false explícito desactiva una categoría.

### PUT /v1/push/preferencias (auth: sí, Firebase Bearer)
Petición parcial, al menos una clave: `{ promociones: false }`. Claves permitidas: reservas, mensajes, promociones; solo booleanos, sin campos extra ni uid.
Se combina de forma transaccional con las anteriores sin perder otros interruptores ni otros campos del perfil. Repetir es idempotente; crea el documento si el usuario no tenía perfil.
Respuesta 200: mapa completo, p. ej. `{ reservas: true, mensajes: true, promociones: false }`.
Errores: 400 `{ error: "VALIDATION_ERROR", details }`; ambos endpoints: 401 `{ error: "MISSING_TOKEN" | "INVALID_TOKEN", message }` y errores de infraestructura habituales.

### Aplicación al envío
Cada llamada interna usa `enviarPush(uid, { tipo: "reservas" | "mensajes" | "promociones", titulo, cuerpo, data?, textos? })`; tipo obligatorio y validado, sin categoría implícita. T05 etiqueta sus avisos de reservas/mensajes; el script manual también indica tipo.
Antes de consultar dispositivos/enviar se leen las preferencias vigentes (sin caché): si la categoría está desactivada, devuelve contadores a cero y no llama a proveedores. Si falla la lectura de preferencias, no envía. No borra tokens ni afecta otras categorías, usuarios o mensajes guardados.
Las llamadas de promociones futuras deberán indicar tipo promociones. Nuevas categorías requieren ampliar el contrato, no se aceptan claves arbitrarias. No cambia el formato de registro de tokens.
