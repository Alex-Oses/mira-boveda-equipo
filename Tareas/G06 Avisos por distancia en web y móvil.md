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
