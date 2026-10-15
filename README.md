# Bóveda del equipo MIRA (Beca)

Memoria compartida del equipo y de nuestras IAs. Aquí está qué hay que hacer, quién lo hace y qué se ha hecho. Se abre con **Obsidian**.

Proyecto actual: **ofertas por geolocalización** (`Proyecto/Ofertas por geolocalización.md`).

## Instalación (una vez por persona)

1. **Acepta la invitación** al repo que te llega por correo o GitHub.
2. **Instala** [Git](https://git-scm.com/downloads) y [Obsidian](https://obsidian.md/download) si no los tienes.
3. **Clona** la bóveda donde quieras:
   ```
   git clone https://github.com/Alex-Oses/mira-boveda-equipo.git
   ```
4. **Ábrela en Obsidian:** "Abrir carpeta como bóveda" y elige la carpeta `mira-boveda-equipo`.
5. **Activa los plugins:** Obsidian pregunta si confías en el autor. Dile que sí (o ve a Ajustes → Plugins de la comunidad → "Activar plugins de la comunidad").
   - El plugin **Obsidian Git** ya viene instalado y configurado: hace pull al abrir y cada 10 min, y commit y push automáticos cada 10 min.
6. **Configura tu IA:** abre `IA - Instrucciones.md` y copia el bloque en tu `CLAUDE.md` o `AGENTS.md` (el global de tu usuario, no el del repo de código), con la ruta de tu clon.
7. **Apúntate** en `Equipo.md` (nombre, GitHub, PC e IA).

## Cada día

**Al empezar**
1. Abre Obsidian (hace pull solo) o ejecuta `git pull`.
2. Lee `Tablero.md`, las sesiones nuevas de `Sesiones/` y tu tarea en `Tareas/`.
3. Coge una tarea libre de tu PC: en la nota pon `estado: en curso` y `responsable: <tu nombre>`, muévela en el Tablero y haz push (en Obsidian, `Ctrl+P` → "Git: Commit and push").

**Mientras trabajas**
- Si eres del PC 1, publica el contrato de la API de cada fase en `Proyecto/Contratos API.md` antes de nada.
- Las decisiones que afectan a otros van a `Decisiones/Decisiones.md`.

**Al terminar**
1. Crea `Sesiones/AAAA-MM-DD TuNombre.md` con la plantilla `Plantillas/Sesión.md` (en Obsidian, `Ctrl+P` → "Insertar plantilla").
2. Actualiza el estado de tu tarea y el Tablero.
3. Haz push.

Más detalle en `Cómo trabajamos.md`.

## Reglas
- No cojas una tarea que esté en curso de otra persona.
- Nada de secretos (claves, `.env`, tokens, service accounts).
- Notas cortas, con enlaces `[[ ]]`. No copies documentos enteros.
- Si Git da un conflicto, quedaos con las dos versiones y avisad.

## Estructura
| Ruta | Qué hay |
|---|---|
| `00 Inicio.md` | Entrada a la bóveda |
| `Tablero.md` | Tareas por fase y estado |
| `Tareas/` | Una nota por tarea (G = geolocalización, T = notificaciones) |
| `Sesiones/` | Resumen de cada sesión de trabajo (una por persona y día) |
| `Decisiones/` | Lo acordado |
| `Proyecto/` | Resumen de MIRA, proyecto actual, contratos de la API y plan de notificaciones |
| `Plantillas/` | Sesión y Tarea |
