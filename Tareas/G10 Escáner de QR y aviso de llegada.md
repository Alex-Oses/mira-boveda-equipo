---
estado: hecho
responsable: Marco (PC 4)
pc: PC 3
fase: 3
depende_de: [G08]
rama: feat/G10-escaner-qr (frontend)
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

## Revisión de Claude (PC 1) de la rama `feat/G10-escaner-qr` (2026-10-15)
`cd6c820`: `npm test` 202/202 y build correcto (pero el build pasa porque el componente no se usa en ningún sitio; ver abajo). El escáner está bien resuelto: cámara trasera, lectura con `jsQR`, bloqueo para no validar dos veces el mismo QR, cámara apagada al validar o desmontar, entrada manual de respaldo y un estado por cada error.

Se programó contra `docs/coordenacion/CONTRATOS-ASUMIDOS.md`, y **el contrato oficial ya está publicado** en [[Contratos API]] (fase 3); G08 (Codex) ya lo implementa y está aprobado. Hay que adaptarlo:
- **[P1] Ruta y cuerpo:** `POST /v1/visitas/validar` con `{ qr }` (no `/v1/ofertas/visitas/validar` con `{ codigo }`).
- **[P1] Formato del QR:** es `v1.<visitaId>.<caducaEnMs>.<firma>`, no `MIRA.<…>.<…>`. `RE_CODIGO`/`esCodigoFormaValida` solo se usan en el mock, pero si alguien los usa para filtrar rechazaría todos los QR reales. Actualizarlos (`/^v1\.[A-Za-z0-9_-]{1,200}\.\d{1,16}\.[A-Za-z0-9_-]{43}$/`) o quitarlos y dejar que valide el servidor.
- **[P1] Respuesta:** la real es `{ visita, aplicar: { texto, descuentoPct }, cliente: { nombre } }`. Adaptar `onValidar` en quien lo monte: `visitaId = visita.id`, la oferta que se aplica en `aplicar` y el nombre del cliente en `cliente.nombre` (que conviene mostrar). Los códigos de estado coinciden (400, 403, 404, 409, 410), así que `estadoDesdeError` sirve; los `error` reales son `QR_INVALIDO`, `OTRO_RESTAURANTE`, `QR_USADO` y `QR_CADUCADO`.
- **Lista de llegadas:** `GET /v1/visitas/restaurante?restauranteId=&fecha=` → `{ data: [Visita + cliente: { nombre }] }`. Cargarla al abrir el escáner y después de cada validación, en vez de llevarla solo en memoria.
- **Falta montarlo** en el panel (dashboard → ofertas) con el `restauranteId` del restaurante activo.
- Menor: si el componente se desmonta mientras `getUserMedia` espera, el stream que llega después no se para. Comprobar si sigue montado antes de guardarlo.
- `jsqr` está en `package.json`; quien una la rama tiene que hacer `npm install`.

## Segunda revisión de Claude: adaptado al contrato oficial (2026-10-15)
`80c751c` (Marco): **aprobado.** `POST /v1/visitas/validar` con `{ qr }`, sin filtrar el formato en el cliente (lo comprueba el servidor); muestra `cliente.nombre` y lo que hay que aplicar (`aplicar`); la lista de llegadas viene de `GET /v1/visitas/restaurante` y se recarga después de cada validación; corregido el stream de la cámara si se desmonta mientras espera el permiso. Montado en el panel de ofertas.
- Probado en el navegador con la entrada manual: validación correcta → «Llegada confirmada», nombre del cliente y oferta; QR usado (409) → «Este QR ya se ha usado». La cámara no se ha probado (sin cámara en este entorno).
- Necesita `npm install` al unir (`jsqr`). Prueba real pendiente de `VISITAS_SECRET` en Render.

## Revisión de Codex: compatibilidad con G08 (2026-10-15)
- **Aprobado `80c751c` del lado del backend, sin hallazgos en el alcance revisado.** Revisión en el worktree propio `codex-review-G10`; no se han modificado las ramas ni el código de Marco.
- `POST /v1/visitas/validar`: ruta y cuerpo `{ qr }` correctos, mediante el cliente HTTP autenticado. Lee `visita.id`/fechas, `cliente.nombre` y prioriza `aplicar.texto`/`descuentoPct` para el resultado; no espera UID ni el formato anterior.
- `GET /v1/visitas/restaurante?restauranteId=…`: lee `{ data: [...] }`, transforma cada `Visita + cliente`, conserva el orden del servidor y usa el día de Madrid por defecto al omitir `fecha`. Carga al abrir y recarga después de validar; los errores de lista muestran el aviso correspondiente.
- Errores de validación: **400 QR_INVALIDO y 404 NOT_FOUND → código no válido; 403 OTRO_RESTAURANTE → otro restaurante; 409 QR_USADO → ya usado; 410 QR_CADUCADO → caducado.** `httpClient` conserva `status`, `estadoDesdeError` lo interpreta y las claves mostradas existen en es/ca/en.
- Comprobación cruzada ejecutada: respuestas HTTP producidas por G08 `ed2b8b8` con Supertest/Firestore simulado, consumidas por el modelo real de G10. Alta y validación correctas, lista tras confirmar, nombre/nivel/descuento/fecha coincidentes, ausencia de UID y los cinco errores anteriores: **OK**. Se inspeccionó además el render del componente y su uso de `aplicar`.
- `npm test`: **211/211**; `npm run build` correcto (aviso de tamaño de chunk); `git diff --check` correcto y worktree limpio. No se ha probado cámara física ni Render: queda la prueba conjunta con G09 tras configurar `VISITAS_SECRET`, publicar el índice sin `--force` y desplegar G08.

## Unido a main (2026-10-15, Izan + Claude)
`main` del frontend `218531a` (G07 + G10 juntos), 227/227 tests y build correcto, desplegado en Vercel (en el código publicado ya están `/v1/ofertas/estadisticas`, `/v1/visitas/validar` y `/v1/visitas/restaurante`). Panel del restaurante → «Ofertas»: editor, escáner y estadísticas.
