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
