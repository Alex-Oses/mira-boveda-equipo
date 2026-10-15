---
estado: libre
responsable:
pc: PC 3
fase: 3
depende_de: [G08]
rama:
---
# G10 Escáner de QR y aviso de llegada

## Objetivo
Que el restaurante escanee el QR desde su panel y vea quién ha llegado.

## Pasos
- Escáner con la cámara en el dashboard (librería de lectura de QR en el navegador). Llama a `visitas/validar`.
- Mostrar el resultado: oferta que hay que aplicar, o error (caducado, ya usado, de otro restaurante).
- Lista de llegadas del día y aviso cuando un cliente está cerca (si lo permite el contrato).

## Terminada cuando
Escaneando el QR de G09 desde otro dispositivo, se valida y aparece en la lista.
