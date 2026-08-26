## 9. Routing & Navigation

> Read this when adding routes, guards, code-splitting, or navigation logic.

Example when `react-router-dom` is chosen — adapt to the router selected in §0:

```jsx
// routes/index.jsx
import { createBrowserRouter, Navigate } from 'react-router-dom';
import { lazy, Suspense } from 'react';

const HomePage = lazy(() => import('@/features/home/components/home-page.jsx'));
const FormPage = lazy(() => import('@/features/form/components/form-page.jsx'));
const NotFoundPage = lazy(() => import('@/components/layout/not-found.jsx'));

export const router = createBrowserRouter([
  { path: '/', element: <Navigate to="/home" replace /> },
  { path: '/home', element: <Suspense fallback={<p>Loading…</p>}><HomePage /></Suspense> },
  { path: '/form', element: <Suspense fallback={<p>Loading…</p>}><FormPage /></Suspense> },
  { path: '*', element: <Suspense fallback={null}><NotFoundPage /></Suspense> },
]);
```

- Use `lazy()` + `Suspense` for every route when code-splitting is desired — Vite splits per lazy boundary automatically.
- Guard: an auth/role guard checks the session (store or server-state query) and redirects to a login or entry route with `?next=...` when needed.
- Keep scroll restoration consistent — either via router `scrollRestoration` or a `useScrollToTop` hook on route change. For step-based flows, handle scroll in a `useEffect` tied to the current step, not inline in event handlers.
- If a flow should survive refresh mid-progress, persist minimal state to `sessionStorage` (avoid `localStorage` for sensitive data) and hydrate on mount.

If no router is chosen (single-page), keep the same lazy and guard principles for conditional views.

---
