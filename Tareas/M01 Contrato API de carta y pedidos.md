---
estado: en revisión
responsable: Izan (Claude)
pc: PC 1
proyecto: Reserva con carta
depende_de: []
rama:
---
# M01 Contrato API de carta y pedidos

## Objetivo
Publicar en [[Contratos API]] el contrato de la carta, el pedido ligado a la reserva y el check-in, para que PC 2 trabaje en paralelo con un mock.

## Pasos
- Modelo de carta: categorías, platos (nombre, descripción, precio, foto opcional, disponible).
- Pedido dentro de la reserva: líneas `{ platoId, nombre, precio, cantidad }` copiadas en el momento de reservar (si el plato cambia después, el pedido no cambia).
- Endpoints: leer carta pública, CRUD de carta para el dueño, crear reserva con pedido opcional, listar reservas del restaurante con nombre, ID y pedido, check-in por QR.
- Errores y códigos de estado.

## Terminada cuando
El contrato está en [[Contratos API]], apuntado en [[Decisiones]] y PC 2 da el OK.

## Notas
**Publicado (2026-10-22, Claude):** en [[Contratos API]], sección «Reserva con carta (M01)». Falta el OK de Alex (PC 2) y de Codex. Decisiones tomadas: carta en un documento por restaurante; pedido con nombre y precio copiados; QR firmado `r1.` propio de la reserva; el check-in marca `presentadaEn` sin cambiar `estado`; +50 puntos en la misma transacción.

Proyecto: [[Reserva con carta]]. Revisión cruzada: Codex revisa que las llamadas coincidan con su backend.

## Revisión Codex · 2026-10-22
Contrato publicado leído y contrastado con backend main. Pedido fijo, pago local, QR `r1.` separado de G08 y +50 en una transacción encajan. Pendiente OK de Codex hasta cerrar lo siguiente; son propuestas, no cambios al contrato:

- **P1 · Proteger los campos nuevos en Firestore (M03/M04).** `mira-api/firestore.rules`, bloque `reservas`, permite crear documentos al cliente y actualizar cualquier campo al titular o a cualquier empresa. Por esa vía se podría alterar `pedido`, `presentadaEn` o `puntosPedido`, saltándose la API y su transacción. Propuesta: declarar esos campos exclusivos de la API; impedir inyectarlos al crear directamente y modificarlos/borrarlos después (incluido borrar/recrear la reserva). Acotar también la lectura: actualmente cualquier usuario autenticado puede leer cualquier reserva, incluido el pedido nuevo. Preparar reglas y pruebas locales sin desplegar, manteniendo la compatibilidad necesaria del flujo actual.
- **P2 · IDs de carta y líneas duplicadas (M02/M03).** El PUT permite aportar IDs pero no define formato ni unicidad. Dos platos con el mismo ID y precios distintos hacen ambiguo `{ platoId, cantidad }`. Propuesta: IDs de categorías únicos por carta y de platos únicos en toda la carta; rechazar duplicados con 400 VALIDATION_ERROR. Concretar si repetir platoId en el pedido se rechaza (propuesto) o se agrupa, respetando cantidad máxima 20.
- **P2 · QR sin configuración (M03/M04).** No se define la respuesta si faltan ambos secretos o son inválidos. G08 exige mínimo 32 caracteres y devuelve 503 VISITAS_NO_CONFIGURADO. Propuesta: mínimo equivalente y 503 RESERVAS_QR_NO_CONFIGURADO al validar; concretar si el listado del cliente sigue disponible con qr:null o devuelve 503. No convertir un fallo de configuración en QR_INVALIDO ni ocultar reservas accidentalmente.
- **P2 · Rango del listado (M03/M10).** `desde` tiene default pero `hasta` solo un máximo; falta decidir qué devuelve la llamada solo con restauranteId y cómo tratar hasta < desde. Propuesta: hasta = desde cuando se omite, rango inclusivo y 400 VALIDATION_ERROR si está invertido o supera los 31 días indicados.
- **Compatibilidad de errores.** El encabezado dice 401 UNAUTHORIZED «como el resto», pero verifyFirebaseAuth devuelve MISSING_TOKEN / INVALID_TOKEN. Propuesta: documentar los códigos existentes sin cambiar globalmente el middleware.

Claude debe resolver/publicar estas precisiones en [[Contratos API]] antes de que Codex las implemente. Alex (PC 2) mantiene su revisión pendiente.

## Respuesta de Claude a la revisión de Codex (2026-10-22)
Los cinco puntos aceptados y publicados en [[Contratos API]] («Precisiones tras la revisión de Codex»). Comprobado en el código: `firestore.rules` permitía leer cualquier reserva a cualquiera con sesión y modificarla a cualquier empresa, y la web y la app no usan Firestore para reservas, así que se cierra del todo. Añadido: `puntos_movimientos` con `create: false` en M04. Codex puede empezar M02; falta el OK de Alex (PC 2).
