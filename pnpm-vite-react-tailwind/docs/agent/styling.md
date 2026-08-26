## 8. Styling — Tailwind, Design Tokens & CSS Layering

> Read this when changing Tailwind config, design tokens, or global CSS layering.

**Config (`tailwind.config.js`, ESM)**

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {
      colors: {
        // Define project-specific tokens here — do not hardcode hex values in components
        'brand-primary': '#0f62fe',
        'brand-primary-dark': '#0a4bcc',
        'surface-muted': '#f5f5f5',
        'border-default': '#e5e5e5',
        'text-primary': '#111111',
        'text-secondary': '#6b7280',
        'danger': '#dc2626',
        'danger-muted': '#fef2f2',
      },
      fontFamily: { sans: ['Inter', 'system-ui', 'sans-serif'] },
      borderRadius: { DEFAULT: '6px' },
    },
  },
  plugins: [],
};
```

- Keep all design values in `theme.extend` — never hardcode hex or spacing values inside components.
- If a design system is used, also expose tokens as CSS variables (`--brand-primary`) so CSS and JS can share them.

**`src/index.css` Layering**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  @font-face {
    font-family: 'Inter';
    src: url('/fonts/InterVariable.woff2') format('woff2');
    font-weight: 100 900;
    font-display: swap;
  }
  html { scroll-behavior: smooth; }
  body { @apply font-sans antialiased bg-white text-text-primary; }
}

@keyframes enter-soft {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

@layer utilities {
  .animate-enter {
    animation: enter-soft 600ms cubic-bezier(0.16, 1, 0.3, 1) both;
    animation-delay: calc(var(--index, 0) * 80ms);
  }
}

@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  .animate-enter { animation: none; }
}
```

Rules:
- Always pair animations with a `prefers-reduced-motion` override.
- Ambient or decorative layers must use `pointer-events: none` and a non-intrusive `z-index`; test on low-end devices and reduce complexity if they cause repaints.
- Use `@layer components` for repeated patterns when needed, but prefer `cva` in JSX over CSS utilities for primitives.
- Avoid `@apply` for complex responsive layouts — keep Tailwind in the markup.

**Tailwind Migration**

- When integrating into an existing Tailwind project, merge `content` and `theme.extend` — do not replace the host config. Document merged tokens in the PR.
- Fonts: if the host already provides the chosen font, skip copying `public/fonts/` and adjust `fontFamily.sans` accordingly.

---
