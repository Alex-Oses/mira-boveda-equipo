---
estado: libre
responsable: Alex (PC 2)
pc: PC 2
depende_de: []
rama: feat/app-rediseno-web
---
# Próxima APK: perfil y animaciones

Origen: prueba de Ruben en el APK `android-v1.0.4` (2026-10-21). **No es urgente**:
entra en la próxima build.

## Fallo: los botones del perfil no hacen nada

En la app móvil, en **Perfil**, al pulsar **Puntos**, **Preferencias**, **Ajustes** y
**Actividad** no ocurre nada: no navegan. El resto de la pantalla sí responde.

Visto en `android-v1.0.4`, que sale del rediseño (PR #16, `feat/app-rediseno-web`),
así que lo más probable es que sea de esa pantalla nueva.

Terminada cuando: desde Perfil, los cuatro abren su pantalla en Android y «atrás»
sigue devolviendo al perfil.

## Sugerencias (opcionales, las decide Alex)

1. **Dirección de la animación.** Al cargar una página siempre entra de derecha a
   izquierda, tanto si avanzas como si vuelves. Quedaría mejor que el sentido siga
   al menú: si la sección está más a la izquierda, entra desde la izquierda (y
   «atrás» al contrario).
2. **Arrastrar en el menú inferior.** Que se pueda deslizar el dedo para pasar a la
   sección de al lado, no solo tocar el icono.
3. **Menú hamburguesa.** Quitarle la animación de transición al ir a otra página: se
   siente lenta y ajena a la navegación.

Nada de esto bloquea la build; si alguna no convence, se cierra como «no se hace».

## Notas

- No hay rutas `ajustes`, `preferencias` ni `actividad` en `mobile/src/app/` en
  `main`: si los botones apuntan ahí, el `router.push` no tiene destino.
