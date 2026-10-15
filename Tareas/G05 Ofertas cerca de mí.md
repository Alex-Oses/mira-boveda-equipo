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


## Revisión de Codex (2026-10-15)
Revisado 95be3ac en worktree propio codex-review-G05. **333/333**. No he modificado el módulo de ofertas. La ruta no persiste posición; requestLogger/accessLog quitan valores de query, y el rate limit usa ruta/IP sin coordenadas. nivelAlcanzado da el nivel inclusivo correcto para 1000/500/100/20 m exactos. Sigue **en revisión** por estos hallazgos:
- **[P2] Clasificar con distancia real, antes de redondear** (`service.js:137-139`). A 1000,4 m devuelve distanciaM 1000 y admite la oferta fuera del radio; a 500,4/100,4/20,4 adelanta el nivel 500/100/20. Reproducido generando posiciones al norte con el mismo radio Haversine. Usar distancia real para radio/nivel/orden y redondear solo el campo de presentación; aclararlo en el contrato y probar ±0,4 m en cada borde, además de la igualdad exacta.
- **[P2] Una carga anterior puede deshacer la invalidación local de caché** (`service.js:115-129`). Reproducido pausando el get de la oferta tras leerla activa, desactivándola con editar (que invalida) y reanudando: la carga vieja repuebla la caché y cerca vuelve a devolver la oferta durante 60 s en la misma instancia. La tolerancia del contrato solo contempla otra instancia. Usar generación/versionado al invalidar y no publicar resultados de una generación anterior; añadir regresión de carga concurrente con editar/borrar.
- **Coste/caché:** dos cerca simultáneas en frío hacen dos consultas completas de ofertas_activas; falta compartir la carga en curso. Además del par oferta/restaurante, se leen los N documentos de bloqueo: el barrido normal son N bloqueos + 2N lecturas individuales (aprox. 3N, no 2N). Conviene compartir la promesa de carga y documentar el coste completo y el pico al caducar. No hay coordenadas del usuario en esa caché.
Sin emulador/Firestore real; reproducción con el servicio y el mock. Pendiente corrección y nueva revisión; sin merge ni despliegue.

## Corrección de la revisión de Codex (2026-10-15, Izan + Claude)
`5f…` en `feat/G05-ofertas-cerca`:
- P2 bordes: radio, nivel y orden con la distancia real; solo se redondea `distanciaM`. Test a ±0,4 m de 20/100/500 m y a 999,6/1000,4 m.
- P2 caché: generación en cada invalidación (una carga vieja no publica) y carga en curso compartida. Tests: carga parada mientras se desactiva la oferta → ni la respuesta ni la caché la devuelven; dos peticiones en frío → una sola consulta de `ofertas_activas`.
- Coste documentado: ~3N lecturas por recarga (N bloqueos + 2N), cada 60 s por instancia con tráfico.
- `npm test` 336/336. Contrato aclarado. Repetir revisión de Codex.

