---
estado: en curso
responsable: Alex (PC 2)
pc: PC 2
fase: 2
depende_de: [G03, G05]
rama: feat/mira-app (frontend)
---
# G06 Avisos por distancia en web y móvil

## Objetivo
Avisar al usuario cuando sube de nivel de oferta al acercarse a un restaurante.

## Reparto (2026-10-15)
- Se planteó pasar el móvil a PC 3 (Ruben) y la web a PC 4 (Marco) (hilo #3 de `docs/coordenacion/`), pero Alex (PC 2) ya estaba trabajando en ella en `feat/mira-app`.
- **Decisión final del usuario: G06 se queda con Alex (PC 2)** (web y móvil). PC 3, de momento, no toca la app; a futuro, llevar el panel del restaurante a la app.

## Pasos
- Web: consultar `ofertas/cerca` al moverse (con throttle) y mostrar un aviso en la página.
- Móvil: ubicación en segundo plano (`expo-location` + `expo-task-manager` / geofencing) con notificación local. Ver [[Plan app móvil y notificaciones]].
- No repetir el aviso del mismo nivel para el mismo restaurante.

## Terminada cuando
Simulando posiciones, salen los avisos de 1 km, 500 m y 100 m una sola vez cada uno.

## Avance 2026-10-15 (Alex, PC 2) — primer plano hecho
- Rama `feat/mira-app` (`ee99e99`). Lógica común `models/avisosOfertasModel.js` (web y app, 6 tests): throttle 30 s / 50 m, un aviso por nivel y restaurante (al alejarse no repite un nivel peor) y registro diario `restaurante:nivel` sin posiciones.
- Web: aviso flotante con «Ver» (`#/r/<id>`). App: aviso en pantalla + notificación local que abre la ficha. `POST /v1/ofertas/:id/avisos` por cada aviso.
- `useUbicacion` (web y app): las instancias comparten el consentimiento (al aceptar en Mi cuenta se activan los avisos sin recargar).
- Probado con GPS simulado contra el mock, en la web y en la app: 900 → 400 → 80 → 15 m da 1000/500/100/20 una vez cada uno; al volver a alejarse no repite; los avisos suman en las estadísticas.
- **Falta: segundo plano** (avisos con la app cerrada: `expo-task-manager` + permiso «Permitir siempre» y su consentimiento). APK con G06 en build `1f69638d`.
- Nota dev: en `npm run dev` el adblock bloquea `CookieBanner.jsx` y la web sale en blanco; usar la versión compilada o desactivar el adblock.
