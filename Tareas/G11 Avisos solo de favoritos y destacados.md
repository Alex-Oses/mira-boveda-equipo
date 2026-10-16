---
estado: propuesta
responsable: Alex (PC 2) + PC 1 (backend)
depende_de: [G05, G06, T10]
rama:
---
# G11 Avisos solo de favoritos y destacados

## Objetivo
Que los avisos de ofertas por distancia (G06) solo salten para restaurantes que el usuario tiene en **favoritos** o que estén **destacados/promocionados** en la web, para no saturar.

## Propuesta (pendiente de acordar: cambia el comportamiento de G06)
- Backend (PC 1): `GET /v1/ofertas/cerca?solo=avisables` (o filtro por defecto en avisos) devuelve solo ofertas de `usuarios/{uid}.favoritos` ∪ restaurantes con `destacado: true`. Requiere sesión para leer favoritos.
- Nuevo campo `destacado` (bool) en el restaurante, que activa el admin desde el panel. Hoy no existe ningún campo de «promocionado».
- Respetar el interruptor `preferenciasPush.promociones` (T10): apagado → sin avisos de distancia.
- Web y app: usar el filtro en `useAvisosOfertas` / tarea de segundo plano. El mapa y «Ofertas cerca de mí» siguen mostrando todo.

## Límite iPhone
La PWA en iOS no tiene ubicación en segundo plano: solo avisa con MIRA abierta (desde PR #7 también como notificación del sistema). Avisos con el móvil en el bolsillo en iPhone = app nativa iOS (Apple Developer, 99 €/año).

## Terminada cuando
Con GPS simulado: un favorito y un destacado avisan; otro restaurante con oferta no avisa; con promociones apagado no avisa nada.
