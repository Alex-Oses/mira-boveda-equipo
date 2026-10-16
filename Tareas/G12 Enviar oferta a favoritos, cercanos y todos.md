---
estado: propuesta
responsable: Alex (PC 2) + PC 1 (backend)
depende_de: [G04, T04, T10, G11]
rama:
---
# G12 Enviar oferta a favoritos, cercanos y todos

## Objetivo
Botones en el editor de ofertas (panel del restaurante y admin) para mandar la oferta como push:
1. **Favoritos y cercanos**: usuarios con el restaurante en `favoritos` + usuarios cerca.
2. **Todos** (solo restaurantes **partner**): todos los usuarios.

## Propuesta (pendiente de acordar)
- Backend (PC 1): `POST /v1/ofertas/:id/enviar { alcance: "favoritos_cercanos" | "todos" }`, solo dueño/admin; `todos` exige `partner: true` en el restaurante (o admin).
- Envío con `enviarPush(uid, { tipo: "promociones", ... })`: **solo a quien tenga `promociones === true`** (decisión T10 / LSSI art. 21). Nadie más.
- «Cercanos»: no guardamos posiciones (contrato G05). Opciones: (a) ciudad del perfil o del restaurante, (b) guardar la última zona aproximada con consentimiento (cambio de privacidad, decisión del equipo).
- Límite: 1 envío por oferta y día; contador en estadísticas (G07).
- Campo nuevo `partner` (bool) en el restaurante, que gestiona el admin. Hoy no existe.

## Dudas para el equipo
- ¿Qué es exactamente «partner» y quién lo activa?
- ¿«Cercanos» por ciudad o guardando zona aproximada?

## Terminada cuando
Desde el panel, «Favoritos y cercanos» llega solo a usuarios con promociones activadas que cumplen; «Todos» solo aparece en partners.
