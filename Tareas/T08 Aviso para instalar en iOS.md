---
estado: libre
responsable:
depende_de: [T06]
rama:
---
# T08 Aviso para instalar en iOS

## Objetivo
Explicar a los usuarios de iPhone cómo instalar la app, porque Safari no lo propone solo.

## Pasos
- Detectar iOS + Safari + que no esté instalada (`navigator.standalone !== true`).
- Banner o modal: "Pulsa Compartir → Añadir a pantalla de inicio", con la opción de cerrarlo y que se recuerde.
- Textos en es/ca/en.

## Terminada cuando
El aviso sale en Safari de iPhone y no sale dentro de la app instalada.
