# Prueba conjunta: ofertas por geolocalización (fases 1-3)

Prueba de punta a punta **en producción** (web: restaurante-mira-frontend.vercel.app · API: mira-api-xveu.onrender.com). Se hace cuando [[G06 Avisos por distancia en web y móvil|G06]] y [[G09 Pantalla de llegada con QR|G09]] (Alex) estén unidas a `main`. Hasta entonces se pueden preparar las partes marcadas con ✅: la API se comprueba localmente con tests y datos simulados; la comprobación de pantallas, cámara y entrega real queda para el recorrido conjunto. **✅ significa disponible para probar, no prueba superada en producción.** Codex no crea/modifica datos de producción salvo petición de Izan; los QR de fixtures locales solo sirven en el backend local.

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
- [ ] Comprobación autenticada de `VISITAS_SECRET` (Izan confirma que ya está configurado en Render): con una cuenta de empresa/admin, un código inventado (`v1.prueba.1.aaaa`) da 400 `QR_INVALIDO`, no 503 `VISITAS_NO_CONFIGURADO`. Sin sesión da 401 y no comprueba el secreto.
- [ ] Ninguna otra oferta activa en el restaurante de prueba. Anotar los contadores iniciales y comparar incrementos, no valores absolutos, en el día de **Europe/Madrid**. Si se publica/cambia una oferta, otra instancia puede tardar hasta 60 s en verla en `/ofertas/cerca`.

## Fase 1: base
1. ✅ **Restaurante** → Panel → «Ofertas» → «Nueva oferta»: título «PRUEBA CONJUNTA», activa, niveles 1 km 5 %, 500 m 10 %, 100 m 15 %, llegada 20 %. Guardar → aparece en la lista como «Activa». Recargar → sigue ahí.
2. ✅ Intentar una **segunda oferta activa** → mensaje de «ya hay otra activa» (409) dentro del formulario.
3. ✅ Poner un nivel peor que el anterior (500 m 3 %) → aviso en el formulario y no deja guardar.
4. ✅ **Cliente web** → Mi cuenta → «Ubicación para ofertas cercanas» → Activar → modal → Aceptar → se ve la posición simulada. Desactivar → desaparece. Volver a activarla.
5. **Cliente app** → lo mismo en Mi cuenta de la app (permiso del sistema «mientras se usa»).

## Fase 2: embudo de ofertas
6. **Cliente** en el punto a ~900 m → aviso de nivel **1 km (5 %)**, una sola vez. Esperar a la siguiente consulta según el throttle de G06; comprobar el nivel devuelto antes de continuar. `/ofertas/cerca` por sí solo no suma avisos: G06 debe hacer un POST `/v1/ofertas/:id/avisos` por cada aviso mostrado.
7. A ~400 m → aviso **500 m (10 %)**; a ~80 m → aviso **100 m (15 %)**. Volver atrás y adelante no repite avisos ya mostrados.
8. A ~1,5 km → no hay aviso.
9. **App en segundo plano** (Alex): caso pendiente de implementación según G06. Probar con los permisos de ubicación y notificación adecuados y consentimiento específico para segundo plano; anotar los avisos locales obtenidos. Probar por separado app en primer plano, segundo plano y cierre forzado: no considerar «app cerrada» equivalente a segundo plano ni dar este paso por superado con una prueba en primer plano. Usar un registro de avisos nuevo de prueba (o el día siguiente) para que la deduplicación de los pasos 6-7 no silencie los mismos niveles; no borrar datos reales del usuario para repetirlo.
10. ✅ **Restaurante** → «Ofertas» → estadísticas: recargar el panel para leer los contadores del día en Madrid. Deben subir solo por los POST de avisos aceptados (204), por nivel; incluir también el nivel 20 m del paso 11. Repetir un POST sí vuelve a sumar: el backend no deduplica por usuario; lo hace G06. Sin G06, esta comprobación se prepara con fixtures locales, no se da por probado el embudo real. Evitar la franja en que el rango UTC del panel aún excluye el día de Madrid (observación conocida en G07).

## Fase 3: llegada al restaurante
11. **Cliente** a ~15 m → nivel de llegada (20 m), pantalla «¡Ya has llegado!» y un **QR**. G09 pide `POST /v1/visitas` con `{ restauranteId, lat, lng }` y Bearer de cliente: 201 `{ visita, qr }`. La API admite distancia real **≤ 50 m** por margen GPS, aunque la pantalla se dispare a 20 m; a más de 50 m devuelve 422 `LEJOS` con `distanciaM` redondeada (50,4 m reales pueden devolver 50). Hace falta oferta activa y vigente; sin ella devuelve 404. Repetir mientras hay una pendiente no caducada devuelve el mismo ID, QR y caducidad original, también con 201, y no suma otra visita.
12. ✅ **Restaurante** → escáner → cámara (o pegar el QR en la entrada manual) → POST `/v1/visitas/validar` con `{ qr }`: 200 `{ visita, aplicar, cliente }`, «Llegada confirmada», **nombre del cliente** y «20 %». La lista se recarga desde GET `/v1/visitas/restaurante`; puede contener visitas pendientes/caducadas además de confirmadas (no aparece únicamente tras validar). Antes de G09, probar con un QR creado mediante tests locales; una creación manual en producción necesita petición de Izan.
   - ✅ Código inventado o firma alterada → 400 `QR_INVALIDO`, «código no válido». No modificar la fecha del QR para simular caducidad: invalida su firma.
13. **Cliente** → G09 consulta GET `/v1/visitas/:id` con la sesión del cliente de esa visita y pasa a «confirmada» en el siguiente sondeo. La API no envía push al validar el QR. El restaurante usa el listado de su restaurante, no este GET exclusivo del cliente.
14. ✅ Escanear **el mismo QR otra vez antes de `caducaEn`**, con la empresa autorizada → 409 `QR_USADO`, «este QR ya se ha usado». Dos validaciones simultáneas deben dar una 200 y otra 409, con una sola confirmación y un único incremento. Tras caducar, el resultado es 410 aunque ya se hubiera validado: caducidad se comprueba antes que uso.
15. ✅ Con otra visita **pendiente**, esperar hasta su `caducaEn` (15 min desde la creación original, sin renovar al reemitir): validar devuelve 410 `QR_CADUCADO` y GET del cliente devuelve `estado: caducada`. Pedir de nuevo crea otra visita/QR con nuevos 15 min y suma una visita. En tests se adelanta el reloj; en producción no editar el token ni cambiar `VISITAS_SECRET` para acelerar la prueba. Un QR viejo con firma manipulada devuelve 400, no 410.
16. ✅ Usar un QR auténtico, existente y aún vigente con una **cuenta de empresa que no sea dueña de ese restaurante y no sea admin** → 403 `OTRO_RESTAURANTE`. Cambiar el restaurante seleccionado no basta si la cuenta también es dueña del restaurante del QR. Firma inválida o caducidad producen antes 400/410.
17. ✅ **Restaurante** → recargar estadísticas y comparar incrementos: `visitas` suma **documentos de visita nuevos**, no reemisiones del QR de una pendiente; `confirmadas` suma una vez al confirmar. Los errores 400/403/404/409/410 y el 422 no incrementan contadores. Cada evento se cuenta en su día en Madrid (una visita creada ayer y confirmada hoy suma la confirmación hoy); la lista de llegadas se filtra por el día de creación. `ofertas_stats` conserva los avisos de todos los niveles al incrementar visitas/confirmadas.

## Alcance de notificaciones
- Los avisos de proximidad de G06 son avisos en página/notificaciones locales. `/ofertas/cerca`, `/ofertas/:id/avisos` y las rutas de visitas no disparan `enviarPush`; un QR confirmado se detecta mediante el sondeo del paso 13.
- T04/T05/T10 se prueban aparte: registrar token Expo/FCM y provocar un evento autorizado de reserva o mensaje. Una reserva recién creada pendiente no avisa; confirmar/cancelar por otra cuenta autorizada sí; la cancelación del propio cliente no le manda aviso. En mensajes se notifica el buzón, sin incluir el contenido privado.
- Las preferencias push por defecto son `{ reservas: true, mensajes: true, promociones: false }`; solo una activación explícita permite promociones. T10 filtra el push del servidor: no demuestra por sí solo que G06 silencie una notificación local. Verificar esa parte al revisar `feat/mira-app`. Aceptación del proveedor no prueba entrega al dispositivo.

## Privacidad (comprobar siempre)
- [ ] En las peticiones del navegador (DevTools → Network), la posición solo va en `GET /v1/ofertas/cerca?lat=&lng=` y en el cuerpo de `POST /v1/visitas`; en nada más.
- [ ] En Firestore no aparece ninguna colección ni campo con posiciones del cliente (`ofertas_stats` solo tiene contadores; `visitas` no tiene lat/lng).
- [ ] Al desactivar ubicación, se para el seguimiento y no se generan nuevos avisos de proximidad (incluido si se desactiva mientras se pide permiso). Esto no desactiva los push de reservas/mensajes ni retira avisos ya entregados; comprobar esos interruptores por separado.

## Al terminar
- Desactivar o borrar la oferta «PRUEBA CONJUNTA».
- Apuntar el resultado de cada paso (✔ / ✖ + qué pasó) en la sección de abajo y crear una sesión en `Sesiones/`.

## Revisión pendiente cuando Alex entregue G06/G09
Estado leído el 2026-10-15: ambas **en curso** en `feat/mira-app`; no se ha revisado aún como entrega final. Cuando Alex las marque «en revisión», Codex revisa con Claude las llamadas a cerca, avisos, visitas y push; Claude revisa pantallas y móvil. Revisar en worktrees propios, sin modificar las ramas de otros PC. Comprobar throttle/deduplicación, consentimiento, cuerpos/respuestas, autenticación, errores, reemisión/caducidad y separación entre notificación local y push.

## Incidencias
_(paso · quién · qué pasó · tarea creada)_

### Comprobación local del PC 1 (Codex · 2026-10-15)
Backend `origin/main` **808ff00**, worktree propio `codex-review-G07`. **376/376 tests** pasan con Firestore/proveedores push simulados. Recorrido adicional con Supertest y Firestore en memoria: correcto. No se han enviado peticiones ni escrito datos en producción; la configuración/entrega real y cámara quedan para la prueba conjunta. ✅ de arriba no se ha convertido en un aprobado de producción.

| Paso | Resultado local | Incidencia / corrección del guion |
|---|---|---|
| 1-3 | ✔ Alta 201; segunda activa 409 `CONFLICT`; nivel que empeora 400 | Sin diferencia de API; la UI se prueba aparte |
| 6-10 | ✔ 900/400/80/15 m → niveles 1000/500/100/20; 1500 m → lista vacía; avisos 204 | Añadidos throttle, caché, nivel 20 y recarga de estadísticas; cada POST aceptado suma, G06 debe deduplicar |
| 11 | ✔ 15 min; repetir pendiente da mismo ID/QR/fecha; 50,4 m → 422 `LEJOS` con `distanciaM: 50` | Diferenciados disparo UI a 20 m y radio API ≤50 m; no contar reemisiones |
| 12, 14-16 | ✔ 200 al confirmar; 400 `QR_INVALIDO`, 403 `OTRO_RESTAURANTE`, 404 `NOT_FOUND`, 409 `QR_USADO`, 410 `QR_CADUCADO` | Precisadas firma, propiedad y precedencia de caducidad; reloj avanzado solo localmente |
| 17 | ✔ Tras 1 POST por nivel + 1 repetido a 1000 m, 3 visitas nuevas y 1 confirmación: `avisos={1000:2,500:1,100:1,20:1}`, `visitas=3`, `confirmadas=1` | Ni reemitir ni los errores suman visitas/confirmadas; conservados los mapas de avisos tras la integración G07/G08 |
| Privacidad / push | ✔ Suite de privacidad y T04/T05/T10 en verde; listado sin UID y visitas sin lat/lng | Separados avisos locales, sondeo del QR y push; segundo plano de G06 sigue pendiente según su nota |

No se ha encontrado un fallo de backend en estos casos; las diferencias eran expectativas del guion y quedan corregidas arriba. Sin tarea de código nueva. Pruebas web/app reales (incluidas entrega de push y permisos) pendientes. La ficha G08 tenía `estado: en revisión` pese al despliegue y al Tablero: alineada a `hecho` con la confirmación de Izan.
