---
estado: libre
responsable:
pc: PC 1
fase: 1
depende_de: []
rama:
---
# G01 Contrato API fase 1

## Objetivo
Publicar el contrato de la API de la fase 1 para que los PC 2 y 3 puedan empezar con datos simulados.

## Pasos
- Definir el modelo `oferta`: restaurante, niveles por distancia (1000/500/100/20 m) con su texto y descuento, vigencia y si está activa.
- Endpoints: CRUD de ofertas del restaurante (con auth de negocio) y lectura pública por restaurante.
- Escribirlo en [[Contratos API]] y añadir respuestas de ejemplo al mock (`npm run mock-api`).

## Terminada cuando
Los PC 2 y 3 confirman que pueden trabajar con el mock.
