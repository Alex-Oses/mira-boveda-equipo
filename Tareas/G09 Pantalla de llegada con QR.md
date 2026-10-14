---
estado: libre
responsable:
pc: PC 2
fase: 3
depende_de: [G06, G08]
rama:
---
# G09 Pantalla de llegada con QR

## Objetivo
A 20 m, mostrar «¡Ya has llegado!» con la oferta y el QR.

## Pasos
- Pedir la visita a la API y pintar el QR (web y móvil).
- Estado de la oferta: pendiente → aplicada (cuando el restaurante escanea) o caducada.
- Botón para regenerarlo si caduca.

## Terminada cuando
El restaurante escanea el QR y en el móvil del cliente pasa a "aplicada".
