---
name: daniel-style-redesign
description: Adoptar efectos, interacciones y tipografía del estilo danielstoopendaal.nl adaptados al perfil financiero/profesional de matiasbalbontin.cl, manteniendo la paleta cálida actual.
metadata:
  type: project
---

# Rediseño de Interacciones — Estilo Daniel Stoopendaal

## Contexto

**Sitio actual:** `index.html` único, HTML/CSS/JS vanilla, paleta crema/navy/terracota, tipografía Cormorant Garamond + Outfit + JetBrains Mono. Sin framework, sin build tools.

**Referencia:** https://danielstoopendaal.nl — tipografía gigante, efectos kinéticos, cursor personalizado, transiciones de scroll dramáticas.

**Enfoque elegido:** Reestructuración quirúrgica — reconstruir Hero y Servicios (requieren markup nuevo), agregar GSAP encima del resto sin tocar HTML de Experiencia, Habilidades, Certificaciones y Contacto.

**Paleta:** se mantiene idéntica (crema `#F7F5F2`, navy `#1A3C5E`, terracota `#C4622D`). No hay cambio a dark mode.

**Tecnología nueva:** GSAP + ScrollTrigger vía CDN (un `<script>` tag, sin build tools).

---

## 1. Hero — Reconstrucción completa

### Estructura HTML nueva

Reemplaza el grid de 2 columnas por un bloque de pantalla completa:

```
[100vh, centrado verticalmente con padding]

  MATÍAS           ← Cormorant Garamond, clamp(5rem, 12vw, 11rem), weight 300
  BALBONTÍN        ← misma escala, weight 700, color navy (#1A3C5E)

  — Contador Auditor  ·  Gestor de Proyectos  ·  Asesor Directivo  ·  Ing. Informática
                   ← JetBrains Mono, 0.78rem, color muted, margin-top 2rem

  [Conversemos]  [Ver trayectoria]   ← botones existentes sin cambio visual
                   ← margin-top 3rem

  ↓ (indicador de scroll, abajo a la derecha, animación loop suave)
```

La foto se elimina del Hero y se traslada a la sección "Sobre Mí" (ver sección 5).

### Animación de entrada (GSAP timeline, se ejecuta al DOMContentLoaded)

1. Estado inicial: todo invisible (`autoAlpha: 0`)
2. `MATÍAS` — reveal via `clipPath` de `inset(100% 0% 0% 0%)` a `inset(0% 0% 0% 0%)`, duración 0.8s, ease `power3.out`
3. `BALBONTÍN` — mismo efecto, delay 0.15s tras "MATÍAS"
4. Roles — `y: 12, autoAlpha: 0` → estado final, duration 0.5s, delay 0.2s tras "BALBONTÍN"
5. Botones CTA — `y: 16, autoAlpha: 0` → estado final, stagger 0.1s, delay 0.1s tras roles
6. Indicador scroll — fade in al final, loop de bounce infinito (`y: 6px`, repeat: -1, yoyo: true)

---

## 2. Cursor personalizado

### Comportamiento

Un `<div id="cursor">` posicionado fixed, pointer-events none, z-index 9999. Sigue al mouse via GSAP `ticker` con lerp (factor 0.12) para retraso suave.

### Estados

| Contexto | Tamaño | Apariencia |
|---|---|---|
| Default | 22px | Borde navy 1.5px, fondo transparente |
| Hover link/botón | 48px | Relleno navy sólido, `mix-blend-mode: difference` |
| Hover nombre Hero | 90px | Relleno navy opacity 0.15, sin borde |
| Click/mousedown | 18px (scale 0.82) | Transición rápida 0.1s, vuelve al estado anterior |

### Implementación

- JS puro + `gsap.ticker.add()` para seguimiento
- `document.querySelectorAll('a, button')` para detectar hovers
- `@media (hover: none)` en CSS oculta el cursor en touch devices

---

## 3. Servicios — Reconstrucción parcial

### Estructura HTML nueva

Reemplaza el grid de tarjetas por filas horizontales apiladas:

```html
<div class="services-list">
  <div class="service-row">
    <span class="service-num">01</span>
    <h3 class="service-title">Asesoría Financiera y Contable</h3>
    <p class="service-desc">Diagnóstico financiero...</p>
    <span class="service-arrow">→</span>
  </div>
  <!-- 02, 03 igual -->
</div>
```

### Comportamiento hover (GSAP)

- Fila por defecto: altura ~80px, borde inferior 1px color-border
- En hover:
  - Altura → ~200px (GSAP, duration 0.4s, ease `power2.out`)
  - Background → navy (`#1A3C5E`)
  - Texto → blanco
  - `.service-desc` se revela desde `y: 12, autoAlpha: 0` → visible
  - `.service-arrow` rota de 0° → 45° (`↗`)
  - Otras filas: opacity → 0.4
- En mouse leave: todo revierte

---

## 4. Transiciones de scroll — Secciones existentes

GSAP ScrollTrigger reemplaza el IntersectionObserver + clase `.fade-in` actual. El HTML de Sobre Mí, Experiencia, Habilidades, Certificaciones y Contacto **no cambia**.

### Patrones por tipo de elemento

| Elemento | Animación de entrada | Parámetros |
|---|---|---|
| `.section-title` | Split por palabra, stagger Y:40→0 + opacity | stagger: 0.08s, duration: 0.65s, ease: `power2.out` |
| `.about-body` | Y:25→0 + opacity | duration: 0.7s |
| `.exp-item` | X:-20→0 + opacity, secuencial | stagger: 0.12s |
| `.cert-tag` | Y:10→0 + opacity, stagger rápido | stagger: 0.04s |
| `.about-number` | Count-up animado (0 → 8) + opacity | duration: 1.2s, ease: `power1.out` |
| `.service-row` (load) | Y:30→0 + opacity, stagger | stagger: 0.1s |

### Configuración ScrollTrigger

- `start: "top 82%"` — entra cuando el elemento está al 82% del viewport
- `once: true` — la animación ocurre una sola vez
- No hay `scrub` (animación no está ligada al scroll speed, se dispara y corre sola)

---

## 5. Foto en sección "Sobre Mí"

La imagen se integra en "Sobre Mí" con reveal de ScrollTrigger:

- Estructura: contenedor con `overflow: hidden` + `clipPath: inset(100% 0% 0% 0%)`
- Al entrar al viewport: clip-path se abre hacia arriba (reveal "cortina")
- La imagen queda a la derecha del párrafo editorial, ratio 4:5, max 420px ancho
- La sección "Sobre Mí" pasa a grid 3 columnas: número decorativo | texto | foto

---

## 6. Lo que NO cambia

- Paleta de colores (todas las variables CSS actuales)
- Tipografías (Cormorant Garamond, Outfit, JetBrains Mono)
- Toggle ES/EN (`toggleLang()`)
- HTML interno de: **Experiencia, Habilidades, Certificaciones, Contacto, Footer** (Sobre Mí sí cambia — agrega columna de foto, ver sección 5)
- Sistema de `data-es` / `data-en` para i18n
- Responsive mobile (se extiende con breakpoints para nuevos componentes)

---

## 7. Orden de implementación

1. Agregar GSAP + ScrollTrigger vía CDN al `<head>`
2. Cursor personalizado (componente independiente, no afecta layout)
3. Hero — nuevo HTML + animación de entrada
4. Servicios — nuevo HTML + hover GSAP
5. ScrollTrigger en secciones existentes (reemplaza IntersectionObserver)
6. Foto en "Sobre Mí" — nueva estructura + reveal animation
7. Ajustes responsive mobile para Hero y Servicios nuevos

---

## Criterios de éxito

- La animación de entrada del Hero completa en < 1.5s
- El cursor no produce lag visible (lerp suave)
- Hover de servicios se siente inmediato (< 0.05s latencia percibida)
- Todas las transiciones de scroll son coherentes en velocidad y dirección
- El toggle ES/EN sigue funcionando en todos los elementos nuevos
- Mobile funcional (cursor oculto, hero tipografía responsive, filas de servicios apiladas)
