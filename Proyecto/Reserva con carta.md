# Reserva con carta (proyecto actual, desde 2026-10-22)

Origen: pizarra y prompt «Reserva de carta» del 2026-10-22. Reparto acordado con Izan el mismo día. Tareas M01 a M11 en el [[Tablero]].

## Qué queremos
Que el cliente pueda **reservar mesa y, si quiere, pedir la comida por adelantado**, y que el restaurante lo tenga en su panel. Además, una landing por secciones y una cabecera más limpia.

## Flujo del cliente (web y app)
1. Reserva como siempre: restaurante, fecha, hora y personas.
2. Al final se le pregunta: «¿Quieres pedir la comida por adelantado?». Si dice que sí, elige platos de la carta del restaurante. Si el restaurante no tiene carta, no se ofrece.
3. El pedido queda **fijo** junto a la reserva: no se edita en la web.
4. En «Mis reservas» ve la reserva con su **QR** y su pedido.
5. Llega al local, el restaurante escanea el QR y la reserva queda como **presentada**. Desde ese momento la cocina se pone en marcha.
6. Paga en el local después de comer: **no hay pago online**. En la web solo queda apuntado.

## Reglas
- El pedido **es un compromiso**, no una sugerencia: el restaurante lo tiene que cumplir.
- Los cambios y los platos agotados se hablan **de palabra al llegar**. La web no gestiona cambios.
- **+50 puntos** de fidelidad si el cliente se presenta (QR escaneado) a una reserva que llevaba pedido. Se dan al escanear, no al reservar.
- La carta la crea el restaurante al darse de alta y la puede editar después.

## Panel del restaurante
- Sección **Pedidos → Reservas**: nombre de quien reserva, ID de la reserva y pedido, si lo hay.
- El escáner de QR **ya existe** ([[G10 Escáner de QR y aviso de llegada]]): se reaprovecha para marcar la reserva como presentada.

## Landing (esquema de la pizarra)
Secciones más compactas («+ optimizadas, menos espacio»), en este orden:
1. Cabecera
2. Foto principal
3. Filtros
4. Promociones
5. Más reservados
6. Para ti (fotos de platos)
7. Top 50 Barna
8. Nuevos
9. Bloque final con imagen: «¡Sube tu restaurante!» (enlace) o «disfruta de las ventajas de ser cliente»

## Cabecera
- Quitar el **botón de la racha** (el popup de la racha sigue saliendo como ahora).
- Quitar los **puntos**: se consultan solo en el perfil.

## Reparto
| PC | Quién | Tareas | Zona del código |
|---|---|---|---|
| PC 1 | Izan (Claude + Codex) | M01, M02, M03, M04, M10, M11 | `mira-api` y panel del restaurante (pedidos y escáner) |
| PC 2 | Alex (Claude) | M05, M06, M09 | modal de reserva, «Mis reservas» (web y app) y alta del restaurante |
| PC 3 | Ruben y Marco (OpenCode) | M07 (Ruben), M08 (Marco) | landing, `Header` y `BottomNav` (solo web) |

**Dentro del PC 1:** Claude hace M01 (contrato), M10 y M11 (frontend del panel y `mobile/`) y revisa el backend; Codex hace M02, M03 y M04 en `mira-api` y revisa que M10 y M11 llamen bien a su API. Prompt de Codex: [[Codex - Reserva con carta]].

**Por qué así:** OpenCode va más lento, así que PC 3 lleva trabajo que **no depende de la API nueva ni toca los archivos de los otros PCs**, y puede ir a su ritmo. M01 (contrato) es lo único que bloquea: PC 2 puede empezar con un mock en cuanto esté publicado.

Si una sección de la landing necesita un endpoint nuevo, PC 3 se lo pide a PC 1 (no lo hace en el backend).

## Fuera de este proyecto
- **Empresas de reparto (JustEat, UberEats) en Firebase**: miniproyecto aparte, después de este.
- Pago online del pedido.
- Editar el pedido desde la web.
