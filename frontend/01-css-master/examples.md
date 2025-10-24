# CSS Master Agent - Examples

> **Casos de uso reales y soluciones**

---

## 📋 Tabla de Contenidos

- [Ejemplo 1: Refactoring CSS Legacy](#ejemplo-1-refactoring-css-legacy)
- [Ejemplo 2: Implementar Dark Mode](#ejemplo-2-implementar-dark-mode)
- [Ejemplo 3: Layout Responsive con Grid](#ejemplo-3-layout-responsive-con-grid)
- [Ejemplo 4: Optimizar Performance de Animations](#ejemplo-4-optimizar-performance-de-animations)
- [Ejemplo 5: Design System con Custom Properties](#ejemplo-5-design-system-con-custom-properties)

---

## Ejemplo 1: Refactoring CSS Legacy

### Problema
```css
/* ❌ CSS Legacy - Problemas múltiples */
#sidebar div.widget ul li a {
  color: #333 !important;
  padding: 5px 10px;
}

#sidebar div.widget ul li a:hover {
  color: #0066cc !important;
  background: #f0f0f0;
}

#content .post h2 {
  font-size: 24px;
  margin-bottom: 10px;
  color: #222;
}
```

**Problemas identificados:**
- ❌ Selectores con IDs (alta especificidad)
- ❌ Selectores anidados > 3 niveles
- ❌ !important innecesarios
- ❌ Magic numbers sin variables
- ❌ No usa naming convention
- ❌ Hardcoded colors

### Solución
```css
/* ✅ CSS Moderno con BEM */

/* Variables (CSS Custom Properties) */
:root {
  --color-text-primary: hsl(0, 0%, 20%);
  --color-text-secondary: hsl(0, 0%, 13%);
  --color-link: hsl(210, 100%, 40%);
  --color-bg-hover: hsl(0, 0%, 94%);

  --space-sm: 0.5rem;
  --space-md: 1rem;

  --font-size-h2: 1.5rem;
}

/* Widget Component */
.widget {}

.widget__list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.widget__link {
  color: var(--color-text-primary);
  padding: var(--space-sm) var(--space-md);
  display: block;
  text-decoration: none;
  transition: background-color 0.2s ease;
}

.widget__link:hover {
  color: var(--color-link);
  background-color: var(--color-bg-hover);
}

/* Post Component */
.post {}

.post__title {
  font-size: var(--font-size-h2);
  margin-bottom: var(--space-md);
  color: var(--color-text-secondary);
}
```

### Mejoras
- ✅ BEM naming convention
- ✅ CSS custom properties (theming)
- ✅ Selectores simples (baja especificidad)
- ✅ Sin !important
- ✅ Variables semánticas
- ✅ Escalable y mantenible

---

## Ejemplo 2: Implementar Dark Mode

### Solución Completa

```css
/* Design Tokens */
:root {
  /* Light mode (default) */
  --color-bg-primary: hsl(0, 0%, 100%);
  --color-bg-secondary: hsl(0, 0%, 98%);
  --color-text-primary: hsl(0, 0%, 13%);
  --color-text-secondary: hsl(0, 0%, 40%);
  --color-border: hsl(0, 0%, 88%);
  --color-link: hsl(210, 100%, 50%);

  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
}

/* Dark mode - System preference */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-primary: hsl(0, 0%, 10%);
    --color-bg-secondary: hsl(0, 0%, 15%);
    --color-text-primary: hsl(0, 0%, 95%);
    --color-text-secondary: hsl(0, 0%, 70%);
    --color-border: hsl(0, 0%, 25%);
    --color-link: hsl(210, 100%, 60%);

    --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.5);
    --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.6);
  }
}

/* Dark mode - Manual toggle */
[data-theme="dark"] {
  --color-bg-primary: hsl(0, 0%, 10%);
  --color-bg-secondary: hsl(0, 0%, 15%);
  --color-text-primary: hsl(0, 0%, 95%);
  --color-text-secondary: hsl(0, 0%, 70%);
  --color-border: hsl(0, 0%, 25%);
  --color-link: hsl(210, 100%, 60%);

  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.5);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.6);
}

/* Aplicación de variables */
body {
  background-color: var(--color-bg-primary);
  color: var(--color-text-primary);
  transition: background-color 0.3s ease, color 0.3s ease;
}

.card {
  background-color: var(--color-bg-secondary);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-md);
}

a {
  color: var(--color-link);
}
```

### JavaScript para Toggle
```javascript
// Toggle dark mode
const toggleTheme = () => {
  const current = document.documentElement.getAttribute('data-theme');
  const next = current === 'dark' ? 'light' : 'dark';
  document.documentElement.setAttribute('data-theme', next);
  localStorage.setItem('theme', next);
};

// Load saved theme
const savedTheme = localStorage.getItem('theme');
if (savedTheme) {
  document.documentElement.setAttribute('data-theme', savedTheme);
}
```

---

## Ejemplo 3: Layout Responsive con Grid

### Dashboard Layout

```css
/* Container */
.dashboard {
  display: grid;
  gap: var(--space-lg);
  padding: var(--space-lg);

  /* Mobile: 1 column */
  grid-template-columns: 1fr;
  grid-template-areas:
    "header"
    "stats"
    "chart"
    "table"
    "sidebar";
}

/* Tablet: 2 columns */
@media (min-width: 768px) {
  .dashboard {
    grid-template-columns: 1fr 1fr;
    grid-template-areas:
      "header header"
      "stats stats"
      "chart chart"
      "table sidebar";
  }
}

/* Desktop: 3 columns */
@media (min-width: 1024px) {
  .dashboard {
    grid-template-columns: 1fr 1fr 300px;
    grid-template-areas:
      "header header header"
      "stats stats sidebar"
      "chart chart sidebar"
      "table table sidebar";
  }
}

/* Components */
.dashboard__header { grid-area: header; }
.dashboard__stats { grid-area: stats; }
.dashboard__chart { grid-area: chart; }
.dashboard__table { grid-area: table; }
.dashboard__sidebar { grid-area: sidebar; }

/* Stats Grid */
.stats {
  display: grid;
  gap: var(--space-md);

  /* Auto-fit pattern */
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}

.stat-card {
  background: var(--color-bg-secondary);
  padding: var(--space-lg);
  border-radius: 8px;
  box-shadow: var(--shadow-sm);
}
```

---

## Ejemplo 4: Optimizar Performance de Animations

### Problema
```css
/* ❌ Animación no performante */
.box {
  transition: all 0.3s ease;
}

.box:hover {
  width: 200px;
  height: 200px;
  margin-left: 20px;
  background: red;
}

@keyframes slideIn {
  from {
    left: -100px;
  }
  to {
    left: 0;
  }
}
```

**Problemas:**
- ❌ `transition: all` (afecta todas las propiedades)
- ❌ Animando width/height (causa reflow)
- ❌ Animando margin/left (causa reflow)
- ❌ No usa GPU acceleration

### Solución
```css
/* ✅ Animación performante */
.box {
  /* Solo animar propiedades específicas */
  transition: transform 0.3s ease, opacity 0.3s ease;

  /* Hint al browser para GPU acceleration */
  will-change: transform;
}

.box:hover {
  /* Usar transform en lugar de width/height/margin */
  transform: scale(1.2);
}

@keyframes slideIn {
  from {
    /* Usar transform en lugar de left/top */
    transform: translateX(-100px);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-in {
  animation: slideIn 0.3s ease;
}

/* Cleanup: Remover will-change después de animación */
.box:not(:hover) {
  will-change: auto;
}
```

### Performance Tips
```css
/* ✅ RÁPIDO - Solo transform y opacity (GPU-accelerated) */
.fast-animation {
  transform: translateX(100px);
  opacity: 0.5;
}

/* ❌ LENTO - Causa layout/paint */
.slow-animation {
  left: 100px;
  width: 200px;
  margin-left: 20px;
  background: red;
}

/* Optimización adicional */
.animated {
  /* Crear nuevo layer */
  transform: translateZ(0);

  /* O usar will-change para hints */
  will-change: transform, opacity;
}
```

---

## Ejemplo 5: Design System con Custom Properties

### Sistema Completo

```css
:root {
  /* === SPACING === */
  --space-3xs: 0.125rem;  /* 2px */
  --space-2xs: 0.25rem;   /* 4px */
  --space-xs: 0.5rem;     /* 8px */
  --space-sm: 0.75rem;    /* 12px */
  --space-md: 1rem;       /* 16px */
  --space-lg: 1.5rem;     /* 24px */
  --space-xl: 2rem;       /* 32px */
  --space-2xl: 3rem;      /* 48px */
  --space-3xl: 4rem;      /* 64px */

  /* === TYPOGRAPHY === */
  --font-sans: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
  --font-serif: Georgia, Cambria, 'Times New Roman', serif;
  --font-mono: 'Fira Code', Consolas, Monaco, monospace;

  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.25rem;    /* 20px */
  --font-size-2xl: 1.5rem;    /* 24px */
  --font-size-3xl: 1.875rem;  /* 30px */
  --font-size-4xl: 2.25rem;   /* 36px */

  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  --line-height-tight: 1.25;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.75;

  /* === COLORS === */
  /* Primary */
  --color-primary-50: hsl(220, 90%, 95%);
  --color-primary-100: hsl(220, 90%, 90%);
  --color-primary-500: hsl(220, 90%, 56%);
  --color-primary-600: hsl(220, 90%, 46%);
  --color-primary-900: hsl(220, 90%, 20%);

  /* Neutral */
  --color-neutral-50: hsl(0, 0%, 98%);
  --color-neutral-100: hsl(0, 0%, 96%);
  --color-neutral-500: hsl(0, 0%, 50%);
  --color-neutral-900: hsl(0, 0%, 13%);

  /* Semantic */
  --color-success: hsl(142, 71%, 45%);
  --color-warning: hsl(38, 92%, 50%);
  --color-error: hsl(0, 84%, 60%);
  --color-info: hsl(199, 89%, 48%);

  /* === BORDERS === */
  --border-radius-sm: 0.25rem;
  --border-radius-md: 0.375rem;
  --border-radius-lg: 0.5rem;
  --border-radius-xl: 0.75rem;
  --border-radius-full: 9999px;

  --border-width-thin: 1px;
  --border-width-medium: 2px;
  --border-width-thick: 4px;

  /* === SHADOWS === */
  --shadow-xs: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);

  /* === Z-INDEX === */
  --z-dropdown: 1000;
  --z-sticky: 1020;
  --z-fixed: 1030;
  --z-modal-backdrop: 1040;
  --z-modal: 1050;
  --z-popover: 1060;
  --z-tooltip: 1070;

  /* === TRANSITIONS === */
  --transition-fast: 150ms;
  --transition-base: 250ms;
  --transition-slow: 350ms;

  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
}

/* === UTILITY CLASSES === */

/* Spacing */
.p-sm { padding: var(--space-sm); }
.p-md { padding: var(--space-md); }
.p-lg { padding: var(--space-lg); }

.m-sm { margin: var(--space-sm); }
.m-md { margin: var(--space-md); }
.m-lg { margin: var(--space-lg); }

/* Typography */
.text-xs { font-size: var(--font-size-xs); }
.text-sm { font-size: var(--font-size-sm); }
.text-base { font-size: var(--font-size-base); }
.text-lg { font-size: var(--font-size-lg); }

.font-medium { font-weight: var(--font-weight-medium); }
.font-semibold { font-weight: var(--font-weight-semibold); }
.font-bold { font-weight: var(--font-weight-bold); }

/* Colors */
.bg-primary { background-color: var(--color-primary-500); }
.text-primary { color: var(--color-primary-500); }

.bg-neutral-50 { background-color: var(--color-neutral-50); }
.text-neutral-900 { color: var(--color-neutral-900); }

/* Borders */
.rounded-sm { border-radius: var(--border-radius-sm); }
.rounded-md { border-radius: var(--border-radius-md); }
.rounded-lg { border-radius: var(--border-radius-lg); }
.rounded-full { border-radius: var(--border-radius-full); }

/* Shadows */
.shadow-sm { box-shadow: var(--shadow-sm); }
.shadow-md { box-shadow: var(--shadow-md); }
.shadow-lg { box-shadow: var(--shadow-lg); }
```

### Uso del Design System

```html
<button class="
  bg-primary
  text-white
  p-md
  rounded-lg
  shadow-md
  font-semibold
  hover:shadow-lg
  transition-base
">
  Click me
</button>

<div class="
  bg-neutral-50
  p-lg
  rounded-md
  shadow-sm
">
  <h2 class="text-2xl font-bold text-neutral-900 m-md">
    Card Title
  </h2>
  <p class="text-base text-neutral-500">
    Card content
  </p>
</div>
```

---

## 🎯 Más Ejemplos

Ver también:
- Container Queries para componentes responsive
- CSS Grid subgrid para layouts complejos
- Custom properties para theming dinámico
- Performance optimization checklist
- Accessibility patterns (focus states, keyboard navigation)

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
