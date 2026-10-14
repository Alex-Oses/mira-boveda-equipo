---
estado: libre
responsable:
pc: PC 3
fase: 1
depende_de: [G01]
rama:
---
# G04 Editor básico de ofertas por distancia

## Objetivo
Que el restaurante cree y edite sus ofertas por niveles desde su dashboard.

## Pasos
- Sección nueva en `components/dashboard/`: lista de ofertas, crear y editar.
- Un formulario por nivel (1 km, 500 m, 100 m y llegada) con texto y descuento. Avisar si un nivel no mejora al anterior.
- Primero contra el mock y después contra la API real (G02).

## Terminada cuando
Se crea una oferta completa y vuelve a aparecer al recargar la página.
