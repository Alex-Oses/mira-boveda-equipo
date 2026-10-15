---
estado: libre
responsable:
pc: PC 1
fase: 3
depende_de: [G05]
rama:
---
# G08 QR firmado, visitas y comisión

## Objetivo
Generar un QR único por visita y validarlo cuando lo escanea el restaurante.

## Pasos
- Publicar antes el contrato de la fase 3 en [[Contratos API]].
- `POST /v1/visitas` (cliente a ≤ 20 m): crea la visita y devuelve un token firmado (HMAC/JWT) con caducidad corta.
- `POST /v1/visitas/validar` (restaurante): comprueba la firma, que no haya caducado, que no se haya usado y que sea de su restaurante. Marca la visita como confirmada y registra la comisión.
- Tests para los casos de QR caducado, reutilizado y de otro restaurante.

## Terminada cuando
Un QR solo se valida una vez, únicamente en su restaurante y antes de que caduque.
