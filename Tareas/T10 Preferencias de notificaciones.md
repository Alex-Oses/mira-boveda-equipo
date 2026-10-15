---
estado: en revisión
responsable: Codex (PC 1)
depende_de: [T04]
rama: feat/T10-preferencias-push
---
# T10 Preferencias de notificaciones

## Objetivo
Que cada usuario elija qué notificaciones recibe.

## Pasos
- API: guardar las preferencias en `usuarios/{uid}` (reservas, mensajes, promociones...) y que `enviarPush` las respete.
- App (`cuenta.js`) y web (Cuenta): interruptores por tipo.

## Terminada cuando
Al desactivar "promociones" dejan de llegar y el resto sigue llegando.

## Entrega de Codex (2026-10-15)
- Contrato GET/PUT /v1/push/preferencias publicado y subido antes de implementar (bóveda 4af79de). PC 2 puede usar el mapa reservas/mensajes/promociones para sus interruptores.
- Rama [feat/T10-preferencias-push](https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/T10-preferencias-push), commit `9f0a452`, desde main 85c8e21 en codex-T10-backend. Pendiente revisión de Claude antes de main.
- Preferencias en usuarios/{uid}.preferenciasPush; GET con defaults true sin escribir, PUT parcial estricto en transacción conservando perfil y otros interruptores. Solo UID del Bearer, también para admin.
- enviarPush exige tipo reservas/mensajes/promociones y lee preferencias sin caché antes de dispositivos/proveedores. Desactivar promociones conserva reservas/mensajes y tokens. Fallo al leer preferencias no envía. Emisores T05 y script adaptados; no se infiere el tipo desde data.ruta.
- **npm test 339/339**, 12 tests nuevos: auth, defaults/perfil ausente, combinación/idempotencia/concurrencia, validación, aislamiento, categorías, integración T05, reactivación y fallos de lectura. git diff --check correcto.
- Docs en mira-api/docs/api.md (T10). Sin cambios de ofertas, índices, reglas ni frontend; sin merge/despliegue. Pruebas con Firestore/Expo/FCM simulados, sin emulador por falta de Java.
- Pendiente PC 2: interruptores de web/app y prueba real conjunta tras revisión/despliegue. Emisores futuros de promociones deberán indicar tipo promociones; el que ya esté en curso no se puede retirar al cambiar una preferencia.

## Revisión de Claude (2026-10-15)
`9f0a452`: `npm test` 339/339. Bien: GET/PUT `/v1/push/preferencias` solo del propio usuario, esquema `strict` con al menos un campo, guardado en transacción con merge, `tipo` obligatorio en `enviarPush` (nadie puede olvidarlo) y comprobado antes de leer dispositivos. Compatibilidad: perfiles sin preferencias siguen recibiendo reservas y mensajes.

**[P2] Promociones activadas por defecto.** `normalizarPreferencias` trata todo lo que no sea `false` como `true`, también `promociones`. Las notificaciones comerciales necesitan consentimiento previo (LSSI art. 21 y RGPD); reservas y mensajes son transaccionales y sí pueden ir activados. Propuesta: `promociones` solo `true` si el usuario lo activa explícitamente (`datos?.promociones === true`); actualizar el contrato en [[Contratos API]] (por defecto `{ reservas: true, mensajes: true, promociones: false }`) y los tests. Avisar al PC 2 para el interruptor.

Menor: `PUT` hace `set(..., { merge: true })` sobre `usuarios/{uid}`; si el perfil no existiera, crearía un documento solo con `preferenciasPush`. En la práctica todo usuario tiene perfil al registrarse; no bloquea.

Sigue **en revisión** hasta corregir el P2.

## Aviso para PC 2 · P2 en corrección (2026-10-15)
**Promociones desactivadas por defecto.** El mapa inicial de GET es `{ reservas: true, mensajes: true, promociones: false }`. Un perfil anterior o una clave ausente nunca activa promociones. Solo `promociones: true` explícito las habilita.
Web/app: mostrar el interruptor de promociones apagado hasta leer un true del servidor. Enviar `{ promociones: true }` solo cuando el usuario lo active, nunca por inicializar la pantalla. Reservas/mensajes mantienen su valor por defecto true. Contrato actualizado antes de corregir la implementación; revisión de Claude pendiente.

## Respuesta de Codex al P2 (2026-10-15)
- Corrección subida a la misma rama, commit [a5f332c](https://github.com/ruben-munoz-bl/restuarante-mira-backend/commit/a5f332c). **340/340**, incluido caso nuevo de perfil/campo/clave ausente y valores no booleanos: no reciben promociones hasta activar true explícitamente.
- Defaults efectivos: `{ reservas: true, mensajes: true, promociones: false }`. Reservas/mensajes no cambian. Prueba de reactivación por usuario y aislamiento ajustada para requerir activación explícita en la otra cuenta.
- Contrato corregido y aviso para PC 2 subidos antes del cambio de código (bóveda 198499b); también documentado en docs/api.md. Interruptor apagado inicialmente, sin PUT true automático al abrir ajustes.
- Sigue en revisión de Claude; sin merge/despliegue. Pendiente UI y prueba conjunta real con PC 2.

## Desplegado (2026-10-15, Izan + Claude)
Revisión de Claude de `a5f332c`: **aprobado**, 340/340; promociones solo con `true` explícito. Desplegado en `main` `0249a99`: `/v1/push/preferencias` responde 401 sin sesión. Falta la parte del PC 2 (interruptores en app y web).
