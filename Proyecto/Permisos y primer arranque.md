# Permisos y primer arranque (web + app)

Pedido por Izan el 2026-10-16. Decisiones tomadas con él:
- **Primer arranque en pasos seguidos**, igual en web y app y también **sin sesión**: 1) cookies → 2) ubicación → 3) notificaciones. Cada paso explica para qué sirve y tiene «Aceptar» y «Ahora no». No se vuelve a mostrar una vez decidido (salvo cambio de versión del texto).
- **«Activar ubicación» lanza directamente el permiso del sistema** (navegador o móvil). La explicación RGPD se da antes, en el primer arranque o en Mi cuenta. Si el permiso está **bloqueado**, explicar cómo desbloquearlo (web) o abrir los ajustes del móvil (app, `Linking.openSettings()`).
- **Bug de la app: vuelve a pedir la ubicación cada vez que se abre.** No es nuestra decisión (se guarda bien en `mira:ubicacion`; `_layout` espera a hidratar el storage), sino el **permiso del sistema**: `useUbicacion` llama a `requestForegroundPermissionsAsync` en cada arranque y `useAuth` pide el de notificaciones al iniciar sesión. Corrección: al arrancar, solo **consultar** (`getForegroundPermissionsAsync`, `Notifications.getPermissionsAsync`); **pedir** solo cuando el usuario pulsa Aceptar/Activar. Si el sistema lo ha revocado, reflejarlo en Mi cuenta. Reproducir primero en un móvil.
- **Guardar las decisiones como consentimientos RGPD**, igual que las cookies: en el dispositivo (para invitados) y en el perfil (`usuarios/{uid}`) cuando hay sesión, con estado, versión del texto y fecha. Es la prueba de consentimiento. Nunca se guardan posiciones.

## Contrato (lo implementa Codex en C01)
`GET /v1/users/me` devuelve, además de lo que ya tiene, `consentimientos` (o `{}`):
```
consentimientos: {
  ubicacion:      { estado: "aceptado" | "rechazado", version: 1, fecha: "2026-10-16T08:00:00.000Z" },
  notificaciones: { estado: "aceptado" | "rechazado", version: 1, fecha: "…" }
}
```
`PUT /v1/users/me` acepta `consentimientos` con uno o los dos campos (zod `strict`; `estado` del enum, `version` entero 1-1000, `fecha` ISO). Se guarda con merge por campo: actualizar uno no borra el otro. Las cookies siguen en `consentimientoCookies`, como ahora.
Regla en el cliente: al iniciar sesión, si el dispositivo y el perfil no coinciden, **gana el más reciente** (`fecha`) y se sincroniza el otro. El permiso del sistema es siempre del dispositivo: un «aceptado» en el perfil no da permiso en un móvil nuevo; allí se vuelve a pedir el permiso del sistema (sin repetir la explicación).

## Tareas
- [[C01 Consentimientos en el perfil (API)]] · Codex (PC 1)
- [[C02 Primer arranque y permisos (web y app)]] · Claude (PC 1)
- [[R01 Revisión de G06 y G09 (unidas sin revisar)]] · Claude + Codex (PC 1)

## Reglas de Firestore: dos ficheros distintos (pendiente de decidir, 2026-10-16)
- **Frontend** `firestore.rules` (13 líneas, BUG-014): **lo bloquea todo** para los clientes (`match /{document=**}`).
- **Backend** `mira-api/firestore.rules` (209 líneas): reglas por colección; permite algunas lecturas desde el cliente (p. ej. cada usuario su perfil) y cierra `consentimientos`, `ofertas`, `visitas`, etc.
- No se sabe cuál está publicada. Publicar la del backend sobre la del frontend **abriría** accesos que hoy pueden estar cerrados. Para C01 no hace falta publicar nada (la API escribe con Admin SDK).
- **Decidir entre todos:** un único fichero como fuente de verdad, revisar qué lecturas directas necesita de verdad la web o la app (si usan `firebase/firestore` en el cliente) y publicar solo después.

