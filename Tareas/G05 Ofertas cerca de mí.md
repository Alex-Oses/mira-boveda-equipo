---
estado: en revisión
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

## Hecho (2026-10-15, Izan + Claude)
- Contrato de la fase 2 en [[Contratos API]].
- Backend, rama `feat/G05-ofertas-cerca` (`95be3ac`): `GET /v1/ofertas/cerca?lat=&lng=`. Candidatos = `ofertas_activas` + coordenadas del restaurante, caché de 60 s que se invalida al crear, editar o borrar. Haversine, radio 1 km, nivel = el de menor distancia alcanzada, orden por cercanía. `rateLimit(60000, 60)`. **La posición no se guarda ni se registra** (los logs quitan la query; test que lo comprueba).
- `npm test` 333/333 (6 tests nuevos: niveles a 900/400/50/10 m, fuera a 1500 m, orden, inactivas/fuera de vigencia/sin coordenadas, validación, caché e invariante de no guardar posición).
- Mock en el frontend (`feat/G01-contrato-ofertas`): `/v1/ofertas/cerca` con la misma respuesta. Comprobado a 900/400/50/10/1500 m.
- Falta: revisión de Codex. Después, desplegar.

