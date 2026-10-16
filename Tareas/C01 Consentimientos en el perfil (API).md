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
