## 15. Performance & Accessibility

> Read this when optimizing bundle, rendering, or accessibility.

### Performance

- **Bundle:** configure `vite build` with `rollupOptions.output.manualChunks` for `react` and vendor chunks when using a bundler analyzer (e.g. `rollup-plugin-visualizer`). Check with `pnpm build -- --analyze` if configured.
- **Code-splitting:** lazy-load routes and heavy features via `lazy()` + `Suspense`. Do not lazy-split tiny primitives where the overhead exceeds the benefit.
- **Images and fonts:** use `font-display: swap` and optionally `<link rel="preload" as="font" type="font/woff2" crossorigin>` for local fonts.
- **Memoization:** `useMemo` for derived lists and transformed catalogs; `useCallback` only when passing handlers to memoized children.
- **Network:** deduplicate fetches via the chosen server-state library (`staleTime`) and cancel superseded requests via `signal`. Debounce rapid input that triggers network calls.
- **CSS:** keep `tailwind.config.js` `content` pruned to actual source files — do not include `dist`. Test decorative layers on low-end devices and simplify if they cause repaints.
- **Vitals target (adjust per project):** LCP < 2.5s, INP < 200ms on a 4G profile.

### Accessibility (WCAG 2.1 AA)

- Use semantic HTML: `main`, `header`, `nav`, `fieldset`/`legend` for grouped controls, `aria-current="step"` for step progress.
- Focus: visible `focus-visible:ring`, logical tab order, `Escape` closes overlays, arrow keys navigate listboxes, focus returns to the trigger on close.
- Labels: every input has `label[htmlFor]`; required indicators use `aria-hidden` plus `aria-required` on the input. Error messages are linked via `aria-describedby` and announced with `aria-live="polite"`.
- Motion: respect `prefers-reduced-motion` for all animations and smooth scrolling. Provide a `usePrefersReducedMotion()` hook for JS-driven motion.
- Color and contrast: verify token pairs with an automated tool (e.g. `axe`). Do not rely on color alone for errors — use icon plus text.
- Test with keyboard only, screen reader, and automated `axe` in CI.

---
