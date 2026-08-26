## 6. Components — UI Primitives, Layout & Features

> Read this when creating or modifying UI primitives, layout, or feature components.

### Primitives (`components/ui/*`) — API

- Lean, accessible, token-driven. Every primitive:
  - uses `cn()` from `lib/cn.js` (e.g. `clsx` + `tailwind-merge`),
  - forwards `ref` when wrapping an interactive element,
  - has a `displayName`,
  - declares `propTypes` (if JS) or typed props (if TS).

```jsx
// lib/cn.js — recommended
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';
export function cn(...inputs) { return twMerge(clsx(inputs)); }

// components/ui/button.jsx — pattern (kebab-case file, PascalCase export)
import { forwardRef } from 'react';
import { cva } from 'class-variance-authority';
import { cn } from '@/lib/cn.js';

const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 disabled:opacity-50',
  {
    variants: {
      variant: { primary: 'bg-brand-primary text-white hover:bg-brand-primary-dark', outline: 'border bg-white hover:border-brand-primary', ghost: 'hover:bg-surface-muted' },
      size: { default: 'h-10 px-4 text-sm', large: 'h-11 px-6 text-base' },
    },
    defaultVariants: { variant: 'primary', size: 'default' },
  }
);

export const Button = forwardRef(({ variant, size, className, loading, children, ...props }, ref) => (
  <button ref={ref} className={cn(buttonVariants({ variant, size }), className)} disabled={props.disabled || loading} {...props}>
    {loading && <span aria-hidden className="animate-spin">◌</span>}
    {children}
  </button>
));
Button.displayName = 'Button';
```

- Accessibility checklist per primitive: text field → `label htmlFor + input id`, `aria-invalid`, `aria-describedby` pointing to `<p id="{id}-error">`; custom listbox → `role="listbox"` + `role="option"` + `aria-selected`, keyboard support (Arrow keys, Enter, Escape, Tab), focus management; checkbox/radio groups → `fieldset` + `legend`.

### Layout (`components/layout/*`)

- `page.jsx`: max-width wrapper, skip-to-content link, and `main` with `id="main-content"`.
- `error-boundary.jsx`: error boundary (class component or `react-error-boundary`) with fallback UI, reset action, and logging to observability.
- App shell mounts providers once — example:
  ```jsx
  import { StrictMode } from 'react';
  import { createRoot } from 'react-dom/client';
  import { RouterProvider } from 'react-router-dom';
  import { ErrorBoundary } from '@/components/layout/error-boundary.jsx';
  import { router } from '@/routes/index.jsx';
  import '@/index.css';

  createRoot(document.getElementById('root')).render(
    <StrictMode>
      <ErrorBoundary>
        <RouterProvider router={router} />
      </ErrorBoundary>
    </StrictMode>
  );
  ```
  Add or remove providers (query client, store, theme) according to the stack chosen in §0.

### Features (`features/<domain>/*`)

- Each feature owns its `components/`, `schemas/`, `hooks/`, and optionally `services/`. Its public API is `features/<domain>/index.js`.
- Keep feature containers small — split large flows into a shell component, step components, and hooks (`useWizard`, `useSubmit`, etc.).
- Never import `document` or `window.matchMedia` directly inside a feature component — isolate those in hooks (e.g. `hooks/use-prefers-reduced-motion.js`) for testability.

---
