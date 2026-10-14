---
estado: libre
responsable:
pc: PC 2
fase: 2
depende_de: [G03, G05]
rama:
---
# G06 Avisos por distancia en web y móvil

## Objetivo
Avisar al usuario cuando sube de nivel de oferta al acercarse a un restaurante.

## Pasos
- Web: consultar `ofertas/cerca` al moverse (con throttle) y mostrar un aviso en la página.
- Móvil: ubicación en segundo plano (`expo-location` + `expo-task-manager` / geofencing) con notificación local. Ver [[Plan app móvil y notificaciones]].
- No repetir el aviso del mismo nivel para el mismo restaurante.

## Terminada cuando
Simulando posiciones, salen los avisos de 1 km, 500 m y 100 m una sola vez cada uno.
