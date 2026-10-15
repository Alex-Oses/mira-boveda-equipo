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
