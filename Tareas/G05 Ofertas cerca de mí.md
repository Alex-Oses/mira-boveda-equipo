---
estado: en curso
responsable: Izan (Claude)
pc: PC 1
fase: 2
depende_de: [G02]
rama: feat/G05-ofertas-cerca
---
# G05 Ofertas cerca de mí

## Objetivo
Un endpoint que, dada una posición, devuelva las ofertas cercanas con el nivel que toca a cada una.

## Pasos
- Publicar antes el contrato de la fase 2 en [[Contratos API]].
- `GET /v1/ofertas/cerca?lat=&lng=`: distancia con haversine, filtrar ofertas activas y devolver el nivel alcanzado.
- **No guardar la posición.** Como mucho, un log sin coordenadas.
- Rate limit, porque se llamará a menudo.

## Terminada cuando
Con posiciones de prueba a 900 m, 400 m y 50 m devuelve el nivel correcto (con test).
