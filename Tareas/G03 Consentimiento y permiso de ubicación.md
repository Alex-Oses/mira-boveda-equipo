---
estado: en curso
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
