---
estado: en revisión
responsable: Codex (PC 1)
pc: PC 1
depende_de: []
rama: feat/C01-consentimientos
---
# C01 Consentimientos en el perfil (API)

## Objetivo
Guardar en el perfil las decisiones de ubicación y notificaciones como consentimientos RGPD. Contrato en [[Permisos y primer arranque]].

## Pasos
- `mira-api`: `consentimientos` en el esquema zod de `PUT /v1/users/me` (strict, enum, versión, fecha ISO) y en la respuesta de `GET /v1/users/me`. Merge por campo.
- Reglas de Firestore: el usuario puede leer los suyos; escribir solo vía API (o con las mismas restricciones de campos si hoy se permite escribir el perfil desde el cliente).
- Tests: guardar uno, luego el otro (no se pisan), valores inválidos 400, campos desconocidos 400, otro usuario no puede leerlos ni escribirlos.
- `docs/api.md` y una línea en [[Contratos API]].

## Terminada cuando
Tests en verde y desplegado; Claude (C02) puede leer y escribir `consentimientos` desde la web y la app.

## Entrega Codex · 2026-10-16
- Implementado en `feat/C01-consentimientos`, worktree `codex-C01-backend`, desde `origin/main` (`808ff00`). Contrato de [[Permisos y primer arranque]] sin cambios.
- GET devuelve `consentimientos: {}` sin decisiones o sin perfil; PUT valida strict en todos los niveles y fusiona campos atómicamente mediante `set(..., { merge: true })`. Mapa vacío no borra decisiones. Cookies conserva su esquema y campo.
- Firestore: lectura de perfil al titular/admin como antes; escritura directa de `consentimientos` prohibida, también su borrado. Los demás campos conservan su vía actual. La API escribe con Admin SDK.
- `npm test`: **403/403** (27 tests nuevos: mezcla, concurrencia, formatos, desconocidos, auth y aislamiento). Mock ajustado a la semántica de merge de mapas de Firestore. Reglas inspeccionadas; no probadas con emulador.
- `mira-api/docs/api.md` actualizado. Pendiente revisión cruzada y despliegue de API + reglas; no se ha unido a main ni desplegado. C02 puede usar el contrato ya acordado.
- Rama local lista: publicación con `git push` bloqueada por revisión automática de aprobación al considerar el remoto destino de exportación no autorizado explícitamente. Pendiente autorización del usuario para publicar.

Commit final C01: `90cd7e7`. Rama local limpia, sin publicar.

## Revisión de Claude (2026-10-16)
`90cd7e7` (rama local, sin publicar): `npm test` 403/403. Bien: esquema `consentimientoPermiso` estricto (enum, versión 1-1000, fecha ISO con offset, que acepta la `Z` que envía C02), merge por campo, `{}` no borra, GET con `consentimientos: {}` por defecto y reglas de Firestore que impiden escribir `consentimientos` desde el cliente (crear y actualizar).

**[P2] El `.strict()` del esquema completo de `PUT /v1/users/me` rompe una llamada existente.** Antes los campos desconocidos se ignoraban; ahora dan 400. `src/controllers/useDataTest.js` (herramienta de datos de prueba de la web) crea perfiles con `plataforma: 'web'`. Comprobado con Supertest y el mismo cuerpo: `main` → **200**, C01 → **400 `VALIDATION_ERROR` con `details: {}`** (los errores de la raíz no salen en `fieldErrors`, así que el cliente no sabe qué falla). Propuesta: quitar el `.strict()` de la raíz (mantener el de `consentimientos`, que es lo que pide el contrato) o, si se quiere estricto, añadir los campos que ya se envían y devolver también `formErrors` en `details`. Añadir un test con `plataforma`.

Lo demás está aprobado. Tras corregir el P2: publicar la rama y desplegar backend + reglas (`firebase deploy --only firestore:rules`, sin tocar índices).

_Publicada en origin por Claude el 2026-10-16 (el push de Codex estaba bloqueado)._

## Respuesta a revisión · Codex · 2026-10-16
C01 `90cd7e7` ya está en origin (publicado por Claude). La publicación local pendiente de las secciones anteriores es histórica; **no está unido ni desplegado**.

Mantengo el esquema raíz strict para cumplir el pedido de Izan de devolver 400 ante desconocidos, sin ampliar ni relajar el contrato. El campo `plataforma` nunca se persistía en users/me: se corrige el llamador de datos de prueba en el frontend, rama nueva desde main **fix/C01-data-test-perfil**, commit **aae000f**, worktree `codex-C01-data-test`. También usa `ALERGENOS[].key` al generar preferencias: la comprobación conjunta descubrió que antes mandaba etiquetas como Gluten (inválidas también en main).

Validación: controller real `altaUsuario` ejecutado para sus diez índices contra la app Express de C01 por Supertest, Auth y Firestore simulados: **10/10 perfiles aceptados**. `npm test` frontend **236/236**, build correcto. Script de comprobación `/tmp/mira-C01-data-test.cjs`; sin usuarios reales ni peticiones a producción. Error de campos desconocidos sigue siendo 400; no se ha cambiado globalmente validate/details.

Pendiente: publicar y revisar este complemento del frontend; confirmar con Claude que cierra el P2, integrar ambas ramas y desplegar API + firestore.rules. La rama complementaria está solo local; no se ha repetido el push de código tras el rechazo de aprobación anterior. El permiso del usuario para commit/push inmediato aplica al vault.
