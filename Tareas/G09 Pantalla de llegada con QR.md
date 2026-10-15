---
estado: en revisión
responsable: Alex (PC 2)
pc: PC 2
fase: 3
depende_de: [G06, G08]
rama: feat/mira-app (frontend)
---
# G09 Pantalla de llegada con QR

## Objetivo
A 20 m, mostrar «¡Ya has llegado!» con la oferta y el QR.

## Reparto (2026-10-15)
- Se planteó pasarla a PC 3 (Ruben) (hilo #3 de `docs/coordenacion/`), pero Alex (PC 2) ya estaba trabajando en ella en `feat/mira-app`.
- **Decisión final del usuario: G09 se queda con Alex (PC 2).** PC 3, de momento, no toca la app; a futuro, llevar el panel del restaurante a la app.

## Pasos
- Pedir la visita a la API y pintar el QR (web y móvil).
- Estado de la oferta: pendiente → aplicada (cuando el restaurante escanea) o caducada.
- Botón para regenerarlo si caduca.

## Terminada cuando
El restaurante escanea el QR y en el móvil del cliente pasa a "aplicada".

## Entrega 2026-10-15 (Alex, PC 2) — en revisión
- Web `27e46d2` (`#/llegada/<id>`) y app `f2df4c2` (`/llegada?id=`): crea la visita (≤ 50 m), pinta el QR (`qrcode` → path SVG), cuenta atrás de 15 min y sondeo cada 4 s hasta «Oferta aplicada» o «caducada» (botón «Pedir otro QR»). 422 LEJOS muestra la distancia; 401/404/503 con mensaje propio. El aviso de llegada de G06 abre esta pantalla.
- Mock: `POST/GET /v1/visitas`, `validar` y lista del restaurante según el contrato (firma «mock»). Probado en la web contra el mock: lejos → QR → validar → «Oferta aplicada».
- Falta probar de verdad cuando esté `VISITAS_SECRET` en Render.
