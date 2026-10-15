---
estado: en revisión
responsable: Izan (Claude), apoyo del PC 1
pc: PC 2
fase: 1
depende_de: [G01]
rama: feat/G03-ubicacion
---
# G03 Consentimiento y permiso de ubicación

## Objetivo
Pedir la ubicación cumpliendo el RGPD, en la web y en la app.

## Pasos
- Pantalla o modal de consentimiento (es/ca/en): para qué se usa, que no se guarda el recorrido y que se puede desactivar.
- Web: `navigator.geolocation.watchPosition`, solo con la página abierta.
- Móvil: `expo-location`, primero en primer plano (el segundo plano va en G06).
- Interruptor en Cuenta para desactivarlo. Guardar solo la preferencia, nunca posiciones.

## Terminada cuando
Se puede aceptar, rechazar y desactivar, y la posición actual aparece en pantalla (sin enviarla todavía).

## Hecho (2026-10-15, Izan + Claude, apoyo del PC 1)
Rama `feat/G03-ubicacion` del frontend.
- **Web** (`2aba98b`): sección «Ubicación para ofertas cercanas» en Mi cuenta, con modal de consentimiento (es/ca/en: para qué, sin guardar recorrido, solo con la página abierta, se puede desactivar). `hooks/useUbicacion.js` usa `watchPosition` solo con el consentimiento aceptado y para el seguimiento al desactivar. La posición se muestra en pantalla y no se envía ni se guarda. Mensajes para permiso bloqueado, sin señal, tiempo agotado y navegador sin soporte. `models/ubicacionModel.js` guarda solo `{ estado, version, fecha }` en `localStorage` (`mira:ubicacion`); si cambia el texto, subir `VERSION_CONSENTIMIENTO` y se vuelve a pedir. 4 tests nuevos; `npm test` 199/199 y build correcto. Probado en el navegador con un GPS simulado: aceptar, ver posición, desactivar (0 seguimientos activos), «Ahora no» y permiso bloqueado.
- **App** (`mobile/`): `expo-location ~57.0.21` (instalado con `npx expo install`, lockfile limpio), plugin en `app.json` con el texto de permiso «mientras se usa» (sin segundo plano). Mismo modelo (el `localStorage` de la app es el shim sobre AsyncStorage), `useUbicacion` con `requestForegroundPermissionsAsync` + `watchPositionAsync` (se cierra al desactivar o salir) y la misma sección en `cuenta.js`. `npx expo lint` sin errores. `expo-doctor` da 3 avisos que ya existían antes (versiones y duplicados), no de G03.
- **Sin probar en un móvil real.** Hace falta un build nuevo (el plugin cambia los permisos nativos), o probar en Expo Go.
- El consentimiento se guarda por dispositivo, no en el perfil, porque el permiso de ubicación también es por dispositivo.
- **Para el PC 2:** G06 puede usar `useUbicacion()` (`activa`, `posicion`) en la web y en la app. El segundo plano se añade en G06.
- Falta: revisión (Codex o el PC 2) y prueba en un móvil.


## Revisión del PC 2 (Alex + Claude, 2026-10-15)
- Revisado `6ac2c96`: permiso solo en primer plano, `watchPositionAsync` se cierra al desactivar/salir (incluida la carrera con el permiso), la posición no sale del dispositivo y solo se guarda `{estado, version, fecha}`. OK.
- Menor (no bloquea): si se acepta en MIRA y luego se deniega el permiso de Android, el consentimiento queda «aceptado» y solo se ve el error.
- Unida en `feat/mira-app` (`b049614`) junto a T03/T06/T07 y el rediseño móvil. Conflictos de i18n, `app.json`, `Cuenta.jsx` y lock resueltos quedándose con los dos lados. `npm test` 199/199, build y `expo lint` OK.
- Prueba en móvil: va en el APK que se está generando con todo junto (build `082b1acc`).

## Revisión de Codex (2026-10-15)
- Revisados web `2aba98b` y app `6ac2c96` en `codex-review-G03`, sin modificar código. Web: **199/199** tests y build correcto (aviso de tamaño de chunk). Móvil: `npm run lint` correcto. `tsc --noEmit` no comprueba el proyecto porque no hay `tsconfig.json` (muestra ayuda); no se cuenta como typecheck superado. Sin prueba en dispositivo real.
- Privacidad: en el código revisado no hay envío, registro ni persistencia de coordenadas; solo estado React para la posición y `{ estado, version, fecha }` para el consentimiento. Textos es/ca/en: finalidad, sin recorrido, primer plano/página abierta y revocación, con variantes móviles usadas en la UI. Web limpia el watch con `clearWatch`; móvil libera una suscripción ya resuelta con `remove()` y evita iniciarla si se desactiva durante la espera del permiso.
- Lockfile móvil verificado contra el commit web: solo alta de `expo-location` 57.0.21 y actualización de sus dependencias `@expo/image-utils` 0.11.5→0.11.6 y `@expo/require-utils` 57.0.5→57.0.6, más la dependencia raíz. Sin otros cambios de paquetes.
- **[P2] Se puede iniciar el seguimiento después de desactivar mientras se consultan los servicios.** `mobile/src/hooks/useUbicacion.js:25-29`: falta comprobar `vivo` tras `await Location.hasServicesEnabledAsync()` y los callbacks de posición/error tampoco lo comprueban. Si se desactiva durante esa espera y resuelve `true`, se llama a `watchPositionAsync` con el consentimiento ya rechazado. Hasta que se resuelva su promesa y se ejecute el `remove()` tardío, un callback puede volver a guardar posición en memoria tras haberla limpiado.
- Reproducción ejecutando el hook original con dobles de hooks/Expo y promesas controladas: permiso concedido → consulta de servicios pendiente → `rechazar` y cleanup → servicios `true` → **1 watch iniciado con `activa: false`** → callback deja `{ lat: 40.4, lng: -3.7, precisionM: 5 }` en el estado. Al resolver la suscripción se llama finalmente a `remove()`. Control: desactivar mientras espera el permiso produce **0 watches**, correcto.
- Propuesta: comprobar `vivo` también después de la consulta de servicios, proteger ambos callbacks y mantener la liberación de la suscripción que llega tarde. Añadir regresiones para desactivar durante permisos, servicios y creación de la suscripción, verificando que los callbacks tardíos no restauran posición/error.
- **Resultado: en revisión, pendiente del P2 móvil y de la prueba real.** No se ha marcado «hecho».
- Coordinación: Alex informa arriba de la integración en `feat/mira-app`; llevar también allí la corrección del P2 móvil y coordinar la prueba del APK. Su control de la espera del permiso es compatible con este hallazgo: el fallo reproducido está en la espera posterior de servicios.

## Corrección del P2 de Codex (2026-10-15, Izan + Claude)
`dce2a73` en `feat/G03-ubicacion`: la secuencia de arranque de la app pasa a `mobile/src/services/seguimientoUbicacion.js` (`iniciarSeguimiento`, sin React, con `expo-location` inyectado). Vuelve a comprobar si sigue activa después del permiso, de los servicios y de la suscripción; los callbacks de posición y error también lo comprueban; una suscripción que llega tarde se elimina al momento. `useUbicacion` solo la llama.
- 5 tests nuevos con un `expo-location` falso y esperas controladas (`test/seguimientoUbicacion.test.js`): flujo normal; desactivar durante el permiso, durante los servicios (el caso de Codex: 0 seguimientos) y durante la suscripción (se elimina y la posición tardía no se guarda); permiso denegado y servicios apagados. `npm test` 204/204, `npx expo lint` correcto.
- **Alex:** como G03 ya está unida en `feat/mira-app`, trae también este commit (o vuelve a unir `feat/G03-ubicacion`).
