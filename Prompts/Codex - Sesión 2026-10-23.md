# Codex · sesión 2026-10-23 (PC 1)

Lee antes: [[Contratos API]] (sección M01), [[M11 Escáner QR para el check-in]] y el aviso de Alex en esa nota.
Repo frontend: `ruben-munoz-bl/restaurante-mira-frontend`. Claude hace M10 y M11 **web** (panel); tú no toques `src/components/dashboard/`.

## 1. Revisión cruzada de Alex (M05, M06, M09)
Rama `feat/M05-M06-M09-reserva-carta` (`b0a871c`). Claude ya comprobó: contrato M01 respetado, 327/327 tests, build y sin conflictos con main.
Tú revisa contra **tu** backend: cuerpo de `POST /v1/reservations` con `pedido`, errores `PLATO_NO_DISPONIBLE`/`SIN_CARTA`, `qr` y `presentadaEn` en `GET /v1/reservations`, y el `PUT /v1/cartas/:id` del editor (límites, ids, `foto` null). Anota hallazgos en la sesión; no cambies su rama.

## 2. Mock para probar M10 y M11 sin backend
Rama `feat/M11-mock-y-app` **desde** `ruben/feat/M05-M06-M09-reserva-carta` (el mock ya tiene cartas allí).
En `scripts/mock-api.mjs` añade, con la forma exacta del contrato:
- `POST /v1/reservations/checkin {qr}` → `{ reserva: {id,codigo,fecha,hora,comensales,presentadaEn,pedido}, cliente:{nombre}, puntosPedido: 50|0 }`; errores del contrato: 400 `QR_INVALIDO`, 403 `OTRO_RESTAURANTE`, 404 `NOT_FOUND`, 409 `QR_USADO`, 409 `RESERVA_CANCELADA`, 409 `FUERA_DE_FECHA`.
- `GET /v1/reservations/restaurante?restauranteId&desde&hasta` → `{ data:[…] }` (sin uid ni email).
Commit propio y avisa a Claude en la nota M11 en cuanto esté (lo necesita para probar).

## 3. M11 en la app móvil
Misma rama. En `mobile/src/components/dashboard/EscanerLlegadas.jsx`: si el QR empieza por `r1.`, llama a `POST /v1/reservations/checkin`; muestra nombre del cliente, hora, comensales y los platos del pedido con total; mensajes claros para cada error. Los `v1.` siguen yendo al flujo de visitas de G10. Test si es lógica pura; `expo lint` en verde.

## 4. M14 en producción (solo preparar)
En `mira-api`, rama `feat/M14-sembrar-produccion`: modo `--produccion --dry-run` que lista qué cartas crearía sin escribir; nunca sobrescribe una carta existente. **No lo ejecutes contra producción**: Izan da el OK.

Al acabar cada punto: `npm test`, build, sesión en `Sesiones/2026-10-23 Codex.md` y estado en el Tablero. Commits en español (Conventional Commits).
