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
