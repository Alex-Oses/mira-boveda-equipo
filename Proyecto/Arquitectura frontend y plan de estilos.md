# Arquitectura frontend y plan de estilos

> **Fecha:** 2026-10-19
> **Estado:** investigación cerrada. Plan aprobado en enfoque, ejecución en espera.
> **Ámbito:** web primero; app móvil después.
> **Autor:** OpenCode (DeepSeek V4.1 Flash Free), a partir de la investigación del repo.

---

## 0. Resumen ejecutivo

- El proyecto tiene **tres productos**: web (Vite + React 18), app móvil (Expo / React Native) y API (Express + Firebase Admin).
- La web usa **13 módulos externos**. No hay router, ni librería de i18n, ni kit de UI, ni framework CSS: todo está hecho a mano.
- Los estilos son **23 ficheros CSS globales**, **11.064 líneas**, **2.933 reglas** y **≈286 KB raw / ≈58 KB gzip** en el último build medido.
- Hay **varios sistemas de tokens en paralelo** (`--*`, `--ops-*`, `--op-*`, `--jam-*`, `--fx-*`, `--rsv-*`, `--dsc-*`, `--lm-*`), **94 selectores de tema oscuro**, **194 colores hex distintos fuera de tokens** y **456 estilos inline**.
- `App.css` es un monolito de **5.251 líneas** con 28 secciones.
- El orden de carga del CSS no es explícito: `tokens.css` acaba **al final** del bundle, después de `App.css` y de los CSS de componentes.
- **Decisión de Rubén:** web primero; plan por fases; enfoque tipo **Tailwind** con componentes ya preestilizados (`.btn`, `.card`, etc.); rendimiento y herramientas de calidad aparcados; esperar para ejecutar.
- **Recomendación técnica:** **Tailwind CSS v4 + una capa de componentes MIRA** (`.btn`, `.btn-primary`, `.card`, `.modal`, `.field`), con los colores y tipografías de MIRA en tokens. DaisyUI queda como acelerador opcional.
- **Acción ya realizada:** se eliminó el worktree viejo `wt-fixes` (limpio y ya fusionado en `main`).
- **Hallazgo importante:** los 30K cambios que aparecen en el editor **no son de MIRA**. Vienen de un repositorio git accidental en `C:\Users\ruben.munoz-blanco.e`. Ver anexo 12. **Resuelto el 20-10-2026** (ver 12.6).

---

## 1. Respuestas y decisiones de Rubén

| # | Pregunta | Respuesta | Implicación |
|---|---|---|---|
| 1 | ¿Web o también móvil? | **Principalmente web; después se adaptará a la app.** | El sistema de tokens y componentes se diseña para web. La app se adaptará después, reutilizando tokens. |
| 2 | ¿Quick wins o plan por fases? | **Tal y como se planteó: plan por fases.** | Se mantiene la hoja de ruta de 5-7 fases, sin reescritura total. |
| 3 | ¿Tokens + `@layer` + CSS Modules, o Tailwind? | **Explicar los conceptos, pero prefiere una librería tipo Tailwind, sencilla de adaptar al estilo MIRA y centralizada.** Ejemplo: una etiqueta de botón ya customizada con nuestros colores. | Se propone **Tailwind v4 + capa de componentes MIRA**. Los conceptos se explican en la sección 8. |
| 4 | ¿Code splitting / rendimiento? | **No sabe qué es; la página va bien de rendimiento.** | **Aparcado.** No se toca el rendimiento por ahora. |
| 5 | ¿Cuándo? | **Espera.** | No se ejecuta nada todavía. Este documento es el plan. |
| 6 | ¿Stylelint + Playwright? | **Apartado.** | **Fuera de alcance** por ahora. Se puede retomar más adelante. |

### Decisiones cerradas

- [x] El ámbito inicial es la **web**.
- [x] Se adopta un **plan por fases**, no una reescritura total.
- [x] El enfoque preferido es **Tailwind-like + componentes centralizados**.
- [x] Rendimiento y code splitting quedan **aparcados**.
- [x] La ejecución **espera** a que Rubén dé el visto bueno.
- [x] La calidad automatizada (stylelint, tests visuales) queda **aparcada**.

---

## 2. Stack y versiones

### 2.1 Entorno

| Herramienta | Versión instalada |
|---|---|
| Node.js | **v24.21.0** |
| npm | **11.19.0** |

El `package.json` web declara `node >=18`; el backend, `>=20`.

### 2.2 Frontend web

`Frontend/restaurante-mira-frontend/package.json`:

| Paquete | Rango declarado | Versión instalada |
|---|---|---|
| `react` | `^18.3.1` | **18.3.1** |
| `react-dom` | `^18.3.1` | **18.3.1** |
| `firebase` | `^12.18.0` | **12.18.0** |
| `zustand` | `^5.0.15` | **5.0.15** |
| `recharts` | `^3.10.1` | **3.10.1** |
| `leaflet` | `^1.9.4` | **1.9.4** |
| `react-leaflet` | `^4.2.1` | **4.2.1** |
| `qrcode` | `^1.5.4` | **1.5.4** |
| `jsqr` | `^1.4.0` | **1.4.0** |
| `vite` (dev) | `^5.4.0` | **5.4.21** |
| `@vitejs/plugin-react` (dev) | `^4.3.0` | **4.7.0** |

### 2.3 Módulos externos realmente importados en la web

Solo estos 13:

| Módulo | Uso |
|---|---|
| `firebase/app`, `firebase/auth` | Autenticación (email, Google) |
| `firebase/messaging` | Notificaciones push web |
| `leaflet`, `leaflet/dist/leaflet.css`, `react-leaflet` | Mapas |
| `recharts` | Gráficas del dashboard |
| `zustand`, `zustand/middleware` | Estado global |
| `qrcode` | Generar QR |
| `jsqr` | Escanear QR con cámara |
| `react`, `react-dom`, `react-dom/client` | Base |

**No hay** React Router, i18next, React Hook Form, TanStack Query, Tailwind, Bootstrap, Material UI, styled-components ni librería de fechas.

### 2.4 App móvil (`mobile/package.json`)

| Paquete | Declarado | Instalado |
|---|---|---|
| `expo` | `~57.0.25` | **57.0.25** |
| `react` | `19.2.3` | **19.2.3** |
| `react-native` | `0.86.3` | **0.86.3** |
| `expo-router` | `~57.0.23` | **57.0.23** |
| `firebase` | `^12.19.0` | **12.19.0** |
| `zustand` | `^5.0.15` | **5.0.15** |
| `react-native-reanimated` | `4.5.1` | **4.5.1** |
| `react-native-worklets` | `0.10.1` | **0.10.1** |
| `react-native-svg` | `15.15.4` | **15.15.4** |
| `react-native-gifted-charts` | `^1.4.78` | **1.4.78** |
| `typescript` | `~6.0.3` | **6.0.3** |
| `eslint` | `^9.0.0` | **9.39.5** |
| Fuentes Expo (Playfair / Jakarta) | `^0.4.2` | **0.4.2** |

> **Aviso:** el `node_modules` móvil está incompleto. Faltan `expo-dev-client`, `expo-device`, `expo-location`, `expo-task-manager`, `qrcode` y `react-native-webview`. Hay paquetes “extraneous” no declarados (`react-native-maps@1.27.2`, `@types/geojson@7946.0.16`). No afecta al plan web, pero conviene arreglarlo antes de tocar la app.

### 2.5 Backend `mira-api`

| Paquete | Declarado | Instalado |
|---|---|---|
| `express` | `^4.18.2` | **4.22.3** |
| `firebase-admin` | `^12.0.0` | **12.7.0** |
| `zod` | `^3.23.8` | **3.25.76** |
| `@google/generative-ai` | `^0.24.1` | **0.24.1** |
| `cors` | `^2.8.5` | **2.8.6** |
| `dotenv` | `^16.4.5` | **16.6.1** |
| `helmet` | `^7.1.0` | **7.2.0** |
| `node-cron` | `^3.0.3` | **3.0.3** |
| `pino` | `^9.4.0` | **9.14.0** |
| `pino-pretty` | `^11.2.2` | **11.3.0** |
| `supertest` (dev) | `^7.3.0` | **7.3.0** |

### 2.6 `yelp-connection`

Declara `@faker-js/faker ^10.6.0` y `firebase-admin ^13.10.0`, pero **no están instalados** (`UNMET DEPENDENCY`). Es un proyecto de scripts; no afecta a producción.

---

## 3. Estructura del proyecto

### 3.1 Vista general

```
Restaurante Mira/
├─ Frontend/
│  ├─ restaurante-mira-frontend/   ← repo git principal (web + móvil en /mobile)
│  ├─ wt-tanda2/                   ← worktree activo (fix/tanda2-ui, ya fusionado)
│  ├─ wt-G04/                      ← worktree de una feature antigua
│  ├─ wt-b34/                      ← carpeta huérfana (ya no es worktree)
│  ├─ wt-jam/                      ← carpeta huérfana (ya no es worktree)
│  └─ wt-ajustes/                  ← carpeta huérfana (ya no es worktree)
├─ restuarante-mira-backend/
│  ├─ mira-api/                    ← API Express
│  └─ yelp-connection/             ← scripts de datos
└─ obsidian/mira-boveda-equipo/    ← bóveda del equipo
```

### 3.2 Frontend web por capas

```
src/
├─ main.jsx                 ← entrada; monta <App />, index.css, Leaflet
├─ App.jsx                  ← orquestador: auth, tema, idioma, rutas hash, puntos, racha, IA
├─ App.css                  ← 5.251 líneas; 28 secciones
├─ index.css                ← reset + @import tokens.css
│
├─ components/              ← 97 JSX + 4 CSS
│  ├─ (42 en la raíz)       ← Header, Hero, SearchBar, cards, Login, Cuenta, Reservas, JAM...
│  ├─ dashboard/            ← 12 ficheros; panel del restaurante (.op-*)
│  ├─ ops/                  ← 20 ficheros; panel de administración (.ops-*)
│  ├─ mira/                 ← 10 ficheros; asistente IA
│  ├─ points/               ← 6 ficheros; MIRA Points
│  ├─ promotions/, tickets/, fiscal/
│
├─ controllers/             ← 4 hooks de lógica de pantalla
├─ models/                  ← 9 módulos de reglas puras
├─ services/                ← 41 módulos (API, auth, puntos, auditoría, IA...)
├─ stores/                  ← 4 stores de Zustand
├─ hooks/                   ← 4 hooks de navegador
├─ i18n/                    ← es.js, ca.js, en.js (1.998 líneas cada uno) + index.jsx
├─ pages/                   ← 4 envoltorios finos
├─ styles/                  ← 17 CSS por dominio + tokens.css
├─ data/                    ← mocks
└─ utils/
```

### 3.3 Routing

No hay React Router. El routing es **hash manual** en `App.jsx`, con **25 rutas**:

```
#/login, #/registro, #/cuenta, #/reservas, #/admin, #/dashboard,
#/jam, #/puntos, #/puntos/historial, #/invitar, #/ticket/:id,
#/llegada/:id, #/mapa, #/descubrir, #/favoritos, #/mensajes,
#/negocio, #/contacto, #/privacidad, #/app, #/demo-paneles,
#/test-reservas, #/recuperar, #/restablecer, #/
```

### 3.4 Componentes y paneles

| Grupo | Ficheros | Namespace CSS | Función |
|---|---:|---|---|
| `components/` raíz | 42 | `.site-*`, `.card`, `.modal`, `.btn-*`, `.streak-*`, `.wheel-*` | Landing, buscador, fichas, auth, cuenta, reservas, JAM, puntos |
| `components/ops/` | 20 | `.ops-*` | Panel de administración (`#/admin`) |
| `components/dashboard/` | 12 | `.op-*` | Panel del restaurante (`#/dashboard`) |
| `components/mira/` | 10 | `.mira-*` | Asistente IA |
| `components/points/` | 6 | `.points-*`, `.streak-*` | MIRA Points |
| `components/fiscal/` | 4 | `.fx-*` | Documento fiscal |
| `components/promotions/` | 2 | `.promo-*` | Ofertas |
| `components/tickets/` | 1 | `.tkd-*` | Detalle de ticket |

### 3.5 App móvil

- Expo Router: **25 rutas** en `mobile/src/app/`.
- Componentes que replican la web: `components/ops`, `components/dashboard`, `points`, `shell`, `ui`.
- **Aquí los estilos sí están centralizados:** `mobile/src/theme/tokens.js` es espejo de `tokens.css`, `theme/ui.js` replica `.btn-cta/.btn-secundario/.btn-texto`, y `ThemeContext` replica `data-theme` + `localStorage 'mira:tema'`.
- Existen dos espejos de tokens de panel: `mobile/src/components/ops/OpsTokens.js` y `mobile/src/components/dashboard/OpTokens.js`.

### 3.6 Backend

Monolito modular Express:

```
src/
├─ app.js, server.js
├─ config/, middlewares/
└─ modules/
   ├─ admin, ai (context/guardrails/handlers/provider)
   ├─ audit, auditoria, contactos, cron
   ├─ dashboard, espera, interactions, invitations, jams
   ├─ mensajes, metricas, negocios, points, promotions
   ├─ reservations, restaurants, reviews, tickets, users
```

Cada módulo tiene su `routes.js`; hay **18 routers** colgando de `/v1`.

### 3.7 Tests y documentación

- **43 ficheros de test** en `test/` con `node --test` (sin Jest/Vitest). La última línea base registrada: **309/309**.
- **70 ficheros** en `docs/bugs/` (`BUGS.md`, plantilla y BUG-001…BUG-137).
- Bóveda Obsidian con `Proyecto/`, `Sesiones/`, `Tareas/`, `Decisiones/`, `Plantillas/`.

---

## 4. Worktrees (`wt-*`)

### 4.1 Qué son

Un **git worktree** es un directorio de trabajo adicional asociado al mismo repositorio. Comparten la base de objetos `.git`, pero cada uno tiene su propio checkout y su propia rama. Sirven para trabajar en paralelo sin hacer `stash` ni cambiar de rama.

### 4.2 Estado actual (2026-10-19)

| Carpeta | Rama | HEAD | Estado |
|---|---|---|---|
| `restaurante-mira-frontend` | `main` | `8aac33e` | Limpio; solo 4 untracked de tooling (`.opencode/`, `AGENTS.md.mira-local`, `docs/bugs/BUG-.md`, `opencode.json`) |
| `wt-tanda2` | `fix/tanda2-ui` | `78c6227` | **Limpio.** La tanda 2 (BUG-146…152) está commiteada y fusionada en `main` (`8aac33e`) |
| `wt-G04` | `feat/G04-editor-ofertas` | `c72b2de` | Limpio; **no fusionado** en `main` |
| `wt-fixes` | `fix/errores-ui-web` | `26bb0c0` | **Eliminado** (limpio y ya fusionado en `main`). La rama sigue existiendo |
| `wt-b34` | — | — | **Carpeta huérfana.** Ya no tiene `.git` ni está en `git worktree list` |
| `wt-jam` | — | — | **Carpeta huérfana.** Ya no tiene `.git` ni está en `git worktree list` |
| `wt-ajustes` | — | — | **Carpeta huérfana.** Ya no tiene `.git` ni está en `git worktree list` |

### 4.3 Hallazgo: `wt-b34`, `wt-jam` y `wt-ajustes` ya no son worktrees

Las tres carpetas **no tienen fichero `.git`**. Al ejecutar `git` dentro de ellas, git sube hasta `C:\Users\ruben.munoz-blanco.e` y opera sobre el **repo accidental del home** (ver anexo 12). Por eso mostraban 140/153 cambios que no tenían nada que ver con MIRA.

No aparecen en `git worktree list` y `git worktree prune --dry-run` no las lista. Son **copias huérfanas** de la tanda anterior. Según el trabajo previo, sus cambios ya se integraron en `wt-tanda2` (commit `78c6227`), pero conviene revisar que no haya nada único antes de borrarlas.

### 4.4 `node_modules` compartido

`wt-tanda2` y `wt-G04` tienen `node_modules` como **junction** al `node_modules` del repo principal. Ahorra disco, pero comparte también la caché de Vite (`node_modules/.vite`). Eso puede provocar comportamientos raros entre worktrees (el dev server que no renderiza en un worktree mientras en otro sí). Si se vuelve a trabajar en paralelo, considerar:

- `cacheDir` distinto por worktree en `vite.config.js`, o
- `npm install` separado en cada worktree, o
- limpiar `node_modules/.vite` al cambiar de worktree.

### 4.5 Limpieza pendiente

- [x] Eliminar `wt-fixes`.
- [ ] Revisar y eliminar `wt-b34`, `wt-jam`, `wt-ajustes` (no son worktrees).
- [ ] Decidir qué hacer con `wt-G04` (rama no fusionada).
- [ ] Opcional: borrar la rama `fix/errores-ui-web` (ya fusionada) con `git branch -d fix/errores-ui-web`.
- [ ] Opcional: `git worktree prune` para limpiar metadatos.

---

## 5. Cómo funcionan los estilos actualmente

### 5.1 Carga de CSS

```
main.jsx
├─ App.jsx
│  ├─ import './App.css'          (5.251 líneas)
│  ├─ import './styles/motion.css'
│  ├─ import './styles/paneles.css'
│  ├─ import './styles/cuenta.css'
│  └─ componentes que importan su CSS:
│     ├─ styles/ops.css           (components/ops/OpsPanel.jsx)
│     ├─ components/dashboard/dashboard.css
│     ├─ components/mira/mira.css
│     ├─ components/ops/opsAuditoria.css
│     ├─ components/fiscal/fiscal.css
│     └─ styles/{detalle, carta, reservas, mapa, jam, descubrir, descargar, llegada, avisosOfertas, logo, incidencias, resolverIncidencia}.css
├─ import './index.css'           (reset + @import tokens.css)
└─ import 'leaflet/dist/leaflet.css'
```

No hay CSS Modules, ni SCSS, ni Tailwind, ni CSS-in-JS. Todo es CSS global con prefijos como namespacing.

### 5.2 Orden real en el bundle

- En el CSS construido, **`tokens.css` aparece casi al final** (offset ≈269.738 de 286 KB), después de `App.css`, `ops.css`, `dashboard.css`, `mira.css`, etc.
- La razón: en `main.jsx`, `import App from './App.jsx'` va **antes** de `import './index.css'`. Los imports de CSS de `App.jsx` y de todos sus componentes se evalúan primero.
- Las variables CSS siguen funcionando (se resuelven en tiempo de cálculo), pero los resets de `index.css` (`body`, `h1`, scrollbar, `:focus-visible`) quedan al final del cascade. El orden depende del grafo de imports, no de una decisión explícita.
- Además, `dashboard.css` tiene un `@import` de Google Fonts (Plus Jakarta + **Material Symbols**). Vite lo hoistea al principio del bundle, así que **todo el mundo descarga Material Symbols**, aunque nunca entre al dashboard.

### 5.3 Tokens y temas

- `styles/tokens.css` (121 líneas) define la paleta Epicure: `--primary`, `--fondo`, `--tinta`, `--verde`, `--oro`, sombras, radios, tipografías, layout.
- Tema oscuro: `[data-theme='dark']` sobre `<html>`, activado desde `App.jsx` y persistido en `localStorage['mira:tema']`.
- Pero **no es la única fuente de tokens**. Hay varios sistemas en paralelo:

| Sistema | Dónde | Tokens propios |
|---|---|---|
| Web/landing | `tokens.css` | `--primary`, `--fondo`, `--tinta`… |
| Admin ops | `styles/ops.css` | `--ops-*` (44 definiciones) |
| Dashboard restaurante | `components/dashboard/dashboard.css` | `--op-*` (129 definiciones) |
| JAM | `styles/jam.css` | `--jam-*` |
| Fiscal | `components/fiscal/fiscal.css` | `--fx-*` |
| Carta | `styles/carta.css` | `--carta-*`, `--oro` |
| Reservas | `styles/reservas.css` | `--rsv-*` |
| Descubrir | `styles/descubrir.css` | `--dsc-*` |
| Logo | `styles/logo.css` | `--lm-*` |

Y las curvas de animación están duplicadas con nombres distintos: `--ease-suave`, `--ease-expo`, `--pn-ease-out`, `--pn-ease-in-out`, `--jam-ease`, `--rsv-ease`, `--mapa-ease`, `--ease-out`.

**Deriva detectada:** `--ops-*` en claro usa superficies azuladas (`#f9f9ff`, `#f0f3ff`, `#e7eeff`), mientras que `--op-*` ya se corrigió a verdes cálidos (`#f9fbf8`, `#f1f4f1`). Son el mismo concepto con valores distintos.

### 5.4 Métricas actuales del CSS

Medición sobre `main` (`8aac33e`), tras la tanda 2:

| Métrica | Valor |
|---|---:|
| Ficheros CSS | **23** |
| Líneas totales | **11.064** |
| Reglas | **2.933** |
| `@media` | **102** |
| Breakpoints distintos | **19** (360, 380, 400, 480, 500, 520, 560, 600, 639, 640, 719, 720, 767, 768, 860, 1024, 1200, 1280) |
| Selectores `[data-theme='dark']` | **94** |
| `!important` | **27** (12 solo en `dashboard.css`) |
| Colores hex en CSS (fuera de tokens) | **647 usos / 194 distintos** |
| `rgba()` | **368** |
| Usos de `var(--…)` | **1.839** |
| Clases distintas | **1.464** |
| Clases definidas en 2+ ficheros | **84** |
| `@keyframes` distintos | **64** |
| Valores de `z-index` | **31** (de −2 a 2000) |
| `position: fixed` | **23** |
| Estilos inline en JSX | **456** (`Dashboard.jsx` solo: **193**) |
| CSS del build | **≈286 KB raw / ≈58 KB gzip** |
| JS del build | **≈2.036 KB raw / ≈586 KB gzip** (un solo chunk) |

`App.css` concentra **5.251 líneas y 28 secciones** (header, hero, buscador, cards, modales, auth, cuenta, reservas, puntos, dashboard, racha, ruleta…).

### 5.5 Duplicaciones concretas

- `.btn-cta` aparece en `App.css`, `mira.css`, `jam.css`, `motion.css` y `detalle.css`.
- `.btn-secundario` y `.btn-texto`, en 4-5 ficheros.
- `.modal`, `.modal-fondo`, `.modal-cerrar`, en `App.css`, `detalle.css` y `motion.css`.
- `.ops-card`, en `ops.css`, `paneles.css`, `opsAuditoria.css` e `incidencias.css`.
- `.ops-kpi`, `.ops-content`, `.dash-chart`, en varios.
- `paneles.css` es un fichero “parche” que aplica animaciones y overrides **a la vez** sobre `.ops-*` y `.op-*`, acoplando los dos paneles.

### 5.6 Animación y el bug reciente

`paneles.css` usaba `animation: pn-aparecer … both` en `.ops-content > .ops-card`. Con `fill-mode: both`, al terminar queda un `transform: matrix(1,0,0,1,0,0)` que **no es `none`**, y eso convierte la tarjeta en *containing block* de los `position: fixed`. Por eso los modales de admin se centraban en la tarjeta (la página) y no en la pantalla. Es el mismo patrón que BUG-130 con `.ruta`.

Esto demuestra el problema de fondo: **una animación definida en un fichero global cambia el comportamiento de posicionamiento de componentes de otro fichero**.

---

## 6. Diagnóstico: por qué cuesta mantenerlo

1. **No hay una única fuente de verdad visual.** Hay 4-9 sistemas de tokens conviviendo.
2. **El cascade depende del orden de imports**, y el orden actual pone los tokens al final. Cualquier refactor puede cambiar el resultado sin tocar una regla.
3. **Todo el CSS es global.** Los prefijos evitan colisiones, pero no las detectan; 84 clases ya viven en 2+ ficheros.
4. **El tema oscuro se resuelve componente a componente** (94 selectores) en lugar de remapear tokens.
5. **`App.css` es un monolito** de 5.251 líneas con 28 dominios distintos.
6. **Todo el CSS se carga siempre.** No hay code splitting por ruta: landing, admin, dashboard, JAM y MIRA se importan estáticamente en `App.jsx`.
7. **Hay estilos inline** (456) que no se pueden tematizar ni adaptar por media query.
8. **No hay guardarraíles**: ni stylelint, ni Prettier, ni tests visuales, ni convención de breakpoints, ni escala de z-index.
9. **Duplicación de fuentes**: Google Fonts en `index.html` + `@import` en `dashboard.css`.
10. **Los worktrees comparten caché de Vite** y pueden provocar comportamientos raros entre entornos.

---

## 7. Glosario: tokens, `@layer` y CSS Modules

Rubén pidió entender estos conceptos antes de decidir. Se explican en simple, y luego se comparan con Tailwind.

### 7.1 Tokens (variables de diseño)

Un token es un **valor con nombre**. En vez de repetir `#16382C` 50 veces, se define una vez:

```css
:root {
  --mira-verde: #16382c;
  --mira-dorado: #fed65b;
  --radio: 12px;
}
```

Y se usa:

```css
.btn-primary {
  background: var(--mira-verde);
  border-radius: var(--radio);
}
```

Ventajas: cambiar un color en un sitio cambia toda la app; el tema oscuro se hace remapeando tokens; hay consistencia.

En MIRA ya existen tokens, pero están **fragmentados** en varios sistemas (`--ops-*`, `--op-*`, `--jam-*`…).

### 7.2 `@layer` (capas de cascade)

CSS decide qué regla gana por **especificidad** y **orden**. `@layer` permite declarar el orden de antemano:

```css
@layer reset, tokens, base, layout, components, utilities, overrides;
```

Lo que esté en `components` gana a lo que esté en `base`, sin importar el orden de los ficheros. Y `utilities` gana a `components`, etc.

Ventaja: dejas de depender del orden de imports y de los `!important`. Es una herramienta de **orden**, no de componentes.

### 7.3 CSS Modules

Un CSS Module es un fichero `Boton.module.css` cuyos nombres de clase se **renombran automáticamente** para que no colisionen:

```css
/* Boton.module.css */
.primary { background: var(--mira-verde); }
```

```jsx
import styles from './Boton.module.css';
<button className={styles.primary}>Reservar</button>
```

Ventaja: scope real por componente; no hay que inventar prefijos. Desventaja: hay que cambiar los `className` y no da componentes ya hechos; sigue siendo CSS manual.

### 7.4 Por qué no convencen para este proyecto

- No dan un **botón ya preestilizado**; hay que escribirlo.
- No reducen por sí solos la duplicación; solo la organizan.
- CSS Modules exige migrar `className` de cada componente.
- `@layer` es una gran base, pero no es una librería de componentes.

**Conclusión:** son fundamentos útiles, y Tailwind los usa por debajo. Pero si el objetivo es “una etiqueta `<button>` que ya salga con nuestros colores”, hace falta una **capa de componentes** (propia o DaisyUI), no solo tokens/layers/modules.

---

## 8. Propuesta: Tailwind adaptado a MIRA

### 8.1 Qué es Tailwind

Tailwind es un framework **utility-first**: en vez de escribir CSS nuevo para cada pantalla, usas clases pequeñas ya generadas:

```jsx
<div className="flex items-center gap-4 rounded-xl bg-white p-6 shadow">
  ...
</div>
```

No es un kit de componentes: no trae un `<Button>` ni un `<Card>`. Trae **utilidades** y un **tema** (colores, fuentes, espacios) que se configura una vez.

Puntos clave:

- **Centraliza** colores, tipografías, radios, sombras y spacing en un tema.
- **Genera solo las clases usadas** (no mete 300 KB de CSS).
- **Se adapta** a la identidad de MIRA: los colores y fuentes se definen con los valores de la marca.
- **Permite componentes**: con `@apply` (o CSS normal) se crean `.btn`, `.btn-primary`, `.card` una sola vez.
- **Convive** con el CSS actual si se configura con capas y prefijo.

### 8.2 Qué es DaisyUI

DaisyUI es una **librería de componentes encima de Tailwind**. Sí trae las etiquetas “súper sencillas”:

```jsx
<button className="btn btn-primary">Reservar</button>
<button className="btn btn-secondary">Ver carta</button>
```

Y temas personalizables:

```css
@plugin "daisyui/theme" {
  name: "mira";
  default: true;
  --color-primary: #16382c;
  --color-secondary: #735c00;
  --color-accent: #fed65b;
  --radius-box: 1rem;
}
```

Ventaja: rapidez; el “botón ya customizado” existe desde el día 1.
Desventaja: su look por defecto es muy reconocible; adaptarlo a la estética editorial de MIRA requiere trabajo. Añade otra capa que mantener.

### 8.3 Arquitectura recomendada

**Tailwind v4 + capa de componentes MIRA.** Sin DaisyUI al principio (se puede probar después).

```
src/
├─ styles/
│  ├─ tailwind.css        ← @import "tailwindcss"; @theme; @custom-variant dark
│  ├─ componentes.css     ← .btn, .btn-primary, .card, .modal, .field, .badge
│  └─ legacy/             ← CSS actual, durante la migración
├─ ui/                    ← componentes React que usan esas clases
│  ├─ Button.jsx
│  ├─ Card.jsx
│  ├─ Modal.jsx
│  └─ Field.jsx
```

Reglas de convivencia:

- **Preflight desactivado** al principio (o controlado) para no romper el reset actual.
- **Prefijo `tw-`** durante la migración, para no colisionar con `.btn` / `.card` existentes.
- **Capas**: `@layer theme, base, components, utilities, legacy;` y el CSS viejo en `legacy` hasta que se migre cada pantalla.
- **Un solo fichero de tema** con los valores de MIRA.
- **Componentes solo para lo repetido** (botón, card, modal, campo, badge); el resto, utilidades de layout.

### 8.4 Ejemplo de tema

```css
/* src/styles/tailwind.css */
@import "tailwindcss";

@theme {
  --color-mira-primary: #16382c;
  --color-mira-primary-dark: #002218;
  --color-mira-gold: #fed65b;
  --color-mira-bg: #f7faf6;
  --color-mira-surface: #ffffff;
  --color-mira-text: #181c1a;
  --color-mira-muted: #414844;
  --color-mira-border: #c1c8c3;
  --color-mira-danger: #67081d;

  --font-sans: "Plus Jakarta Sans", system-ui, sans-serif;
  --font-display: "Playfair Display", Georgia, serif;

  --radius-mira: 12px;
  --radius-mira-lg: 24px;
}

@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
```

> El ejemplo es orientativo. Tailwind v4 usa configuración CSS-first; los nombres exactos de las variables se pueden ajustar.

### 8.5 Ejemplo de botón “súper sencillo”

```css
/* src/styles/componentes.css */
@layer components {
  .btn {
    @apply inline-flex items-center justify-center gap-2
           rounded-xl px-5 py-3 font-semibold
           transition-colors duration-200
           disabled:cursor-not-allowed disabled:opacity-50;
  }

  .btn-primary {
    @apply bg-mira-primary text-white hover:bg-mira-primary-dark;
  }

  .btn-secondary {
    @apply border border-mira-primary text-mira-primary
           hover:bg-mira-primary/10;
  }

  .btn-ghost {
    @apply text-mira-primary hover:bg-mira-primary/10;
  }

  .card {
    @apply rounded-2xl border border-mira-border bg-mira-surface shadow-sm;
  }

  .modal {
    @apply fixed inset-0 z-50 grid place-items-center bg-black/50 p-4;
  }

  .modal-panel {
    @apply max-h-[calc(100dvh-2rem)] w-full max-w-lg overflow-y-auto
           rounded-2xl bg-mira-surface p-6 shadow-xl;
  }
}
```

Uso en JSX:

```jsx
<button className="btn btn-primary">Reservar</button>
<button className="btn btn-secondary">Ver carta</button>

<div className="card p-6">
  <h2 className="font-display text-2xl text-mira-text">MIRA</h2>
  <p className="text-mira-muted">...</p>
</div>
```

Esto es exactamente lo que pedía Rubén: **una etiqueta simple, ya con los colores de MIRA, definida una sola vez**.

### 8.6 Cómo se mantiene la identidad

- Los **colores, fuentes y radios** salen de tokens MIRA, no de Tailwind por defecto.
- La **capa de componentes** define el look editorial (botones, cards, modales).
- Las **utilidades** solo se usan para layout (flex, grid, spacing, responsive).
- Las **pieles** se hacen remapeando tokens por ámbito:

```css
[data-skin="ops"] {
  --color-mira-primary: #005134;
  --radius-mira: 8px;
}

[data-skin="jam"] {
  --color-mira-accent: #fed65b;
}
```

Así ops sigue sobrio, JAM festivo y la landing editorial, pero todos comparten el mismo vocabulario.

### 8.7 Comparativa de opciones

| Opción | Ventaja | Riesgo | Veredicto |
|---|---|---|---|
| **Tailwind v4 + capa MIRA** | Centraliza tema y componentes; migración incremental; control total del look | Hay que escribir la capa de componentes; convivencia con CSS viejo | ✅ **Recomendado** |
| **Tailwind + DaisyUI** | `btn`, `card`, `modal` desde el día 1; temas | Look por defecto reconocible; otra dependencia; adaptar el estilo cuesta | ⚠️ Buen piloto, no base |
| **Tokens + `@layer` + CSS Modules** | Sin dependencias; control total | Más trabajo manual; no da componentes hechos | ⚠️ Alternativa si no se quiere Tailwind |
| **Bootstrap / Bulma** | Componentes clásicos, fáciles | Estética difícil de llevar a MIRA; CSS extra | ❌ No |
| **CSS-in-JS (Emotion/styled)** | Scoping y temas | Runtime, bundle, complejidad | ❌ No |
| **Reescritura total** | Resultado limpio | Meses, riesgo alto, bugs visuales | ❌ No |

### 8.8 Plan de adopción (cuando Rubén diga “ya”)

1. **Piloto aislado** en una pantalla autocontenida (propuesta: **JAM**, que ya tiene su propio CSS y una piel distinta).
2. Definir el tema MIRA y los componentes `.btn`, `.card`, `.modal`, `.field`.
3. Comparar capturas antes/después en claro/oscuro y 390/768/1440.
4. Si el resultado convence, migrar **landing** (header, hero, buscador, cards, footer).
5. Después **auth, cuenta, reservas**.
6. Después **ops, dashboard, MIRA**.
7. Al final, retirar el CSS viejo y los alias `--ops-*` / `--op-*`.

**No** migrar todo a la vez. Cada pantalla migrada debe quedar visualmente igual (o mejor) y con menos CSS.

---

## 9. Objetivos

### 9.1 Objetivo general

Unificar la capa visual de la web en un sistema **centralizado, reutilizable y tematizable**, sin perder la identidad de cada zona (landing editorial, ops sobrio, dashboard, JAM festivo), y dejarlo preparado para reutilizarlo en la app móvil.

### 9.2 Corto plazo (cuando se apruebe; 1-2 semanas)

- [ ] Aprobar el enfoque y la pantalla piloto.
- [ ] Crear rama `feat/design-system-tailwind` (o similar).
- [ ] Instalar Tailwind CSS v4 + `@tailwindcss/vite`.
- [ ] Configurar Preflight desactivado/controlado y prefijo `tw-` durante la convivencia.
- [ ] Crear `src/styles/tailwind.css` con los tokens MIRA (colores, fuentes, radios, sombras).
- [ ] Crear `src/styles/componentes.css` con `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.card`, `.modal`, `.field`, `.badge`.
- [ ] Pilotar en **JAM** (o en el header/hero de la landing).
- [ ] Capturas antes/después en claro/oscuro y 390/768/1440.
- [ ] Documentar convenciones en esta bóveda.

### 9.3 Medio plazo (1-2 meses)

- [ ] Migrar la landing: header, hero, buscador, cards, footer.
- [ ] Migrar auth, cuenta y reservas.
- [ ] Unificar el tema oscuro por tokens (objetivo: bajar de 94 selectores a <15).
- [ ] Unificar las paletas de ops y dashboard (hoy una es azulada y la otra verde cálida).
- [ ] Partir `App.css` por dominio (≤600 líneas por fichero).
- [ ] Estandarizar breakpoints (640/768/1024/1280) y escala de z-index.
- [ ] Reducir estilos inline (456 → <100), empezando por `Dashboard.jsx` (193).

### 9.4 Largo plazo

- [ ] Migrar ops, dashboard, JAM y MIRA.
- [ ] Adaptar la app móvil al mismo sistema de tokens.
- [ ] Documentar el design system completo (tokens + componentes + ejemplos).
- [ ] Opcional (aparcado): tests visuales, stylelint, code splitting.

### 9.5 Métricas de éxito

| Métrica | Hoy | Objetivo |
|---|---:|---:|
| Líneas en `App.css` | 5.251 | ≤ 600 por fichero |
| Colores hex fuera de tokens | 194 distintos | < 30 |
| Selectores dark | 94 | < 15 |
| `!important` | 27 | 0-5 |
| Breakpoints | 19 | 4-6 |
| z-index distintos | 31 | 6-8 |
| Clases en 2+ ficheros | 84 | < 10 |
| CSS inicial (gzip) | ≈58 KB | 35-40 KB (si se retira CSS viejo) |
| Estilos inline | 456 | < 100 |

### 9.6 Fuera de alcance por ahora

- Code splitting y optimización de rendimiento (decisión de Rubén: la página va bien).
- Stylelint, Prettier y tests visuales (aparcado).
- Reescritura total del frontend.
- Migración de la app móvil antes de cerrar la web.

---

## 10. Riesgos y preguntas abiertas

### Riesgos

1. **Convivencia Tailwind + CSS viejo**: si no se usan capas y prefijo, las clases pueden colisionar. Mitigación: `@layer legacy`, prefijo `tw-` y migración por pantalla.
2. **Preflight**: el reset de Tailwind puede cambiar tipografías y márgenes. Mitigación: desactivarlo al principio.
3. **Pérdida de identidad**: si se usan utilidades sin capa de componentes, la UI se vuelve genérica. Mitigación: `.btn`, `.card`, `.modal` con tokens MIRA.
4. **Markup más ruidoso**: muchas clases en el JSX. Mitigación: componentes React (`<Button>`, `<Card>`) que encapsulan las clases.
5. **Dos sistemas durante la migración**: hay que resistir la tentación de mezclar en el mismo componente. Mitigación: regla “pantalla migrada = solo Tailwind; pantalla no migrada = solo CSS viejo”.
6. **El repo del home** (anexo 12) puede seguir generando ruido en el editor y confundir a las herramientas.

### Preguntas para Rubén

1. ¿Pantalla piloto: **JAM** (autocontenida, piel distinta) o **landing** (más visible)?
2. ¿**Tailwind + capa MIRA** o quieres probar primero **DaisyUI** para ver el “botón ya hecho”?
3. ¿Prefijo `tw-` durante la migración, o sin prefijo desde el principio?
4. ¿Preflight desactivado al principio, o dejar que Tailwind resetee y ajustar?
5. ¿Cuándo empezamos? (Ahora está en “espera”.)
6. ¿Quieres que elimine el `.git` accidental del home? (Ver anexo 12; es destructivo solo para el historial, no para los ficheros).

---

## 11. Anexo: inventario CSS

| Fichero | Líneas |
|---|---:|
| `src/App.css` | 5.251 |
| `src/styles/detalle.css` | 717 |
| `src/components/mira/mira.css` | 644 |
| `src/styles/ops.css` | 531 |
| `src/components/dashboard/dashboard.css` | 523 |
| `src/styles/carta.css` | 494 |
| `src/styles/cuenta.css` | 372 |
| `src/styles/descubrir.css` | 360 |
| `src/styles/descargar.css` | 280 |
| `src/styles/reservas.css` | 261 |
| `src/styles/motion.css` | 256 |
| `src/styles/jam.css` | 240 |
| `src/styles/mapa.css` | 177 |
| `src/styles/paneles.css` | 166 |
| `src/components/fiscal/fiscal.css` | 146 |
| `src/components/ops/opsAuditoria.css` | 126 |
| `src/styles/tokens.css` | 121 |
| `src/index.css` | 112 |
| `src/styles/logo.css` | 98 |
| `src/styles/llegada.css` | 98 |
| `src/styles/avisosOfertas.css` | 73 |
| `src/styles/incidencias.css` | 10 |
| `src/styles/resolverIncidencia.css` | 8 |
| **Total** | **11.064** |

---

## 12. Anexo: por qué aparecen 30K cambios en git

### 12.1 Causa

El directorio de usuario `C:\Users\ruben.munoz-blanco.e` **es un repositorio git**:

- `.git` propio en `C:\Users\ruben.munoz-blanco.e\.git`.
- Rama actual: `feat/parkings-simple`.
- Un único commit: `9737e82 acabada`.
- **37 ficheros trackeados**, casi todos de `Documents/Servicios de Voz e Interfaces` y `Documents/Clase/acceso_a_datos_RA1`.
- **No tiene `.gitignore`.**

Ese repo ve **todo** lo que hay en la carpeta de usuario: `AppData/`, `.ssh/`, `.cache/`, `.expo/`, `node_modules/`, `Documents/`, etc. `git status` normal colapsa los directorios no trackeados en una línea (140 entradas), pero VS Code y otras UIs los expanden y cuentan cada fichero. La medición con `git status --porcelain -uall` da **381.145 entradas** en el home. De ahí los “30K” (o más) que aparecen en el editor.

**No es un problema del repo de MIRA.** El repo de MIRA (`Frontend/restaurante-mira-frontend`) está limpio salvo 4 untracked de tooling.

### 12.2 Por qué parece un `git init` accidental

- El commit es **initial** (`commit (initial): acabada`), sin remoto.
- La rama `feat/parkings-simple` no tiene relación con el proyecto de Servicios de Voz; parece un nombre heredado de otro contexto.
- La carpeta `Documents/Servicios de Voz e Interfaces` **ya tiene su propio `.git`**. Es decir, el repo del home es redundante para ese proyecto.
- No hay `.gitignore`, así que el repo “ve” toda la carpeta de usuario.

### 12.3 Soluciones

**Opción A — Si el repo del home es accidental (recomendada):**

1. Hacer copia de seguridad del historial (opcional): comprimir `C:\Users\ruben.munoz-blanco.e\.git`.
2. Eliminar solo la carpeta `.git` del home:
   ```powershell
   Remove-Item "C:\Users\ruben.munoz-blanco.e\.git" -Recurse -Force
   ```
   Esto **no borra ningún fichero**; solo el historial de ese repo accidental. El proyecto `Documents/Servicios de Voz e Interfaces` conserva su propio `.git`.
3. Abrir VS Code en `C:\Users\ruben.munoz-blanco.e\Documents\Restaurante Mira` (o en un `.code-workspace`), no en la carpeta de usuario.

**Opción B — Si quieres conservar el historial:**

1. Mover el `.git` del home a `Documents/Servicios de Voz e Interfaces` (o inicializar un repo allí y commitear).
2. Eliminar el `.git` del home.
3. Añadir un `.gitignore` al proyecto.

**Opción C — Si decides mantener el repo del home:**

Añadir un `.gitignore` que ignore todo menos lo que quieras trackear:

```gitignore
# Ignorar todo
*

# Pero trackear estos ficheros/directorios
!Documents/Servicios de Voz e Interfaces/
!Documents/Clase/acceso_a_datos_RA1
!.gitignore
```

Aun así, no es recomendable tener un repo en la raíz del usuario.

> **Importante:** **no ejecutes `git clean -fdx` en el home**. Borraría 381K ficheros no trackeados, incluidos documentos personales. La solución pasa por quitar el `.git` accidental, no por limpiar el working tree.

### 12.4 Worktrees huérfanos

Las carpetas `wt-b34`, `wt-jam` y `wt-ajustes` **no tienen `.git`**. Al ejecutar `git` dentro de ellas, git sube al repo del home y muestra sus cambios. Por eso parecían tener 140/153 cambios de `Documents/...`.

No están en `git worktree list`. Son copias huérfanas. Antes de borrarlas, confirma que no tengan nada único; según el trabajo previo, sus cambios ya están en `wt-tanda2` (commit `78c6227`).

### 12.5 Estado actual de worktrees

```
C:/Users/.../Frontend/restaurante-mira-frontend   8aac33e [main]
C:/Users/.../Frontend/wt-G04                      c72b2de [feat/G04-editor-ofertas]
C:/Users/.../Frontend/wt-tanda2                   78c6227 [fix/tanda2-ui]
```

`wt-fixes` fue eliminado (limpio y ya fusionado). La rama `fix/errores-ui-web` sigue existiendo y se puede borrar con `git branch -d fix/errores-ui-web`.

### 12.6 Limpieza ejecutada (20-10-2026)

Se eliminó el `.git` accidental del home siguiendo la opción A:

- Copia de seguridad: `Documents/Restaurante Mira/_backups/home-git-2026-10-20.zip` (82 entradas, ~1 MB).
- También se guardaron `home-git-status.txt`, `home-git-head.txt` y `home-git-uncommitted.diff` en `_backups/`.
- `git` desde `C:\Users\ruben.munoz-blanco.e` ya responde `not a git repository`.
- Los repos de MIRA (frontend, backend y bóveda) no se tocaron y siguen funcionando.
- Las carpetas `wt-b34`, `wt-jam` y `wt-ajustes` siguen existiendo como carpetas sueltas; ya no resuelven a ningún repo. Se pueden borrar cuando se confirme que no guardan nada único.

Con esto deberían desaparecer los 30K cambios del editor. Si VS Code sigue mostrándolos, hay que cerrar y volver a abrir la carpeta de trabajo.

---

## 13. Próximo paso

Cuando Rubén dé el visto bueno:

1. Elegir la pantalla piloto.
2. Crear la rama y el tema Tailwind.
3. Migrar solo esa pantalla.
4. Revisar capturas y decidir si se extiende.

Mientras tanto, este documento queda como referencia y el código **no se toca**.
