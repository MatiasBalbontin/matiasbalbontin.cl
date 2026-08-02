# Rediseño Interacciones — Estilo Daniel Stoopendaal

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rediseñar `index.html` con tipografías Fraunces + Space Grotesk + Space Mono, hero de tipografía total con GSAP, cursor personalizado, servicios con hover expansivo, y ScrollTrigger en todas las secciones, manteniendo la paleta cálida y el toggle ES/EN.

**Architecture:** Todo vive en un único `index.html` (HTML + CSS inline + JS inline). GSAP y ScrollTrigger se cargan vía CDN. El HTML de Hero y Servicios se reconstruye; el resto de las secciones solo recibe JS de animación encima.

**Tech Stack:** HTML5, CSS3 (variables + clip-path + grid), GSAP 3 + ScrollTrigger (CDN), Google Fonts CDN (Fraunces, Space Grotesk, Space Mono). Sin build tools.

## Global Constraints

- Archivo único: `index.html` — no crear archivos CSS o JS separados
- Paleta intacta: `#F7F5F2` (bg), `#1A3C5E` (navy), `#C4622D` (terra), `#6B6560` (muted), `#DDDAD5` (border)
- Toggle ES/EN (`toggleLang()`) debe seguir funcionando en todos los elementos nuevos — todo texto bilingüe lleva `data-es` y `data-en`
- GSAP CDN: `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js` y `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js`
- Google Fonts: Fraunces `ital,opsz,wght@0,9..144,300..900;1,9..144,300..900`, Space Grotesk `wght@300;400;500;700`, Space Mono `wght@400;700`
- Mobile breakpoint: `max-width: 768px`
- `cursor: none` en body para desktop; cursor visible en touch (`@media (hover: none)`)

---

### Task 1: GSAP CDN + Swap de tipografías

**Files:**
- Modify: `index.html` — `<head>` (líneas 16–18 Google Fonts, líneas 36–39 variables CSS)

**Interfaces:**
- Produce: variables `--font-display`, `--font-body`, `--font-mono` actualizadas; scripts GSAP disponibles globalmente como `gsap` y `ScrollTrigger`

- [ ] **Step 1: Reemplazar el bloque de Google Fonts**

Busca las líneas con `Cormorant+Garamond` y `Outfit` y `JetBrains+Mono` y reemplaza el `<link>` de Google Fonts por:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,300..900;1,9..144,300..900&family=Space+Grotesk:wght@300;400;500;700&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
```

- [ ] **Step 2: Agregar scripts de GSAP justo antes de `</head>`**

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
```

- [ ] **Step 3: Actualizar las tres variables de fuente en `:root`**

Busca el bloque `--font-display` / `--font-body` / `--font-mono` y reemplaza:

```css
--font-display: 'Fraunces', Georgia, serif;
--font-body:    'Space Grotesk', sans-serif;
--font-mono:    'Space Mono', monospace;
```

- [ ] **Step 4: Registrar ScrollTrigger en el bloque `<script>` principal**

Al inicio del bloque `<script>` (antes de cualquier función), agrega:

```javascript
gsap.registerPlugin(ScrollTrigger);
```

- [ ] **Step 5: Verificar en el navegador**

Abre `index.html` en el navegador. Verifica:
- El título "Matías Balbontín" usa Fraunces (serif con personalidad, no Cormorant)
- El cuerpo usa Space Grotesk (grotesca limpia)
- Las etiquetas monoespacio usan Space Mono
- No hay errores en consola sobre GSAP

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: swap fonts to Fraunces/Space Grotesk/Space Mono, add GSAP CDN"
```

---

### Task 2: Cursor personalizado

**Files:**
- Modify: `index.html` — agregar CSS al bloque `<style>`, agregar `<div id="cursor">` al `<body>`, agregar función `initCursor()` al bloque `<script>`

**Interfaces:**
- Produce: `initCursor()` — función a llamar desde `DOMContentLoaded`
- Consume: `gsap` global (Task 1)

- [ ] **Step 1: Agregar CSS del cursor al final del bloque `<style>` (antes del cierre `</style>`)**

```css
/* ============================================================
   CURSOR PERSONALIZADO
   ============================================================ */
body { cursor: none; }

#cursor {
    position: fixed;
    top: 0; left: 0;
    width: 22px; height: 22px;
    border-radius: 50%;
    border: 1.5px solid var(--color-navy);
    background: transparent;
    pointer-events: none;
    z-index: 9999;
    transform: translate(-50%, -50%);
    transition: width 0.25s ease, height 0.25s ease,
                background 0.25s ease, border-color 0.25s ease,
                opacity 0.25s ease;
    mix-blend-mode: normal;
}
#cursor.is-hovering {
    width: 48px; height: 48px;
    background: var(--color-navy);
    border-color: var(--color-navy);
    mix-blend-mode: difference;
}
#cursor.is-hovering-hero-name {
    width: 90px; height: 90px;
    background: rgba(26, 60, 94, 0.15);
    border: none;
}
#cursor.is-clicking {
    transform: translate(-50%, -50%) scale(0.82);
}
@media (hover: none) {
    #cursor { display: none; }
    body { cursor: auto; }
}
```

- [ ] **Step 2: Agregar el elemento del cursor al inicio del `<body>` (antes de `<nav>`)**

```html
<div id="cursor" aria-hidden="true"></div>
```

- [ ] **Step 3: Agregar la función `initCursor()` al bloque `<script>` (antes de `DOMContentLoaded`)**

```javascript
function initCursor() {
    const cursor = document.getElementById('cursor');
    if (!cursor || window.matchMedia('(hover: none)').matches) return;

    let mouseX = window.innerWidth / 2;
    let mouseY = window.innerHeight / 2;
    let curX = mouseX;
    let curY = mouseY;
    const LERP = 0.12;

    window.addEventListener('mousemove', e => {
        mouseX = e.clientX;
        mouseY = e.clientY;
    });

    gsap.ticker.add(() => {
        curX += (mouseX - curX) * LERP;
        curY += (mouseY - curY) * LERP;
        gsap.set(cursor, { x: curX, y: curY });
    });

    // Hover sobre links y botones
    document.querySelectorAll('a, button').forEach(el => {
        el.addEventListener('mouseenter', () => cursor.classList.add('is-hovering'));
        el.addEventListener('mouseleave', () => cursor.classList.remove('is-hovering'));
    });

    // Hover sobre el nombre del hero (se añade después de que el Hero exista)
    const heroName = document.querySelector('.hero-name');
    if (heroName) {
        heroName.addEventListener('mouseenter', () => {
            cursor.classList.remove('is-hovering');
            cursor.classList.add('is-hovering-hero-name');
        });
        heroName.addEventListener('mouseleave', () => cursor.classList.remove('is-hovering-hero-name'));
    }

    // Click feedback
    document.addEventListener('mousedown', () => cursor.classList.add('is-clicking'));
    document.addEventListener('mouseup', () => cursor.classList.remove('is-clicking'));
}
```

- [ ] **Step 4: Llamar `initCursor()` dentro de `DOMContentLoaded`**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // ... código existente ...
    initCursor();
});
```

- [ ] **Step 5: Verificar en el navegador**

- El cursor nativo desaparece y aparece el círculo navy
- Al pasar sobre un link/botón: se expande a 48px y rellena navy
- Al pasar sobre el hero name: se expande a 90px con baja opacidad
- Al hacer click: escala rápida a 0.82 y vuelve
- En DevTools → emulación touch: cursor nativo vuelve, `#cursor` oculto

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add custom cursor with lerp tracking and hover states"
```

---

### Task 3: Hero — Reconstrucción completa

**Files:**
- Modify: `index.html` — CSS del `#hero` (reemplazar), HTML de `<section id="hero">` (reemplazar), agregar `initHeroAnimation()` al `<script>`

**Interfaces:**
- Produce: `initHeroAnimation()` — función a llamar desde `DOMContentLoaded`
- Consume: `gsap` global (Task 1); elementos `.hero-name`, `.hero-name-line-1`, `.hero-name-line-2`, `.hero-roles`, `.hero-cta`, `.scroll-indicator`

- [ ] **Step 1: Reemplazar TODO el bloque CSS de `#hero` y clases hero relacionadas**

Busca el bloque que comienza con `/* HERO */` (o `#hero {`) y reemplaza hasta el final de las clases hero (`.hero-img-wrap`, `.hero-img-placeholder`) por:

```css
/* ============================================================
   HERO — Tipografía total
   ============================================================ */
#hero {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    max-width: var(--max-width);
    margin: 0 auto;
    padding: 9rem 2rem 5rem;
    position: relative;
}

.hero-name {
    font-family: var(--font-display);
    font-size: clamp(4.5rem, 12vw, 11rem);
    line-height: 0.95;
    letter-spacing: -0.02em;
    margin-bottom: 2.5rem;
}

.hero-name-line-1 {
    display: block;
    font-weight: 300;
    color: var(--color-text);
    overflow: hidden;
}

.hero-name-line-2 {
    display: block;
    font-weight: 700;
    color: var(--color-navy);
    overflow: hidden;
}

.hero-name-inner {
    display: block;
}

.hero-roles {
    display: flex;
    flex-wrap: wrap;
    gap: 0 1.5rem;
    margin-bottom: 3rem;
    opacity: 0;
}

.role-item {
    font-family: var(--font-mono);
    font-size: 0.78rem;
    letter-spacing: 0.12em;
    color: var(--color-muted);
    text-transform: uppercase;
    display: flex;
    align-items: center;
    gap: 0.5rem;
}
.role-item::before {
    content: '—';
    color: var(--color-terra);
}

.hero-cta {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
    opacity: 0;
}

.scroll-indicator {
    position: absolute;
    bottom: 2.5rem;
    right: 2rem;
    font-family: var(--font-mono);
    font-size: 0.7rem;
    letter-spacing: 0.15em;
    color: var(--color-muted);
    text-transform: uppercase;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.4rem;
    opacity: 0;
}
.scroll-indicator::after {
    content: '';
    display: block;
    width: 1px;
    height: 36px;
    background: var(--color-muted);
}
```

- [ ] **Step 2: Reemplazar el HTML de `<section id="hero">` completo**

```html
<section id="hero">

    <h1 class="hero-name">
        <span class="hero-name-line-1">
            <span class="hero-name-inner">Matías</span>
        </span>
        <span class="hero-name-line-2">
            <span class="hero-name-inner">Balbontín</span>
        </span>
    </h1>

    <div class="hero-roles">
        <span class="role-item" data-es="Contador Auditor" data-en="CPA &amp; Auditor">Contador Auditor</span>
        <span class="role-item" data-es="Gestor de Proyectos" data-en="Project Manager">Gestor de Proyectos</span>
        <span class="role-item" data-es="Asesor Directivo" data-en="Strategic Advisor">Asesor Directivo</span>
        <span class="role-item" data-es="Ing. Informática (cursando)" data-en="Computer Eng. (in progress)">Ing. Informática (cursando)</span>
    </div>

    <div class="hero-cta">
        <a href="#contacto" class="btn-primary"
           data-es="Conversemos"
           data-en="Let's Talk">Conversemos</a>
        <a href="#experiencia" class="btn-secondary"
           data-es="Ver trayectoria"
           data-en="My Journey">Ver trayectoria</a>
    </div>

    <div class="scroll-indicator" aria-hidden="true">scroll</div>

</section>
```

- [ ] **Step 3: Agregar `initHeroAnimation()` al bloque `<script>`**

```javascript
function initHeroAnimation() {
    const tl = gsap.timeline({ defaults: { ease: 'power3.out' } });

    tl.fromTo('.hero-name-line-1 .hero-name-inner',
        { y: '105%' },
        { y: '0%', duration: 0.85 }
    )
    .fromTo('.hero-name-line-2 .hero-name-inner',
        { y: '105%' },
        { y: '0%', duration: 0.85 },
        '-=0.65'
    )
    .to('.hero-roles',
        { autoAlpha: 1, y: 0, duration: 0.5 },
        '-=0.25'
    )
    .fromTo('.hero-cta a',
        { y: 16, autoAlpha: 0 },
        { y: 0, autoAlpha: 1, stagger: 0.1, duration: 0.45 },
        '-=0.2'
    )
    .to('.scroll-indicator',
        { autoAlpha: 1, duration: 0.4 },
        '-=0.1'
    );

    // Bounce loop del scroll indicator
    gsap.to('.scroll-indicator', {
        y: 6, repeat: -1, yoyo: true,
        duration: 0.9, ease: 'power1.inOut',
        delay: 1.8
    });
}
```

- [ ] **Step 4: Llamar `initHeroAnimation()` en `DOMContentLoaded`**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    document.getElementById('footer-year').textContent = new Date().getFullYear();
    initHeroAnimation();
    initCursor();
    // initScrollFade(); ← comentar o eliminar esta línea
});
```

- [ ] **Step 5: Verificar en el navegador**

- Al cargar: pantalla en negro/invisible un instante, luego "Matías" sube desde abajo, luego "Balbontín", luego roles, luego botones
- El nombre ocupa ~80% del ancho en desktop
- El scroll indicator aparece al final y hace bounce suave
- El toggle ES/EN cambia los textos de `.role-item` y `.hero-cta a`

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: rebuild hero with full-width typography and GSAP entrance animation"
```

---

### Task 4: Servicios — Filas con hover GSAP

**Files:**
- Modify: `index.html` — CSS de `#servicios` y `.service-*` (reemplazar), HTML de `<section id="servicios">` (reemplazar bloque interno), agregar `initServiciosHover()` al `<script>`

**Interfaces:**
- Produce: `initServiciosHover()` — función a llamar desde `DOMContentLoaded`
- Consume: `gsap` global (Task 1); elementos `.service-row`, `.service-num`, `.service-title`, `.service-desc`, `.service-arrow`

- [ ] **Step 1: Reemplazar el CSS de `#servicios` y `.service-*`**

Busca el bloque que empieza con `#servicios { padding:` y reemplaza hasta el final de `.service-desc` por:

```css
/* ============================================================
   SERVICIOS — Filas horizontales
   ============================================================ */
#servicios { padding: var(--pad-section); }

.services-list {
    border-top: 1px solid var(--color-border);
}

.service-row {
    display: grid;
    grid-template-columns: 60px 1fr auto 32px;
    align-items: center;
    gap: 2rem;
    padding: 0 1rem;
    height: 80px;
    overflow: hidden;
    border-bottom: 1px solid var(--color-border);
    cursor: pointer;
    position: relative;
    transition: background 0.4s ease;
}

.service-num {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    letter-spacing: 0.18em;
    color: var(--color-terra);
    transition: color 0.3s;
    flex-shrink: 0;
}

.service-title {
    font-family: var(--font-display);
    font-size: clamp(1.2rem, 2.5vw, 1.75rem);
    font-weight: 600;
    color: var(--color-text);
    line-height: 1.1;
    transition: color 0.3s;
}

.service-desc {
    font-family: var(--font-body);
    font-size: 0.88rem;
    color: var(--color-muted);
    line-height: 1.65;
    grid-column: 2;
    grid-row: 2;
    padding-bottom: 1.25rem;
    visibility: hidden;
    opacity: 0;
    transition: color 0.3s;
}

.service-arrow {
    font-size: 1.3rem;
    color: var(--color-terra);
    transition: color 0.3s, transform 0.3s;
    display: inline-block;
    flex-shrink: 0;
}
```

- [ ] **Step 2: Reemplazar el HTML interno de `<section id="servicios">`**

Mantener `<section id="servicios">` y `<div class="container">` pero reemplazar el contenido interior:

```html
<p class="section-label" data-es="Lo que ofrezco" data-en="What I offer">Lo que ofrezco</p>
<h2 class="section-title" data-es="Servicios" data-en="Services">Servicios</h2>

<div class="services-list">

    <div class="service-row">
        <span class="service-num">01</span>
        <h3 class="service-title"
            data-es="Asesoría Financiera y Contable"
            data-en="Financial &amp; Accounting Advisory">Asesoría Financiera y Contable</h3>
        <p class="service-desc"
           data-es="Diagnóstico financiero, mitigación de riesgos y profesionalización de PYMEs. Auditoría de procesos contables y tributarios bajo normativa vigente."
           data-en="Financial diagnosis, risk mitigation, and PYME professionalization. Audit of accounting and tax processes under current regulations.">
            Diagnóstico financiero, mitigación de riesgos y profesionalización de PYMEs. Auditoría de procesos contables y tributarios bajo normativa vigente.
        </p>
        <span class="service-arrow" aria-hidden="true">→</span>
    </div>

    <div class="service-row">
        <span class="service-num">02</span>
        <h3 class="service-title"
            data-es="Gestión y Dirección de Proyectos"
            data-en="Project Management &amp; Direction">Gestión y Dirección de Proyectos</h3>
        <p class="service-desc"
           data-es="Evaluación de oportunidades de negocio, gestión de cartera de clientes y verificación de protocolos internos. Metodologías ágiles aplicadas."
           data-en="Business opportunity assessment, client portfolio management, and internal protocol verification. Applied agile methodologies.">
            Evaluación de oportunidades de negocio, gestión de cartera de clientes y verificación de protocolos internos. Metodologías ágiles aplicadas.
        </p>
        <span class="service-arrow" aria-hidden="true">→</span>
    </div>

    <div class="service-row">
        <span class="service-num">03</span>
        <h3 class="service-title"
            data-es="Transformación Digital e IA"
            data-en="Digital Transformation &amp; AI">Transformación Digital e IA</h3>
        <p class="service-desc"
           data-es="Análisis de datos con Power BI, automatización de procesos y uso estratégico de inteligencia artificial. Integración de plataformas cloud (Azure IoT)."
           data-en="Data analysis with Power BI, process automation, and strategic use of AI. Cloud platform integration (Azure IoT).">
            Análisis de datos con Power BI, automatización de procesos y uso estratégico de inteligencia artificial. Integración de plataformas cloud (Azure IoT).
        </p>
        <span class="service-arrow" aria-hidden="true">→</span>
    </div>

</div>
```

- [ ] **Step 3: Agregar `initServiciosHover()` al bloque `<script>`**

```javascript
function initServiciosHover() {
    const rows = document.querySelectorAll('.service-row');
    const EXPANDED_HEIGHT = 200;
    const COLLAPSED_HEIGHT = 80;

    rows.forEach(row => {
        const num   = row.querySelector('.service-num');
        const title = row.querySelector('.service-title');
        const desc  = row.querySelector('.service-desc');
        const arrow = row.querySelector('.service-arrow');

        row.addEventListener('mouseenter', () => {
            // Expandir esta fila
            gsap.to(row, {
                height: EXPANDED_HEIGHT,
                backgroundColor: '#1A3C5E',
                duration: 0.4, ease: 'power2.out'
            });
            gsap.to([num, title, arrow], { color: '#FFFFFF', duration: 0.3 });
            gsap.to(desc, { autoAlpha: 1, y: 0, duration: 0.35, ease: 'power2.out' });
            gsap.set(desc, { visibility: 'visible' });
            gsap.to(arrow, { rotation: 45, duration: 0.3, ease: 'power2.out' });

            // Atenuar otras filas
            rows.forEach(other => {
                if (other !== row) gsap.to(other, { opacity: 0.4, duration: 0.3 });
            });
        });

        row.addEventListener('mouseleave', () => {
            gsap.to(row, {
                height: COLLAPSED_HEIGHT,
                backgroundColor: 'transparent',
                duration: 0.4, ease: 'power2.out'
            });
            gsap.to([num, title], { color: '', duration: 0.3 });
            gsap.to(num, { color: '#C4622D', duration: 0.3 });
            gsap.to(arrow, { color: '#C4622D', rotation: 0, duration: 0.3 });
            gsap.to(desc, { autoAlpha: 0, duration: 0.2 });

            rows.forEach(other => gsap.to(other, { opacity: 1, duration: 0.3 }));
        });
    });

    // Animación de entrada de las filas al cargar (vía ScrollTrigger)
    gsap.from('.service-row', {
        y: 30, autoAlpha: 0, stagger: 0.1, duration: 0.6, ease: 'power2.out',
        scrollTrigger: {
            trigger: '.services-list',
            start: 'top 82%',
            once: true
        }
    });
}
```

- [ ] **Step 4: Llamar `initServiciosHover()` en `DOMContentLoaded`**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    document.getElementById('footer-year').textContent = new Date().getFullYear();
    initHeroAnimation();
    initCursor();
    initServiciosHover();
});
```

- [ ] **Step 5: Verificar en el navegador**

- Las 3 filas aparecen apiladas con borde inferior, altura ~80px
- Al hacer hover en fila 01: se expande a ~200px, fondo navy, texto blanco, descripción visible, flecha rota 45°, otras filas al 40% opacidad
- Al salir: todo revierte suavemente
- Al hacer scroll hasta Servicios: las filas entran con stagger desde abajo

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: rebuild servicios as expandable rows with GSAP hover"
```

---

### Task 5: ScrollTrigger en secciones existentes

**Files:**
- Modify: `index.html` — eliminar CSS de `.fade-in` / `.fade-in.visible`, eliminar `initScrollFade()`, agregar `initScrollAnimations()` al `<script>`

**Interfaces:**
- Produce: `initScrollAnimations()` — función a llamar desde `DOMContentLoaded`
- Consume: `gsap` + `ScrollTrigger` globales (Task 1); `.section-title`, `.about-body`, `.about-number`, `.exp-item`, `.cert-tag`, `.skills-grid > div`, elementos de `.contact-grid`

- [ ] **Step 1: Eliminar el CSS de `.fade-in` y `.fade-in.visible`**

Busca y elimina el bloque:
```css
.fade-in {
    opacity: 0;
    transform: translateY(18px);
    transition: opacity 0.55s ease, transform 0.55s ease;
}
.fade-in.visible { opacity: 1; transform: translateY(0); }
```

- [ ] **Step 2: Eliminar la función `initScrollFade()` completa del `<script>`**

Busca y elimina la función entera `function initScrollFade() { ... }` y la llamada `initScrollFade()` en `DOMContentLoaded`.

- [ ] **Step 3: Agregar utilidad `splitWords()` al bloque `<script>`**

```javascript
function splitWords(selector) {
    document.querySelectorAll(selector).forEach(el => {
        const words = el.textContent.trim().split(/\s+/);
        el.innerHTML = words.map(w =>
            `<span class="sw-wrap" style="display:inline-block;overflow:hidden;vertical-align:bottom">` +
            `<span class="sw-inner" style="display:inline-block">${w}</span></span>`
        ).join(' ');
    });
}
```

- [ ] **Step 4: Agregar `initScrollAnimations()` al bloque `<script>`**

```javascript
function initScrollAnimations() {

    // — Section titles (word split) —
    splitWords('.section-title');
    document.querySelectorAll('.section-title').forEach(el => {
        gsap.from(el.querySelectorAll('.sw-inner'), {
            y: 40, autoAlpha: 0,
            stagger: 0.08, duration: 0.65, ease: 'power2.out',
            scrollTrigger: { trigger: el, start: 'top 82%', once: true }
        });
    });

    // — Section labels —
    gsap.from('.section-label', {
        x: -16, autoAlpha: 0, duration: 0.5, ease: 'power2.out',
        scrollTrigger: { trigger: '.section-label', start: 'top 88%', once: true }
    });

    // — About body —
    gsap.from('.about-body', {
        y: 25, autoAlpha: 0, duration: 0.7, ease: 'power2.out',
        scrollTrigger: { trigger: '.about-body', start: 'top 82%', once: true }
    });

    // — About number count-up —
    const numValEl = document.querySelector('.about-number');
    if (numValEl) {
        const supEl = numValEl.querySelector('sup');
        const supHTML = supEl ? supEl.outerHTML : '';
        ScrollTrigger.create({
            trigger: numValEl,
            start: 'top 82%',
            once: true,
            onEnter: () => {
                gsap.fromTo({ val: 0 }, { val: 8 }, {
                    duration: 1.2, ease: 'power1.out',
                    onUpdate: function () {
                        numValEl.innerHTML = Math.round(this.targets()[0].val) + supHTML;
                    }
                });
            }
        });
    }

    // — Experiencia items —
    gsap.from('.exp-item', {
        x: -20, autoAlpha: 0, stagger: 0.12, duration: 0.6, ease: 'power2.out',
        scrollTrigger: { trigger: '.exp-list', start: 'top 82%', once: true }
    });

    // — Habilidades columnas —
    gsap.from('.skills-grid > div', {
        y: 20, autoAlpha: 0, stagger: 0.15, duration: 0.6, ease: 'power2.out',
        scrollTrigger: { trigger: '.skills-grid', start: 'top 82%', once: true }
    });

    // — Certificaciones tags —
    gsap.from('.cert-tag', {
        y: 10, autoAlpha: 0, stagger: 0.04, duration: 0.4, ease: 'power2.out',
        scrollTrigger: { trigger: '.certs-wrap', start: 'top 82%', once: true }
    });

    // — Contacto —
    gsap.from('.contact-headline', {
        y: 30, autoAlpha: 0, duration: 0.7, ease: 'power2.out',
        scrollTrigger: { trigger: '.contact-headline', start: 'top 82%', once: true }
    });
    gsap.from('.contact-link', {
        x: -12, autoAlpha: 0, stagger: 0.1, duration: 0.5, ease: 'power2.out',
        scrollTrigger: { trigger: '.contact-links', start: 'top 82%', once: true }
    });
    gsap.from('.whatsapp-col', {
        y: 20, autoAlpha: 0, duration: 0.6, ease: 'power2.out',
        scrollTrigger: { trigger: '.whatsapp-col', start: 'top 82%', once: true }
    });
}
```

- [ ] **Step 5: Actualizar `DOMContentLoaded` para llamar `initScrollAnimations()`**

```javascript
document.addEventListener('DOMContentLoaded', () => {
    document.getElementById('footer-year').textContent = new Date().getFullYear();
    initHeroAnimation();
    initCursor();
    initServiciosHover();
    initScrollAnimations();
});
```

- [ ] **Step 6: Verificar en el navegador**

Scroll lento por la página completa y verificar:
- Los títulos de sección entran palabra a palabra desde abajo
- Los ítems de Experiencia entran desde la izquierda en secuencia
- El número "8" hace count-up al entrar
- Los tags de certificaciones hacen stagger rápido
- No hay elementos que se queden pegados en `opacity: 0`
- No hay errores de consola relacionados con ScrollTrigger

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: replace IntersectionObserver with GSAP ScrollTrigger animations"
```

---

### Task 6: Sobre Mí — Foto con reveal animado

**Files:**
- Modify: `index.html` — CSS de `.about-grid` (agregar columna foto), HTML de `<section id="sobre-mi">` (agregar columna), agregar animación foto en `initScrollAnimations()`

**Interfaces:**
- Consume: `gsap` + `ScrollTrigger` (Task 1); `initScrollAnimations()` ya definida (Task 5)
- Require: imagen en `imagenes/imagen-principal.jpg` (si no existe, usar placeholder `<div>`)

- [ ] **Step 1: Actualizar CSS de `.about-grid` para 3 columnas**

Busca `.about-grid {` y reemplaza:

```css
.about-grid {
    display: grid;
    grid-template-columns: 140px 1fr 320px;
    gap: 4rem;
    align-items: start;
}

.about-photo-wrap {
    aspect-ratio: 4 / 5;
    overflow: hidden;
    clip-path: inset(100% 0% 0% 0%);
}

.about-photo-wrap img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
}

.about-photo-placeholder {
    width: 100%;
    height: 100%;
    min-height: 280px;
    background: var(--color-bg-alt);
    border: 2px dashed var(--color-border);
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: var(--font-mono);
    font-size: 0.7rem;
    color: var(--color-muted);
    letter-spacing: 0.1em;
}
```

- [ ] **Step 2: Actualizar el HTML de la sección `#sobre-mi`**

Agregar la columna de foto al `.about-grid`:

```html
<div class="about-grid">

    <!-- Columna 1: número decorativo -->
    <div class="about-number-col">
        <div class="about-number">8<sup>+</sup></div>
        <p class="about-sublabel"
           data-es="años de experiencia"
           data-en="years of experience">años de experiencia</p>
    </div>

    <!-- Columna 2: párrafo editorial -->
    <p class="about-body"
       data-es="Soy Contador Auditor con más de 8 años asesorando finanzas, proyectos y dirección empresarial. Hoy amplío ese perfil hacia la tecnología: estudio Ingeniería Informática combinando <strong>rigor financiero con visión digital</strong> para ayudar a empresas a tomar mejores decisiones."
       data-en="I'm a certified CPA with 8+ years advising on finance, projects, and corporate strategy. Today I'm expanding that profile into technology: studying Computer Engineering and combining <strong>financial rigor with digital vision</strong> to help businesses make better decisions.">
        Soy Contador Auditor con más de 8 años asesorando finanzas, proyectos y dirección empresarial. Hoy amplío ese perfil hacia la tecnología: estudio Ingeniería Informática combinando <strong>rigor financiero con visión digital</strong> para ayudar a empresas a tomar mejores decisiones.
    </p>

    <!-- Columna 3: foto con reveal -->
    <div class="about-photo-wrap">
        <!--
            Cuando tengas imagen-principal.jpg, reemplaza el placeholder por:
            <img src="imagenes/imagen-principal.jpg" alt="Matías Balbontín">
        -->
        <div class="about-photo-placeholder">[ FOTO ]</div>
    </div>

</div>
```

- [ ] **Step 3: Agregar la animación de reveal de foto en `initScrollAnimations()`**

Al final de la función `initScrollAnimations()`, antes del cierre `}`:

```javascript
// — Foto Sobre Mí — reveal clip-path
gsap.to('.about-photo-wrap', {
    clipPath: 'inset(0% 0% 0% 0%)',
    duration: 0.9, ease: 'power3.out',
    scrollTrigger: {
        trigger: '.about-photo-wrap',
        start: 'top 82%',
        once: true
    }
});
```

- [ ] **Step 4: Verificar en el navegador**

- La sección "Sobre Mí" muestra 3 columnas: número | párrafo | foto
- Al hacer scroll hasta la sección: la foto se revela desde abajo hacia arriba (efecto cortina)
- El placeholder `[FOTO]` es visible mientras no haya imagen real
- En móvil (< 768px): la foto se oculta o apila (verificar responsive en Task 7)

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add photo column to Sobre Mi with ScrollTrigger clip-path reveal"
```

---

### Task 7: Responsive mobile

**Files:**
- Modify: `index.html` — bloque `@media (max-width: 768px)` al final del CSS

**Interfaces:**
- Consume: clases `.hero-name`, `.service-row`, `.about-grid`, `.about-photo-wrap`, `#cursor` definidas en Tasks 3–6

- [ ] **Step 1: Reemplazar el bloque `@media (max-width: 768px)` completo**

Busca el bloque que empieza con `@media (max-width: 768px) {` y reemplaza todo su contenido:

```css
@media (max-width: 768px) {

    /* Hero */
    #hero {
        padding: 7rem 1.5rem 5rem;
        min-height: 100dvh;
    }
    .hero-name {
        font-size: clamp(3.2rem, 15vw, 5.5rem);
        margin-bottom: 2rem;
    }
    .hero-roles {
        flex-direction: column;
        gap: 0.5rem;
        margin-bottom: 2rem;
    }
    .scroll-indicator { display: none; }

    /* Sobre mí */
    .about-grid {
        grid-template-columns: 1fr;
        gap: 1.5rem;
    }
    .about-number-col { display: none; }
    .about-photo-wrap { display: none; }

    /* Servicios */
    .service-row {
        grid-template-columns: 40px 1fr 24px;
        gap: 1rem;
        padding: 0 0.5rem;
    }
    .service-title { font-size: 1.05rem; }

    /* Experiencia */
    .exp-item { grid-template-columns: 1fr; gap: 0.5rem; }

    /* Habilidades */
    .skills-grid { grid-template-columns: 1fr; gap: 2rem; }

    /* Contacto */
    .contact-grid { grid-template-columns: 1fr; gap: 3rem; }

    /* Nav */
    .nav-links { display: none; }
}
```

- [ ] **Step 2: Verificar en el navegador con DevTools (iPhone SE y iPad)**

Abre DevTools → Toggle device toolbar → iPhone SE (375px):
- Hero: nombre visible y grande, botones apilados si necesario
- Servicios: filas legibles, descripción aparece al tap (hover no aplica en touch)
- "Sobre Mí": sin columna de foto, sin número decorativo
- Nav: solo logo + botón de idioma

Revisar iPad (768px):
- Que ningún layout se rompa en el breakpoint exacto

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "fix: responsive mobile layout for new hero, servicios, and sobre mi"
```

---

## Self-Review

**Spec coverage:**
- [x] Tipografías Fraunces + Space Grotesk + Space Mono → Task 1
- [x] GSAP + ScrollTrigger CDN → Task 1
- [x] Cursor personalizado (4 estados, lerp) → Task 2
- [x] Hero tipografía total, animación de entrada → Task 3
- [x] Servicios filas horizontales con hover expansivo → Task 4
- [x] ScrollTrigger en todas las secciones existentes → Task 5
- [x] Count-up del número "8+" → Task 5
- [x] Foto trasladada a Sobre Mí con reveal → Task 6
- [x] Toggle ES/EN preservado en todos los elementos nuevos → verificado en cada task
- [x] Responsive mobile → Task 7

**Placeholder scan:** Sin TBD ni TODO en código. El placeholder de foto es intencional (imagen pendiente) y está documentado con comentario.

**Type/name consistency:**
- `initHeroAnimation()` → definida en Task 3, llamada en Task 3 ✓
- `initCursor()` → definida en Task 2, llamada en Task 3 ✓
- `initServiciosHover()` → definida en Task 4, llamada en Task 4 ✓
- `initScrollAnimations()` → definida en Task 5, extendida en Task 6, llamada en Task 5 ✓
- `splitWords()` → definida en Task 5, usada en Task 5 ✓
- `.hero-name-line-1 / .hero-name-line-2 / .hero-name-inner` → CSS Task 3, HTML Task 3, JS Task 3 ✓
- `.about-photo-wrap` → CSS Task 6, HTML Task 6, JS Task 6 ✓
- `.service-row / .service-desc / .service-arrow` → CSS Task 4, HTML Task 4, JS Task 4 ✓
