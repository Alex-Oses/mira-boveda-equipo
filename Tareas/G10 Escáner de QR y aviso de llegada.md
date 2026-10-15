---
estado: libre
responsable:
pc: PC 3
fase: 3
depende_de: [G08]
rama:
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
