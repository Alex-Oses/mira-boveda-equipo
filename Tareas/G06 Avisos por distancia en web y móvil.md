---
<<<<<<< HEAD
estado: en curso
responsable: Alex (PC 2)
pc: PC 2
=======
estado: libre
responsable: Ruben (PC 3, móvil) + Marco (PC 4, web)
pc: PC 3 / PC 4
>>>>>>> c700157 (reparto web/app (G06/G09 a PC 3, web a PC 4) y G04 a hecho con prueba real)
fase: 2
depende_de: [G03, G05]
rama: feat/mira-app (frontend)
---
# G06 Avisos por distancia en web y móvil

## Objetivo
Avisar al usuario cuando sube de nivel de oferta al acercarse a un restaurante.

## Reparto (2026-10-15, decisión del usuario, hilo #3)
- **Móvil (Ruben, PC 3):** parte de `mobile/`, rama de base `feat/mira-app`. Se apoya en `useUbicacion`/`seguimientoUbicacion` de G03 y en las notificaciones locales (el push de segundo plano va con el plan T01–T10 de Alex).
- **Web (Marco, PC 4):** consultar `ofertas/cerca` con throttle y aviso en la página.

## Pasos
- Web: consultar `ofertas/cerca` al moverse (con throttle) y mostrar un aviso en la página.
- Móvil: ubicación en segundo plano (`expo-location` + `expo-task-manager` / geofencing) con notificación local. Ver [[Plan app móvil y notificaciones]].
- No repetir el aviso del mismo nivel para el mismo restaurante.

## Terminada cuando
Simulando posiciones, salen los avisos de 1 km, 500 m y 100 m una sola vez cada uno.
