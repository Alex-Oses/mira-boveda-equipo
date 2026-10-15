---
estado: en revisión
responsable: Codex (PC 1)
pc: PC 1
fase: 3
depende_de: [G05]
rama: feat/G08-visitas-qr
---
# G08 QR firmado, visitas y comisión

## Objetivo
Generar un QR único por visita y validarlo cuando lo escanea el restaurante.

## Pasos
- Publicar antes el contrato de la fase 3 en [[Contratos API]].
- `POST /v1/visitas` (cliente a ≤ 50 m, según contrato fase 3): crea la visita y devuelve un token firmado (HMAC/JWT) con caducidad corta.
- `POST /v1/visitas/validar` (restaurante): comprueba la firma, que no haya caducado, que no se haya usado y que sea de su restaurante. Marca la visita como confirmada y registra la comisión.
- Tests para los casos de QR caducado, reutilizado y de otro restaurante.

## Terminada cuando
Un QR solo se valida una vez, únicamente en su restaurante y antes de que caduque.

## Entrega de Codex (2026-10-15)
- Rama [feat/G08-visitas-qr](https://github.com/ruben-munoz-bl/restuarante-mira-backend/tree/feat/G08-visitas-qr), commit `ed2b8b8`, desde main 0249a99 en worktree propio codex-G08-backend. Pendiente revisión de Claude antes de main.
- Módulo visitas (routes/service/logica), cuatro endpoints del contrato. Oferta pública + relectura transaccional, radio real ≤50 m, copia nivel 20 m; sin persistir ni registrar posición.
- QR HMAC-SHA256 base64url con timingSafeEqual; 15 min; reemisión conserva caducidad original (confirmado por Izan, texto determinista idéntico). Bloqueo auxiliar visitas_pendientes por hash cliente/restaurante evita duplicados de altas simultáneas.
- Validación en orden de contrato; confirmación única y comisión actual en transacción (incluye 0 %, fallback COMISION_PCT si falta campo). Al restaurante solo cliente.nombre; ningún endpoint devuelve UID. GET proyecta caducada sin alterar confirmadas.
- visitas y confirmadas se incrementan atómicamente con FieldValue.increment en ofertas_stats del día de cada evento en Madrid; merge conserva avisos. No se han tocado rutas de avisos/estadísticas ni módulos de ofertas.
- **npm test 370/370**, 21 nuevas; sintaxis y git diff --check correctos. Primera ejecución: ECONNRESET en test de webhook no modificado; aislado 13/13 y suite completa final en verde.
- Mock compartido: añadido soporte de admin.firestore.FieldValue.increment y resolución de incrementos en set/update; conserva cola de transacciones. Atención al unir con el trabajo de estadísticas de Claude.
- **Antes de desplegar:** configurar VISITAS_SECRET aleatorio (mínimo 32 caracteres) solo en Render. Falta o corto → 503 para crear/validar, sin secreto por defecto. Reglas cerradas para visitas y visitas_pendientes.
- **Índice nuevo:** visitas, restauranteId ASC + fecha ASC + creada DESC, añadido a firestore.indexes.json; publicar y esperar antes de usar listado. Avisado al usuario; nunca --force ni borrar índices existentes.
- Docs §22 ter antes de «## 23. Resumen». Sin merge/despliegue; no emulador por falta de Java ni pruebas contra Firestore real. Pendiente prueba conjunta de QR con PC 2/PC 4 tras revisión.

## Revisión de Claude (2026-10-15)
`ed2b8b8`: **aprobado.** `npm test` 370/370 (21 tests de visitas). Cumple el contrato de la fase 3:
- QR `v1.<id>.<caduca>.<firma>` con HMAC-SHA256, formato canónico estricto (solo 43 caracteres base64url, sin variantes de la misma firma) y `timingSafeEqual`; sin `VISITAS_SECRET` de 32+ caracteres responde 503 en vez de firmar con algo débil.
- Validar: firma → caducidad → existe → dueño (`OTRO_RESTAURANTE`) → pendiente, con la confirmación, la comisión y el contador en una transacción; además comprueba que la caducidad del QR coincide con la de la visita. Un QR solo vale una vez.
- Crear: oferta vigente releída en la transacción, ≤ 50 m, nivel de 20 m copiado, una visita pendiente por cliente y restaurante con bloqueo propio (`visitas_pendientes`), y reemitir no alarga la caducidad. La posición no se guarda (va en el body, que no se registra).
- Nunca devuelve el `uid` del cliente; solo `cliente.nombre`.

**Antes de desplegar:**
1. `VISITAS_SECRET` en Render (32+ caracteres; por ejemplo, `openssl rand -base64 48`).
2. Publicar el índice nuevo de `visitas` (restauranteId, fecha, creada desc) **sin `--force`**, igual que con T04.

**Al unir con mi rama `feat/G07-estadisticas-api`:** los dos hemos cambiado `FieldValue.increment` en `test/helpers/mockFirebase.js`. Me quedo con mi versión, porque fusiona en profundidad los mapas con incrementos (como Firestore con `merge: true`); con la tuya, `set({ avisos: { 500: inc } }, { merge: true })` borraría `avisos.1000`. También cambiaré tu `contar()` por `ofertas.contarVisita()` para que solo haya un sitio que escriba `ofertas_stats` (en la transacción hace falta la versión con `tx`, así que añadiré un parámetro opcional).
