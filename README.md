# EC Game — Software Lab

Sitio estático (una sola página, un solo archivo) que funciona como **catálogo de descargas**
para PC y móvil: juegos, programas, IA, APKs, drivers y sistemas operativos.

Cada categoría se abre en un **modal tipo ventana de escritorio** (se puede arrastrar, redimensionar
y maximizar), y los programas individuales disparan un **selector de sistema operativo** que
redirige a la descarga oficial del proyecto.

- **Autor:** Estalingrado Corp — Facundo Nicolás Escobar (Project Manager)
- **Dominio de marca:** EC Game / Software Lab
- **Licencia:** ©2027 Estalingrado Corp — Todos los derechos reservados
- **Repo:** `github.com/nicotips27/ec-download`

---

## Índice

1. [Quick start](#quick-start)
2. [Stack y dependencias externas](#stack-y-dependencias-externas)
3. [Mapa del archivo `index.html`](#mapa-del-archivo-indexhtml)
4. [Arquitectura: los 8 sistemas](#arquitectura-los-8-sistemas)
5. [Referencia de datos](#referencia-de-datos)
6. [Catálogo de contenido](#catálogo-de-contenido)
7. [Cómo extender el sitio](#cómo-extender-el-sitio)
8. [Sistema de diseño](#sistema-de-diseño)
9. [Notas de mantenimiento y trampas conocidas](#notas-de-mantenimiento-y-trampas-conocidas)
10. [Despliegue](#despliegue)

---

## Quick start

No hay build. No hay `package.json`. No hay dependencias que instalar.

```bash
# Clonar
git clone https://github.com/nicotips27/ec-download.git
cd ec-download

# Abrir directamente en el navegador
start index.html          # Windows
xdg-open index.html       # Linux
open index.html           # macOS
```

O servírlo con cualquier servidor estático, que es lo recomendado para que las URLs
de GitHub Pages y los `iframe` se comporten igual que en producción:

```bash
python -m http.server 8000
# http://localhost:8000
```

Todo el JavaScript corre **al final del `<body>`** sin `type="module"`, así que no hay
dependencias de CORS ni módulos que resolver. Funciona con `file://`.

---

## Stack y dependencias externas

HTML5 + CSS3 + JavaScript vanilla (ES2018ish: `const`/`let`, arrow functions, template
literals, `IntersectionObserver`, `requestAnimationFrame`). **Cero frameworks, cero librerías
de terceros propias.** Todo el JavaScript está inline en un solo `<script>`.

Todo lo externo se carga por CDN o URL absoluta:

| Recurso | URL | Uso | ¿Critico? |
|---|---|---|---|
| Google Fonts — Orbitron (400/700/900) | `fonts.googleapis.com` | Tipografía display (títulos, botones) | Sí, si cae la fuente cambia el layout |
| Google Fonts — Rajdhani | `fonts.googleapis.com` | **Cargada pero sin uso real** (ver [notas](#notas-de-mantenimiento-y-trampas-conocidas)) | No |
| Font Awesome 6.4.0 | `cdnjs.cloudflare.com` | Todos los iconos (`fas`, `fab`) | **Sí** — sin FA no hay icono en toda la página |
| Imágenes Unsplash | `images.unsplash.com` | Fondos del carousel | Sí, para el hero |
| Imágenes Tumblr | `64.media.tumblr.com` / `lh3.googleusercontent.com` | Logo, screenshot de Demiurgo Box, arte de EC Download | Sí, para el logo |
| Vimeo Player | `player.vimeo.com` | Video explicativo de Demiurgo Box dentro del modal | No |
| myinstants.com | `www.myinstants.com/media/sounds/kaching.mp3` | Sonido de "caja registradora" (`new Audio`, precargado) | No |
| GitHub Pages del proyecto | `nicotips27.github.io/Demiurgo-box/` | Botón "JUGAR" del modal Demiurgo | No |
| Enlace externo del nav | `nicotips27.github.io/Estalingrado-corp-Buscador/` | Item "Estalingrado Corp" del menú | No |

**Consecuencia práctica:** el sitio es 100 % dependiente de red para verse bien. No hay
fallback de imagen ni `alt` decorativo consistente. Si un CDN cae, se ve el hueco.

---

## Mapa del archivo `index.html`

2442 líneas, 4 secciones. Este es el mapa completo:

| Líneas | Bloque |
|---|---|
| 1–9 | `<head>`: metadatos, título, favicon (Tumblr), 2 `<link>` de CDN |
| **10–1367** | **CSS completo** (1358 líneas, inline en `<style>`) |
| 11–37 | Reset `*` + design tokens en `:root` + `body` |
| 39–76 | Fondo animado (`.bg-animation`) y partículas (`.particles` / `.particle` + `@keyframes float`) |
| 78–144 | `header`, `.logo`, `nav` |
| 145–218 | `.hero`, `.cta-button`, `.platforms` |
| 220–403 | Carousel: slides, flechas, dots (`@keyframes dotFill`) |
| 404–418 | Breakpoint `768px` + `480px` del carousel/hero |
| 423–452 | `.section-title` |
| 446–452 | `.grid` |
| 454–536 | `.card` y variantes (`.card-icon`, `.card-button`) |
| 537–600 | `.badge` |
| 602–660 | `.mobile-menu-btn`, `.fade-in` |
| 661–740 | `.help-fab`, `.help-menu`, `.about-*` (overlay, modal, titlebar, body, logo) |
| 741–889 | `.game-modal-*`: overlay, caja, titlebar, dots, body, hero, heading, features |
| 890–949 | `@keyframes` de entrada/salida de modales (`modalIn`, `modalOut`, `overlayIn`, `overlayOut`) |
| 927–949 | `.modal-resize-handle` (el tirador de resize, inyectado por JS) |
| 950–1009 | `.game-modal-titlebar`, `.gm-dot*` |
| 1010–1042 | `.feature-block`, `.feature-icon` |
| 1043–1143 | Breakpoint `768px` de modales + `.license-tag` / `.license-free` / `.license-paid` |
| 1065–1143 | `.os-picker*` |
| 1157–1215 | `.ec-construction*` (popup genérico reutilizado por 3 modales) |
| 1216–1259 | `.header-search-*` y `.hsd-*` (buscador) |
| 1260–1291 | Items del dropdown de búsqueda |
| 1292–1333 | Breakpoint `768px` del buscador |
| 1334–1367 | Pantalla de carga: `.loader-wrapper`, anillo, barra, `@keyframes fillProgress` / `blinkStatus` |
| **1371–1870** | **Markup del `<body>`** |
| 1373–1385 | Loader (pantalla de "INICIANDO SISTEMA") |
| 1387–1388 | `.bg-animation` + contenedor `#particles` |
| 1390–1418 | `<header>`: logo, buscador, botón móvil, `<nav>` |
| 1420–1457 | Carousel del hero (3 slides + dots) |
| 1459–1469 | `<section id="inicio">` — hero "SOFTWARE LAB" |
| 1471–1485 | `<section id="ec-games">` — card de Demiurgo Box |
| 1487–1497 | `<section id="software">` — grid de 6 categorías |
| 1500 | Modal Demiurgo Box (video Vimeo + link "JUGAR") |
| 1503–1540 | Modal **Sistemas Operativos** |
| 1541–1617 | Modal **Programas** |
| 1618–1688 | Modal **Juegos** |
| 1689–1734 | Modal **APK Android** |
| 1735–1746 | Modal **Drivers** |
| 1747–1806 | Modal **Inteligencia Artificial** |
| 1809–1816 | `#osPicker` — "¿Qué sistema usás?" |
| 1819–1827 | `#ecConstruction` — popup "en construcción" de EC Download |
| 1830–1848 | `#adDownloadModal` — popup de anuncio + contraseña ZIP + countdown |
| 1851–1861 | `#demiurgoComingSoon` — popup "en construcción" de Demiurgo |
| 1864 | `.help-fab` — botón `?` con menú (Soporte / Sobre esta página) |
| 1867 | `#aboutModal` — créditos |
| 1870 | `<footer>` — logo + 5 redes sociales |
| **1872–2440** | **JavaScript completo** (568 líneas, inline en `<script>`) |
| 1873–1893 | Lógica del loader |
| 1895–1896 | Generación de partículas |
| 1898–1930 | Carousel (IIFE) |
| 1932–1935 | Scroll reveal, smooth scroll, header al scrollear |
| 1937–1958 | Modales de categoría: `openCatModal` / `closeCatModal` |
| 1960–1996 | Menú help + `#aboutModal` |
| 1998–2005 | `#demiurgoComingSoon` |
| 2006–2007 | Tecla `Escape` + click fuera |
| 2009–2040 | Menú móvil |
| 2042–2122 | `pcLinks` + `showOSPicker` / `closeOSPicker` |
| 2124–2131 | `#ecConstruction` |
| 2133–2178 | `#adDownloadModal` |
| 2180–2289 | **Buscador en tiempo real** (`searchIndex` + filtering) |
| 2290–2368 | **Window drag & resize** (mouse + touch) |
| 2370–2416 | **Maximizar / restaurar** (punto verde) |
| 2418–2439 | `resetModalStyles` + monkey patch de `closeCatModal` |

---

## Arquitectura: los 8 sistemas

No hay módulos ni separación de concerns: es un script lineal que se ejecuta entero una vez
al cargar. Aun así, el código se organiza en 8 sistemas independientes que se comunican
mediante el DOM (clases CSS e IDs), nunca mediante imports.

### 1. Pantalla de carga (1873–1893, CSS 1296–1367)

Overlay a pantalla completa con logo, anillo giratorio, barra de progreso y texto de estado.

- Al disparar `window.load`, rota 3 textos de estado cada 500 ms:
  `"Estableciendo conexión..."` → `"Verificando base de datos..."` → `"Acceso Concedido."`
- A los **1800 ms** fijos agrega `.hidden` al loader (que lo saca con `opacity` + `visibility`).
- La barra de progreso se anima con `@keyframes fillProgress`, no con JS.

Es decorativo: el tiempo es un `setTimeout` constante, no espera a que carguen los recursos.
Si algo falla, el loader se oculta igual.

### 2. Fondo animado y partículas (1895–1896, CSS 39–76)

50 `<div class="particle">` de 2×2 px generados al azar (`left`, `animation-delay`,
`animation-duration` entre 10–20 s) que suben y rotan con `@keyframes float`. El contenedor
tiene `z-index: -1`, así que quedan detrás de todo.

### 3. Carousel del hero (1898–1930, CSS 220–418)

IIFE auto-contenida. 3 slides con `background-image` de Unsplash y Ken Burns (efecto zoom
de 7 s al activarse, ver la transición en `.carousel-slide`).

- Autoplay de **5000 ms**.
- `mouseenter` / `mouseleave` sobre el contenedor pausan el avance (`paused`) sin matar el
  `setInterval`.
- Cada click manual (dot o flecha) reinicia el autoplay: `stopAuto()` → `goTo()` → `startAuto()`.
- `goTo(idx)` alterna la clase `.active` en el slide y su dot correspondiente.

### 4. Scroll reveal, smooth scroll y header (1932–1935)

- `IntersectionObserver` con `threshold: 0.1` y `rootMargin: '0px 0px -50px 0px'` agrega
  `.visible` a cada `.fade-in` cuando entra en viewport (animación en CSS 661).
- Todos los `a[href^="#"]` interceptan el click y hacen `scrollIntoView({ behavior: 'smooth' })`.
  Esto incluye el nav, el CTA y los botones-card con ancla.
- Listener de `scroll` que cambia el `background` del `header` entre opaco y translúcido
  según `pageYOffset > 100`.

### 5. Sistema de modales tipo ventana (1937–1996 + 2290–2439)

Es la pieza más grande del sitio. Los 7 modales de categoría comparten **el mismo estilo
de "ventana de escritorio"** (macOS):

| Pieza | Comportamiento |
|---|---|
| `.game-modal-overlay` | Fondo oscurecido. Click en el overlay cierra, **pero** solo si `modalInteraction === false` (evita que cerrar un dragCounts como click fuera) |
| `.game-modal` | La caja de la ventana |
| `.game-modal-titlebar` | Barra de título: 3 dots + nombre. Es el **handle de arrastre** |
| `.gm-dot-red` | Cerrar (`onclick="closeCatModal()"`) |
| `.gm-dot-yellow` | In decorativo, sin acción |
| `.gm-dot-green` | Maximizar / restaurar (bind en 2410) |
| `.modal-resize-handle` | Tirador de resize, **inyectado por JS** (2350–2354) a cada modal |
| `.closing` | Clase que dispara la animación de salida; se limpia en `animationend` o a los 500 ms (fallback) |

**Maximizar (2371–2408):** guarda el estado previo del modal en `dataset.restore*` (position,
left, top, width, height, maxWidth, margin) y lo aplica a `94vw × 94vh` con `position: fixed`
en `3vw/3vh`. Al restaurar, devuelve los valores guardados. La transición dura 300 ms y se
limpia a los 350 ms.

**Drag (2301–2318):** al hacer `mousedown` en el titlebar, convierte el modal a
`position: absolute`, captura el offset inicial (`dragData = {ox, oy, l, t}`) y delega el
movimiento a un listener global de `document`. Si el modal estaba maximizado, **primero lo
restaura**. Ignora el click si empieza sobre un `.gm-dot` / `.about-dot`.

**Resize (2320–2329):** mismo patrón desde el tirador, con mínimos de `350×250 px`.

**Mouse + touch:** cada gesture se registra dos veces (`mousedown` + `touchstart` con
`{passive: false}` para poder hacer `preventDefault`), y el movimiento con `mousemove`/`mouseup`
y `touchmove`/`touchend` sobre `document`.

`getPos(e)` (2293) normaliza los dos casos: usa `e.clientX/clientY`, y si no existe cae a
`e.touches[0]`. Devuelve `null` si no hay ninguno de los dos.

**Estado global:** `activeModal`, `dragMode` (`'drag' | 'resize'`), `dragData` y
`modalInteraction`. Este último se pone en `true` al iniciar un gesto y se resetea **150 ms
después** de soltarlo (`onPointerEnd`), para que el evento `click` que sigue al drag ya no
cuente como click sobre el overlay.

**Al abrir** (`openCatModal`, 1937): agrega `.active`, bloquea `document.body.style.overflow`,
y en el siguiente `requestAnimationFrame` **maximiza automáticamente** el modal. Por eso
todos los modales aparecen casi a pantalla completa en vez de como una caja chica.

**Al cerrar** (`closeCatModal`, 1948): itera **todos** los `.game-modal-overlay.active`, resetea
los `iframe` con `iframe.src = iframe.src` (truco para reiniciar el video de Vimeo),
agrega `.closing`, y desbloquea el scroll del body.

**Botón Maximizar:** después de `resetModalStyles` (2419) y un monkey patch de `closeCatModal`
(2434–2439) que envuelve la función original, llama `onPointerEnd()` y limpia los estilos
inline de todos los modales 350 ms después, para que el siguiente `openCatModal` vuelva a
maximizar limpio.

### 6. Selector de sistema operativo (2042–2122, CSS 1065–1143)

Es el flujo de descarga de un programa individual:

```
click en card  →  showOSPicker('firefox')  →  popup "¿Qué sistema usás?"
                                            →  botón Windows / Linux → window.open(url, '_blank')
```

- `pcLinks[appId]` da `{ windows, linux, paid }`. Los botones se **crean por JS** con
  `document.createElement`, no están en el HTML.
- Si falta el link de una plataforma, ese botón simplemente no se crea.
- Si faltan ambos, se muestra "No disponible para descarga directa en este momento."
- `paid: true` agrega un tercer botón **"EC DOWNLOAD — Gratis"** que, en vez de descargar,
  abre el popup `#ecConstruction` ("en construcción"). Es el placeholder para el futuro
  mirror propio de los binaries.
- La tecla `Escape` cierra este popup (2006).

### 7. Buscador en tiempo real (2180–2289, CSS 1216–1291)

Input en el header con dropdown de resultados.

- `searchIndex` (2181–2244) es un array de 62 entradas con shape
  `{ name, desc, icon, cat, action }`, donde `action` es una **función** (closure) que decide
  si abrir un modal de categoría o el OS picker.
- Filtrado por `input`: `name` + `desc` + `cat` con `.includes()`, case-insensitive,
  top 8 resultados.
- Cada resultado se inyecta con `innerHTML` (string template) y después se le bindea el
  click por `data-idx`, que ejecuta `r.action()` y limpia el input.
- Click fuera del `.header-search-wrap` cierra el dropdown (2280).
- `Escape` en el input cierra y limpia (2286).

### 8. UI Miscelánea

| Sistema | Detalle |
|---|---|
| **Menú móvil** (2009–2040) | `#mobileMenuBtn` alterna `.active` en `#navMenu` y cambia el icono `fa-bars` ⇄ `fa-times`. Se cierra al clickear un link o fuera del nav. |
| **Help FAB** (1864, 1960) | Botón `?` flotante. `toggleHelpMenu()` alterna `#helpMenu` con 2 items: "Soporte" (link externo a un FAQ de Wix) y "Sobre esta página" (abre `#aboutModal`). Click fuera lo cierra (2007). |
| **About modal** (1961–1996) | Mismo estilo de ventana que los modales de categoría. Hace `resetModalStyles` completo al cerrar, incluidos los dots verdes. |
| **`#adDownloadModal`** (1830–1848, 2137–2178) | Popup de anuncio: muestra una **contraseña de ZIP**, un GIF de anuncio, y un **countdown de 10 s** antes de habilitar el botón "DESCARGAR AHORA". Reproduce un efecto de sonido de caja registradora. |
| **Tecla `Escape`** (2006) | Cierra **todo** lo que esté abierto de golpe, en este orden: `closeCatModal`, `closeAboutModal`, `closeOSPicker`, `closeECConstruction`, `closeDemiurgoComingSoon`, `closeAdDownload`. |
| **Footer** (1870) | Logo + 5 redes: Facebook, X, YouTube, Tumblr, Telegram. |

---

## Referencia de datos

Hay exactamente **dos** estructuras de datos en todo el sitio. Ambas son JS plano, sin
schema ni validación. Si querés agregar un programa o una app, estos son los dos lugares
que tenés que tocar.

### `pcLinks` (2043–2085)

Diccionario `appId → { windows, linux, paid }`. **41 entradas.**

| Campo | Tipo | Regla |
|---|---|---|
| `appId` | `string` | Slug en minúsculas, sin espacios ni acentos. Debe coincidir **exactamente** con el argumento de `showOSPicker()`. Artículos con guiones: `'7zip'`, `'notepadpp'`, `'githubdesktop'`, `'winterminal'`, `'gpt4all'`, `'janai'`, `'msoffice'`, `'lmstudio'`. |
| `windows` | `string \| null` | URL oficial de descarga para Windows. `null` = la plataforma no aparece. |
| `linux` | `string \| null` | Ídem para Linux. Muchos programas usan **el mismo link** en ambas plataformas (o un link a la home en vez de al `.exe`). |
| `paid` | `boolean` | Opcional. Si es `true`, agrega el botón "EC DOWNLOAD — Gratis" que abre `#ecConstruction`. |

```js
// Ejemplo de entrada válida
chrome: { windows: 'https://www.google.com/chrome/', linux: 'https://www.google.com/chrome/' },
// Ejemplo de entrada con pago
kaspersky: { windows: 'https://www.kaspersky.com/downloads', linux: null, paid: true },
// Ejemplo de entrada solo-Windows
rufus: { windows: 'https://rufus.ie/downloads/', linux: null },
```

### `searchIndex` (2181–2244)

Array de objetos. **62 entradas.** Es lo que el usuario ve en el dropdown del buscador.

| Campo | Tipo | Regla |
|---|---|---|
| `name` | `string` | Lo que se muestra en negrita. También es el campo de búsqueda principal. |
| `desc` | `string` | Una línea corta. Se busca con `includes()`. |
| `icon` | `string` | Clases de Font Awesome **sin** la clase `fa` base, ej. `'fas fa-brain'`, `'fab fa-windows'`. Se inyecta como `<i class="${icon}">`. |
| `cat` | `string` | Etiqueta de grupo, se muestra como `cat · desc` y también se busca. Valores actuales: `Categoría`, `EC Game`, `Programas`, `IA Local`, `IA`, `Programadores`, `Sistemas Operativos`, `Drivers`, `Juegos`, `APK Android`. |
| `action` | `function` | Closure sin argumentos. Llama a `openCatModal(id)` o `showOSPicker(appId)`. |

```js
// Entrada que abre un modal de categoría
{ name: 'Kimi AI', desc: 'IA gratuita para lectura de documentos largos',
  icon: 'fas fa-book-reader', cat: 'IA', action: () => openCatModal('ia') },

// Entrada que abre el OS picker
{ name: 'Firefox', desc: 'Navegador privado y rápido',
  icon: 'fab fa-firefox-browser', cat: 'Programas', action: () => showOSPicker('firefox') },
```

**Correlación entre las dos estructuras:** `searchIndex.action()` es el único puente entre el
buscador y el resto del sitio. Las cards del HTML (dentro de los modales) llaman a
`showOSPicker()` directamente con `onclick`, sin pasar por `searchIndex`. Son dos caminos
independientes al mismo destino, y hay que actualizar los dos al agregar algo nuevo.

---

## Catálogo de contenido

| Modal | ID | Cards | Vía de descarga | Notas |
|---|---|---|---|---|
| Demiurgo Box | `modal-demiurgo` | — | — | Modal de presentación: features, video Vimeo, botón "JUGAR" al juego real |
| Sistemas Operativos | `modal-os` | 17 | Link directo `<a>` | Windows 10/11, Linux (principiantes + avanzados + ligeros), macOS |
| Programas | `modal-programas-unificados` | 56 | `showOSPicker` (35) | Browsers, oficina, diseño, desarrollo, IA local, seguridad, multimedia |
| Juegos | `modal-juegos-unificados` | 47 | Link directo | Open Source (SupertuxKart, Wesnoth, Xonotic, Minetest, OpenTTD, Freeciv, Veloren…) y comerciales (Steam, Epic, GOG), más free-to-play |
| APK Android | `modal-apk` | 36 | Link directo | Mensajería, F-Droid, Termux, NewPipe, Aurora Store, Obtainium, tweaked apps |
| Drivers | `modal-drivers` | 6 | `showOSPicker` (6) | NVIDIA, AMD, Intel, Realtek, Logitech G HUB, Snappy Driver Installer |
| Inteligencia Artificial | `modal-ia` | 49 | Link directo | Gratuitas (ChatGPT Free, Gemini, Kimi, DeepSeek, Leonardo, Hugging Face…), premium (Claude Pro, Midjourney, Runway, ElevenLabs…) y local (Ollama, llama.cpp, PrivateGPT, LocalAI, text-generation-webui) |

Notá la diferencia de Approach: los 3 modales de contenido (SOs, Juegos, APKs) usan
`<a href="..." class="card-button">DESCARGAR</a>` directo a la fuente, mientras que Programas
y Drivers usan el OS picker porque necesitan elegir plataforma. La sección IA **también** usa
links directos a `<a>` aunque muchas entradas son claramente multiplataforma — es el modo
más fácil de agregar y el que menos fricción da al usuario.

Además hay un botón de badge que se abre desde el carousel del hero:

- `juegos-unificados` (Juegos)
- `programas-unificados` (Programas)
- `ia` (Inteligencia Artificial)

---

## Cómo extender el sitio

### Agregar un programa (flujo completo)

Necesitás **3 cambios**. Los dos primeros son imprescindibles, el tercero es recomendado.

**1. Agregá la entrada en `pcLinks`** (dentro del bloque 2043–2085):

```js
myapp: { windows: 'https://ejemplo.com/download/win', linux: 'https://ejemplo.com/download/linux' },
```

**2. Agregá la card en el HTML** del modal correspondiente (dentro de las líneas 1541–1617):

```html
<div class="card">
  <span class="license-tag license-free">GRATUITO</span>
  <div class="card-icon"><i class="fas fa-cube"></i></div>
  <h3>MyApp</h3>
  <p>Descripción corta de una línea.</p>
  <button class="card-button" onclick="showOSPicker('myapp')">DESCARGAR</button>
</div>
```

**3. Agregá la entrada en `searchIndex`** (dentro del bloque 2181–2244):

```js
{ name: 'MyApp', desc: 'Descripción corta', icon: 'fas fa-cube',
  cat: 'Programas', action: () => showOSPicker('myapp') },
```

> Si el `appId` de `pcLinks` no coincide exactamente con el string que pasás a
> `showOSPicker()`, el popup abre y muestra "No disponible para descarga directa"
> — no hay error en consola, así que es un fallo silencioso.

### Agregar una categoría nueva

1. **HTML:** copiá el bloque de un modal existente (ej. `modal-drivers`, 1735–1746),
   cambiá el `id` a `modal-miclase` y el título. Es un `.game-modal-overlay` con
   `.game-modal` > `.game-modal-titlebar` (+ `.game-modal-dots` con los 3 dots) +
   `.game-modal-body`.
2. **Card en la home:** agregá un `<div class="card fade-in" onclick="openCatModal('miclase')">`
   en la grid de `.categories#software` (1489–1496).
3. **Buscador:** agregá la entrada en `searchIndex` con `action: () => openCatModal('miclase')`.
4. **Carousel (opcional):** si querés que sea slide del hero, duplicá un `.carousel-slide`
   (1424–1450) + su `.carousel-dot` (1452–1456) y actualizá el `data-slide` del dot nuevo.

`openCatModal` no necesita registro: resuelve `document.getElementById('modal-' + id)`, así
que el `id` del HTML es la única fuente de verdad.

### Cambiar textos de la pantalla de carga

`statusTexts` en 1875 (array de 3 strings) y el `setTimeout(..., 1800)` en 1892 que decide
cuánto tarda en cerrarse. El ancho de la barra está en el `@keyframes fillProgress` (1354),
no en JS.

### Cambiar el diseño global

Todo el color del sitio sale de las custom properties en `:root` (17–30). Cambiar
`--primary` y `--secondary` recolorea buttons, iconos, bordes, dots y el glow del hero.
Los colores de las cards y tags de licencia están hardcodeados en sus clases
(454–600, 1144–1156), no en tokens.

---

## Sistema de diseño

**Tokens** (`:root`, 17–30):

| Token | Valor | Rol |
|---|---|---|
| `--primary` | `#66c0f0` | Azul principal: títulos, iconos, acentos |
| `--secondary` | `#1a9fff` | Azul saturado: gradientes |
| `--dark` | `#171a21` | Fondo del header |
| `--darker` | `#1b2838` | Fondo del body y hero |
| `--card-bg` | `#2a475e` | Fondo de card |
| `--card-hover` | `#316282` | Hover de card |
| `--glass` | `rgba(255,255,255,0.03)` | Overlay translúcido |
| `--text-primary` | `#c7d5e0` | Texto normal |
| `--text-secondary` | `#8f98a0` | Texto secundario |
| `--text-muted` | `#6d7b8f` | Texto terciario, captions |
| `--accent-green` | `#a4d007` | Acento verde (badges "UNIFICADO", licencias gratis) |
| `--border-color` | `rgba(102,192,240,0.15)` | Bordes sutiles |

**Tipografía:**
- **Orbitron** (700/900) para todo lo display: títulos de sección, botones, logo, headings
  de modal, breadcrumbs. 43 usos, casi todos en `font-family` inline.
- **Rajdhani** se descarga pero **no se usa**: aparece una sola vez, como 4to fallback
  del stack del `body` (33), después de `-apple-system`, `BlinkMacSystemFont` y `'Segoe UI'`.
- El stack del body es: `-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Rajdhani', sans-serif`.

**Convenciones de clases:**
- Prefijos por componente: `.game-modal-*`, `.about-*`, `.carousel-*`, `.hsd-*` (header search),
  `.os-picker-*`, `.ec-construction-*`, `.loader-*`, `.gm-dot-*`.
- Las cards son `.card > (.badge | .card-icon + h3 + p + a.card-button)`.
- Las licencias usan `.license-tag` + `.license-free` (verde) o `.license-paid` (rojo).
- Las secciones se identifican por `id`: `inicio`, `ec-games`, `software`.
- Botones: `.cta-button` (principal, hero), `.card-button` (dentro de card),
  `.carousel-btn` (carousel), `.os-btn` (picker), `.ec-close-btn` (popup),
  `.os-picker-close` (cancelar).

**Responsive:** dos breakpoints, `768px` y `480px`. Ambos están inline en el `<style>`
(404, 612, 1043, 1292) y mobile-first por partes: el menú de navegación se oculta detrás
del `#mobileMenuBtn` y el carousel pierde flechas.

---

## Notas de mantenimiento y trampas conocidas

Cosas que vas a tropezar si tocás el código. No están arregladas porque el sitio funciona,
pero conviene saberlas.

### 1. Dos funciones muertas (no tienen ningún trigger)

- **`startAdDownload(url, pass)`** (2137) — el popup de anuncio con contraseña ZIP y
  countdown de 10 s. **Nunca se la llama.** El HTML del popup existe (1830–1848) y el
  listener del overlay también (2178), pero ningún botón del sitio lo invoca. Para que
  aparezca hay que agregarle un `onclick` a algún botón.
- **`showDemiurgoComingSoon()`** (1999) — el popup "Demiurgo en construcción". Tampoco
  tiene trigger; el modal de Demiurgo (1500) enlaza directo al juego real.

Además, la página llama `closeDemiurgoComingSoon()` y `closeAdDownload()` en el handler de
`Escape` (2006), así que ambos modales están cableados a un lado del flujo y sucios del otro.

### 2. Inyección de HTML sin escapar en el buscador

`index.html:2258` interpola el valor crudo del input del usuario dentro de `innerHTML`:

```js
searchDropdown.innerHTML = `<div class="hsd-empty">...Sin resultados para "${this.value}"</div>`;
```

`this.value` no pasa por escape. Un input como `<img src=x onerror=alert(1)>` se ejecuta.
Como es un sitio público sin auth, el riesgo es baixo (auto-XSS), pero si alguna vez se
copia este patrón a otro contexto hay que usar `textContent` o escapar. El resto del
renderizado del dropdown (2260–2268) **sí** usa datos propios, no input del usuario, así
que ahí no hay problema.

### 3. `closeCatModal` está monkey-patcheada

`index.html:2434–2439` reasigna la función global después de declararla:

```js
const origCloseCatModal = closeCatModal;
closeCatModal = function() { onPointerEnd(); origCloseCatModal(); setTimeout(resetModalStyles, 350); };
```

Funciona porque los `onclick` inline y los listeners resuelven `closeCatModal` en scope
global **al momento de ejecutarse**, no al momento de bindearse. Pero es frágil: si
alguien captura la función en una variable antes de esta línea, o la mueve antes del
patch, el reset de estilos deja de correr. Si tocás esta zona, consideráOSE integrar
el `resetModalStyles` directamente dentro de `closeCatModal` y borrar el patch.

### 4. Truco del iframe de Vimeo

`index.html:1950` recarga el video con `iframe.src = iframe.src`. Funciona, pero
re-descarga el iframe entero cada vez que se cierra el modal, en vez de solo llamar al
API de Vimeo o usar `pause()`. Si algún día hay más de un iframe, esto escala mal.

### 5. El modal se maximiza solo al abrir

`openCatModal` (1944) llama a `toggleMaximizeModal` en el `requestAnimationFrame`
siguiente. Efecto: **todos** los modales aparecen al 94 % del viewport, no como una caja
centrada. Si querés un modal centrado chico, hay que sacar esa llamada — peroojo: el
tamaño chico es exactamente el estado "restaurado", así que el usuario lo vería al
tamaño de la ventana minus el padding.

### 6. `pcLinks` es un bottleneck de mantenimiento

Los links viven en **un solo objeto** de 41 entradas. Agregar una app obliga a editar el
HTML de la card **y** `pcLinks` **y** `searchIndex`, tres lugares separados sin validación
de por medio. Un typo en el `appId` produce el mensaje "No disponible para descarga"
sin error visible. Considerar mover el link al `data-download` de la card y derivar
`pcLinks` del DOM, o al revés.

### 7. Links de descarga que ya no coinciden con la card

Algunas entradas de `pcLinks` apuntan a la **home** en vez de al archivo de descarga
directa (típicamente cuando el proyecto no tiene un link estable por plataforma):
`lmstudio` → `lmstudio.ai`, `gpt4all` → `gpt4all.io`, `janai` → `jan.ai`, `brave` (Windows)
→ `brave.com/download/`. También hay apps marcadas `paid: true` que en realidad tienen
descarga gratuita pública en su sitio oficial. Vale la pena una pasada de verificación
de URLs.

### 8. Rajdhani: fuente descargada sin usar

Ver [Sistema de diseño](#sistema-de-diseño). Se puede sacar del `<link>` de Google Fonts
(8) y del stack del `body` (33) para ahorrar una request.

### 9. No hay red de seguridad

Cero tests, cero linting, cero CI, cero `package.json`. El único chequeo posible es abrir
el archivo y hacer click en todo, en particular: los 7 modales, arrastrar + redimensionar
+ maximizar cada uno, el buscador con acentos y sin resultados, el OS picker en sus
tres variantes (Windows-only, ambas plataformas, y `paid`), y la tecla `Escape` con
varios modales abiertos a la vez.

---

## Despliegue

El sitio es un único archivo estático. Sirve en cualquier hosting sin build.

**GitHub Pages:** subí `index.html` a la rama `gh-pages` (o a la raíz de la rama principal)
y activá Pages en la config del repo. No hay `.nojekyll` ni archivos `_assets`, así que
GitHub Jekyll no interfiere con nada.

**El resto:** Netlify, Vercel, Cloudflare Pages, un `nginx` cualquiera — subís el archivo
y listo. Si se sirve en un subdirectorio, ojo con que los links absolutos a
`nicotips27.github.io` (nav y botón de Demiurgo) son externos y no se rompen, pero
cualquier link relativo futuro sí lo haría.

**Health check:** no aplica — no hay backend, ni base de datos, ni variables de entorno,
ni claves de API. El texto "Verificando base de datos" del loader es decorativo.
