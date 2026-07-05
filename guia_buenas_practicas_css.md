# Guía de buenas prácticas CSS para sitios web

## Objetivo

Esta guía resume buenas prácticas para escribir CSS claro, mantenible, accesible y adaptable a distintos tamaños de pantalla. Está organizada en tres niveles:

```text
1. Básico: fundamentos visuales, selectores, cajas, espaciado, colores, tipografía, estados simples.
2. Intermedio: layouts, responsividad, componentes, formularios, dropdowns, contratos de estilos y paletas.
3. Avanzado: arquitectura CSS, tokens, capas de cascada, temas, container queries, animaciones, drag and drop y escalabilidad.
```

La meta no es memorizar propiedades, sino aprender a diseñar sistemas visuales consistentes.

---

# Parte I: fundamentos básicos

## 1. Qué es CSS

CSS, o Cascading Style Sheets, define cómo se presenta el HTML: colores, tamaños, márgenes, layout, animaciones, estados visuales y adaptación a pantallas.

HTML responde a la pregunta:

```text
¿Qué contenido existe?
```

CSS responde a:

```text
¿Cómo se ve ese contenido?
¿Cómo se adapta?
¿Cómo comunica estado?
¿Cómo mantiene consistencia visual?
```

Ejemplo mínimo:

```html
<button class="button">
    Guardar cambios
</button>
```

```css
.button {
    padding: 0.75rem 1rem;
    border: 0;
    border-radius: 0.5rem;
    background: #2563eb;
    color: white;
    font-weight: 600;
    cursor: pointer;
}
```

---

## 2. Regla práctica: separa estructura, estilo y comportamiento

Una base sana:

```text
HTML        estructura y semántica
CSS         presentación visual
JavaScript  comportamiento e interacción
```

Evita usar HTML solo para lograr apariencia visual.

Menos recomendable:

```html
<div onclick="save()" style="background: blue; color: white;">
    Guardar
</div>
```

Mejor:

```html
<button class="button" type="button">
    Guardar
</button>
```

```css
.button {
    background: var(--color-primary);
    color: var(--color-on-primary);
}
```

```javascript
document.querySelector(".button").addEventListener("click", save);
```

Ventajas:

```text
Mejor accesibilidad.
Mejor mantenibilidad.
Mejor reutilización.
Mejor compatibilidad con lectores de pantalla.
```

---

## 3. Normaliza el box model

El box model define cómo se calculan ancho, alto, padding, border y margin.

Recomendación común:

```css
*,
*::before,
*::after {
    box-sizing: border-box;
}
```

Con esto, si defines:

```css
.card {
    width: 300px;
    padding: 1rem;
    border: 1px solid #ddd;
}
```

el ancho total se mantiene en `300px`, en lugar de sumar padding y borde por fuera.

---

## 4. Usa un reset base, pero no borres todo sin criterio

Un reset elimina diferencias entre navegadores. Pero un reset agresivo puede romper estilos nativos útiles.

Base razonable:

```css
*,
*::before,
*::after {
    box-sizing: border-box;
}

html {
    line-height: 1.5;
    -webkit-text-size-adjust: 100%;
}

body {
    margin: 0;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    background: var(--color-bg);
    color: var(--color-text);
}

img,
picture,
svg,
video,
canvas {
    display: block;
    max-width: 100%;
}

button,
input,
textarea,
select {
    font: inherit;
}
```

Evita eliminar outline globalmente:

```css
/* Mala práctica */
*:focus {
    outline: none;
}
```

Si necesitas personalizarlo, reemplázalo por un foco visible:

```css
:focus-visible {
    outline: 3px solid var(--color-focus);
    outline-offset: 3px;
}
```

---

## 5. Usa clases para estilos reutilizables

Evita depender demasiado de etiquetas o IDs.

Menos flexible:

```css
#main-button {
    background: blue;
}
```

Más reutilizable:

```css
.button {
    background: var(--color-primary);
}
```

Los IDs tienen especificidad alta y dificultan sobrescribir estilos. Úsalos para anclas o JavaScript cuando corresponda, no como base de diseño.

---

## 6. Entiende la cascada y la especificidad

CSS decide qué regla gana usando:

```text
1. Origen e importancia.
2. Capas de cascada, si existen.
3. Especificidad.
4. Orden de aparición.
```

Ejemplo:

```css
.button {
    color: blue;
}

.card .button {
    color: red;
}
```

La segunda regla gana porque es más específica.

Evita la guerra de especificidad:

```css
/* Mala señal */
main section.product-list div.card button.button.primary {
    color: red !important;
}
```

Mejor:

```css
.button--primary {
    color: var(--color-on-primary);
}
```

Regla práctica:

```text
Usa selectores cortos.
Evita encadenamientos largos.
Evita !important salvo casos controlados.
```

---

## 7. Unidades básicas: px, rem, em, %, vw, vh

Usa cada unidad con intención.

```text
px     bordes finos, detalles específicos.
rem    tipografía, spacing global, tamaños consistentes.
em     tamaños relativos al componente.
%      dimensiones relativas al contenedor.
vw/vh  dimensiones relativas al viewport.
```

Recomendación:

```css
:root {
    font-size: 16px;
}

.card {
    padding: 1rem;
    border-radius: 0.75rem;
}

.card__title {
    font-size: 1.25rem;
}
```

Evita fijar demasiadas dimensiones en `px` si el diseño debe adaptarse.

---

## 8. Espaciado consistente

No inventes márgenes distintos en cada componente. Define una escala.

```css
:root {
    --space-1: 0.25rem;
    --space-2: 0.5rem;
    --space-3: 0.75rem;
    --space-4: 1rem;
    --space-6: 1.5rem;
    --space-8: 2rem;
    --space-12: 3rem;
}
```

Uso:

```css
.card {
    padding: var(--space-6);
}

.card + .card {
    margin-top: var(--space-4);
}
```

Regla práctica:

```text
El espaciado debe parecer sistemático.
Si todo tiene medidas arbitrarias, el sitio se ve inconsistente.
```

---

## 9. Tipografía básica

Define una jerarquía clara.

```css
:root {
    --font-sans: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;

    --text-xs: 0.75rem;
    --text-sm: 0.875rem;
    --text-md: 1rem;
    --text-lg: 1.125rem;
    --text-xl: 1.5rem;
    --text-2xl: 2rem;
}

body {
    font-family: var(--font-sans);
    font-size: var(--text-md);
}

h1,
h2,
h3 {
    line-height: 1.15;
    text-wrap: balance;
}

p {
    max-width: 70ch;
}
```

`ch` es útil para ancho de lectura. Un párrafo de `60ch` a `75ch` suele ser más legible que un bloque de texto que cruza toda la pantalla.

---

## 10. Colores básicos

Usa colores semánticos, no solo nombres visuales.

Menos mantenible:

```css
.alert {
    background: red;
}
```

Más mantenible:

```css
:root {
    --color-danger: #dc2626;
    --color-danger-bg: #fee2e2;
    --color-danger-text: #7f1d1d;
}

.alert--danger {
    background: var(--color-danger-bg);
    color: var(--color-danger-text);
    border-color: var(--color-danger);
}
```

Semántico significa que el token describe el rol, no el color exacto.

```text
--color-primary
--color-success
--color-warning
--color-danger
--color-bg
--color-surface
--color-text
--color-muted
```

---

## 11. Hovers básicos

Un hover debe comunicar interactividad sin exagerar.

```css
.button {
    background: var(--color-primary);
    color: var(--color-on-primary);
    transition:
        background-color 160ms ease,
        transform 160ms ease,
        box-shadow 160ms ease;
}

.button:hover {
    background: var(--color-primary-hover);
    transform: translateY(-1px);
    box-shadow: 0 0.5rem 1rem rgb(0 0 0 / 0.12);
}

.button:active {
    transform: translateY(0);
    box-shadow: none;
}
```

Buenas prácticas:

```text
El hover no debe ser la única pista de interactividad.
El estado focus también debe estar diseñado.
Evita animaciones largas en botones frecuentes.
```

---

## 12. Estados focus, active y disabled

No diseñes solo el estado normal.

```css
.button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    min-height: 2.75rem;
    padding: 0 1rem;
    border: 0;
    border-radius: 0.5rem;
    background: var(--color-primary);
    color: var(--color-on-primary);
    font-weight: 600;
    cursor: pointer;
}

.button:hover {
    background: var(--color-primary-hover);
}

.button:focus-visible {
    outline: 3px solid var(--color-focus);
    outline-offset: 3px;
}

.button:active {
    transform: translateY(1px);
}

.button:disabled {
    opacity: 0.55;
    cursor: not-allowed;
}
```

---

## 13. Links bien diseñados

No elimines el subrayado sin ofrecer otra señal clara.

```css
a {
    color: var(--color-link);
    text-decoration-thickness: 0.08em;
    text-underline-offset: 0.18em;
}

a:hover {
    color: var(--color-link-hover);
}

a:focus-visible {
    outline: 3px solid var(--color-focus);
    outline-offset: 3px;
}
```

Para navegación puedes usar otro estilo, pero debe quedar claro que es clickeable.

---

## 14. Tarjetas básicas

HTML:

```html
<article class="card">
    <img class="card__image" src="cover.jpg" alt="Portada del juego">
    <div class="card__body">
        <h2 class="card__title">Daggerheart</h2>
        <p class="card__text">
            Juego de rol de fantasía con énfasis narrativo.
        </p>
        <a class="card__link" href="/games/daggerheart">
            Ver detalle
        </a>
    </div>
</article>
```

CSS:

```css
.card {
    overflow: hidden;
    border: 1px solid var(--color-border);
    border-radius: 1rem;
    background: var(--color-surface);
    box-shadow: 0 0.25rem 1rem rgb(0 0 0 / 0.06);
}

.card__image {
    width: 100%;
    aspect-ratio: 16 / 9;
    object-fit: cover;
}

.card__body {
    display: grid;
    gap: var(--space-3);
    padding: var(--space-4);
}

.card__title {
    margin: 0;
    font-size: var(--text-lg);
}

.card__text {
    margin: 0;
    color: var(--color-muted);
}
```

---

## 15. Despliegue simple con details/summary

Para contenido desplegable básico, usa HTML nativo.

```html
<details class="disclosure">
    <summary class="disclosure__summary">
        Ver detalles
    </summary>

    <div class="disclosure__content">
        <p>
            Este contenido se muestra al abrir el bloque.
        </p>
    </div>
</details>
```

```css
.disclosure {
    border: 1px solid var(--color-border);
    border-radius: 0.75rem;
    background: var(--color-surface);
}

.disclosure__summary {
    padding: var(--space-4);
    cursor: pointer;
    font-weight: 600;
}

.disclosure__summary:hover {
    background: var(--color-surface-hover);
}

.disclosure__content {
    padding: 0 var(--space-4) var(--space-4);
    color: var(--color-muted);
}
```

Ventajas:

```text
Funciona sin JavaScript.
Tiene semántica de apertura/cierre.
Es suficiente para FAQs y bloques simples.
```

---

# Parte II: nivel intermedio

## 16. Layout con Flexbox

Flexbox sirve para distribuir elementos en una dimensión: fila o columna.

Ejemplo: barra de navegación.

```html
<header class="site-header">
    <a class="site-header__brand" href="/">
        Mi sitio
    </a>

    <nav class="site-header__nav" aria-label="Principal">
        <a href="/productos">Productos</a>
        <a href="/precios">Precios</a>
        <a href="/contacto">Contacto</a>
    </nav>
</header>
```

```css
.site-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--space-4);
    padding: var(--space-4) var(--space-6);
    border-bottom: 1px solid var(--color-border);
}

.site-header__brand {
    font-weight: 800;
    color: var(--color-text);
    text-decoration: none;
}

.site-header__nav {
    display: flex;
    align-items: center;
    gap: var(--space-4);
}
```

Usa Flexbox para:

```text
Botoneras.
Navbar.
Elementos alineados.
Cards internas.
Pilas verticales simples.
```

---

## 17. Layout con Grid

Grid sirve para distribuir elementos en dos dimensiones: filas y columnas.

Ejemplo: grilla responsive de tarjetas.

```css
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
    gap: var(--space-6);
}
```

HTML:

```html
<section class="card-grid">
    <article class="card">...</article>
    <article class="card">...</article>
    <article class="card">...</article>
</section>
```

La expresión:

```css
minmax(min(100%, 18rem), 1fr)
```

significa:

```text
La tarjeta no debe ser más ancha que el contenedor.
La tarjeta debería medir al menos 18rem si hay espacio.
Si hay más espacio, las columnas pueden crecer.
```

Este patrón evita muchos media queries innecesarios.

---

## 18. Cuándo usar Flexbox y cuándo Grid

```text
Flexbox:
- Una fila o una columna.
- Distribuir espacio entre elementos.
- Alinear contenido interno.

Grid:
- Filas y columnas al mismo tiempo.
- Layouts de página.
- Galerías y paneles.
- Áreas con estructura.
```

Ejemplo de layout principal:

```css
.page {
    display: grid;
    grid-template-columns: 16rem minmax(0, 1fr);
    min-height: 100dvh;
}

.sidebar {
    border-right: 1px solid var(--color-border);
}

.content {
    padding: var(--space-6);
}

@media (width < 48rem) {
    .page {
        grid-template-columns: 1fr;
    }

    .sidebar {
        border-right: 0;
        border-bottom: 1px solid var(--color-border);
    }
}
```

---

## 19. Responsividad: piensa en rangos, no en dispositivos

Evita diseñar para modelos específicos de teléfono.

Menos recomendable:

```css
@media (width: 390px) {
    ...
}
```

Mejor:

```css
@media (width < 40rem) {
    ...
}

@media (40rem <= width < 64rem) {
    ...
}

@media (width >= 64rem) {
    ...
}
```

Regla práctica:

```text
El breakpoint debe aparecer donde el diseño se rompe,
no donde termina una marca de teléfono.
```

---

## 20. Cómo calcular tamaños responsivos

### 20.1 Usa `clamp()`

`clamp(min, preferido, max)` permite tamaños fluidos con límites.

```css
.hero__title {
    font-size: clamp(2rem, 5vw, 4.5rem);
}
```

Significa:

```text
No menos de 2rem.
Crece usando 5vw.
No más de 4.5rem.
```

Ejemplo más controlado:

```css
:root {
    --text-fluid-xl: clamp(2rem, 1.2rem + 3vw, 4rem);
}

.hero__title {
    font-size: var(--text-fluid-xl);
}
```

### 20.2 Fórmula práctica

Si quieres que un título mida:

```text
32px en 360px de viewport
64px en 1200px de viewport
```

puedes usar:

```css
.hero__title {
    font-size: clamp(2rem, 1.143rem + 3.81vw, 4rem);
}
```

Cálculo aproximado:

```text
Diferencia de tamaño: 64 - 32 = 32px
Diferencia de viewport: 1200 - 360 = 840px
Pendiente: 32 / 840 = 0.0381
En CSS: 3.81vw
```

En la práctica, no necesitas calcular todo manualmente para cada caso. Define una escala fluida y reutilízala.

### 20.3 Usa `min()`, `max()` y `clamp()`

```css
.container {
    width: min(100% - 2rem, 72rem);
    margin-inline: auto;
}

.sidebar {
    width: clamp(14rem, 20vw, 22rem);
}
```

Esto evita contenedores pegados al borde en móvil y demasiado anchos en escritorio.

---

## 21. Contenedores reutilizables

Un layout base útil:

```css
.container {
    width: min(100% - 2rem, 72rem);
    margin-inline: auto;
}

.section {
    padding-block: clamp(3rem, 6vw, 6rem);
}

.stack {
    display: grid;
    gap: var(--stack-gap, var(--space-4));
}

.cluster {
    display: flex;
    flex-wrap: wrap;
    gap: var(--cluster-gap, var(--space-3));
    align-items: center;
}
```

Uso:

```html
<section class="section">
    <div class="container stack" style="--stack-gap: 1.5rem;">
        <h2>Características</h2>
        <p>Contenido de la sección.</p>
    </div>
</section>
```

Estos patrones reducen CSS repetitivo.

---

## 22. Paletas de colores

Una paleta útil no es solo un grupo de colores bonitos. Debe cubrir roles.

### 22.1 Roles mínimos

```text
Background principal.
Superficie elevada.
Texto principal.
Texto secundario.
Borde.
Primario.
Primario hover.
Primario texto.
Éxito.
Advertencia.
Error.
Foco.
```

Ejemplo:

```css
:root {
    --color-bg: #f8fafc;
    --color-surface: #ffffff;
    --color-surface-hover: #f1f5f9;

    --color-text: #0f172a;
    --color-muted: #475569;
    --color-border: #cbd5e1;

    --color-primary: #2563eb;
    --color-primary-hover: #1d4ed8;
    --color-on-primary: #ffffff;

    --color-success: #16a34a;
    --color-warning: #d97706;
    --color-danger: #dc2626;

    --color-focus: #f59e0b;
}
```

### 22.2 Método simple para crear paleta

```text
1. Elige un color primario.
2. Define fondos neutros.
3. Define texto principal y secundario.
4. Crea variantes hover/active.
5. Define colores semánticos: success, warning, danger.
6. Verifica contraste.
7. Prueba en modo claro y oscuro.
```

### 22.3 Usa OKLCH o HSL para ajustar tonos

OKLCH puede ser útil porque permite modificar luminosidad, croma y tono de forma más predecible que hex puro.

```css
:root {
    --blue-50: oklch(97% 0.03 255);
    --blue-500: oklch(58% 0.21 255);
    --blue-700: oklch(45% 0.20 255);
}
```

Con `color-mix()` puedes derivar estados:

```css
.button {
    background: var(--color-primary);
}

.button:hover {
    background: color-mix(in oklch, var(--color-primary), black 12%);
}
```

Consejo:

```text
Usa color-mix() para sistemas modernos,
pero conserva tokens explícitos cuando necesites control visual estricto.
```

### 22.4 Contraste

Para texto normal, apunta al menos a contraste AA:

```text
Texto normal: 4.5:1.
Texto grande: 3:1.
Componentes e indicadores visuales importantes: 3:1.
```

No uses color como única forma de comunicar estado. Agrega texto, iconos o patrones.

---

## 23. Tema claro y oscuro

Define tokens semánticos. Luego cambia valores.

```css
:root {
    color-scheme: light;

    --color-bg: #f8fafc;
    --color-surface: #ffffff;
    --color-text: #0f172a;
    --color-muted: #475569;
    --color-border: #cbd5e1;
}

[data-theme="dark"] {
    color-scheme: dark;

    --color-bg: #020617;
    --color-surface: #0f172a;
    --color-text: #e2e8f0;
    --color-muted: #94a3b8;
    --color-border: #334155;
}
```

Uso:

```html
<html lang="es" data-theme="dark">
```

También puedes respetar el sistema del usuario:

```css
@media (prefers-color-scheme: dark) {
    :root {
        color-scheme: dark;

        --color-bg: #020617;
        --color-surface: #0f172a;
        --color-text: #e2e8f0;
        --color-muted: #94a3b8;
        --color-border: #334155;
    }
}
```

Regla práctica:

```text
No cambies componentes uno por uno.
Cambia tokens.
```

---

## 24. Formularios

HTML:

```html
<form class="form">
    <label class="field">
        <span class="field__label">Correo</span>
        <input class="field__control" type="email" placeholder="nombre@dominio.com">
        <span class="field__hint">Usaremos este correo para iniciar sesión.</span>
    </label>

    <button class="button" type="submit">
        Continuar
    </button>
</form>
```

CSS:

```css
.form {
    display: grid;
    gap: var(--space-4);
}

.field {
    display: grid;
    gap: var(--space-2);
}

.field__label {
    font-weight: 600;
}

.field__control {
    width: 100%;
    min-height: 2.75rem;
    padding: 0.625rem 0.75rem;
    border: 1px solid var(--color-border);
    border-radius: 0.5rem;
    background: var(--color-surface);
    color: var(--color-text);
}

.field__control:hover {
    border-color: var(--color-muted);
}

.field__control:focus {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: 0 0 0 3px color-mix(in oklch, var(--color-primary), transparent 75%);
}

.field__hint {
    color: var(--color-muted);
    font-size: var(--text-sm);
}
```

Estado inválido:

```css
.field__control[aria-invalid="true"] {
    border-color: var(--color-danger);
}

.field__error {
    color: var(--color-danger);
    font-size: var(--text-sm);
}
```

---

## 25. Dropdown simple

Para menús reales, normalmente se necesita JavaScript para accesibilidad completa. Para un caso simple, puedes partir así:

```html
<div class="dropdown">
    <button class="dropdown__trigger" type="button" aria-expanded="false">
        Opciones
    </button>

    <div class="dropdown__menu">
        <a href="/perfil">Perfil</a>
        <a href="/configuracion">Configuración</a>
        <a href="/salir">Salir</a>
    </div>
</div>
```

```css
.dropdown {
    position: relative;
    display: inline-block;
}

.dropdown__trigger {
    min-height: 2.5rem;
    padding: 0 1rem;
    border: 1px solid var(--color-border);
    border-radius: 0.5rem;
    background: var(--color-surface);
    cursor: pointer;
}

.dropdown__menu {
    position: absolute;
    z-index: 20;
    inset-block-start: calc(100% + 0.5rem);
    inset-inline-end: 0;
    display: none;
    min-width: 12rem;
    padding: 0.5rem;
    border: 1px solid var(--color-border);
    border-radius: 0.75rem;
    background: var(--color-surface);
    box-shadow: 0 1rem 2rem rgb(0 0 0 / 0.12);
}

.dropdown__menu a {
    display: block;
    padding: 0.625rem 0.75rem;
    border-radius: 0.5rem;
    color: var(--color-text);
    text-decoration: none;
}

.dropdown__menu a:hover {
    background: var(--color-surface-hover);
}

.dropdown[data-open="true"] .dropdown__menu {
    display: block;
}
```

JavaScript mínimo:

```javascript
const dropdown = document.querySelector(".dropdown");
const trigger = dropdown.querySelector(".dropdown__trigger");

trigger.addEventListener("click", () => {
    const isOpen = dropdown.dataset.open === "true";

    dropdown.dataset.open = String(!isOpen);
    trigger.setAttribute("aria-expanded", String(!isOpen));
});
```

Buenas prácticas:

```text
Actualiza aria-expanded.
Permite cerrar con Escape.
Permite navegación con teclado si es un menú de aplicación.
No escondas contenido crítico solo en hover.
```

---

## 26. Tooltips

Un tooltip no debe contener información indispensable. Debe ser ayuda secundaria.

```html
<span class="tooltip">
    <button class="icon-button" type="button" aria-describedby="tip-save">
        ?
    </button>
    <span class="tooltip__content" id="tip-save" role="tooltip">
        Guarda los cambios localmente.
    </span>
</span>
```

```css
.tooltip {
    position: relative;
    display: inline-flex;
}

.tooltip__content {
    position: absolute;
    z-index: 30;
    inset-block-end: calc(100% + 0.5rem);
    inset-inline-start: 50%;
    width: max-content;
    max-width: 16rem;
    padding: 0.5rem 0.75rem;
    border-radius: 0.5rem;
    background: var(--color-text);
    color: var(--color-bg);
    font-size: var(--text-sm);
    opacity: 0;
    transform: translateX(-50%) translateY(0.25rem);
    pointer-events: none;
    transition:
        opacity 120ms ease,
        transform 120ms ease;
}

.tooltip:hover .tooltip__content,
.tooltip:focus-within .tooltip__content {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
}
```

---

## 27. Badges y chips

```html
<span class="badge badge--success">
    Activo
</span>
```

```css
.badge {
    display: inline-flex;
    align-items: center;
    min-height: 1.5rem;
    padding-inline: 0.625rem;
    border-radius: 999px;
    font-size: var(--text-xs);
    font-weight: 700;
}

.badge--success {
    background: color-mix(in oklch, var(--color-success), white 85%);
    color: color-mix(in oklch, var(--color-success), black 35%);
}

.badge--warning {
    background: color-mix(in oklch, var(--color-warning), white 85%);
    color: color-mix(in oklch, var(--color-warning), black 35%);
}

.badge--danger {
    background: color-mix(in oklch, var(--color-danger), white 85%);
    color: color-mix(in oklch, var(--color-danger), black 35%);
}
```

---

## 28. Tablas legibles

```css
.table-wrapper {
    overflow-x: auto;
    border: 1px solid var(--color-border);
    border-radius: 0.75rem;
}

.table {
    width: 100%;
    border-collapse: collapse;
    font-size: var(--text-sm);
}

.table th,
.table td {
    padding: 0.75rem 1rem;
    border-bottom: 1px solid var(--color-border);
    text-align: start;
    vertical-align: top;
}

.table th {
    background: var(--color-surface-hover);
    font-weight: 700;
}

.table tr:hover td {
    background: color-mix(in oklch, var(--color-surface-hover), transparent 20%);
}
```

Consejo:

```text
En móvil, permite scroll horizontal antes que romper la tabla.
Para datos complejos, evalúa vista de tarjetas.
```

---

## 29. Contratos de estilos

Un contrato de estilos define qué clases, tokens, variantes y estados puede usar un componente. Sirve para que diseño y desarrollo hablen el mismo idioma.

### 29.1 Contrato mínimo de un botón

```text
Componente: Button

Clases públicas:
- .button
- .button--primary
- .button--secondary
- .button--danger
- .button--sm
- .button--md
- .button--lg

Estados soportados:
- :hover
- :focus-visible
- :active
- :disabled
- [aria-busy="true"]

Tokens usados:
- --button-bg
- --button-color
- --button-border
- --button-radius
- --button-height
```

CSS:

```css
.button {
    --button-bg: var(--color-primary);
    --button-color: var(--color-on-primary);
    --button-border: transparent;
    --button-radius: 0.5rem;
    --button-height: 2.75rem;

    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    min-height: var(--button-height);
    padding-inline: 1rem;
    border: 1px solid var(--button-border);
    border-radius: var(--button-radius);
    background: var(--button-bg);
    color: var(--button-color);
    font-weight: 700;
    text-decoration: none;
    cursor: pointer;
}

.button--secondary {
    --button-bg: var(--color-surface);
    --button-color: var(--color-text);
    --button-border: var(--color-border);
}

.button--danger {
    --button-bg: var(--color-danger);
    --button-color: white;
}

.button--sm {
    --button-height: 2.25rem;
    padding-inline: 0.75rem;
    font-size: var(--text-sm);
}

.button--lg {
    --button-height: 3.25rem;
    padding-inline: 1.25rem;
    font-size: var(--text-lg);
}
```

Ventajas:

```text
Los componentes se personalizan sin romper su estructura.
El equipo sabe qué variantes existen.
Se reducen estilos improvisados.
```

---

## 30. Convenciones de nombres

Elige una convención y mantenla.

### Opción BEM

```css
.card {}
.card__title {}
.card__body {}
.card--featured {}
```

### Opción utilitaria propia

```css
.stack {}
.cluster {}
.container {}
.text-muted {}
```

### Opción híbrida recomendada

```text
Componentes:
.card
.card__title
.card--featured

Patrones de layout:
.container
.stack
.cluster
.grid-auto

Utilidades pequeñas:
.sr-only
.text-muted
.visually-hidden
```

Evita mezclar muchas convenciones sin necesidad.

---

## 31. Utilidades útiles

```css
.visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
}

.text-muted {
    color: var(--color-muted);
}

.flow > * + * {
    margin-block-start: var(--flow-space, 1rem);
}

.full-bleed {
    width: 100vw;
    margin-inline-start: 50%;
    transform: translateX(-50%);
}
```

---

## 32. Animaciones intermedias

Una animación debe aclarar estado, no distraer.

```css
@keyframes fade-in-up {
    from {
        opacity: 0;
        transform: translateY(0.5rem);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.toast {
    animation: fade-in-up 180ms ease-out;
}
```

Respeta reducción de movimiento:

```css
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        scroll-behavior: auto !important;
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```

---

# Parte III: nivel avanzado

## 33. Arquitectura CSS por capas

Cuando un proyecto crece, organiza CSS por responsabilidades.

```text
settings   tokens: colores, spacing, tipografía.
base       reset, body, headings, links.
layout     container, grid, stack, shell.
components botones, cards, forms, navbars.
utilities  helpers pequeños.
overrides  excepciones controladas.
```

Con CSS Cascade Layers:

```css
@layer reset, tokens, base, layout, components, utilities, overrides;

@layer reset {
    *,
    *::before,
    *::after {
        box-sizing: border-box;
    }

    body {
        margin: 0;
    }
}

@layer tokens {
    :root {
        --color-bg: #f8fafc;
        --color-text: #0f172a;
        --space-4: 1rem;
    }
}

@layer base {
    body {
        background: var(--color-bg);
        color: var(--color-text);
    }
}

@layer components {
    .button {
        padding: var(--space-4);
    }
}

@layer utilities {
    .text-muted {
        color: var(--color-muted);
    }
}
```

Las capas ayudan a controlar prioridad sin subir especificidad.

---

## 34. Design tokens

Un token es una decisión de diseño con nombre.

```text
Color:       --color-primary
Spacing:     --space-4
Radius:      --radius-md
Typography:  --font-size-lg
Shadow:      --shadow-sm
Z-index:     --z-dropdown
```

Ejemplo:

```css
:root {
    /* Colors */
    --color-blue-600: #2563eb;
    --color-slate-950: #020617;
    --color-white: #ffffff;

    /* Semantic colors */
    --color-primary: var(--color-blue-600);
    --color-on-primary: var(--color-white);
    --color-text: var(--color-slate-950);

    /* Spacing */
    --space-1: 0.25rem;
    --space-2: 0.5rem;
    --space-4: 1rem;
    --space-8: 2rem;

    /* Radius */
    --radius-sm: 0.375rem;
    --radius-md: 0.5rem;
    --radius-lg: 1rem;

    /* Shadows */
    --shadow-sm: 0 0.25rem 0.75rem rgb(0 0 0 / 0.08);
    --shadow-md: 0 0.75rem 2rem rgb(0 0 0 / 0.12);
}
```

### 34.1 Tokens primitivos vs semánticos

```text
Primitivo: --blue-600: #2563eb;
Semántico: --color-primary: var(--blue-600);
Componente: --button-bg: var(--color-primary);
```

Regla:

```text
Los componentes deben usar tokens semánticos o tokens propios,
no valores hex dispersos.
```

---

## 35. Contratos de estilos avanzados

Un contrato avanzado debe documentar:

```text
1. Nombre del componente.
2. HTML esperado.
3. Clases públicas.
4. Variables CSS permitidas.
5. Variantes.
6. Estados.
7. Slots.
8. Reglas responsive.
9. Accesibilidad.
10. Qué no debe modificarse desde fuera.
```

Ejemplo para `Card`:

```html
<article class="card card--interactive">
    <img class="card__media" src="image.jpg" alt="">
    <div class="card__body">
        <h2 class="card__title">Título</h2>
        <p class="card__description">Descripción.</p>
    </div>
</article>
```

Contrato:

```text
Clases públicas:
- .card
- .card--interactive
- .card--horizontal
- .card__media
- .card__body
- .card__title
- .card__description

Variables públicas:
- --card-padding
- --card-radius
- --card-bg
- --card-border
- --card-shadow

Estados:
- .card--interactive:hover
- .card:focus-within
- [aria-disabled="true"]

Restricción:
- No seleccionar .card > div > h2 desde fuera.
- No depender del orden interno salvo que esté documentado.
```

CSS:

```css
.card {
    --card-padding: var(--space-4);
    --card-radius: var(--radius-lg);
    --card-bg: var(--color-surface);
    --card-border: var(--color-border);
    --card-shadow: none;

    overflow: hidden;
    border: 1px solid var(--card-border);
    border-radius: var(--card-radius);
    background: var(--card-bg);
    box-shadow: var(--card-shadow);
}

.card__body {
    display: grid;
    gap: var(--space-2);
    padding: var(--card-padding);
}

.card--interactive {
    transition:
        transform 160ms ease,
        box-shadow 160ms ease;
}

.card--interactive:hover {
    --card-shadow: var(--shadow-md);

    transform: translateY(-2px);
}

.card--horizontal {
    display: grid;
    grid-template-columns: 12rem 1fr;
}

@media (width < 40rem) {
    .card--horizontal {
        grid-template-columns: 1fr;
    }
}
```

---

## 36. Container queries

Los media queries responden al viewport. Los container queries responden al tamaño del contenedor.

Esto permite que un componente cambie según el espacio real donde vive.

```css
.product-card-wrapper {
    container-type: inline-size;
    container-name: product-card;
}

.product-card {
    display: grid;
    gap: var(--space-4);
}

@container product-card (width >= 36rem) {
    .product-card {
        grid-template-columns: 12rem 1fr;
        align-items: center;
    }
}
```

HTML:

```html
<div class="product-card-wrapper">
    <article class="product-card">
        <img src="cover.jpg" alt="">
        <div>
            <h2>Producto</h2>
            <p>Descripción corta.</p>
        </div>
    </article>
</div>
```

Útil para:

```text
Cards reutilizadas en sidebar y contenido principal.
Widgets embebidos.
Componentes de dashboard.
Listados que cambian según el ancho disponible.
```

---

## 37. `@supports` y mejora progresiva

No todos los navegadores soportan lo mismo al mismo tiempo. Usa `@supports` para habilitar mejoras.

```css
.card {
    background: #ffffff;
}

@supports (background: color-mix(in oklch, white, black)) {
    .card {
        background: color-mix(in oklch, var(--color-surface), var(--color-primary) 4%);
    }
}
```

Ejemplo con container queries:

```css
.grid {
    display: grid;
    grid-template-columns: 1fr;
}

@supports (container-type: inline-size) {
    .grid-wrapper {
        container-type: inline-size;
    }

    @container (width >= 48rem) {
        .grid {
            grid-template-columns: repeat(2, 1fr);
        }
    }
}
```

---

## 38. Z-index controlado

No uses números arbitrarios.

Menos recomendable:

```css
.modal {
    z-index: 999999;
}
```

Mejor:

```css
:root {
    --z-base: 0;
    --z-dropdown: 100;
    --z-sticky: 200;
    --z-overlay: 300;
    --z-modal: 400;
    --z-toast: 500;
}

.dropdown {
    z-index: var(--z-dropdown);
}

.modal {
    z-index: var(--z-modal);
}
```

Regla:

```text
El z-index debe pertenecer a una escala.
No debe resolverse por desesperación.
```

---

## 39. Modales

Los modales requieren HTML, CSS y normalmente JavaScript. Si puedes usar `<dialog>`, úsalo.

```html
<dialog class="modal" id="settings-modal">
    <form method="dialog" class="modal__panel">
        <header class="modal__header">
            <h2>Configuración</h2>
            <button class="icon-button" value="cancel" aria-label="Cerrar">
                ×
            </button>
        </header>

        <div class="modal__body">
            <p>Opciones del sistema.</p>
        </div>

        <footer class="modal__footer">
            <button class="button button--secondary" value="cancel">
                Cancelar
            </button>
            <button class="button" value="confirm">
                Guardar
            </button>
        </footer>
    </form>
</dialog>
```

```css
.modal {
    width: min(100% - 2rem, 36rem);
    padding: 0;
    border: 0;
    border-radius: 1rem;
    background: transparent;
}

.modal::backdrop {
    background: rgb(2 6 23 / 0.64);
    backdrop-filter: blur(4px);
}

.modal__panel {
    overflow: hidden;
    border-radius: 1rem;
    background: var(--color-surface);
    color: var(--color-text);
    box-shadow: var(--shadow-md);
}

.modal__header,
.modal__footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--space-4);
    padding: var(--space-4);
}

.modal__body {
    padding: var(--space-4);
}
```

JavaScript:

```javascript
const modal = document.querySelector("#settings-modal");
const openButton = document.querySelector("[data-open-settings]");

openButton.addEventListener("click", () => {
    modal.showModal();
});
```

---

## 40. Skeleton loaders

```html
<div class="skeleton-card" aria-hidden="true">
    <div class="skeleton skeleton--media"></div>
    <div class="skeleton skeleton--title"></div>
    <div class="skeleton skeleton--line"></div>
    <div class="skeleton skeleton--line skeleton--line-short"></div>
</div>
```

```css
.skeleton-card {
    display: grid;
    gap: var(--space-3);
    padding: var(--space-4);
}

.skeleton {
    border-radius: 0.5rem;
    background:
        linear-gradient(
            90deg,
            var(--color-surface-hover),
            color-mix(in oklch, var(--color-surface-hover), white 20%),
            var(--color-surface-hover)
        );
    background-size: 200% 100%;
    animation: skeleton-loading 1.2s ease-in-out infinite;
}

.skeleton--media {
    aspect-ratio: 16 / 9;
}

.skeleton--title {
    width: 60%;
    height: 1.25rem;
}

.skeleton--line {
    width: 100%;
    height: 0.875rem;
}

.skeleton--line-short {
    width: 75%;
}

@keyframes skeleton-loading {
    from {
        background-position: 200% 0;
    }

    to {
        background-position: -200% 0;
    }
}

@media (prefers-reduced-motion: reduce) {
    .skeleton {
        animation: none;
    }
}
```

---

## 41. Drag and drop

CSS puede diseñar los estados visuales, pero el comportamiento requiere HTML Drag and Drop API o una librería.

### 41.1 Drag and drop de archivos

HTML:

```html
<label class="dropzone" for="file-input" data-dragging="false">
    <input class="dropzone__input" id="file-input" type="file" multiple>
    <span class="dropzone__title">Arrastra archivos aquí</span>
    <span class="dropzone__hint">o haz clic para seleccionarlos</span>
</label>
```

CSS:

```css
.dropzone {
    display: grid;
    place-items: center;
    gap: var(--space-2);
    min-height: 12rem;
    padding: var(--space-6);
    border: 2px dashed var(--color-border);
    border-radius: 1rem;
    background: var(--color-surface);
    color: var(--color-muted);
    text-align: center;
    cursor: pointer;
    transition:
        border-color 160ms ease,
        background-color 160ms ease,
        transform 160ms ease;
}

.dropzone:hover {
    border-color: var(--color-primary);
    background: color-mix(in oklch, var(--color-primary), white 94%);
}

.dropzone[data-dragging="true"] {
    border-color: var(--color-primary);
    background: color-mix(in oklch, var(--color-primary), white 88%);
    color: var(--color-text);
    transform: scale(1.01);
}

.dropzone__input {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
}

.dropzone__title {
    font-weight: 800;
    color: var(--color-text);
}

.dropzone__hint {
    font-size: var(--text-sm);
}
```

JavaScript:

```javascript
const dropzone = document.querySelector(".dropzone");
const input = document.querySelector(".dropzone__input");

for (const eventName of ["dragenter", "dragover"]) {
    dropzone.addEventListener(eventName, (event) => {
        event.preventDefault();
        dropzone.dataset.dragging = "true";
    });
}

for (const eventName of ["dragleave", "drop"]) {
    dropzone.addEventListener(eventName, (event) => {
        event.preventDefault();
        dropzone.dataset.dragging = "false";
    });
}

dropzone.addEventListener("drop", (event) => {
    const files = [...event.dataTransfer.files];

    console.log(files);
});
```

Buenas prácticas:

```text
Previene el comportamiento por defecto en dragover y drop.
Muestra claramente el estado de arrastre.
Permite alternativa por click con input file.
Valida tipo y tamaño de archivo en frontend y backend.
```

### 41.2 Reordenar tarjetas con drag and drop

HTML:

```html
<ul class="kanban-list">
    <li class="kanban-card" draggable="true">Tarea 1</li>
    <li class="kanban-card" draggable="true">Tarea 2</li>
    <li class="kanban-card" draggable="true">Tarea 3</li>
</ul>
```

CSS:

```css
.kanban-list {
    display: grid;
    gap: var(--space-3);
    min-height: 12rem;
    padding: var(--space-3);
    border: 1px solid var(--color-border);
    border-radius: 1rem;
    background: var(--color-surface-hover);
    list-style: none;
}

.kanban-card {
    padding: var(--space-4);
    border: 1px solid var(--color-border);
    border-radius: 0.75rem;
    background: var(--color-surface);
    box-shadow: var(--shadow-sm);
    cursor: grab;
    user-select: none;
}

.kanban-card:active {
    cursor: grabbing;
}

.kanban-card[data-dragging="true"] {
    opacity: 0.5;
    transform: rotate(1deg);
}

.kanban-list[data-drag-over="true"] {
    outline: 3px dashed var(--color-primary);
    outline-offset: 0.25rem;
}
```

JavaScript mínimo:

```javascript
const list = document.querySelector(".kanban-list");

list.addEventListener("dragstart", (event) => {
    const card = event.target.closest(".kanban-card");

    if (!card) {
        return;
    }

    card.dataset.dragging = "true";
    event.dataTransfer.effectAllowed = "move";
});

list.addEventListener("dragend", (event) => {
    const card = event.target.closest(".kanban-card");

    if (!card) {
        return;
    }

    card.dataset.dragging = "false";
    list.dataset.dragOver = "false";
});

list.addEventListener("dragover", (event) => {
    event.preventDefault();
    list.dataset.dragOver = "true";

    const draggingCard = list.querySelector('[data-dragging="true"]');
    const targetCard = event.target.closest(".kanban-card");

    if (!draggingCard || !targetCard || draggingCard === targetCard) {
        return;
    }

    const targetBox = targetCard.getBoundingClientRect();
    const shouldInsertAfter = event.clientY > targetBox.top + targetBox.height / 2;

    list.insertBefore(
        draggingCard,
        shouldInsertAfter ? targetCard.nextSibling : targetCard
    );
});

list.addEventListener("dragleave", () => {
    list.dataset.dragOver = "false";
});

list.addEventListener("drop", (event) => {
    event.preventDefault();
    list.dataset.dragOver = "false";
});
```

Advertencia:

```text
El drag and drop nativo puede requerir trabajo adicional para accesibilidad con teclado.
Para tableros complejos, considera una librería que soporte teclado, anuncios ARIA y touch.
```

---

## 42. Estilos para estados de aplicación

No diseñes solo componentes aislados. Diseña estados:

```text
Cargando.
Vacío.
Error.
Sin permisos.
Sin conexión.
Datos parciales.
Confirmación.
```

Ejemplo de empty state:

```html
<section class="empty-state">
    <div class="empty-state__icon" aria-hidden="true">📄</div>
    <h2>No hay documentos</h2>
    <p>Cuando subas un documento, aparecerá en esta lista.</p>
    <button class="button" type="button">Subir documento</button>
</section>
```

```css
.empty-state {
    display: grid;
    justify-items: center;
    gap: var(--space-3);
    padding: clamp(3rem, 8vw, 6rem);
    border: 1px dashed var(--color-border);
    border-radius: 1rem;
    background: var(--color-surface);
    text-align: center;
}

.empty-state__icon {
    display: grid;
    place-items: center;
    width: 4rem;
    height: 4rem;
    border-radius: 999px;
    background: var(--color-surface-hover);
    font-size: 2rem;
}
```

---

## 43. Performance CSS

Buenas prácticas:

```text
Evita selectores extremadamente complejos.
Evita animar width, height, top, left cuando puedas animar transform u opacity.
Usa imágenes con tamaño adecuado.
Evita sombras gigantes en listas extensas.
No cargues fuentes innecesarias.
Divide CSS crítico y CSS diferido si el sitio es grande.
```

Mejor animar:

```css
.panel {
    transform: translateY(0);
    opacity: 1;
}
```

Menos recomendable:

```css
.panel {
    top: 120px;
    height: 400px;
}
```

Para contenido pesado fuera de pantalla:

```css
.long-section {
    content-visibility: auto;
    contain-intrinsic-size: 600px;
}
```

Úsalo con criterio y prueba que no afecte accesibilidad, búsqueda interna o mediciones de layout.

---

## 44. Accesibilidad visual

Revisa:

```text
Contraste suficiente.
Focus visible.
Tamaño de click razonable.
Texto que no dependa de color.
Estados hover equivalentes para teclado.
Animaciones reducibles.
Layout usable con zoom.
```

Ejemplo de área clickeable:

```css
.icon-button {
    display: inline-grid;
    place-items: center;
    width: 2.75rem;
    height: 2.75rem;
    border: 0;
    border-radius: 999px;
    background: transparent;
    color: var(--color-text);
    cursor: pointer;
}

.icon-button:hover {
    background: var(--color-surface-hover);
}

.icon-button:focus-visible {
    outline: 3px solid var(--color-focus);
    outline-offset: 3px;
}
```

---

## 45. CSS para impresión

Si el sitio tiene reportes, tickets, órdenes o documentos, diseña impresión.

```css
@media print {
    body {
        background: white;
        color: black;
    }

    .site-header,
    .site-footer,
    .button,
    .no-print {
        display: none !important;
    }

    .content {
        width: auto;
        margin: 0;
        padding: 0;
    }

    a {
        color: black;
        text-decoration: underline;
    }

    a[href]::after {
        content: " (" attr(href) ")";
        font-size: 0.85em;
    }
}
```

---

## 46. Organización de archivos

Para proyecto pequeño:

```text
styles/
    main.css
```

Para proyecto mediano:

```text
styles/
    main.css
    tokens.css
    base.css
    layout.css
    components.css
    utilities.css
```

Para proyecto grande:

```text
styles/
    00-settings/
        tokens.css
    01-reset/
        reset.css
    02-base/
        typography.css
        links.css
    03-layout/
        container.css
        grid.css
        shell.css
    04-components/
        button.css
        card.css
        form.css
        modal.css
    05-utilities/
        visibility.css
        spacing.css
    06-overrides/
        vendor.css
```

Ejemplo de `main.css`:

```css
@import "./00-settings/tokens.css";
@import "./01-reset/reset.css";
@import "./02-base/typography.css";
@import "./03-layout/container.css";
@import "./04-components/button.css";
@import "./04-components/card.css";
@import "./05-utilities/visibility.css";
```

En proyectos con build tools, usa bundling/minificación según stack.

---

## 47. Linting y calidad

Usa herramientas para detectar:

```text
Propiedades duplicadas.
Selectores inválidos.
Colores fuera del sistema.
Especificidad excesiva.
Uso accidental de !important.
Orden inconsistente.
```

Ejemplo conceptual de reglas:

```json
{
    "rules": {
        "declaration-no-important": true,
        "selector-max-id": 0,
        "selector-max-compound-selectors": 4,
        "color-named": "never"
    }
}
```

Regla de equipo:

```text
Si un color, sombra, radio o espaciado no existe como token,
debe justificarse o agregarse al sistema.
```

---

## 48. Checklist de buenas prácticas CSS

```text
[ ] El sitio usa box-sizing: border-box.
[ ] Hay tokens para colores, spacing, tipografía, radius y sombras.
[ ] Los colores son semánticos.
[ ] El contraste fue revisado.
[ ] El foco visible existe.
[ ] El diseño funciona con teclado.
[ ] Los layouts usan Flexbox/Grid con criterio.
[ ] La responsividad se define por ruptura del diseño, no por dispositivos.
[ ] Se usa clamp(), min() o max() donde corresponde.
[ ] Las animaciones respetan prefers-reduced-motion.
[ ] No hay !important innecesarios.
[ ] Los z-index pertenecen a una escala.
[ ] Los componentes tienen contratos de clases, variantes y estados.
[ ] Los dropdowns/modales no dependen solo de hover.
[ ] Hay estilos para estados vacíos, error y carga.
[ ] El CSS está dividido por responsabilidad.
```

---

# Parte IV: ejemplo integrado

## 49. Base CSS inicial para un proyecto

```css
@layer reset, tokens, base, layout, components, utilities;

@layer reset {
    *,
    *::before,
    *::after {
        box-sizing: border-box;
    }

    body,
    h1,
    h2,
    h3,
    p {
        margin: 0;
    }

    img,
    svg,
    video {
        display: block;
        max-width: 100%;
    }

    button,
    input,
    textarea,
    select {
        font: inherit;
    }
}

@layer tokens {
    :root {
        color-scheme: light;

        --font-sans: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;

        --color-bg: #f8fafc;
        --color-surface: #ffffff;
        --color-surface-hover: #f1f5f9;
        --color-text: #0f172a;
        --color-muted: #475569;
        --color-border: #cbd5e1;
        --color-primary: #2563eb;
        --color-primary-hover: #1d4ed8;
        --color-on-primary: #ffffff;
        --color-danger: #dc2626;
        --color-focus: #f59e0b;

        --space-1: 0.25rem;
        --space-2: 0.5rem;
        --space-3: 0.75rem;
        --space-4: 1rem;
        --space-6: 1.5rem;
        --space-8: 2rem;
        --space-12: 3rem;

        --text-sm: 0.875rem;
        --text-md: 1rem;
        --text-lg: 1.125rem;
        --text-xl: 1.5rem;
        --text-2xl: clamp(2rem, 1.2rem + 3vw, 4rem);

        --radius-md: 0.5rem;
        --radius-lg: 1rem;

        --shadow-sm: 0 0.25rem 0.75rem rgb(0 0 0 / 0.08);
        --shadow-md: 0 0.75rem 2rem rgb(0 0 0 / 0.12);

        --z-dropdown: 100;
        --z-modal: 400;
        --z-toast: 500;
    }
}

@layer base {
    html {
        line-height: 1.5;
        -webkit-text-size-adjust: 100%;
    }

    body {
        min-height: 100dvh;
        font-family: var(--font-sans);
        background: var(--color-bg);
        color: var(--color-text);
    }

    a {
        color: var(--color-primary);
        text-decoration-thickness: 0.08em;
        text-underline-offset: 0.18em;
    }

    :focus-visible {
        outline: 3px solid var(--color-focus);
        outline-offset: 3px;
    }
}

@layer layout {
    .container {
        width: min(100% - 2rem, 72rem);
        margin-inline: auto;
    }

    .section {
        padding-block: clamp(3rem, 6vw, 6rem);
    }

    .stack {
        display: grid;
        gap: var(--stack-gap, var(--space-4));
    }

    .grid-auto {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
        gap: var(--space-6);
    }
}

@layer components {
    .button {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
        min-height: 2.75rem;
        padding-inline: 1rem;
        border: 0;
        border-radius: var(--radius-md);
        background: var(--color-primary);
        color: var(--color-on-primary);
        font-weight: 700;
        text-decoration: none;
        cursor: pointer;
        transition:
            background-color 160ms ease,
            transform 160ms ease,
            box-shadow 160ms ease;
    }

    .button:hover {
        background: var(--color-primary-hover);
        transform: translateY(-1px);
        box-shadow: var(--shadow-sm);
    }

    .button:active {
        transform: translateY(0);
        box-shadow: none;
    }

    .button:disabled {
        opacity: 0.55;
        cursor: not-allowed;
    }

    .card {
        overflow: hidden;
        border: 1px solid var(--color-border);
        border-radius: var(--radius-lg);
        background: var(--color-surface);
        box-shadow: var(--shadow-sm);
    }

    .card__body {
        display: grid;
        gap: var(--space-3);
        padding: var(--space-4);
    }

    .card__title {
        font-size: var(--text-lg);
        line-height: 1.2;
    }
}

@layer utilities {
    .text-muted {
        color: var(--color-muted);
    }

    .visually-hidden {
        position: absolute;
        width: 1px;
        height: 1px;
        overflow: hidden;
        clip-path: inset(50%);
        white-space: nowrap;
    }
}

@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        scroll-behavior: auto !important;
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```

---

## 50. Resumen de reglas principales

```text
1. Diseña con tokens, no con valores sueltos.
2. Usa nombres semánticos para colores y roles.
3. Prefiere clases reutilizables sobre selectores largos.
4. Controla especificidad antes de necesitar !important.
5. Usa Flexbox para una dimensión y Grid para dos.
6. Usa clamp(), min() y max() para tamaños fluidos.
7. Define breakpoints cuando el diseño se rompe.
8. Crea contratos de estilo para componentes reutilizables.
9. Diseña estados: hover, focus, active, disabled, loading, empty y error.
10. Respeta accesibilidad: contraste, foco visible, teclado y reducción de movimiento.
11. Usa capas de cascada y estructura de archivos cuando el proyecto crezca.
12. Para interacciones complejas, CSS diseña estados; JavaScript gestiona comportamiento.
```

---

# Fuentes de referencia recomendadas

```text
- MDN Web Docs: CSS Guides.
- MDN Web Docs: CSS Box Model.
- MDN Web Docs: Flexbox.
- MDN Web Docs: CSS Grid Layout.
- MDN Web Docs: @media.
- MDN Web Docs: Container Queries.
- MDN Web Docs: CSS Custom Properties.
- MDN Web Docs: @layer.
- MDN Web Docs: CSS Colors, oklch() y color-mix().
- MDN Web Docs: prefers-reduced-motion.
- MDN Web Docs: HTML Drag and Drop API.
- W3C WCAG 2.2: contrast minimum.
- W3C Design Tokens Community Group.
```
