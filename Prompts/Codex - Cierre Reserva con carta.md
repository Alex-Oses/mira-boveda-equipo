# Codex · cierre de Reserva con carta (PC 1, 2026-10-23)

Reparto: **Codex = backend `mira-api`**, **Claude = frontend web y app**. No toques el frontend salvo lo indicado.
Nada de merge a main ni despliegue sin el OK de Izan; sube tus ramas y avisa en la bóveda.

## 1. M14 · sembrar cartas en producción — **HECHO por Alex (705 cartas, por secciones). Sáltalo**; empieza por el punto 2.
Rama `feat/M14-sembrar-produccion`. En producción `GET /v1/cartas/:id` da 404 para todos.
- Modo `--produccion` con `--dry-run` por defecto; escribir solo con `--confirmar`.
- Nunca sobrescribe una carta existente; idempotente.
- **Por secciones** (petición de Alex): Entrantes, Principales, Postres, Bebidas…, como `cartaLibro()` del frontend; ids válidos `^[A-Za-z0-9_-]{1,40}$`, precios 0,01–999,99, alérgenos de la lista.
- Deja en la sesión la salida del dry-run (nº de restaurantes, ejemplo de una carta). **No ejecutes `--confirmar`**: lo lanza Izan tras revisarlo Claude.

## 2. M15 · cerrar y editar promociones (para Rubén)
Rama `feat/M15-promociones-api`. Hoy no hay `PUT` ni `DELETE`, y una campaña caducada sigue `activa` y bloquea la siguiente con 409.
- `PUT /v1/promotions/:id` (empresa dueña o admin): editar `fechaFin`, `presupuestoTotal`, `estado: "pausada" | "finalizada"`.
- El 409 de «ya hay una activa» debe ignorar las caducadas (`fechaFin` pasada) o finalizarlas al crear.
- Documenta el contrato en [[Contratos API]] (sección M15) y avisa a Rubén en [[M15 Editor de promociones en el panel]].

## 3. Revisión de M05/M06/M09 contra tu backend
Ya en `main` del frontend. Revisa `POST /v1/reservations` con `pedido`, `PLATO_NO_DISPONIBLE`/`SIN_CARTA`, `qr`/`presentadaEn` en la lista y el `PUT /v1/cartas/:id` del editor. Anota hallazgos; los arregla Claude.

Al acabar cada punto: `npm test`, sesión en `Sesiones/2026-10-23 Codex.md`, Tablero y push de la bóveda. Commits en español (Conventional Commits).
