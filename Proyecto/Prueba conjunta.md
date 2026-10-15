# Prueba conjunta: ofertas por geolocalización (fases 1-3)

Prueba de punta a punta **en producción** (web: restaurante-mira-frontend.vercel.app · API: mira-api-xveu.onrender.com). Se hace cuando [[G06 Avisos por distancia en web y móvil|G06]] y [[G09 Pantalla de llegada con QR|G09]] (Alex) estén unidas a `main`. Hasta entonces se pueden probar ya las partes marcadas con ✅.

Contratos: [[Contratos API]]. Si algo falla, se apunta abajo en **Incidencias** y como tarea nueva si hace falta.

## Quién hace qué
| Papel | Quién | Necesita |
|---|---|---|
| Restaurante | Ruben o Marco | Cuenta de empresa dueña de un restaurante **con coordenadas** en Firestore (p. ej. «Super Restaurante Mira», `IrSLEIhEfkt0qvJEbSRI`; comprobar que tiene `coordenadas.latitud/longitud`) |
| Cliente web | Izan | Cuenta de cliente, Chrome de escritorio |
| Cliente app | Alex | Cuenta de cliente distinta, APK con `feat/mira-app` |
| PC 1 (apoyo) | Claude y Codex | Revisan respuestas de la API y los contadores |

**Simular la ubicación** (para no tener que ir físicamente):
- Web (Chrome): DevTools → ⋮ → More tools → **Sensors** → Location → «Other…» y escribir latitud/longitud.
- Android: Opciones de desarrollador → «Aplicación de ubicación simulada» (con una app de mock GPS).
- Para cada paso, usar puntos al norte del restaurante: 1 grado de latitud ≈ 111 195 m (900 m ≈ +0,00809; 400 m ≈ +0,0036; 80 m ≈ +0,00072; 15 m ≈ +0,000135).

## Antes de empezar
- [ ] `main` del backend y del frontend desplegados (Render y Vercel), con G06/G09 incluidas.
- [ ] `VISITAS_SECRET` activo: en el escáner, un código inventado (`v1.prueba.1.aaaa`) da «código no válido», no error del servidor.
- [ ] Ninguna otra oferta activa en el restaurante de prueba.

## Fase 1: base
1. ✅ **Restaurante** → Panel → «Ofertas» → «Nueva oferta»: título «PRUEBA CONJUNTA», activa, niveles 1 km 5 %, 500 m 10 %, 100 m 15 %, llegada 20 %. Guardar → aparece en la lista como «Activa». Recargar → sigue ahí.
2. ✅ Intentar una **segunda oferta activa** → mensaje de «ya hay otra activa» (409) dentro del formulario.
3. ✅ Poner un nivel peor que el anterior (500 m 3 %) → aviso en el formulario y no deja guardar.
4. ✅ **Cliente web** → Mi cuenta → «Ubicación para ofertas cercanas» → Activar → modal → Aceptar → se ve la posición simulada. Desactivar → desaparece. Volver a activarla.
5. **Cliente app** → lo mismo en Mi cuenta de la app (permiso del sistema «mientras se usa»).

## Fase 2: embudo de ofertas
6. **Cliente** en el punto a ~900 m → aviso de nivel **1 km (5 %)**, una sola vez.
7. A ~400 m → aviso **500 m (10 %)**; a ~80 m → aviso **100 m (15 %)**. Volver atrás y adelante no repite avisos ya mostrados.
8. A ~1,5 km → no hay aviso.
9. **App en segundo plano** (Alex): con la app cerrada o en segundo plano, acercarse → notificación local de cada nivel.
10. ✅ **Restaurante** → «Ofertas» → estadísticas: el día de hoy suma los avisos de los pasos 6-9 por nivel.

## Fase 3: llegada al restaurante
11. **Cliente** a ~15 m → pantalla «¡Ya has llegado!» con la oferta de llegada (20 %) y un **QR**. A más de 50 m, pedir la visita debe decir que está lejos (422 con la distancia).
12. ✅ **Restaurante** → escáner → cámara (o pegar el código del QR en la entrada manual) → «Llegada confirmada», con el **nombre del cliente** y «20 %». Aparece en «Llegadas de hoy».
13. **Cliente** → la pantalla pasa a «confirmada» sola, en unos segundos.
14. ✅ Escanear **el mismo QR otra vez** → «este QR ya se ha usado» (409).
15. Esperar 15 min sin escanear un QR nuevo (o usar uno viejo) → «caducado» (410); el cliente puede pedir otro.
16. ✅ Un QR de **otro restaurante** (con otra cuenta de empresa) → «es de otro restaurante» (403).
17. ✅ **Restaurante** → estadísticas: hoy cuenta las visitas (QR generados) y las confirmadas.

## Privacidad (comprobar siempre)
- [ ] En las peticiones del navegador (DevTools → Network), la posición solo va en `GET /v1/ofertas/cerca?lat=&lng=` y en el cuerpo de `POST /v1/visitas`; en nada más.
- [ ] En Firestore no aparece ninguna colección ni campo con posiciones del cliente (`ofertas_stats` solo tiene contadores; `visitas` no tiene lat/lng).
- [ ] Al desactivar la ubicación en Mi cuenta, dejan de llegar avisos.

## Al terminar
- Desactivar o borrar la oferta «PRUEBA CONJUNTA».
- Apuntar el resultado de cada paso (✔ / ✖ + qué pasó) en la sección de abajo y crear una sesión en `Sesiones/`.

## Incidencias
_(paso · quién · qué pasó · tarea creada)_
