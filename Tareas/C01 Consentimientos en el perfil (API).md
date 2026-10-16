---
estado: libre
responsable: Codex (PC 1)
pc: PC 1
depende_de: []
rama:
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
