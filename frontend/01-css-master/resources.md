# CSS Master Agent - Resources

> **Referencias al Dev-Kit padre y recursos externos**

---

## 📚 Referencias al Dev-Kit Padre

### Documentación Core
Este agente se fundamenta en los siguientes documentos del dev-kit:

#### [CODE-QUALITY-METRICS.md](../../../docs/CODE-QUALITY-METRICS.md)
- **Complejidad Ciclomática**: Aplicada a selectores CSS
- **Maintainability Index**: CSS mantenible y escalable
- **Duplicación de Código**: DRY en CSS
- **LCOM (Cohesión)**: Componentes CSS cohesivos

**Aplicación en CSS:**
```css
/* ✅ BAJA COMPLEJIDAD - Selectores simples */
.button { }
.button--primary { }

/* ❌ ALTA COMPLEJIDAD - Selectores anidados */
nav > ul > li > a:not(.active):hover { }
```

#### [FAULT-TOLERANCE-GUIDE.md](../../../docs/FAULT-TOLERANCE-GUIDE.md)
- **Graceful Degradation**: CSS funciona sin JavaScript
- **Progressive Enhancement**: Features modernas con fallbacks
- **Resilience**: CSS robusto ante errores

**Aplicación en CSS:**
```css
/* Progressive Enhancement */
.grid {
  /* Fallback: Flexbox */
  display: flex;
  flex-wrap: wrap;
}

@supports (display: grid) {
  .grid {
    /* Enhancement: Grid */
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  }
}
```

#### [LESSONS-LEARNED-GODOT.md](../../../docs/LESSONS-LEARNED-GODOT.md)
- **Patrones aplicados**: Observer para themes, Factory para components
- **Optimizaciones**: Performance CSS similar a game performance
- **SOLID principles**: Single Responsibility en componentes CSS

#### [MICROSERVICES-ARCHITECTURE.md](../../../docs/MICROSERVICES-ARCHITECTURE.md)
- **Separation of Concerns**: CSS modular por componentes
- **Service Mesh**: CSS architecture mesh (design system)
- **Observability**: Monitoring CSS performance

---

## 🛠️ Herramientas Recomendadas

### Linting & Formatting
- **Stylelint**: [https://stylelint.io/](https://stylelint.io/)
  ```bash
  npm install --save-dev stylelint stylelint-config-standard
  ```

- **Prettier**: [https://prettier.io/](https://prettier.io/)
  ```bash
  npm install --save-dev prettier
  ```

### Build Tools
- **PostCSS**: [https://postcss.org/](https://postcss.org/)
  - Autoprefixer
  - PostCSS Preset Env
  - cssnano (minification)

- **PurgeCSS**: [https://purgecss.com/](https://purgecss.com/)
  ```bash
  npm install --save-dev @fullhuman/postcss-purgecss
  ```

### Performance Analysis
- **CSS Stats**: [https://cssstats.com/](https://cssstats.com/)
- **Project Wallace**: [https://www.projectwallace.com/](https://www.projectwallace.com/)
- **Lighthouse**: Built into Chrome DevTools
- **WebPageTest**: [https://www.webpagetest.org/](https://www.webpagetest.org/)

### Visual Regression Testing
- **Percy**: [https://percy.io/](https://percy.io/)
- **Chromatic**: [https://www.chromatic.com/](https://www.chromatic.com/)
- **BackstopJS**: [https://github.com/garris/BackstopJS](https://github.com/garris/BackstopJS)

### Accessibility
- **axe DevTools**: Chrome/Firefox extension
- **Pa11y**: [https://pa11y.org/](https://pa11y.org/)
- **WAVE**: [https://wave.webaim.org/](https://wave.webaim.org/)

---

## 📖 Learning Resources

### Official Documentation
- **MDN CSS Reference**: [https://developer.mozilla.org/en-US/docs/Web/CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- **W3C CSS Specifications**: [https://www.w3.org/Style/CSS/](https://www.w3.org/Style/CSS/)
- **Can I Use**: [https://caniuse.com/](https://caniuse.com/)

### Guides & Tutorials
- **CSS Tricks**: [https://css-tricks.com/](https://css-tricks.com/)
- **Smashing Magazine**: [https://www.smashingmagazine.com/category/css](https://www.smashingmagazine.com/category/css)
- **Web.dev CSS**: [https://web.dev/learn/css/](https://web.dev/learn/css/)
- **Modern CSS Solutions**: [https://moderncss.dev/](https://moderncss.dev/)

### Books
- **CSS Secrets** - Lea Verou
- **Refactoring UI** - Adam Wathan & Steve Schoger
- **Every Layout** - Heydon Pickering & Andy Bell
- **Inclusive Components** - Heydon Pickering

### Video Courses
- **CSS for JavaScript Developers** - Josh Comeau
- **Advanced CSS and Sass** - Jonas Schmedtmann (Udemy)
- **CSS Grid** - Wes Bos (free)
- **Flexbox** - Wes Bos (free)

---

## 🎓 CSS Methodologies

### BEM (Block Element Modifier)
- **Official**: [https://getbem.com/](https://getbem.com/)
- **Naming Convention**: [https://getbem.com/naming/](https://getbem.com/naming/)

### SMACSS (Scalable and Modular Architecture for CSS)
- **Official**: [http://smacss.com/](http://smacss.com/)

### ITCSS (Inverted Triangle CSS)
- **Introduction**: [https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture/](https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture/)

### OOCSS (Object-Oriented CSS)
- **Official**: [http://oocss.org/](http://oocss.org/)

### Atomic CSS
- **Tailwind CSS**: [https://tailwindcss.com/](https://tailwindcss.com/)
- **Tachyons**: [https://tachyons.io/](https://tachyons.io/)

---

## 🎨 CSS Frameworks & Libraries

### Utility-First
- **Tailwind CSS**: [https://tailwindcss.com/](https://tailwindcss.com/)
- **UnoCSS**: [https://unocss.dev/](https://unocss.dev/)

### Component Libraries
- **Bootstrap**: [https://getbootstrap.com/](https://getbootstrap.com/)
- **Bulma**: [https://bulma.io/](https://bulma.io/)
- **Foundation**: [https://get.foundation/](https://get.foundation/)

### CSS-in-JS
- **Styled Components**: [https://styled-components.com/](https://styled-components.com/)
- **Emotion**: [https://emotion.sh/](https://emotion.sh/)
- **Stitches**: [https://stitches.dev/](https://stitches.dev/)
- **Vanilla Extract**: [https://vanilla-extract.style/](https://vanilla-extract.style/)

---

## 🔗 Related Dev-Kit Skills

Este agente complementa y se integra con:

### Frontend Skills
- **HTML Semantic Agent** (#2) - Estructura HTML semántica
- **JavaScript Vanilla Specialist** (#3) - Interactividad
- **Web Performance Optimizer** (#6) - Performance general
- **Accessibility Champion** (#38) - Accesibilidad

### Code Quality Skills
- **Clean Code Enforcer** (#18) - Código limpio
- **Naming Convention Specialist** (#20) - Naming consistente
- **Code Review Bot** (#21) - Revisión de código

### Testing Skills
- **E2E Test Designer** (#28) - Testing visual
- **Performance Test Architect** (#29) - Performance testing

---

## 🌐 Browser Support

### Compatibilidad Recomendada
```json
{
  "browserslist": [
    "defaults",
    "not IE 11",
    "last 2 versions",
    "> 0.5%",
    "Firefox ESR",
    "not dead"
  ]
}
```

### Feature Detection
```css
@supports (display: grid) {
  /* CSS Grid supported */
}

@supports not (display: grid) {
  /* Fallback */
}
```

### Autoprefixer Configuration
```javascript
// postcss.config.js
module.exports = {
  plugins: {
    autoprefixer: {
      browsers: ['last 2 versions', '> 1%'],
      grid: 'autoplace'
    }
  }
}
```

---

## 📊 Performance Budgets

### CSS Bundle Size
- **Critical CSS**: < 14KB (TCP slow start)
- **Total CSS**: < 50KB gzipped
- **Unused CSS**: < 10%

### Rendering Metrics
- **First Contentful Paint (FCP)**: < 1.8s
- **Largest Contentful Paint (LCP)**: < 2.5s
- **Cumulative Layout Shift (CLS)**: < 0.1

---

## 🔍 Debugging Tools

### Browser DevTools
- **Chrome DevTools**: CSS debugging, performance
- **Firefox DevTools**: CSS Grid/Flexbox inspector
- **Safari Web Inspector**: iOS debugging

### Extensions
- **CSS Peeper**: Extract CSS
- **WhatFont**: Identify fonts
- **ColorZilla**: Color picker
- **Responsive Viewer**: Multi-device preview

### Online Tools
- **CodePen**: [https://codepen.io/](https://codepen.io/)
- **JSFiddle**: [https://jsfiddle.net/](https://jsfiddle.net/)
- **CSS Grid Generator**: [https://cssgrid-generator.netlify.app/](https://cssgrid-generator.netlify.app/)
- **Flexbox Generator**: [https://flexboxgenerator.com/](https://flexboxgenerator.com/)

---

## 📝 Templates & Boilerplates

### Starter Templates
```bash
# Create new project with CSS architecture
~/dev-kit/scripts/new-project-from-spec.sh my-app web typescript

# Copy CSS architecture template
cp -r ~/dev-kit/skills/frontend/01-css-master/templates/ ./src/styles/
```

### Directory Structure
```
src/
├── styles/
│   ├── base/
│   │   ├── _reset.css
│   │   ├── _typography.css
│   │   └── _utilities.css
│   ├── components/
│   │   ├── _button.css
│   │   ├── _card.css
│   │   └── _modal.css
│   ├── layout/
│   │   ├── _grid.css
│   │   ├── _header.css
│   │   └── _footer.css
│   ├── themes/
│   │   ├── _variables.css
│   │   └── _dark-mode.css
│   └── main.css
```

---

**Versión:** 1.0.0 | **Última actualización:** 2025-10-24
