---
<<<<<<< HEAD
estado: en curso
responsable: Alex (PC 2)
pc: PC 2
=======
estado: libre
responsable: Ruben (PC 3, Opencode)
pc: PC 3
>>>>>>> c700157 (reparto web/app (G06/G09 a PC 3, web a PC 4) y G04 a hecho con prueba real)
fase: 3
depende_de: [G06, G08]
rama: feat/mira-app (frontend)
---
# G09 Pantalla de llegada con QR

## Objetivo
A 20 m, mostrar «¡Ya has llegado!» con la oferta y el QR.

## Reparto (2026-10-15, decisión del usuario, hilo #3)
- **Móvil (Ruben, PC 3):** en `mobile/`, rama de base `feat/mira-app` (la web de esta pantalla, si hace falta, la retoma PC 2 o PC 4; avisar en el hilo).

## Pasos
- Pedir la visita a la API y pintar el QR (web y móvil).
- Estado de la oferta: pendiente → aplicada (cuando el restaurante escanea) o caducada.
- Botón para regenerarlo si caduca.

## Terminada cuando
El restaurante escanea el QR y en el móvil del cliente pasa a "aplicada".
