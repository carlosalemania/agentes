# CSS Master Agent - Checklist

> **Lista de verificación para auditoría y desarrollo CSS**

---

## 🔍 Auditoría CSS

### Arquitectura
- [ ] ¿Se usa una metodología consistente (BEM, SMACSS, ITCSS)?
- [ ] ¿La estructura de archivos está organizada lógicamente?
- [ ] ¿Los nombres de clases son descriptivos y semánticos?
- [ ] ¿Hay un design system con tokens/variables?
- [ ] ¿Se evitan IDs para styling?
- [ ] ¿Los selectores tienen baja especificidad (< 3 niveles)?

### Performance
- [ ] ¿Las animaciones usan solo `transform` y `opacity`?
- [ ] ¿Se usa `will-change` apropiadamente?
- [ ] ¿Hay critical CSS para above-the-fold?
- [ ] ¿Se ha eliminado CSS no utilizado?
- [ ] ¿Los selectores son eficientes?
- [ ] ¿Se evita `@import` en favor de `<link>`?
- [ ] ¿Las imágenes de fondo son optimizadas?
- [ ] ¿Se usa `contain` o `content-visibility` donde aplica?

### Responsive Design
- [ ] ¿Se usa mobile-first approach?
- [ ] ¿Los breakpoints están basados en contenido?
- [ ] ¿Hay fluid typography (clamp, calc)?
- [ ] ¿Las imágenes son responsive (srcset, picture)?
- [ ] ¿Se usan unidades relativas (rem, em, %)?
- [ ] ¿Funciona bien en todos los tamaños de pantalla?
- [ ] ¿Se consideran Container Queries donde aplica?

### Accesibilidad
- [ ] ¿El contraste de colores cumple WCAG 2.1 (mínimo 4.5:1)?
- [ ] ¿Los estados de focus son visibles?
- [ ] ¿Se puede navegar solo con teclado?
- [ ] ¿No hay scroll horizontal no intencionado?
- [ ] ¿El texto es legible (tamaño mínimo 16px)?
- [ ] ¿Funciona con zoom al 200%?
- [ ] ¿Se respeta `prefers-reduced-motion`?

### Mantenibilidad
- [ ] ¿Hay documentación de decisiones complejas?
- [ ] ¿Los magic numbers están explicados o son variables?
- [ ] ¿No hay duplicación de código?
- [ ] ¿Se usan CSS custom properties para valores repetidos?
- [ ] ¿El código es consistente en todo el proyecto?
- [ ] ¿Se evita `!important` excepto para utilities?
- [ ] ¿Hay comments útiles (por qué, no qué)?

### Modern CSS
- [ ] ¿Se usan CSS custom properties?
- [ ] ¿Se aprovechan CSS Grid y Flexbox?
- [ ] ¿Se usan logical properties para i18n?
- [ ] ¿Se considera cascade layers (@layer)?
- [ ] ¿Hay fallbacks para features modernas?

---

## 🛠️ Desarrollo de Componentes

### Planning
- [ ] Definir estados del componente (default, hover, active, disabled)
- [ ] Identificar variantes necesarias
- [ ] Determinar breakpoints responsive
- [ ] Planear accesibilidad (ARIA, keyboard)
- [ ] Considerar dark mode desde el inicio

### Implementation
- [ ] Crear estructura HTML semántica
- [ ] Aplicar naming convention (BEM)
- [ ] Usar variables para valores repetidos
- [ ] Implementar mobile-first
- [ ] Añadir estados interactivos
- [ ] Optimizar para performance
- [ ] Documentar uso del componente

### Testing
- [ ] Probar en múltiples browsers (Chrome, Firefox, Safari, Edge)
- [ ] Validar responsive en diferentes tamaños
- [ ] Verificar accesibilidad (axe DevTools)
- [ ] Testar con keyboard navigation
- [ ] Verificar con screen reader
- [ ] Probar dark mode si aplica
- [ ] Validar performance (Lighthouse)

---

## 🎨 Layout Design

### Grid Layout
- [ ] ¿Usa CSS Grid apropiadamente?
- [ ] ¿Las grid areas están nombradas semánticamente?
- [ ] ¿Se aprovecha auto-placement donde aplica?
- [ ] ¿Hay fallback para navegadores antiguos?
- [ ] ¿Es responsive con grid-template-areas?

### Flexbox
- [ ] ¿Flexbox es la herramienta correcta (vs Grid)?
- [ ] ¿Se evita flex-grow/shrink no intencionado?
- [ ] ¿Hay wrap apropiado para responsive?
- [ ] ¿Se usa gap en lugar de margins?

### Spacing
- [ ] ¿Se usa un sistema de spacing consistente?
- [ ] ¿Los valores vienen de variables?
- [ ] ¿Se usa gap/grid-gap en lugar de margins cuando es posible?
- [ ] ¿El spacing es responsive?

---

## 🌗 Theming

### Setup
- [ ] CSS custom properties definidas en `:root`
- [ ] Naming semántico (--color-primary, no --color-blue)
- [ ] Dark mode implementado
- [ ] Fallbacks para navegadores sin soporte
- [ ] Sistema de colores con escalas (50-900)

### Implementation
- [ ] Todas las propiedades usan variables
- [ ] Transiciones suaves entre temas
- [ ] No hay colores hardcodeados
- [ ] Respeta `prefers-color-scheme`
- [ ] Toggle manual funciona correctamente

---

## 🚀 Performance Optimization

### CSS File Size
- [ ] Minificado para producción
- [ ] Gzipped/Brotli compression
- [ ] Critical CSS inline en `<head>`
- [ ] Non-critical CSS cargado async
- [ ] Unused CSS removido

### Rendering Performance
- [ ] Animaciones en properties GPU-accelerated
- [ ] No hay layout thrashing
- [ ] `will-change` usado apropiadamente
- [ ] Selectores optimizados
- [ ] Fonts optimizados (preload, font-display)

### Loading Strategy
- [ ] Critical CSS inline
- [ ] Above-the-fold optimizado
- [ ] Lazy loading para below-the-fold
- [ ] Preload para fonts críticas
- [ ] Resource hints (preconnect, dns-prefetch)

---

## 📱 Responsive Checklist

### Breakpoints
- [ ] Mobile: < 640px
- [ ] Tablet: 640px - 1024px
- [ ] Desktop: > 1024px
- [ ] Large Desktop: > 1280px
- [ ] Custom breakpoints basados en contenido

### Testing Devices
- [ ] iPhone SE (375px)
- [ ] iPhone 12/13 Pro (390px)
- [ ] iPad (768px)
- [ ] iPad Pro (1024px)
- [ ] Desktop (1440px)
- [ ] Large Desktop (1920px)

### Features
- [ ] Touch targets mínimo 44x44px (móvil)
- [ ] No hover states solo en móvil
- [ ] Gestures apropiados (swipe, pinch)
- [ ] Viewport meta tag configurado
- [ ] No fixed widths inflexibles

---

## 🔧 Tooling

### Linting
- [ ] Stylelint configurado
- [ ] Pre-commit hooks activos
- [ ] CI/CD valida CSS
- [ ] Reglas de proyecto documentadas

### Build Process
- [ ] PostCSS configurado
- [ ] Autoprefixer activo
- [ ] CSS minification
- [ ] PurgeCSS para remover unused
- [ ] Source maps en desarrollo

### Testing
- [ ] Visual regression testing (Percy, Chromatic)
- [ ] Accessibility testing (axe, Pa11y)
- [ ] Performance testing (Lighthouse CI)
- [ ] Cross-browser testing

---

## ✅ Pre-Release Checklist

### Code Quality
- [ ] Pasa linting sin errores
- [ ] No hay CSS no utilizado
- [ ] No hay !important innecesarios
- [ ] Naming consistente en todo el proyecto
- [ ] Documentación actualizada

### Performance
- [ ] Lighthouse score > 90
- [ ] Bundle size aceptable (< 50KB gzipped)
- [ ] Critical CSS optimizado
- [ ] Animations performantes

### Cross-Browser
- [ ] Chrome (últimas 2 versiones)
- [ ] Firefox (últimas 2 versiones)
- [ ] Safari (últimas 2 versiones)
- [ ] Edge (últimas 2 versiones)
- [ ] Mobile browsers

### Accessibility
- [ ] WCAG 2.1 Level AA compliance
- [ ] Keyboard navigation funciona
- [ ] Screen reader compatible
- [ ] Contrast ratios cumplen
- [ ] Zoom 200% funciona

### Documentation
- [ ] Componentes documentados
- [ ] Design system actualizado
- [ ] Ejemplos de uso
- [ ] Breaking changes documentados

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
