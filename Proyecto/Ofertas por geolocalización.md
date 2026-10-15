# Proyecto actual: ofertas por geolocalización (web + app)

Propuesta aprobada el 2026-10-14.

## Qué hace
- El usuario comparte su ubicación, con consentimiento. No habrá inicio de sesión por SMS.
- Las ofertas mejoran al acercarse a un restaurante: **1 km → 500 m → 100 m**.
- A **20 m** sale el aviso «¡Ya has llegado!» con la oferta y un **QR único por visita**, firmado y con caducidad.
- El restaurante escanea el QR desde su panel. Así queda confirmado que el cliente llegó por MIRA, se aplica la oferta y cuenta para la comisión y los informes.
- En la web la ubicación solo se sigue con la página abierta. En la app móvil también en segundo plano.
- El GPS no es fiable a 20 m, así que **la prueba real es el QR**.
- **RGPD:** consentimiento explícito, opción de desactivarlo y no guardar el recorrido.

## Equipo: 4 personas en 4 PC
| PC | Quién | Parte |
|---|---|---|
| PC 1 | Izan + Claude + Codex | Backend y datos: API de ofertas por distancia, ofertas cerca de mí, QR (generar y validar), visitas y comisión |
| PC 2 | Alex (+ Claude) | App del cliente (`mobile/`, `feat/mira-app`): consentimiento (hecho, G03), avisos por distancia (G06), pantalla de llegada (G09), notificaciones T01–T10 |
| PC 3 | Ruben | Panel del restaurante: editor de ofertas (G04, hecho). A futuro: **llevar el panel del restaurante a la app** |
| PC 4 | Marco | Web del proyecto de ofertas: revisión/merge del editor (G04), estadísticas (G07), escáner de QR (G10) |

**Regla clave:** al empezar cada fase, el PC 1 publica el **contrato de la API** en [[Contratos API]]. Así los PC 2 y 3 pueden trabajar con datos simulados (`npm run mock-api`).

## Fases
Cada fase termina con la unión de ramas, una revisión cruzada y una prueba conjunta.
1. **Base:** modelo de ofertas, consentimiento de ubicación y editor básico.
2. **Embudo de ofertas:** ofertas por distancia y avisos al acercarse.
3. **Llegada al restaurante:** QR, escáner, visitas y comisión.

El detalle de cada tarea está en el [[Tablero]].

## Relación con las notificaciones
Los avisos por distancia de la app móvil (en segundo plano) necesitan notificaciones. Para eso está el plan anterior: [[Plan app móvil y notificaciones]] (tareas T01-T10). Las notificaciones locales (las que lanza la propia app al entrar en la zona) no necesitan backend. Las push sí.

## Riesgos
- Geolocalización en segundo plano en Android e iOS: permisos y revisión de las tiendas.
- Render gratis se duerme: la primera petición de "ofertas cerca" puede tardar.
- No guardar el recorrido: solo eventos sueltos ("visita confirmada").
