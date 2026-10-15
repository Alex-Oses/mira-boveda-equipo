# Plan: app móvil y notificaciones

Objetivo: que MIRA funcione como app en Android e iOS y pueda enviar notificaciones push.

## Enfoque
- **Android:** la app Expo de `mobile/`, con notificaciones por Expo Push Service (por debajo, FCM) y builds con EAS.
- **iOS:** la web como PWA instalada en la pantalla de inicio, con Web Push (iOS 16.4+). No necesita cuenta de Apple. Limitaciones: el usuario tiene que instalarla desde Safari y activar las notificaciones pulsando un botón.
- **Backend:** un módulo push en mira-api que guarda los tokens (expo y web) y envía cada notificación por su canal. Ver [[Decisiones]].

## Fases y tareas
1. Preparar la app: [[T01 Configurar EAS y app.json|T01]], [[T02 Firebase y FCM para Android|T02]]
2. Registrar el dispositivo: [[T03 Registrar token push en la app|T03]]
3. Backend: [[T04 Módulo push en mira-api|T04]], [[T05 Notificar reservas y mensajes|T05]]
4. iOS / web: [[T06 Convertir la web en PWA|T06]], [[T07 Web Push en la PWA|T07]], [[T08 Aviso para instalar en iOS|T08]]
5. Pruebas y extras: [[T09 Builds y pruebas en móviles|T09]], [[T10 Preferencias de notificaciones|T10]]

## Reparto (encaja con [[Ofertas por geolocalización]])
- **PC 2 (app del cliente):** T01, T02, T03, T06, T07, T08 y T09
- **PC 1 (backend):** T04, T05 y T10 (parte de la API)

T04 une a los dos PC: el PC 2 necesita el endpoint de tokens, así que el contrato que hay en [[Decisiones]] hay que cerrarlo el primer día. Mientras tanto, el PC 2 puede sacar el token y mostrarlo por consola.

## Más adelante
Con cuenta de Apple Developer: build de iOS con EAS y TestFlight usando la misma app Expo.
