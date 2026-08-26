## 14. Lint & Format (ESM + Tailwind)

> Read this when changing ESLint, Prettier, Husky, or lint-staged config.

- **ESLint** (flat config `eslint.config.js`, ESM) — base set:
  - `eslint:recommended`, `react-hooks` (flat), `react-refresh/vite`, `jsx-a11y` (`eslint-plugin-jsx-a11y`)
  - `eslint-config-prettier` (disables conflicting rules)
- **Prettier** with `prettier-plugin-tailwindcss` (class sorting) and optionally `prettier-plugin-organize-imports`.
- **Config example:**
  ```js
  // eslint.config.js
  import js from '@eslint/js';
  import globals from 'globals';
  import reactHooks from 'eslint-plugin-react-hooks';
  import reactRefresh from 'eslint-plugin-react-refresh';
  import jsxA11y from 'eslint-plugin-jsx-a11y';
  import prettier from 'eslint-plugin-prettier/recommended';

  export default [
    { ignores: ['dist', 'coverage'] },
    {
      files: ['**/*.{js,jsx,ts,tsx}'],
      languageOptions: { globals: globals.browser, parserOptions: { ecmaFeatures: { jsx: true } } },
      plugins: { 'jsx-a11y': jsxA11y },
      extends: [js.configs.recommended, reactHooks.configs['recommended-latest'], reactRefresh.configs.vite],
      rules: { 'no-console': ['warn', { allow: ['warn', 'error'] }], 'jsx-a11y/alt-text': 'error' },
    },
    prettier,
  ];
  ```
- **Scripts:**
  ```json
  {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
  ```
- **Husky + lint-staged:**
  ```json
  {
    "lint-staged": {
      "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
      "*.{css,md,json}": ["prettier --write"]
    }
  }
  ```
  Pre-commit runs `lint-staged`; pre-push runs `pnpm test` and optionally `pnpm build`.
- **CI:** fails on `lint` or `test` failure.

---
