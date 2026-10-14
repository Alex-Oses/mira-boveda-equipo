# Cómo trabajamos

## Compartir la bóveda
- La bóveda vive en un **repo privado de GitHub** que todos clonamos y abrimos con Obsidian.
- Sincronizamos con el plugin **Obsidian Git** (pull al abrir, commit y push cada 10 min) o a mano: `git pull` al empezar y `git push` al terminar.
- No se suben `.obsidian/workspace*.json`, porque cada uno tiene su propia disposición de ventanas.

## Al empezar a trabajar
1. `git pull` de la bóveda.
2. Lee el [[Tablero]] y las sesiones nuevas desde la última vez que entraste.
3. Coge una tarea **libre**: en su nota pon `estado: en curso` y `responsable: <tu nombre>`, muévela en el Tablero y haz push **enseguida**, para que nadie la coja a la vez.

## Mientras trabajas
- Una rama de código por tarea: `feat/T03-registro-token`, `fix/...`.
- Si tomas una decisión que afecta a los demás (un endpoint, el nombre de una colección, una librería), apúntala en [[Decisiones]].

## Al terminar
1. Crea una nota en `Sesiones/` con la plantilla [[Sesión]] y llámala `AAAA-MM-DD Nombre`. Entre 10 y 20 líneas: qué hiciste, qué falta y qué problemas hubo.
2. Actualiza la nota de la tarea (estado y enlace a la rama o PR) y el [[Tablero]].
3. `git push`.

## Para evitar conflictos de Git
- Cada sesión es un archivo nuevo con tu nombre, así que nunca chocan.
- El Tablero se toca poco y en líneas cortas. Si hay conflicto, quedaos con las dos líneas.
- No copies documentos enteros: resúmenes y enlaces.
- Nada de secretos (claves, `.env`, tokens) en la bóveda.
